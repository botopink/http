# Decision 623 — (a): `io/http`'s `fetch` has one typed surface, and `fetch(url)` is removed in the same commit.

The example calls std's one `fetch` and reads the answer with this library's codecs. `import {io.http} from "std";` binds std's `http` module; the `http` package of the imports below is a source string, not a name, so the two never meet. A short GET is the request record with its defaults, `http.Request(method: .Get, url: …)`; `login` fills the other fields, a `Cookie` header written with `cookie.formatHeader` and a `bytes` body, and propagates a failure with `try await`. `fetch` answers `Result<http.Response, http.HttpError>`, and `report` matches the error by variant: `NotAllowedOnHost`, `Network(_)` and `Timeout`, with no text to compare. An HTTP error status is a response, `Ok` with its status, which `status.reasonPhrase` names (`404 Not Found`).

The address is the local discard port, where nothing listens, so both calls answer a `Network` error and the program prints `network error` twice. `http.fetch("http://127.0.0.1:9/")`, the removed `fetch(url)`, is refused by the checker's own type error, `expected Request, got string`.

Cells in this repository that prove it: none yet. The library itself never calls `fetch`; the std cells `http_fetch_request` and `http_fetch_url_removed` are botopink-lang's.

[draft/main.bp](draft/main.bp)
