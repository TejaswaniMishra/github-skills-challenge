GitHub Challenge


Hey there!

Your challenge is ready. Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!


# AIOps Monitoring Assessment

## SCENARIO

This project monitors a payment service. The idea is to notice when the service starts having problems before it affects more requests. The AIOps part combines the service metrics and log information, finds unusual records, and sends those findings through a small event-processing workflow.

## WorkFlow
Operational Data -> Anomaly Detection -> Event Generation -> Producer -> Topic ->Consumer -> AIOps Output
The assessment uses a lightweight Python-based simulation rather than a real Kafka or Airflow
deployment.

## Operational data

The sample data is in `data/service_data.json`. It has 10 records for the payment service. Each record includes response time, CPU percentage, memory percentage, log level, and a log message.

Most records look normal. They have response times between 120 and 150 ms, CPU below 50 percent, memory in the low-to-mid 50s, and INFO logs. Two records show problems. One has a response time of 610 ms and a payment timeout message. The other has a response time of 640 ms, CPU at 94 percent, memory at 91 percent, and a database timeout message.

## Detection results

The detector uses limits of 500 ms for response time and 80 percent for both CPU and memory. It correctly classified 8 records as normal and found 2 anomalous records.

The first anomaly was flagged for high response time. The second was flagged for high response time, high CPU, and high memory. The ERROR log messages were also identified after the detector was corrected to check for both WARNING and ERROR log levels. No normal records were incorrectly flagged in this data.

One limitation is that the detector uses fixed thresholds. A better version could make the limits configurable or compare the current values with a normal baseline for each service.

## Event flow

`aiops_pipeline.py` loads the JSON records and passes each one to `AnomalyDetector`.
When a record is anomalous, the detector creates an event containing the service, reason, source record, and other event details.

`EventProducer` publishes the event to an `EventTopic`. The topic is an in-memory list that acts like a simple message broker. `EventConsumer` reads the messages from that same topic. The pipeline then uses the consumed events to produce the final output.

The main files are `src/anomaly_detector.py`, `src/event_producer.py`,
`src/event_topic.py`, `src/event_consumer.py`, and `src/aiops_pipeline.py`.

## Problems found and corrected

At first, the producer and consumer used different topic objects. The producer put events on `service-events`, while the consumer read from `anomaly-events`, so the consumer received nothing. I corrected `src/aiops_pipeline.py` so both components use the same `anomaly-events` topic.

The detector originally checked WARNING logs but the sample data used ERROR logs. I updated the existing check in `src/anomaly_detector.py` to recognize both levels. The original architecture and components were kept.

## Final run

After the corrections, the workflow processed 10 records, detected 2 anomalies, published 2 events, and consumed 2 events. The final output showed the slow payment requests, high resource use, and timeout messages from the payment service.

## Reproducing the result

Run these commands from the project root:

1. Run the workflow with `python3 src/aiops_pipeline.py`.
2. Run the tests with `PYTHONPATH=.:src python3 -m pytest -q`.

The workflow should report 10 records processed, 2 anomalies detected, and 2 events
consumed. The tests should pass as well.

## Validation

I ran the provided validation after making the corrections. The test suite completed with 9 passing tests. These tests cover normal and anomalous records, event creation, publishing, consumption, and the complete pipeline result.

The workflow also completed successfully with 10 records processed, 2 anomalies detected, and 2 events consumed. The final output included the response-time, CPU,memory, and log problems from the payment service.

I also ran Python compilation for `src` and `tests`, and checked the repository diff for whitespace errors. Both checks passed.
