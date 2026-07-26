# ruststream-grafana

A provisioning-ready Grafana dashboard for [RustStream](https://github.com/powersemmi/ruststream)
services instrumented with the core crate's `otel` feature. Import
[`dashboards/ruststream.json`](dashboards/ruststream.json) (Grafana 10+), pick your
Prometheus-compatible datasource (Prometheus, Mimir, VictoriaMetrics, Grafana Cloud), and the
panels light up per handler.

This README doubles as the human-readable **metrics contract**: the dashboard expects exactly the
instruments below, which the `otel` feature's `consume_layer()` / `publish_layer()` /
`observe_health()` emit.

## The metrics contract

OpenTelemetry instrument names on the left; the Prometheus rendering the panels query on the
right (dots to underscores, `_total` on counters, unit suffixes on histograms).

| OpenTelemetry instrument | Prometheus name | Kind | Meaning |
|---|---|---|---|
| `messaging.client.consumed.messages` | `messaging_client_consumed_messages_total` | counter | deliveries received, per handler |
| `messaging.process.duration` | `messaging_process_duration_seconds` | histogram | handler processing time |
| `ruststream.messages.processed` | `ruststream_messages_processed_total` | counter | settlements, `outcome` = `ack` / `nack_requeue` / `nack_drop` / `retry_after` |
| `ruststream.messages.in_flight` | `ruststream_messages_in_flight` | up-down counter | deliveries inside handlers (pool saturation vs `workers(n)`) |
| `ruststream.message.queue_time` | `ruststream_message_queue_time_seconds` | histogram | publish-to-handler-start lag (needs the publish-time header stamp, on by default) |
| `messaging.client.sent.messages` | `messaging_client_sent_messages_total` | counter | publishes; failures carry `error.type` |
| `messaging.client.operation.duration` | `messaging_client_operation_duration_seconds` | histogram | the publish operation |
| `ruststream.message.payload.size` | `ruststream_message_payload_size_bytes` | histogram | published payload sizes |
| `ruststream.app.state` | `ruststream_app_state` | gauge | lifecycle state, 0/1 per `state` attribute (wire with `otel.observe_health(running.health())`) |

Common attributes: `messaging.destination.name` (the handler / subscription name, the dashboard's
per-handler variable), `messaging.system` (set it with `.messaging_system("kafka")` on the
builder), `error.type` on failed publishes.

## Layout

- **Overview** - app state, consumed and published throughput.
- **Per handler** - processing-duration quantiles, in-flight vs the worker pool, queue-time lag,
  payload sizes.
- **Failures and outcomes** - settlement outcomes and the non-ack rate.

## Versioning

The dashboard tracks the metrics contract of the core crate's `otel` feature, not the crate API;
it versions on its own cadence. Breaking metric renames in the core bump the major version here.

## License

Apache-2.0, same as the core crate.
