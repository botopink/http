# http — conformance

> Path: `repository/http/conformance/`
> Sibling (AGENTS): [`./AGENTS.md`](AGENTS.md)

This reference describes `http`, the codecs of HTTP semantics that rakun and `rakun-ssr` both read and write
(decision 196): the `Cookie` and `Set-Cookie` lines, content negotiation, the extension to media-type table, the
reason phrases, HTTP-dates, byte ranges and `Cache-Control`. It is pure botopink, compiled from one source for erlang
and commonJS, so a server on one target and a client on the other agree byte for byte. A program imports a module as
a namespace with `import {cookie, accept} from "http";` and each name from the module that declares it
(decisions 206, 337, 763). Every example is a conformance cell (decision 461); the library holds no feature cell yet
— they come with front 155's Conformance step (decision 465) — and the decision cells below show the points the
decisions fix, each as a `draft/main.bp` until front 166 promotes it to a cell (decision 649). A table is fenced as
`text`.

| Section | What |
|---|---|
| *cookie* | reading a `Cookie` header, writing a `Set-Cookie` line and a `Cookie` request header |
| *accept* | `Accept`, `Accept-Encoding` and `Accept-Language`: the weight, the ranges, the negotiation |
| *mime* | the one extension to `Content-Type` table |
| *status* | the one reason-phrase table |
| *date* | HTTP-date, read in three forms and written in one |
| *byteRange* | a `Range: bytes=…` request against a size, and the `Content-Range` of the answer |
| *cacheControl* | a `Cache-Control` response value, built and rendered |
| Decisions | [`modules/decisions-taken/`](modules/decisions-taken/): The ledger's index: one `<slug>-<N>/` per code-bearing row of this repository's generated [`../decisions-taken.md`](../decisions-taken.md) (front 166, decisions 646–651), its `docs.md` walking through one example program, `draft/main.bp` until front 166 s4 promotes it to a cell (649) |

The package declares `pub mod` lines in `src/root.bp`, an `"entry"` and a `"files"` list in `botopink.json`, and its
sources write `Array<T>`; the language has retired all three (every `.bp` under `src/` is a module, a library has no
entry, the manifest names no files, and the type is `T[]`; decisions 592, 733, 703, 641). The library's codemod is
pending, and this reference describes the API in the spelling in force. The module `lexical` (character classes) is
internal: `root.bp` does not export it.

The decision cells that show the library as a whole: [http_one_cookie_reader-196](modules/decisions-taken/http_one_cookie_reader-196/draft/main.bp) (a request handled through
`cookie`, `accept`, `mime` and `status`), [own_module_import_without_from-206](modules/decisions-taken/own_module_import_without_from-206/draft/main.bp) and
[shorthand_import_refused-337](modules/decisions-taken/shorthand_import_refused-337/draft/main.bp) (how a module of a package is named), [libraries_run_at_build_on_wasm-594](modules/decisions-taken/libraries_run_at_build_on_wasm-594/draft/main.bp)
(a library runs at build on the wasm comptime runtime) and [fetch_one_typed_surface-623](modules/decisions-taken/fetch_one_typed_surface-623/draft/main.bp) (the typed
`fetch(Request(…))` that replaces `fetch(url)`; the call sites of the `http` library move with it).

## cookie

The `Cookie` request header and the `Set-Cookie` line (RFC 6265). Decision 196 makes this module the one cookie
reader of rakun and `rakun-ssr`: rakun's four `Cookie:` readers and the server's are deleted for it
([http_one_cookie_reader-196](modules/decisions-taken/http_one_cookie_reader-196/draft/main.bp)).

`cookie.CookieAttributes(path, domain, maxAge, httpOnly, secure, sameSite)` holds the attributes of one `Set-Cookie`
line. `maxAge` is a `?i32`: `null` omits `Max-Age` (a session cookie), `0` or less expires the cookie on arrival.
`domain` and `sameSite` empty omit the attribute.

Reading:

- `cookie.parse(header)` answers every `name=value` pair of a `Cookie` header as `#(string, string)` tuples, in header
  order, each name once and without the refused ones.
