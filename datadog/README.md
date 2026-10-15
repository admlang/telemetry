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
dd.serviceName = "shop"                                         // DD_SERVICE, else service.name
dd.env = "prod"                                                 // DD_ENV
dd.version = "1.4.0"                                            // DD_VERSION, else service.version
dd.tags = ["team:payments"]                                     // with DD_TAGS
try Telemetry.config(TelemetryConfig{exporters: [dd], requireConsent: false})
try await Telemetry.start()
```

Counters go as counts of what was added between batches, gauges and toggles as gauges, and
histograms as distributions. Log records go to the logs intake with the trace and span they
were written in, error reports as error logs with their kind, message and stack, and usage
events as Datadog events. Spans go to the trace intake, grouped by trace, with the APM stats
Datadog builds its request, error and latency metrics from; set `dd.sampleRate` to the
service's `TelemetryConfig.sampling` so those count every request. A span's `operation.name`,
`resource.name` and `span.type` attributes set the operation, resource and type; the span an
error was reported in goes as failed, with the error.

`dd.intake = "https://proxy.internal:3834"` sends every request through a proxy instead of the
site's hosts.

## Tests

`adm test telemetry/datadog` runs against a loopback intake. `ADM_TEST_DATADOG_KEY` and
`ADM_TEST_DATADOG_SITE` add sends to a real account under the service and environment
`adm-test`.
