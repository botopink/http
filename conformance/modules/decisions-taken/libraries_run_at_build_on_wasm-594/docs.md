# Decision 594 — Every library runs at build on the wasm comptime runtime (434), jhonstart first; a function only Erlang or Node can run is a located error there.

The example writes four headers inside `comptime`, which runs at build on the wasm comptime runtime whatever `--target` the build uses, and splices each answer into the program as a literal. `cookie.serialize` (with std's `encoding.percentEncode`), `status.reasonPhrase`, `cacheControl.staticFile`, `accept.qValue` and `mime.contentTypeOf` are written in botopink, so each has a wasm form. The line written at build is the one written at run (`line == cookie.serialize(…)` prints `true` on every target). The program prints `theme=dark; Path=/; Max-Age=3600; Secure; SameSite=Lax`, `true`, `Too Many Requests`, `public, max-age=31536000, immutable`, `125` and `application/wasm`.

The library has one host cell, `wide` in `src/lexical.bp`, a template on erlang and JavaScript only, and `date` and `byteRange` reach it. A `comptime date.formatHttpDate(…)` is therefore refused at build, at the call, naming `lexical.wide` as the cell with no wasm form, on an erlang build as on any other, until the library gives `wide` a wasm form (the decision's point (2), a step of front 155).

Cells in this repository that prove it: none yet. Its feature cells come with front 155's Conformance step (decision 465).

[draft/main.bp](draft/main.bp)
