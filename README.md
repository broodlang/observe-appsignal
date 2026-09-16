# observe-appsignal

[AppSignal](https://www.appsignal.com) as an [`observe`](../observe) provider for
[Brood](https://broodlang.org): logs, metrics and errors over AppSignal's
[public HTTP endpoints](https://docs.appsignal.com/api/public-endpoint/). No agent, no
extension — three wire formats over `http`.

## Usage

```brood
(observe/start {:provider (observe-appsignal/provider
                            {:api-key (os/env "APPSIGNAL_FRONTEND_KEY")   ; errors + metrics
                             :log-key (os/env "APPSIGNAL_LOG_KEY")        ; logs
                             :group "myapp"})
                :env "prod" :revision (os/env "FLY_MACHINE_VERSION") :app "myapp"})
(log/add-backend (observe/log-backend {:min-level :info}))
(observe/attach-hatch)
```

AppSignal uses two keys and so does this: the app's **front-end API key** (App settings →
Push & deploy) for `/errors` and `/metrics/json`, and a **log source key** (Logging → add a
source of type *HTTP JSON*) for `/logs/json`. Without `:log-key`, logs are not sent at all
(quietly — pair with `observe-local` to keep them) and never under the wrong key.

What lands where:

| observe | AppSignal | shape |
|---|---|---|
| log record | `/logs/json` (NDJSON) | `timestamp` RFC 3339, `group`, `severity`, `message`, `hostname`, `attributes` (the record's `:meta`, scalar, ≤100, plus `env` and `revision`) |
| metric | `/metrics/json` | `name` (sanitised to `[a-zA-Z0-9-_]`), `metricType` `gauge`/`counter`/`timing`, `value`, string `tags` |
| error report | `/errors` (one per request) | `timestamp`, `action`, `namespace`, `error {name message backtrace}`, `revision`, `tags`, `params`, `environment {hostname env app}`, `language "brood"` |

`:transport` (a `(fn (url body) -> response)`) replaces `http/post` for a test; the suite
runs against a recording one and never touches the network.

## Development

`nest fetch` resolves `../observe`. The suite is `tests/observe-appsignal_test.blsp`;
`nest format` before committing.
