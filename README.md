# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

## AIOps assessment

This project monitors a `payment-service`. AIOps detects slow requests and high
resource usage, then converts them into events for operational response.

### Main files

- [`data/service_data.json`](data/service_data.json) contains the service data.
- [`src/anomaly_detector.py`](src/anomaly_detector.py) checks for anomalies.
- [`src/event_producer.py`](src/event_producer.py) sends anomaly events.
- [`src/event_topic.py`](src/event_topic.py) stores events in memory.
- [`src/event_consumer.py`](src/event_consumer.py) receives events.
- [`src/aiops_pipeline.py`](src/aiops_pipeline.py) runs the full process.
- [`tests/test_aiops_pipeline.py`](tests/test_aiops_pipeline.py) tests the code.

The workflow is simple: the data is checked, an event is created for an
anomaly, the producer sends it to the topic, and the consumer passes it to the
AIOps output.
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
- Error logs are now flagged as concerning events.

### Part 4: Event-flow check

- An event contains the timestamp, service name, reasons, and original record.
- The producer sends every detected event.
- The `anomaly-events` topic stores the events.
- The consumer reads events from the same topic.
- The pipeline returns the consumed events as the AIOps result.
- Both detected anomalies were sent, received, and returned successfully.

### Run

```bash
python -m pip install -r requirements.txt
python -m pytest --verbose
python src/aiops_pipeline.py
```

Expected result: 10 records processed, 2 anomalies detected, and 2 events
consumed by the downstream AIOps output.

### Part 5: Problems found and fixed

- The detector was checking for `WARNING`, but the data contained `ERROR`.
  I changed it to recognise both levels. The two error records now include a
  log-related reason.
- The producer and consumer were using different topics. Because of this, the
  consumer received no events. I changed the pipeline so both use the shared
  `anomaly-events` topic.
- After these fixes, the complete workflow works as expected.
- The final check passed all 9 tests.
- The pipeline processed 10 records, found 2 anomalies, and consumed 2 events.

---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)
.