- `cookie.get(header, name)` answers the value of one cookie as `?string`: `null` when the header does not carry it,
  when its first occurrence is refused, or when `name` is not an RFC 9110 token. The result is read with the `??`
  operator, never with a method ([optional_typescript_operators-330](modules/decisions-taken/optional_typescript_operators-330/draft/main.bp)).

The header is split on `;`; each chunk is trimmed of SP and HTAB, and an empty chunk is skipped. A chunk is a pair
when its name, the text before the first `=`, is an RFC 9110 token and its value is an RFC 6265 `cookie-value` —
cookie-octets, optionally inside one pair of `"` (stripped) — that percent-decodes to a text with no control
character. A chunk with no `=` is not a cookie. A chunk that breaks another rule is refused, and a refused chunk
still claims its name: the first occurrence of a name decides (RFC 6265 § 5.4, decision 181), so `a=%zz; a=2` reads
`a` as absent and never as `2`; a later duplicate is how a cookie-tossing attacker overrides a value
([duplicate_cookie_first_wins-181](modules/decisions-taken/duplicate_cookie_first_wins-181/draft/main.bp)). Names are case-sensitive. The escapes are checked in the library, so
a trailing lone `%` is refused on both targets alike.

Writing:

- `cookie.serialize(name, value, attrs)` answers one `Set-Cookie` field value: `name=<percent-encoded value>`, then
  `Path`, `Domain`, `Max-Age`, `HttpOnly`, `Secure` and `SameSite` in that order, each only when set. The value is
  percent-encoded, so every value round-trips through `parse`.
- `cookie.formatHeader(pairs)` answers a `Cookie` request header carrying the pairs in order, `name=value; …`, each
  value encoded as `serialize` does. A name given twice is written twice; the reader keeps the first.

Writing never answers an `Error`: a header line that cannot be written correctly is a programming error, so the call
panics instead of producing a line. It panics for a name that is not a token; a value outside printable ASCII (there
is no byte type in a cookie, so encode the value first, with base64url for instance); a `Path` with `;` or a
character outside printable ASCII; a `Domain` that is not a host name; a `SameSite` other than `Strict`, `Lax` or
`None`; `SameSite=None` without `Secure`; a `__Secure-` name without `Secure`; and a `__Host-` name without `Secure`,
`Path=/` and an empty `Domain` (RFC 6265bis § 4.1.3).

Decision 294 puts a typed `Cookie<T>` in this package, declared once (`Cookie<SessionId>("session", httpOnly: true,
…)`), with hooks that are nouns in cardume's family (decision 400, `cookieValue`, `cookieState`, `cookieSetter`,
`cookieReset`) over request state held as atoms (decision 295). The module does not declare `Cookie<T>` yet; what it
offers is the codec those declarations call. The cells [cookie_declared_once_typed-294](modules/decisions-taken/cookie_declared_once_typed-294/draft/main.bp),
[cookie_hooks_are_nouns-400](modules/decisions-taken/cookie_hooks_are_nouns-400/draft/main.bp) and [request_state_atoms-295](modules/decisions-taken/request_state_atoms-295/draft/main.bp) show the decided surface.

## accept

Proactive negotiation (RFC 9110 § 12): `Accept`, `Accept-Encoding` and `Accept-Language`. Decision 196 makes this
module the one q-value parser; the three q-value parsers are deleted for it
([http_one_cookie_reader-196](modules/decisions-taken/http_one_cookie_reader-196/draft/main.bp)).

A quality is read in thousandths (`q=0.5` is 500), so every comparison is integral. One grammar reads it for media,
encoding and language alike (decision 182): `qvalue = ( "0" [ "." 0*3DIGIT ] ) / ( "1" [ "." 0*3("0") ] )`, at most
three decimals, digits only, no sign and no whitespace inside. Anything malformed reads as 0, so a range the client
did not clearly accept is never chosen ([strict_q_value_grammar-182](modules/decisions-taken/strict_q_value_grammar-182/draft/main.bp)). An element without a `q` weighs 1000. A
list element that does not parse — a media range that is not `type/subtype` tokens (`*/html` included), a parameter
that is not `token=token` or `token="quoted"`, a coding that is not a token, a language range outside RFC 4647
§ 2.1 — is dropped and never repaired.

