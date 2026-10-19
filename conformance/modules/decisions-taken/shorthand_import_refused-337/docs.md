# Decision 337 — The shorthand import goes; `mod m;` binds the namespace `m` in the module that declares it.

The example prints the session a request carries. `draft/session.bp` is a module of the package, so it declares `sessionOf` as `pub` and reads the header with the library's `cookie.get`. `main` imports that function by the module's path: `import {session.sessionOf};`, the module first and then the item, with no `from`. Every other import names its source: `import {cookie.get} from "http";` reaches the library and `io.terminal` comes `from "std"`. The program prints `a1`, then `anonymous` for a header with no `SID`.

The refused spelling is the shorthand. `import {sessionOf};` has no `from`, and its first segment does not name a module of the package. It is refused with `shorthand-import` at the item. Only `session` declares `sessionOf` as `pub`, so the error gives the fix: `write import {session.sessionOf};`, which is the line the example uses. A bare `import {get};` does not reach the library either, even though `http` declares that name; it has to be written `from "http"`.

337 (2)'s `mod m;` namespace is replaced by decision 592 (1): every `.bp` under `src/` is a module, its path its name, and there is no `mod` / `pub mod`; with 337 (3) there is no combined form and no re-export syntax. The library's own `src/root.bp` still lists its modules with `mod` / `pub mod`; that goes with the codemod of decision 592 over the library.

Cells in this repository that prove it: none yet. The library's modules already import each other by path (`import {lexical.sub};` in `src/mime.bp`); its feature cells come with front 155's Conformance step (decision 465).

[draft/main.bp](draft/main.bp)
[draft/session.bp](draft/session.bp)
