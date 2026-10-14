# adm.telemetry.datadog

A Datadog exporter for the `Telemetry` service, in ADM over `std.net.http`: metrics, logs,
events and traces to the intake APIs of a Datadog site, with no agent in between.

```bash
adm get admlang:adm.telemetry.datadog
```

```adm
use (
	std.services.telemetry::(Telemetry, TelemetryConfig)
	adm.telemetry.datadog
)

let dd = new datadog.Exporter(apiKey, site = "datadoghq.eu")   // DD_API_KEY, DD_SITE when left out
dd.service = "shop"
dd.env = "prod"
dd.version = "1.4.0"
dd.tags = ["team:payments"]
let config = new TelemetryConfig()
config.exporters = [dd]
try await Telemetry.start(config)
```

## Tests

`adm test telemetry/datadog` runs against a loopback intake. `ADM_TEST_DATADOG_KEY` and
`ADM_TEST_DATADOG_SITE` add sends to a real account under the service name `adm-test`.
