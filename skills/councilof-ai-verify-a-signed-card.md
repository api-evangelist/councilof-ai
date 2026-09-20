---
name: councilof-ai-verify-a-signed-card
description: Verify a Council of AI Ed25519-signed measurement card and check a card's Merkle inclusion in the public root, offline, against the pinned did:web key — reporting VALID, INVALID or UNCHECKABLE and never collapsing the last two.
api: openapi/councilof-ai-gspc-read-surfaces-openapi.yml
operations:
  - getDid
  - getVerifyRecipe
  - getCardIndex
  - getCard
  - getRoot
  - getProof
  - getBoard
method: generated
generated: '2026-09-19'
grounding: Every operationId above exists verbatim in openapi/councilof-ai-gspc-read-surfaces-openapi.yml (the spec at https://councilof.ai/api/openapi.json). The rule, the pinned-key hex and the float caveat are quoted from /signed/HOW-TO-VERIFY.md as fetched 2026-09-19.
---

# Verify a signed measurement card

"You do not need our code, our permission, or our word for any of this. Everything below runs against the published bytes." Verification is free, keyless and offline-capable.

## 1. Pin the key first — this step is not optional

`GET /.well-known/did.json` (`getDid`; the provider's tools read it from `https://csoai.org/.well-known/did.json`, `did:web:csoai.org`). Take the `verificationMethod` whose `id` ends in `#card-attestation-1`, decode `publicKeyJwk.x` (base64url) — the recipe prints it as `d4cb0eaa16d5f50bf7633a36aa34fe09a55e124b9316ded2abdb122bb9c37e38`. Every published card MUST carry that exact `pubkey`. A card that verifies only against the key it ships with proves self-consistency, not authenticity: "anyone can alter a body, sign it with a key they generated a second ago, and it will verify."

## 2. Find the card

- `GET /signed/card_index.json` (`getCardIndex`) — the frozen signed-card index: `n_cards` must equal `n_cells` and `cards[].length`; if they disagree, "neither number is quotable".
- `GET /signed/cards/{id}.json` (`getCard`) — one signed card by id.
- `GET /signed/HOW-TO-VERIFY.md` (`getVerifyRecipe`) — the rule, kept current by the provider.

## 3. Apply the rule

```
preimage  == json.dumps(body, sort_keys=True, separators=(',',':'), ensure_ascii=True).encode('utf-8')
id        == sha256(preimage).hexdigest()
signature == Ed25519(preimage) under the pinned key
```

Caveat the recipe flags: the preimage was produced by CPython's `json.dumps`, which renders an integral float as `0.0`; JavaScript `JSON.stringify`, Go `encoding/json` and RFC 8785 render `0`. A naive non-Python verifier therefore reports a **false INVALID**. Use the provider's zero-dependency verifiers (`/signed/verify-card.mjs`, `/verifier/card-v0-verify.mjs`, exit codes VALID 0 / INVALID 1 / UNCHECKABLE 2) or `pip install "csoai-gspc[verify]"` then `csoai-gspc verify <sha256>`.

## 4. Report three states, never two

- **VALID** — body reproduces its id and the signature verifies under the pinned key;
- **INVALID** — altered body, wrong key, bad signature;
- **UNCHECKABLE** — no Ed25519 backend, unfetchable card, malformed file.

"I could not check" is not "it is forged." Never present UNCHECKABLE as INVALID, and never present a card's own embedded key as proof.

## 5. Optional: check inclusion in the public root

`GET /root.json` (`getRoot`) is a separate Merkle envelope (`merkle_root`, `card_count`, `as_of`, `sig_ed25519` when signed) over **card-v0 leaves** — a different corpus from the signed measurement cards; the provider says the three card counts on the estate share no members and must never be added. `GET /api/proof?sha=<64hex>` (`getProof`) answers VALID (included) / INVALID (not a leaf) / UNCHECKABLE (proof endpoint unreachable) for free; `?bundle=1` on the same path is an x402-metered proof bundle (HTTP 402 first).

## 6. Cross-check the board

`GET /api/gspc` (`getBoard`) — a measured axis should be backed by cards you can verify. If it is not, the provider's own doctrine says to treat the cell as UNMEASURED, and its dispute route (https://councilof.ai/dispute/) answers "with a re-measurement, never a defence".
