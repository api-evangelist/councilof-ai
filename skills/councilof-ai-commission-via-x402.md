---
name: councilof-ai-commission-via-x402
description: Discover Council of AI's x402 doors, read a 402 challenge and its free preview, rehearse settlement on the zero-priced free door, and only then commission a signed artefact from your own wallet — knowing exactly what money buys and does not buy.
api: openapi/councilof-ai-public-api-openapi.yml
operations:
  - get__api_x402
  - x402_free_door
  - x402_request_attestation
  - get__api_commissions
  - get__api_receipt-status
  - get__api_revenue
method: generated
generated: '2026-09-19'
grounding: Every operationId above exists verbatim in openapi/councilof-ai-public-api-openapi.yml. The challenge shape, the observed amounts and the campaign fields are from live 402 responses on 2026-09-19 (no purchase was made); the six-step flow is quickstart.json.
---

# Commission an artefact through x402

Agents pay per artefact — issuance, assembly, cadence — in USDC on Base (`eip155:8453`); the board and verification stay free. The provider "holds no key for you and never settles on your behalf": payment comes from the caller's wallet via an x402 client (quickstart names `@x402/fetch` + `@x402/evm ExactEvmScheme`).

## 1. Discover (free)

- `GET /api/x402` (`get__api_x402`) — the machine catalog of the metered rail ("NO AMOUNTS HERE"); also `GET /.well-known/x402.json` (`x402Version 2`, `mode live`, `payTo`, `resources[]`).
- The OpenAPI marks every door with `x-payment-info: {protocols: [x402]}` and a top-level `x-x402` block naming the ten doors.

## 2. Read the challenge (free)

Call a door without payment, e.g. `GET /api/request-attestation?subject=<id>` (`x402_request_attestation`). You get **HTTP 402**, a `PAYMENT-REQUIRED` header (base64 of the body) and a body:

- `accepts[0]` — `scheme exact`, `network eip155:8453`, `asset 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` (USDC, 6 decimals), `payTo 0x212686404A7D1E1fD88F35eD6200c3aF7A78ae31`, `amount` in atomic units, `maxTimeoutSeconds 300`;
- `accepts[0].csoai_pricing` — `product_id`, `sku_id`, `tier`, `pricing_basis`, `normal_amount_atomic`, `offered_amount_atomic`, campaign dates, `fresh_compute_excluded`;
- `extensions["offer-receipt"].info.offers[]` — a server-signed JWS (EdDSA, `kid did:web:csoai.org#board-attestation-1`) you can verify against `https://csoai.org/.well-known/did.json` **before** paying;
- `csoai.preview` — the free part of the deliverable (for request-attestation: the signed cards already on file for the subject).

**The amount in `accepts[]` is the only price that exists.** Observed 2026-09-19: `10000` atomic (0.01 USDC) under campaign `csoai-launch-30d-20260911`, normal `20000`. Do not quote a price from any document, including this one.

## 3. Rehearse on the free door (free)

`GET /api/free-door` (`x402_free_door`) is a live 402 route with `amount "0"` — "it settles, and charges nothing; proves the rail." Run your full pay-and-retry loop here first. `preview=true` / omitting `bundle=1` on the other doors returns unsigned free versions.

## 4. Settle and retry

Sign an authorisation for `accepts[0]` from your own wallet and retry the **same URL** with the payment header. Settlement is on Base mainnet; on-chain facts are public and final.

## 5. Receive

`200` JSON `{card, verify, signed, unsigned_reason, bytes, note}`; the `x-payment-response` header carries the settlement response and, when the facilitator names a payer and the signing key is present, a signed receipt. Then:

- `GET /api/commissions` (`get__api_commissions`) — per-subject `fulfillment` (`QUEUED` | `RETRIEVABLE` | `UNFULFILLABLE`) and `retrieval` links; "a settled request is not a measurement until a signed card is retrievable";
- `GET /api/receipt-status` (`get__api_receipt-status`) and `GET /api/receipts?payer=0x…` — your receipts; `UNRECORDED` means no store is bound, never that you did not pay;
- `GET /api/revenue` (`get__api_revenue`) — the provider's own settlement truth (`settled_usdc`, `excludes_self`).

## What money buys — and does not

Payment buys issuance/assembly/cadence and a durable signature. It **never** mints a MEASURED cell, buys a rank, a grade, a certificate or a filled cell (`x-x402.not`); re-measurement is free and cannot be expedited (Firewall Charter). No refund window is published — treat a settled purchase as final and use the free preview and the free door to avoid paying for the wrong thing. A 402 is not delivery; a receipt proves the server signed those bytes, not by itself that money moved — check `transaction` against the chain.
