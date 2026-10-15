# adm.telemetry.statsd

A StatsD exporter for the `Telemetry` service, in ADM: the program's metrics sent as StatsD
lines to Graphite, Telegraf, Netdata or any other StatsD server.

```bash
adm get admlang:adm.telemetry.statsd
```

```adm
use (
	std.services.telemetry::(Telemetry, TelemetryConfig)
	adm.telemetry.statsd
)

let metrics = new statsd.Exporter("udp://127.0.0.1:8125")   // tcp://, unix:///path, unixgram:///path
metrics.prefix = "shop."
metrics.resourceTags = ["service.name"]
try Telemetry.config(TelemetryConfig{exporters: [metrics], requireConsent: false})
try await Telemetry.start()
```

Counters go as `c` with what was added since the last batch, gauges and toggles as `g`,
histograms as `ms` samples (time units, in milliseconds) or `h` samples, each line standing for
the values of one range through its sample rate. Tags go in DogStatsD form (`|#key:value`) by
default; `metrics.tagStyle` picks `TagStyle.Influx`, `TagStyle.Graphite` or `TagStyle.None`
(labels in the name) for servers that read those.

## Tests

`adm test telemetry/statsd` sends to a loopback server over UDP, TCP and a Unix socket.
`ADM_TEST_STATSD` names a real server (`udp://host:8125`); the suite then sends metrics under
the prefix `adm-test.`.
