# ADM telemetry exporters

Official [ADM](https://github.com/admlang/adm) libraries that send what the `Telemetry` service
of `std.services.telemetry` collects (metrics, logs, errors, usage events, spans) to a vendor.
The service itself ships OTLP, file and console exporters; a vendor's own ingestion API changes
with the vendor, so each is a library, written in ADM over `std.net.http`, published as
`adm.telemetry.<vendor>` and signed by the `admlang` publisher.

| Library | Vendor | What it sends |
|---|---|---|
| [`adm.telemetry.datadog`](datadog/) | Datadog | metrics (series and distributions), logs, events, traces |
| [`adm.telemetry.sentry`](sentry/) | Sentry, GlitchTip and self-hosted servers | errors as events with stack traces, spans as transactions, check-ins for crons |

A vendor that takes OTLP (Honeycomb, Grafana Cloud, New Relic, Dynatrace, Elastic) needs no
library: point the service's `OTLPExporter` at its endpoint with its headers.

## Shared shape

Each library is one exporter type the program adds to the service's configuration. It batches
what the service hands it, keeps to the vendor's limits and `Retry-After` pauses, drops what
the vendor refuses for good, and maps the service's records onto the vendor's conventions
(resource attributes to tags, the error's type and message to the vendor's exception fields).
Nothing else in the program changes between vendors.

## Tests

Each suite runs against a loopback server that answers as the vendor does. With a key the
suite also sends to a real account, under a service name of its own:

```bash
ADM_TEST_DATADOG_KEY=... ADM_TEST_DATADOG_SITE=datadoghq.eu adm test telemetry/datadog
ADM_TEST_SENTRY_DSN=https://...@o1.ingest.sentry.io/1 adm test telemetry/sentry
```
