# Decision 400 — A cookie's hooks are nouns in cardume's family — `cookieValue`, `cookieState`, `cookieSetter`, `cookieReset`.

The hooks are the frameworks' (rakun's and jhonstart's, front 150), and this library holds what each of them takes and does: the declared `Cookie<T>`. The example declares `sessionCookie` once and writes the four things the hooks spell, as `http` answers them. `valueOf` is what a page's `use cookieValue(sessionCookie)` hands it: `?SessionId`, `null` both when the header has no `session` and when its value does not decode, so a page never sees a malformed cookie. `setLine` is the `Set-Cookie` line a middleware, a route handler or an action writes through `use cookieSetter(sessionCookie)` or `cookieState`'s `.set(v)`, and `resetLine` the line `use cookieReset(sessionCookie)` sends; neither takes an attribute, which always comes from the declaration. The program prints `s-42`, `no session` twice, then the two lines with the declaration's `HttpOnly`, `Secure` and `SameSite`.

The names are nouns, in cardume's family (`atomValue`, `atomSetter`): `use` is the activation, so there is no `useCookieValue`. A page reads only: `cookieState`, `cookieSetter` or `cookieReset` in a render is refused at build.

Cells in this repository that prove it: none yet. `Cookie<T>`, `cookie.read`, `cookie.write` and `cookie.clear` arrive with front 155 s1; until that step lands, the example does not build. The page-hook imports (the frameworks' `use` hooks over a declared cookie or atom) wait on front 165, the rakun workspace that gathers those frameworks. The hooks come with front 150's jhonstart step 12.

[draft/main.bp](draft/main.bp)