- `accept.qValue(text)` answers the per-mille value of a `qvalue` text: `1` is 1000, `0.8` is 800, `1.000` is 1000;
  a malformed text (an empty one, ` 0.5`, `.5`, `0.1234`, `1.5`, `2`, `0.5a`, `-0`) is 0.
- `accept.MediaRange(mainType, subType, params, q)` is one element of an `Accept` header. `mainType` and `subType` are
  lowercase (`*` for a wildcard); `params` are the media-type parameters before the weight, names lowercase and values
  unquoted (a `charset` value lowercase); `q` is per mille.
- `accept.parseAccept(header)` answers the media ranges in header order, without the ones that do not parse.
- `accept.mediaQuality(accept, mediaType)` answers the weight the header gives one media type, through its most
  specific matching range, the first of equally specific ones. An empty header is `*/*` (1000, RFC 9110 § 12.5.1);
  a `mediaType` that is not a concrete `type/subtype` weighs 0. A range matches when its types match and the media type
  carries every parameter of the range with the same value; specificity is 1 for `*/*`, 2 for `type/*`, 3 for
  `type/subtype`, plus 10 per parameter of the range.
- `accept.negotiateMedia(accept, offered)` answers the offered media type the header weighs highest, ties going to the
  order of `offered`, and `""` when it accepts none of them.
- `accept.tokenQuality(header, token)` answers the weight a token list such as `Accept-Encoding` gives a token,
  compared case-insensitively: its own element, else `*`, else 0. `identity` is the exception RFC 9110 § 12.5.3 makes:
  1000 unless the header excludes it (`identity;q=0`, or `*;q=0` without an `identity` element), so an empty header
  accepts `identity` only.
- `accept.negotiateToken(header, offered)` answers the offered token the header weighs highest, ties going to the order
  of `offered`, `""` when every weight is 0.
