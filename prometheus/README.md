# adm.telemetry.prometheus

A Prometheus exporter for the `Telemetry` service, in ADM over `std.net.http`: the program's
metrics pushed by Remote Write or to a Pushgateway, or served for scraping on a listener of
their own.

```bash
adm get admlang:adm.telemetry.prometheus
```

```adm
use (
	std.services.telemetry::(Telemetry, TelemetryConfig)
	adm.telemetry.prometheus
)

let metrics = new prometheus.Exporter("https://prometheus.internal/api/v1/write")
metrics.headers["Authorization"] = "Bearer {token}"
try Telemetry.config(TelemetryConfig{exporters: [metrics], requireConsent: false})
try await Telemetry.start()
```

`Exporter` speaks remote write 1.0, or 2.0 with `metrics.version = prometheus.RemoteWriteVersion.V2`,
to Prometheus (`--web.enable-remote-write-receiver`), Mimir, Cortex, Thanos Receive,
VictoriaMetrics or Grafana Cloud. Counters go as `<name>_total`, gauges and toggles as gauges,
histograms as classic buckets and, with `metrics.nativeHistograms = true`, as native histograms.
Names take their unit (`_seconds`, `_bytes`); every series carries `job` and `instance`, and
the other resource attributes go on `target_info`.

A job that ends before a scrape pushes to a Pushgateway instead:

```adm
let gateway = new prometheus.Pushgateway("http://pushgateway:9091", job = "nightly-import")
// ... and once the job is done:
try gateway.delete()
```

Scraping instead of pushing: `Endpoint` serves `/metrics` on a listener of its own,
`127.0.0.1:9464` unless given another address. Any other address needs a bearer token or
required client certificates; it answers in protobuf (with native histograms), OpenMetrics or
the text format, as the scraper asks.

```adm
let page = new prometheus.Endpoint("0.0.0.0:9464")
page.token = scrapeToken                          // Prometheus: authorization.credentials
try page.start()
```

## Tests

`adm test telemetry/prometheus` runs remote write, a push and scrapes against loopback servers. `ADM_TEST_PROMETHEUS` names a Prometheus server started with
`--web.enable-remote-write-receiver`; the suite then writes series under the job `adm-test` and
reads them back through the query API.
