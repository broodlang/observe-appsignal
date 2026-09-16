# observe-appsignal — guidance for Claude

AppSignal's three public endpoints as an `observe` `Provider`. Batching, buffering and the
request worker are `observe`'s — do not add any here. The module is the wire formats plus
`post`; read its header for the two-key trap.

- The endpoints answer 401 for a bad key (observed 2026-09-16) and 200 for a body they will
  later drop silently, so a shape error is invisible in production: the tests in
  `tests/observe-appsignal_test.blsp` are the only check on the payloads. Change a field
  only with the docs page open (`docs.appsignal.com/api/public-endpoint/…`,
  `…/logging/endpoints/http-json`).
- Metric names must match `[a-zA-Z0-9-_]`; `metric-name` repairs rather than drops. Brood's
  regex has no `[a-z]` ranges — it is written over codepoints on purpose.
- `json/decode` yields keyword keys; the tests assert on `:name`, not `"name"`.
- Run the one test file (the full suite is blocked on this machine) and `nest format`
  before committing. No trailing `!`, no abbreviations.