- `accept.LanguageRange(tag, q)` is one element of an `Accept-Language` header: the range as written (its case is not
  normalised here, the locale grammar is the `i18n` package's) and its weight.
- `accept.parseAcceptLanguage(header)` answers the ranges highest weight first and in header order among equal
  weights. A range weighing 0 is kept, because it excludes; the caller skips it when choosing.

## mime

The one extension to media-type table. The type is chosen by extension, case-insensitively, never by sniffing the
bytes. A text type carries `; charset=utf-8`, every other type none. An unknown extension is
`application/octet-stream`, the type that asks a browser to download rather than render, so an unlisted file is never
executed as a guess ([http_one_cookie_reader-196](modules/decisions-taken/http_one_cookie_reader-196/draft/main.bp)).

- `mime.extensionOf(pathOrExt)` answers the extension a path or an extension names, lowercase and without the dot: the
  text after the last `.` of the last `/` segment (`a/b.min.JS` is `js`, `.png` is `png`), or the whole segment when it
  has no `.` (`README` is `readme`).
- `mime.contentTypeOf(pathOrExt)` answers the `Content-Type` of a path or an extension: `index.html` is
  `text/html; charset=utf-8`, `logo.PNG` is `image/png`, `x.bin` is `application/octet-stream`.
- `mime.isText(mediaType)` is true for a `text/*` media type, parameters and case ignored.

The table covers `html`, `htm`, `css`, `js`, `mjs`, `txt`, `json`, `map`, `xml`, `svg`, `png`, `jpg`, `jpeg`, `gif`,
`webp`, `avif`, `ico`, `woff`, `woff2`, `wasm` and `pdf`.

## status

The one reason-phrase table. `status.reasonPhrase(code)` answers the phrase RFC 9110 § 15 registers for a status code,
plus 103 (RFC 8297), 425 (RFC 8470), 428, 429, 431 and 511 (RFC 6585) and 451 (RFC 7725): `404` is `Not Found`. Any
other code answers `""` — 418, which § 15.5.19 leaves unused, included — because RFC 9112 § 4 makes the reason phrase
optional, and a phrase invented for a code is a wire text two servers would spell differently
([http_one_cookie_reader-196](modules/decisions-taken/http_one_cookie_reader-196/draft/main.bp)).

## date

HTTP-date (RFC 9110 § 5.6.7), in epoch milliseconds (`i64`). The epoch is computed in botopink, a days-from-civil
count with no bitwise operator, so both targets answer the same instant.

- `date.formatHttpDate(epochMillis)` answers the one form a sender generates, IMF-fixdate
  (`Sun, 06 Nov 1994 08:49:37 GMT`), truncated to the second over std's UTC calendar. A year outside 0000–9999 has no
  four-digit spelling and panics.
- `date.parseHttpDate(text, nowMillis)` answers `Result<i64, string>`: the epoch milliseconds the text names, or an
  `Error` naming the text and what is wrong with it. It reads the three forms a recipient accepts — IMF-fixdate, the
  obsolete RFC 850 form (`Sunday, 06-Nov-94 08:49:37 GMT`) and asctime (`Sun Nov  6 08:49:37 1994`) — exactly as the
  grammar spells them: case-sensitive names, fixed widths and single spaces, `GMT`, a real calendar day, and a day
  name that agrees with the date. Anything else is an `Error`; a validator a server cannot read is ignored by its
  caller, never guessed at. `nowMillis` is read only for the two-digit year of an RFC 850 date (the century of the
  current year, or the one before when that is more than 50 years ahead), so the function stays pure; a caller passes
  the clock's reading.

## byteRange

A `Range: bytes=…` request against a representation size (RFC 9110 § 14). One range only. Sizes and offsets are
`i64`, so a file larger than 2 GiB is an ordinary file.

`byteRange.ByteRange(status, first, last)` is the answer, and `status` is what the server sends:

```text
206  first..last, inclusive, last clamped to size - 1, for a satisfiable first-last, first- or -suffix
416  the one range is valid but starts at or past the end (first >= size), is the suffix -0,
     or the representation is empty
200  ignore the header and send the whole representation: no header, a unit other than bytes,
     more than one range, and anything that does not parse (whitespace inside the range, a sign,
     last < first, a number past the i64 range)
```

`first` and `last` are -1 for 200 and 416. `byteRange.parseRange(header, size)` answers the `ByteRange`;
`byteRange.contentRange(r, size)` answers its `Content-Range`: `bytes 0-499/1234` for 206, `bytes */1234` for 416 and
`""` for 200. A range unit the server does not understand is ignored (§ 14.2), and so is a range set, since a
`multipart/byteranges` body is not built.

## cacheControl

A `Cache-Control` response value, built (RFC 9111 § 5.2.2). Start from `cacheControl.directives()`, which sets
nothing; add with the `with*` functions, each answering a new `CacheDirectives`; write with `cacheControl.render(d)`.

- Scope and ages: `withPublic`, `withPrivate`, `withMaxAge(d, seconds)` and `withSharedMaxAge(d, seconds)`. An age is
  a number of seconds, 0 or more.
- Flags: `withNoStore`, `withNoCache`, `withMustRevalidate`, `withProxyRevalidate`, `withNoTransform` and
  `withImmutable` (RFC 8246).
- `cacheControl.render(d)` writes the directives in one order, joined by `, `, so two builders that set the same
  directives write the same bytes; it answers `""` when nothing is set (send no header):

```text
public | private, no-store, no-cache, max-age, s-maxage,
must-revalidate, proxy-revalidate, no-transform, immutable
```

- `cacheControl.staticFile(maxAge, immutable)` answers the policy of a static file: `no-cache` (store, but revalidate
  every use) for an age of 0 or less, else `public, max-age=<age>` with `, immutable` when the file's name changes
  with its content.

Contradictions panic rather than reach a cache: `public` with `private`, a negative age, `immutable` without a
`max-age` to be immutable for, and `no-store` with anything that keeps a copy (`public`, `max-age`, `s-maxage`,
`immutable`).
