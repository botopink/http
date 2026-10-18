# Decision 330 — `?T` is read with TypeScript's operators and has no methods; `result` goes; a type declared in a type is its associated type.

The example reads a visitor's session and the date it was last seen from a `Cookie` header. `cookie.get` answers `?string`, and every optional is read with an operator, never a method. `Visit.read` returns `null` when there is no `SID`; past that `if` the session is a `string`, so `session!` there would be refused, `!` over a value that is never `null`. `Seen` is declared in the body of `Visit`, so it is `Visit`'s associated type, written `Visit.Seen` in the signature, in the constructor and in the type of `visits`.

`date.parseHttpDate` answers a `Result`, and the program reads it with that type's own methods, `at.isOk()` and `at.unwrapOr(nowMillis)`, and at the end `.isError()`; there is no `result` namespace to call instead. In `main`, `visits?.[0]` indexes an optional array and answers `?Visit.Seen`. An integer literal is `i32`, so `epoch`, the `i64` zero, comes from std's `clock.millis(0)`. `first?.session ?? "anonymous"` reads a member only when the visit is there. `first?.at` reads a member that is itself `?i64`, and the result is `?i64`, not `??i64`, because `?.` flattens. `greeting(true)?.("a1")` calls an optional function value, and `greeting(false)?.("a1") ?? "no greeting"` falls back because the function is `null`. `first!.session` takes the visit when the program knows it is there; a `null` at that point would stop the program with the location and the expression, the same on every target. The parentheses around `(first?.at ?? epoch) > epoch` are required: `??` next to `||` with no parentheses is a compile error, and so is `?.` on a value that is never `null` (`header?.length()`).

The program prints `a1`, `784111777000`, `anonymous`, `welcome back a1`, `no greeting`, `a1`, `true` and `true`.

Cells in this repository that prove it: none yet. The library already reads its optionals with `??` (`pieces.at(0) ?? ""` in `src/accept.bp`); its feature cells come with front 155's Conformance step (decision 465).

[draft/main.bp](draft/main.bp)
