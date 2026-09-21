# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

## AIOps assessment

This project monitors a `payment-service`. AIOps detects slow requests and high
resource usage, then converts them into events for operational response.

### Components and workflow

- Data: [`data/service_data.json`](data/service_data.json)
- Detection: [`src/anomaly_detector.py`](src/anomaly_detector.py)
- Producer/topic/consumer: [`src/event_producer.py`](src/event_producer.py),
  [`src/event_topic.py`](src/event_topic.py), [`src/event_consumer.py`](src/event_consumer.py)
- Pipeline: [`src/aiops_pipeline.py`](src/aiops_pipeline.py)
- Tests: [`tests/test_aiops_pipeline.py`](tests/test_aiops_pipeline.py)

```text
Data -> Detect -> Produce -> Topic -> Consume -> AIOps output
```

### Data observations

- Timestamps are ISO-8601, one record per minute from `10:00` to `10:09`.
- Metrics: `response_time_ms`, `cpu_percent`, `memory_percent`.
- Logs: `log_level`, `message`.
- `service` identifies the monitored service.
- Normal records: `10:00`-`10:04` and `10:07`-`10:09`.
  Latency: 120-150 ms; CPU: 42-50%; memory: 51-57%; log level: `INFO`.
- Anomalies: `10:05` and `10:06`.
  - `10:05`: 610 ms, 75% CPU, 70% memory; payment timeout (`ERROR`).
  - `10:06`: 640 ms, 94% CPU, 91% memory; database timeout (`ERROR`).
- Detection thresholds: latency >500 ms, CPU >80%, memory >80%.

### Part 3: Detection report

- Processed: 10 records.
- Detected: 2 anomalies (`10:05`, `10:06`).
- Reasons:
  - `10:05`: high response time; `ERROR` payment timeout; 610 ms, 75% CPU,
    70% memory.
  - `10:06`: high response time, high CPU, high memory; `ERROR` database
    timeout; 640 ms, 94% CPU, 91% memory.
- Normal records were not flagged.
- No metric anomaly was missed.
- Limitation: the detector checks for `WARNING`, not `ERROR`, so error logs are
  not flagged directly. Improvement: include `ERROR` as a concerning log level.

### Part 4: Event-flow verification

- **Event/message:** anomaly record containing timestamp, service, reasons, and
  source data.
- **Producer:** publishes each detected event.
- **Topic:** shared in-memory `anomaly-events` queue.
- **Consumer:** reads events from that topic.
- **AIOps output:** `run_pipeline()` returns the consumed events for reporting.

Execution result: both detected anomalies were published, consumed, and
returned downstream. The complete flow passed through the shared topic:

```text
Detector (2) -> Producer -> anomaly-events -> Consumer (2) -> AIOps output (2)
```

### Run

```bash
python -m pip install -r requirements.txt
python -m pytest --verbose
python src/aiops_pipeline.py
```

Expected result: 10 records processed, 2 anomalies detected, and 2 events
consumed by the downstream AIOps output.

---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)
.