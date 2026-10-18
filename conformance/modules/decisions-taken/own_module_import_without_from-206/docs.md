# Decision 206 — `import {…} from "<name>"` resolves only to a package — std, a bundled package or a declared dependency.

The example has a module of its own named `cookie` (`draft/cookie.bp`, which holds `describe`), and the `http` library it depends on has a `cookie` module too, so each import line says which one it means. `import {cookie.formatHeader, cookie.get} from "http";` names a package, so it reaches the library's `cookie` module and never the module beside `main`, even though that module has the same name. `import {cookie.describe};` is the brace form with no `from`: the module's path, then the item, so it reaches this package's own `cookie.bp`. Every `.bp` file of the package is one of its modules, named by its path, so nothing declares it. `io.terminal` comes `from "std"`, a package like `http`.

`main` writes a header with the library's `formatHeader`, reads it with the library's `get` and prints it through the package's `describe`: `SID is a1`, then `theme is absent`. Had the example written `import {cookie.describe} from "http";`, the import would look for `describe` in the library, which does not declare it. Writing `from` with a module of this package as the source (`import {describe} from "cookie";`) is refused with `module-import-with-from` at the source string, and the fix is the brace form the example uses.

Cells in this repository that prove it: none yet. The library's own modules already import each other in the brace form (`import {lexical.charOf, …};` in `src/cookie.bp`); its feature cells come with front 155's Conformance step (decision 465).

[draft/main.bp](draft/main.bp)
[draft/cookie.bp](draft/cookie.bp)
