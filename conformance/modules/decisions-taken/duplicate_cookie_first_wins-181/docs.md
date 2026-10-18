# Decision 181 — † A duplicated cookie name: first wins, everywhere (RFC 6265 § 5.4); `rakun-session`'s `session_cookie` changes from las

The example reads one `Cookie` header that carries `SID` twice. `cookie.get("SID=first; SID=second", "SID")` answers `first`: the later duplicate, which is how a cookie-tossing attacker would override a session, is never read. The first occurrence decides even when it is refused: in `SID=%zz; SID=second` the first value is a malformed escape, so `SID` reads as absent (`?? "absent"` prints `absent`), never as `second`. `cookie.parse` answers each name once, its first value, in header order, so `theme=dark; SID=a1; theme=light; lang=pt` prints `theme=dark, SID=a1, lang=pt`. The last two lines write a header with `SID` twice through `cookie.formatHeader` (`SID=a1; SID=b2`) and read it back: the reader keeps `a1`. There is one reader in this library, so every consumer that reads cookies through it (rakun's session store included, whose last-wins reader goes with front 155 s2) answers the same.

Cells in this repository that prove it: none yet. The rule is tested in `test/cookie_test.bp` (suite `http:`); the library's feature cells come with front 155's Conformance step (decision 465).

[draft/main.bp](draft/main.bp)
