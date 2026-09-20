---
name: councilof-ai-read-the-board
description: Read the live GSPC AI-governance board from Council of AI without an account or key, quote its counts by field path, and never turn an unmeasured slot into a number.
api: openapi/councilof-ai-public-api-openapi.yml
operations:
  - get__api_gspc
  - get__api_state
  - get__api_axis-register
  - get__api_methodology
  - get__api_observability
method: generated
generated: '2026-09-19'
grounding: Every operationId above exists verbatim in openapi/councilof-ai-public-api-openapi.yml. Field names and the quoting rules are taken from the live GET /api/gspc and /api/state responses (2026-09-19), llms.txt and api-docs.
---

# Read the GSPC board

Public, keyless, CORS-open. `GET https://councilof.ai/api/gspc` is the authority; every other surface (MCP `board_totals`, the A2A `gspc-board` skill, the badges, the PyPI reader) derives from it. Responses are edge-cached for 300 s (`cache-control: public, max-age=300`).

## 1. Fetch the board

- `GET /api/gspc` (`get__api_gspc`) — the whole board: `totals`, `axes[]`, `domains[]`, `limitations`, `doi`, `site_attestation`.
- `GET /api/gspc?axis=<name>` — one axis. The `axis` query is documented on api-docs, not in the OpenAPI. An unknown name returns **404** `{"error":"unknown axis","known":[…]}` — the error body lists every valid name, so use it as the enumeration.

## 2. Quote counts by field path — never compose one

The provider's rule, repeated on every surface: quote `totals.public_count` (e.g. `"23 axis · 22 measured"`) or the long form `totals.count_grammar`. Two numbers always travel together:

- `totals.axes` counts **slots** on the board;
- `totals.measured_axes` counts slots with a real run behind them; `totals.unmeasured_axes` makes the gap visible.

Quote both, or quote the smaller. Do not add `by_family` counts to get a total, and do not carry a count forward from an older response. `GET /api/state` (`get__api_state`) is the machine-readable list of "the numbers a lane may quote", each with `source`, `as_of` and `as_of_field` — if a number is not in that payload it is not established.

## 3. Read an axis honestly

Each `axes[]` row carries `family` (`gspc` | `financial`), `status` (`MEASURED` | `UNMEASURED`), `n`, an accuracy/interval, and a leader separation state (`SEPARATED` | `TIE`). Rules the provider states in the payload itself:

- an **UNMEASURED** axis is a first-class answer — "a declared slot with no run behind it" — never an error and never a zero;
- **TIE is TIE**; UNTESTED is not a win; the 8 `financial` axes are fact runs, not leader scores;
- nothing is quotable below the provider's usable-n threshold; the payload says which cells are quotable.

`GET /api/axis-register` (`get__api_axis-register`) is the register of axis definitions; `GET /api/methodology` (`get__api_methodology`) states the grading rules ("no model judges another model", canaries excluded, transport failures counted against the provider, not the model).

## 4. Check freshness before you rely on it

`GET /api/observability` (`get__api_observability`) reports every connector's `as_of` and `age_hours` and names the `stalest_connector`. `as_of` values are read out of the artefacts, never `new Date()`; two calls any interval apart return identical `as_of` values.

## What this is not

Measurement, not certification. The board is not a compliance verdict, a "safe" label or a ranking for sale; the provider's `explicitly_not` list is `[certification, accreditation, conformity-assessment, legal-determination, enforcement]`. There is no rate limit to negotiate and no 429 in the contract; unknown `/api/*` paths return a JSON 404 with a `hint`.
