# http — conformance/

- `docs.md` — the library's reference, in the pattern of botopink-lang's `conformance/docs.md`: one section per public
  module of `src/`, each describing in prose what the module offers and how it fails, no fenced `botopink` (every
  example is a cell, 461), the decision cells linked where they show a point:
  - `cookie` — reading the `Cookie` header, writing `Set-Cookie` and the request header.
  - `accept` — the q-value grammar, `Accept`, `Accept-Encoding`, `Accept-Language` and their negotiation.
  - `mime` — the extension to `Content-Type` table.
  - `status` — the reason-phrase table.
  - `date` — HTTP-date, three forms read, one written.
  - `byteRange` — `Range: bytes=…` against a size, and `Content-Range`.
  - `cacheControl` — building and rendering a `Cache-Control` value.
- `modules/decisions-taken/<slug>-<N>/` — front 166's decision cells (decision 651: the subtree is that front's alone): one
  per code-bearing row of `../decisions-taken.md`, a `docs.md` (first line generated) and, in phase 1, `draft/main.bp` (decision
  649; no `src/`, no manifest, so `botopink cell check` finds no cell there); kept by `scripts/decision-cells.py` of the meta
  repository and its CI check 7.
- `modules/botopink.json` — the workspace manifest (`"*"`, the library's targets, `http` by path), front 166's.
