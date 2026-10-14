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
st.release = "shop@1.4.0"
st.environment = "prod"
st.sampleRate = 0.2                                // of transactions; errors always go
let config = new TelemetryConfig()
config.exporters = [st]
try await Telemetry.start(config)
```

## Tests

`adm test telemetry/sentry` runs against a loopback server. `ADM_TEST_SENTRY_DSN` adds sends to
a real project.
