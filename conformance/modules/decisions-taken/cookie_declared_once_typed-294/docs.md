# Decision 294 — A cookie is declared once, typed:

The example declares two cookies once each, as module values, with everything the browser sees written in the declaration: `sessionCookie` is a `Cookie<SessionId>` named `"session"`, HTTP-only, secure, for thirty days; `themeCookie` a `Cookie<Theme>` that scripts may read. The name is a string, the browser's, written once; no call later repeats it or an attribute. `T` is a type the library can encode: `SessionId` is a one-field record and `Theme` an enum.

`cookie.read(decl, header)` answers `?T`: the session as a `SessionId` (`s-42`), the theme as `Theme.Dark`, the first occurrence deciding (181), and `null` for `theme=Blue`, which does not decode to a `Theme`. `cookie.write(sessionCookie, …)` writes the `Set-Cookie` line with the declaration's attributes, and `cookie.clear(themeCookie)` the line that expires it. A page reads such a cookie through the framework's hooks (decision 400, `use cookieValue(sessionCookie)`), and the pair jar and `pairValue(jar, "…")` leave page code. The age is written `days(30)`, as the decision writes it, imported by its leaf path `import {io.clock.days} from "std";`, beside std's other duration builders (`millis`, `seconds`, `minutes`, `hours`). The departure: std's `io.clock` has no `days` yet, so the example does not build until it does.

Cells in this repository that prove it: none yet. `Cookie<T>`, `cookie.read`, `cookie.write` and `cookie.clear` arrive with front 155 s1; until that step lands, the example does not build.

[draft/main.bp](draft/main.bp)
