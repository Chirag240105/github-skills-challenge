# AIOps Operational Monitoring Workflow

## Task 1: Project setup complete

This repository was opened in GitHub Codespaces from the forked assessment repository, not the original source repository. The project structure was reviewed and the major workflow components were identified before execution.

The repository includes:

- `data/service_data.json`: operational data for the monitored service
- `src/anomaly_detector.py`: anomaly detection logic for metrics and log events
- `src/event_producer.py`: publishes anomaly events
- `src/event_topic.py`: in-memory topic simulation
- `src/event_consumer.py`: consumes events from the topic
- `src/aiops_pipeline.py`: orchestrates the end-to-end flow

This project simulates a lightweight AIOps workflow for a payment service. The service emits telemetry in the form of timing metrics and log entries. The goal is to identify abnormal operational behaviour, turn those observations into anomaly events, and push the events through a simplified producer/topic/consumer pipeline before they reach the downstream AIOps processing stage.

## Scenario

The monitored service is a `payment-service` that handles customer transactions. The operational problem being addressed is degraded service health during peak or faulted conditions: slow responses, elevated CPU and memory usage, and error-level log output. AIOps in this assessment is used to detect unusual patterns early, represent them as structured events, and pass them through a simple event stream so they can be interpreted and acted on in the same way as a larger production monitoring pipeline.

## Operational data

The repository includes synthetic telemetry records in `data/service_data.json`. Each record contains:

- `timestamp`: ISO timestamp showing when the sample was captured
- `service`: name of the monitored service
- `response_time_ms`: latency metric for the request
- `cpu_percent`: CPU utilization metric
- `memory_percent`: memory utilization metric
- `log_level`: log severity (`INFO`, `WARNING`, `ERROR`)
- `message`: textual log message

### Metrics vs log information

The metric fields are:

- `response_time_ms`
- `cpu_percent`
- `memory_percent`

The log information is:

- `log_level`
- `message`

The `timestamp` field and `service` field provide contextual metadata for each observation rather than the monitored values themselves.

## Observed normal and abnormal behaviour

The operational data shows a repetitive sequence of healthy transactions from around `2026-09-20T10:00:00` through `2026-09-20T10:04:00`.

Normal behaviour includes:

- response times around 120-145 ms
- CPU near 42-50%
- memory near 51-57%
- `INFO` log entries indicating successful payment processing

Unusual behaviour includes:

- the record at `2026-09-20T10:05:00`, where response time reaches 610 ms and the service logs an `ERROR`
- the record at `2026-09-20T10:06:00`, where CPU and memory spike to 94% and 91% respectively and the service reports a database timeout

These later observations clearly stand out from the normal baseline and match the expected failure pattern.

## Anomaly detection findings

The detection logic in `src/anomaly_detector.py` flags records when any of the following indicators are present:

- `response_time_ms > 500`
- `cpu_percent > 80`
- `memory_percent > 80`
- a `ERROR` or `WARNING` log event is present

Using this logic, the pipeline identifies two anomalies in the dataset: the timeout records at `10:05:00` and `10:06:00`. Both anomalies include clear reasons such as `High response time`, `High CPU utilization`, `High memory utilization`, and `Error log detected`. No normal records were incorrectly marked as anomalous, and no expected anomalies were missed for this dataset.

One limitation of the current approach is that it relies on static thresholds rather than historical baselines. A service with a generally slow but acceptable load profile might be flagged incorrectly, while a subtle but significant change in behaviour could be missed if the thresholds are not tuned over time.

## Event-processing flow

The workflow is implemented in the following components:

- `src/anomaly_detector.py`: evaluates each record and emits an anomaly event when thresholds are exceeded
- `src/event_producer.py`: pushes the event to an in-memory topic
- `src/event_topic.py`: stores the messages in a lightweight topic simulation
- `src/event_consumer.py`: reads the topic to simulate downstream consumption
- `src/aiops_pipeline.py`: orchestrates the full operational data → detection → event → producer → topic → consumer → output flow

The event structure contains the service name, timestamp, event type, and human-readable reasons. This makes the generated anomaly readable and understandable to downstream monitoring systems.

## Final workflow result

The corrected pipeline produces the expected end-to-end result:

- `Records processed: 10`
- `Anomalies detected: 2`
- `Events consumed: 2`

The final output represents the payment-service operational issue caused by timeouts and resource saturation.

## Issues identified and corrected

Two issues were affecting the assessment workflow:

1. `src/anomaly_detector.py` checked for `WARNING` log levels but appended the reason text `Error log detected`. This was a logic bug and meant the system would not correctly classify error-level log events.
2. `src/aiops_pipeline.py` created different topic objects for the producer and consumer, preventing the consumer from receiving events produced by the detector.

The fixes kept the existing architecture intact while making the workflow behave as expected.

## Reproduction steps

1. Open the repository in GitHub Codespaces or a local Python environment.
2. Create a virtual environment if needed and install dependencies:

   ```bash
   python -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   ```

3. Run the validation tests:

   ```bash
   pytest -q
   ```

4. Run the end-to-end AIOps pipeline:

   ```bash
   python src/aiops_pipeline.py
   ```

5. Review the output to confirm the anomaly records are detected and processed through the in-memory event flow.

---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)


