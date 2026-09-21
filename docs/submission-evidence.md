# Submission Evidence

This document records the checks completed for the AIOps assessment. The application
uses the provided Python components and does not use an external broker or service.

## What the workflow detects

The workflow reads payment-service records from `data/service_data.json`. It checks
response time, CPU usage, memory usage, and log level. Records above the configured
limits or with WARNING or ERROR logs become anomaly events.

## Validation completed

From the repository root, I ran:

    python3 src/aiops_pipeline.py

The final output was:

    Records processed: 10
    Anomalies detected: 2
    Events consumed: 2

The output listed two payment-service anomalies. The first had high response time and
a payment timeout log. The second had high response time, high CPU, high memory, and a
database connection timeout log.

I also ran:

    PYTHONPATH=.:src python3 -m pytest -q

The result was 9 passing tests. Python compilation and `git diff --check` also passed.

## Event-flow evidence

The detector generated two events. The producer published both to the shared
`anomaly-events` topic. The consumer read both events from that topic, and
`aiops_pipeline.py` displayed them in the final output.

## Screenshots to attach to the submission

The written results above are the reproducible evidence in the repository. For the
submission screenshots, capture these terminal or editor views in VS Code:

1. `data/service_data.json` showing the normal and abnormal metrics and logs.
2. `python3 src/aiops_pipeline.py` showing the anomaly reasons and final output.
3. The event-flow trace or the producer, topic, and consumer source files.
4. `PYTHONPATH=.:src python3 -m pytest -q` showing `9 passed`.

The screenshots should be added to the pull request or submission form without changing
the application data or source code just to produce a different result.