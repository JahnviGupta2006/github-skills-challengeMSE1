# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!

# AIOps Scenario

This project simulates monitoring for a `payment-service`. Operational records
contain service metrics and log information. An anomaly detector evaluates each
record, creates an anomaly event for abnormal behaviour, and sends that event
through an in-memory producer, topic, and consumer to the downstream AIOps
output.

# Analyse Logs and Metrics

The analysis below is based on all 10 observations in `data/service_data.json`.

# 1. Metrics

These numeric fields measure the service's runtime behaviour:

| Field | Meaning | Observed values |
| --- | --- | --- |
| `response_time_ms` | Request response time in milliseconds | `120-640` |
| `cpu_percent` | CPU utilisation percentage | `42-94%` |
| `memory_percent` | Memory utilisation percentage | `51-91%` |

`service` is a service identifier or dimension, rather than a metric. The other fields are operational metadata or log content.

# 2. Log information

| Field | Role |

| `timestamp` | Time at which the observation and log event occurred |
| `service` | Service that emitted the observation |
| `log_level` | Event severity (`INFO` or `ERROR`) |
| `message` | Human-readable description of the event |

# 3. Timestamp usage

The timestamps use an ISO 8601-style date and time format: `YYYY-MM-DDTHH:MM:SS`. All records are for `2026-09-20`, are ordered chronologically, and are exactly one minute apart from `10:00:00` through `10:09:00`. This makes it possible to correlate a log event with the metric values captured at that minute and to see that the errors are short-lived rather than a continuous condition. The timestamps do not include a timezone offset.

# 4. Observations that appear normal

The records at `10:00`-`10:04` and `10:07`-`10:09` appear normal because each has `log_level: INFO` and the same successful message, `Payment request processed successfully`. Across these records, response time is `120-150 ms`, CPU is `42-50%`, and memory is `51-57%`, showing a relatively stable baseline.

# 5. Observations that appear unusual

The observations at `10:05` and `10:06` are unusual:

| Timestamp | Log evidence | Metric evidence |

| `2026-09-20T10:05:00` | `ERROR`: `Payment service timeout` | Response time `610 ms`, CPU `75%`, memory `70%` |
| `2026-09-20T10:06:00` | `ERROR`: `Database connection timeout` | Response time `640 ms`, CPU `94%`, memory `91%` |

These two records are clear outliers compared with the surrounding baseline. They combine error-level events and timeout messages with a roughly four-times increase in response time and substantially higher CPU and memory utilisation. The return to normal-looking values at `10:07` suggests the incident lasted about two minutes. Since this is synthetic data and no alert thresholds are supplied, “unusual” here is based on the strong contrast with the other observations and the accompanying `ERROR` messages.


---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

# Identify Anomalies

The provided anomaly-detection component was used to process all observations in
`data/service_data.json`.

# Detection Results

The pipeline processed 10 operational records and detected 2 anomalies.

| Timestamp | Metric information | Log information | Detection reasons |

| `2026-09-20T10:05:00` | Response time: `610 ms`; CPU: `75%`; memory: `70%` | Level: `ERROR`; message: `Payment service timeout` | High response time; concerning log level |
| `2026-09-20T10:06:00` | Response time: `640 ms`; CPU: `94%`; memory: `91%` | Level: `ERROR`; message: `Database connection timeout` | High response time; high CPU utilization; high memory utilization; concerning log level |

The detector uses these thresholds:

- Response time above `500 ms`
- CPU utilization above `80%`
- Memory utilization above `80%`
- Log level of `ERROR` or `WARNING`

Normal observations were not flagged. The records from `10:00` to `10:04`
and `10:07` to `10:09` had `INFO` logs, successful messages, response times
between `120` and `150 ms`, CPU utilization between `42%` and `50%`, and memory
utilization between `51%` and `57%`.

No expected anomalies were missed. The two timeout records at `10:05` and
`10:06` were detected because they contained concerning `ERROR` logs and
abnormal metric values. No normal event was incorrectly flagged.

The detection result includes the timestamp, service name, anomaly type, source
record, and reasons for each anomaly, making it possible to understand why each
record was flagged.

# Limitation and Possible Improvement

The detector uses fixed thresholds, so it may miss gradual performance
degradation or flag a legitimate short-term spike. A rolling baseline or
configurable thresholds based on historical service behaviour would improve the
detection approach.


# Verify the AIOps Event Flow

The provided AIOps pipeline was executed with:

```bash
python -m src.aiops_pipeline
```

```text
Records processed: 10
Anomalies detected: 2
Events consumed: 2
```

The event flow was verified as follows:

The anomaly detector identified abnormal records at 2026-09-20T10:05:00 and 2026-09-20T10:06:00.
For each abnormal record, the detector created an event with type ANOMALY.
The EventProducer published each event to the anomaly-events topic.
The EventTopic stored the published events.
The EventConsumer received both events from the same topic.
The consumed events were returned by the pipeline and printed by the downstream AIOps output.

The detected events were:

- `2026-09-20T10:05:00`: High response time and concerning `ERROR` log level.
- `2026-09-20T10:06:00`: High response time, high CPU utilization, high memory utilization, and concerning `ERROR` log level.


The workflow initially had two issues:

1. The anomaly detector checked for `WARNING`, but the operational data contained `ERROR` log events. The detector was corrected to identify both `WARNING` and `ERROR` as concerning log levels.
2. The producer and consumer were connected to different topic instances. They were corrected to use the same `anomaly-events` topic.

The corrected workflow processed 10 records and detected 2 anomalies:

- `2026-09-20T10:05:00`: high response time and concerning `ERROR` log level.
- `2026-09-20T10:06:00`: high response time, high CPU utilization, high memory utilization, and concerning `ERROR` log level.

# Execute the End-to-End Pipeline

The complete event flow is:

```text
Operational Data
    -> Anomaly Detection
    -> Anomaly Event
    -> Event Producer
    -> anomaly-events Topic
    -> Event Consumer
    -> AIOps Output
  ```

  The final output represents the payment-service timeout conditions at `10:05`
  and `10:06`, including the metric thresholds exceeded and the related error
  messages. Both anomaly events were published and consumed successfully.

  # Reproduce the Demonstration

  From the repository root:

  ```bash
  python -m venv .venv/calculations
  source .venv/calculations/bin/activate
  python -m pip install -r requirements.txt
  python -m pip install pytest coverage pytest-cov
  python -m src.aiops_pipeline
  python -m pytest --cov=src --verbose
  ```

  The data source is `data/service_data.json`. The pipeline loads all 10 records,
  passes them to `AnomalyDetector`, publishes detected events through
  `EventProducer` to the `anomaly-events` `EventTopic`, and retrieves them with
  `EventConsumer` for the final AIOps output.
