# Decision 346 — A `Bytes` primitive:

The example serves two files whose bodies are `bytes`, the type a binary payload has where it enters a program. `bytes.fromUtf8("hello")` encodes a text into bytes; there is no other way from `string` to `bytes`, and `val b: bytes = "hello";` is a type mismatch, not a conversion. `describe` asks the library which media type the path has (`mime.contentTypeOf`) and decodes the body only when `mime.isText` says it is text: `body.toUtf8()` answers a `Result`, `Ok(text)` here and an `Error` for bytes that are not UTF-8, so the program prints `hello.txt: text/plain; charset=utf-8, hello`. The PNG body is never turned into a `string`: it prints `pixel.png: image/png, binary`. On erlang the bytes are a binary, on commonJS a `Uint8Array`, and in wasm a buffer in memory.

Cells in this repository that prove it: none yet. The library holds no `bytes` yet: its cookie writer refuses a value outside printable ASCII because "there is no byte type" (`src/cookie.bp`), and its HTTP/1.1 head parsers wait on the type (`AGENTS.md`). They come once 144 B-12 (`bytes`) lands, with front 155.

[draft/main.bp](draft/main.bp)
