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

### Run

```bash
python -m pip install -r requirements.txt
python -m pytest --verbose
python src/aiops_pipeline.py
```

Expected result: 10 records processed and 2 anomalies detected. The supplied
baseline reports 0 consumed events because producer and consumer use different
in-memory topics.

---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)
