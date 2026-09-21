# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!

# Analyse Logs and Metrics

The analysis below is based on all 10 observations in `data/service_data.json`.

# 1. Metrics

These numeric fields measure the service's runtime behaviour:

 Field             |    Meaning                            |  Observed values 
 
`response_time_ms` | Request response time in milliseconds | 120-640  `cpu_percent`      | CPU utilisation percentage            | 42-94% 
 `memory_percent`  | Memory utilisation percentage         | 51-91% 

`service` is a service identifier or dimension, rather than a metric. The other fields are operational metadata or log content.

### 2. Log information

| Field | Role |
| --- | --- |
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

 Timestamp              | Log evidence                       | Metric evidence                           

  `2026-09-20T10:05:00` |`ERROR`: `Payment service timeout` | Response time `610 ms`, CPU `75%`, memory `70%` 
`2026-09-20T10:06:00`  | `ERROR`: `Database connection timeout` | Response time `640 ms`, CPU `94%`, memory `91%` 

These two records are clear outliers compared with the surrounding baseline. They combine error-level events and timeout messages with a roughly four-times increase in response time and substantially higher CPU and memory utilisation. The return to normal-looking values at `10:07` suggests the incident lasted about two minutes. Since this is synthetic data and no alert thresholds are supplied, “unusual” here is based on the strong contrast with the other observations and the accompanying `ERROR` messages.


---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

