# AIOps Assessment

## Scenario

- This project monitors a small `payment-service`.
- AIOps checks its metrics and logs, finds problems, and sends anomaly events.

## Data and observations

- Data is in [`data/service_data.json`](data/service_data.json).
- It has 10 records from `10:00` to `10:09`.
- Metrics: `response_time_ms`, `cpu_percent`, and `memory_percent`.
- Logs: `log_level` and `message`.
- Records from `10:00`-`10:04` and `10:07`-`10:09` look normal.
- At `10:05`, response time was 610 ms and there was a payment timeout.
- At `10:06`, response time was 640 ms, CPU was 94%, memory was 91%, and
  there was a database timeout.

## Detection results

- Limits are 500 ms response time, 80% CPU, and 80% memory.
- 10 records were checked and 2 anomalies were found at `10:05` and `10:06`.
- The detector reports high metrics and concerning `ERROR` or `WARNING` logs.
- Normal records were not flagged.

## Event flow

- The detector creates an event with the timestamp, service, reasons, and data.
- The producer sends it to the in-memory `anomaly-events` topic.
- The consumer reads it from the same topic.
- The pipeline returns the event as the AIOps output.
- Main files are [`src/anomaly_detector.py`](src/anomaly_detector.py),
  [`src/event_producer.py`](src/event_producer.py),
  [`src/event_topic.py`](src/event_topic.py),
  [`src/event_consumer.py`](src/event_consumer.py), and
  [`src/aiops_pipeline.py`](src/aiops_pipeline.py).

## Final result

- 10 records processed.
- 2 anomalies detected.
- 2 events published and 2 consumed.
- The output showed the payment and database timeout problems.
- All 9 tests passed.

## Task completion notes

- Task 1: reviewed the repository and identified the data, detector, event
  files, pipeline, and tests.
- Task 2: checked the metrics, logs, timestamps, normal records, and unusual
  records.
- Task 3: confirmed that the detector found the two expected anomalies and
  explained the reasons.
- Task 4: confirmed that an event moved from the producer to the topic, then
  to the consumer and AIOps output.
- Task 5: found and fixed the incorrect log-level check and the topic mismatch.
- Task 6: ran the complete pipeline and confirmed 2 consumed events.
- Task 8: ran the provided tests and checked the final pipeline output.
- The source files and test files were kept in the original project structure.
- The workflow was tested from the repository root using Python and pytest.

## Problems fixed

- The detector checked only `WARNING`, but the data used `ERROR`. It now checks
  both levels.
- The producer and consumer used different topics. They now use the shared
  `anomaly-events` topic.

## Limitation

- The detector uses fixed limits. It could be improved by learning normal
  behaviour automatically.

## Reproduce

Run these commands from the repository folder:

```bash
python -m pip install -r requirements.txt
python -m pytest --verbose
python src/aiops_pipeline.py
```

Expected result: 10 records processed, 2 anomalies detected, and 2 events
consumed.

For evidence, screenshots can show the data file, anomaly output, event
publish/consume output, final pipeline output, and the passing test results.

## Validation

- Run `python -m pytest --verbose` to check the detector, producer, topic,
  consumer, and full pipeline tests.
- Run `python src/aiops_pipeline.py` to check the final workflow.
- The validation result was 9 tests passed.
- The final workflow processed 10 records, found 2 anomalies, and consumed
  2 events successfully.
