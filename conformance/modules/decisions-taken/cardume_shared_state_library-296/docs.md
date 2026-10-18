# Decision 296 — `cardume` is a library of its own

The example puts the two kinds of declared state side by side. `cartCount` is a cardume atom, `import {atom} from "cardume";`: cardume is a library of its own and its core imports neither framework, so a plain program can declare one. `sessionCookie` is a `Cookie<SessionId>`, imported from `http`: the cookie stays in `http` and does not move into cardume. Each is identified by its declaration, with no string key; the cookie's `"session"` is the browser's name, not a key into a store. Reading the atom's value needs a store, and the stores are the frameworks' bridges (`rakun-cardume` per request, `jhonstart-cardume` per page), so the program only reads the cookie: `s-42`.

Cells in this repository that prove it: none yet. `Cookie<T>`, `cookie.read`, `cookie.write` and `cookie.clear` arrive with front 155 s1; until that step lands, the example does not build. The page-hook imports (the frameworks' `use` hooks over a declared cookie or atom) wait on front 165, the rakun workspace that gathers those frameworks. The `cardume` repository holds no code yet (front 136).

[draft/main.bp](draft/main.bp)
