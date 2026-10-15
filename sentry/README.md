# adm.telemetry.sentry

A Sentry exporter for the `Telemetry` service, in ADM over `std.net.http`: errors as Sentry
events with stack traces and breadcrumbs, spans as transactions, and check-ins for scheduled
jobs. It speaks the envelope protocol, so it works with sentry.io, self-hosted Sentry and
GlitchTip.

```bash
adm get admlang:adm.telemetry.sentry
```

```adm
use (
	std.services.telemetry::(Telemetry, TelemetryConfig)
	adm.telemetry.sentry
)

let st = new sentry.Exporter(dsn)                 // SENTRY_DSN when left out
st.release = "shop@1.4.0"                          // SENTRY_RELEASE, else service@version
st.environment = "prod"                            // SENTRY_ENVIRONMENT, else production
st.sampleRate = 0.2                                // of traces; errors always go
try Telemetry.config(TelemetryConfig{exporters: [st], requireConsent: false})
try await Telemetry.start()
```

Errors given to `Telemetry.report` (and records written with `Logging.fatal`) become events with
their stack, the log records and usage events before them as breadcrumbs, and their fields as
extra data; `user.id`, `user.email`, `user.name` and `client.address` fill the user,
`url.full`, `http.request.method` and `url.query` the request. Spans become transactions: a span
with no parent, or with a parent from another program's `traceparent`, is one, with the spans
inside it as children; its `sentry.op` attribute is the operation. With `st.logs = true` log
records also go as Sentry logs.

A scheduled job reports its runs as check-ins:

```adm
let nightly = st.monitor("nightly-import")
nightly.crontab = "0 3 * * *"                      // creates or updates the monitor in Sentry
nightly.maxRuntime = 30min
try nightly.run(importOrders)                      // in progress, then ok or error
```

## Tests

`adm test telemetry/sentry` runs against a loopback server. `ADM_TEST_SENTRY_DSN` adds sends to
a real project (Sentry or GlitchTip): an error, a transaction, a log record and a check-in under
the environment `adm-test`.
