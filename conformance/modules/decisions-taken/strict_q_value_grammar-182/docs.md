# Decision 182 — † One strict q-value grammar for media, encoding and language: at most three decimals, digits only; anything malformed r

The example runs ten weights through `accept.qValue`, which answers per mille. `1`, `0.8`, `0.125` and `1.000` are the grammar's own forms and print `1000`, `800`, `125` and `1000`. The other six are malformed and each prints `0`: `0.1234` has four decimals, `1.5` is above one, `.5` has no leading digit, ` 0.5` has a space, `0.5a` a letter and `-0` a sign. Nothing is rounded or trimmed into a weight.

The last three lines show that media, encodings and languages read the same grammar. In `application/json;q=0.0001, text/html;q=0.5` the JSON weight is malformed, so it weighs 0 and `negotiateMedia` picks `text/html`. In `gzip;q=1.5, br;q=0.9` `gzip` weighs 0 and `negotiateToken` picks `br`. `parseAcceptLanguage("pt-BR;q=0.9, en;q=0.95555, fr")` prints `fr:1000, pt-BR:900, en:0`: an element without `q` weighs 1000, and `en`'s five decimals give it 0, which excludes it.

Cells in this repository that prove it: none yet. The grammar is tested in `test/accept_test.bp` (suite `http:`); the library's feature cells come with front 155's Conformance step (decision 465).

[draft/main.bp](draft/main.bp)
