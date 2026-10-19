# Decision 732 — (a): `Async<T>` is the one name of an `async` body's return type; `Task` is no name of the prelude.

Every body the example runs later is typed `Async<…>`, the one name of an `async` body's return type. `fetch` of std's `io.http` answers `Async<Result<http.Response, http.HttpError>>`, and `request` keeps that value without awaiting it, its type written out; `statusOf` is an `async fn` answering `Async<Result<i32, http.HttpError>>` and awaits it; `ready`, without the marker, returns an `Async<i32>` itself through `Async.resolve`. `Task` is no name of the prelude, so the program declares a record of its own called `Task` and nothing collides with it. The address is the local discard port, where nothing listens, so `fetch` answers a `Network` error: the program prints `write the report`, `network error` and `200 OK` from `status.reasonPhrase`.

Departure: the compiler still reads 553 (2)'s marker as `Task<T>`: `async fn … -> Async<…>` is refused as `marker-type-mismatch`, its hint naming `Task<T>`, so the example does not build until 144 B-137's codemod and the checker's rename land.

Cells in this repository that prove it: none yet. The library itself declares no `async` body; [fetch_one_typed_surface-623](../fetch_one_typed_surface-623/docs.md) writes `fetch`'s answer with the same name.

[draft/main.bp](draft/main.bp)
