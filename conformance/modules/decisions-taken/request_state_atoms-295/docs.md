# Decision 295 — Request state is atoms, read and written through `use`

Request state is atoms, and cookies follow the same pattern, which is how this library meets the decision: a value's identity is its declaration, never a string key read at run time. The example declares two cookies of different types, `sessionCookie` and `localeCookie`, and reads both from one header by declaration: `cookie.read(sessionCookie, header)` answers `?SessionId` and `cookie.read(localeCookie, header)` `?Locale`, each typed by its declaration, so the program prints `s-42` and `pt-BR`. Nothing like `LocalKey<T>(name)` exists, so two parts of a program cannot meet on one name at run time: a second declaration is a second value, and the name `"session"` is written once, in its declaration. The atoms themselves (`atom(…)`, `use atomSetter(currentUser)` in a middleware, `use atomValue(currentUser)` in a page) are cardume's and rakun's.

Cells in this repository that prove it: none yet. `Cookie<T>`, `cookie.read`, `cookie.write` and `cookie.clear` arrive with front 155 s1; until that step lands, the example does not build. The page-hook imports (the frameworks' `use` hooks over a declared cookie or atom) wait on front 165, the rakun workspace that gathers those frameworks. The atoms come with front 136 (cardume) and rakun's request store with front 150.

[draft/main.bp](draft/main.bp)
