# adm.telemetry.graylog

GELF for the `Logging` service, in ADM: the program's log records sent as GELF messages to a
Graylog input over UDP, TCP or HTTP.

```bash
adm get admlang:adm.telemetry.graylog
```

```adm
use (
	std.services.logging::*
	adm.telemetry.graylog
)

try Logging.config(
	LogConfig{
		writers: [
			LogWriterConfig{
				writer:  new graylog.Writer("tcp+tls://graylog.internal:12201"),
				encoder: new graylog.Encoder(),
			},
		],
	}
)
```

`Writer` takes `udp://` (compressed and chunked as GELF needs, `compression` picks gzip, zlib or
none), `tcp://` and `tcp+tls://` (null-terminated messages) and `http://` or `https://` URLs of
a GELF HTTP input. `Encoder` writes the record's first line as `short_message`, the whole of a
longer one as `full_message`, its level as the syslog severity, where it was written as
`_module`, `_file`, `_line` and `_function`, its trace and span, and every field as `_<name>`;
`encoder.host` names the source (the application's name by default).

## Tests

`adm test telemetry/graylog` sends to loopback GELF inputs over UDP, TCP and HTTP, and through the
`Logging` service.
`ADM_TEST_GRAYLOG` names a real input (`udp://host:12201`); the suite then sends messages with
the source `adm-test`.
