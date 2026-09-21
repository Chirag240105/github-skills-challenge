# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!


---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)


# AIOps Operational Monitoring Workflow

## Summary
- Service: `payment-service`
- Problem: slow response times, increased CPU/memory usage, error-level logs
- Goal: detect abnormal behavior, convert it into anomaly events, and confirm the producer/topic/consumer flow works

## Setup and structure
- Forked repo confirmed and opened in Codespaces
- Main files reviewed:
  - `data/service_data.json`
  - `src/anomaly_detector.py`
  - `src/event_producer.py`
  - `src/event_topic.py`
  - `src/event_consumer.py`
  - `src/aiops_pipeline.py`

## Operational data
- Metrics:
  - `response_time_ms`
  - `cpu_percent`
  - `memory_percent`
- Log fields:
  - `log_level`
  - `message`
- Timestamp usage:
  - ISO timestamps sampled once per minute
  - Used to track behavior over time and spot abrupt changes
- Normal behavior:
  - `2026-09-20T10:00:00` to `2026-09-20T10:04:00`
  - `2026-09-20T10:07:00` to `2026-09-20T10:09:00`
  - response time ~120-150 ms, CPU ~42-57%, memory ~51-57%, `INFO` messages
- Unusual behavior:
  - `2026-09-20T10:05:00`: 610 ms, `ERROR`, timeout
  - `2026-09-20T10:06:00`: 640 ms, 94% CPU, 91% memory, `ERROR`, database timeout

## Detection findings
- Detected anomalies:
  1. `2026-09-20T10:05:00` — `High response time`, `Error log detected`
  2. `2026-09-20T10:06:00` — `High response time`, `High CPU utilization`, `High memory utilization`, `Error log detected`
- Checks passed:
  - abnormal metrics identified
  - relevant log events identified
  - normal records not incorrectly flagged
  - no expected anomaly missed
- Limitation:
  - static thresholds can miss gradual issues or trigger false positives under different normal baselines

## Event flow
- Producer: publishes anomaly events
- Topic: in-memory queue for events
- Consumer: reads events from the topic
- Event/message: includes service, timestamp, type, and reasons
- Verified flow:
  - operational data processed
  - anomaly detected
  - anomaly event created
  - event published to topic
  - event consumed
  - downstream AIOps output generated

## Issues fixed
- `src/anomaly_detector.py`: corrected log-level reason mapping
- `src/aiops_pipeline.py`: fixed shared topic usage so producer and consumer communicate correctly

## Final result
- `Records processed: 10`
- `Anomalies detected: 2`
- `Events consumed: 2`

## Validation
```bash
cd /workspaces/github-skills-challenge
pytest -q
python src/aiops_pipeline.py
```

## Reproduction
1. Open the repo in Codespaces or a local Python environment.
2. Create a venv and install requirements:
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   ```
3. Run validation:
   ```bash
   pytest -q
   ```
4. Run the workflow:
   ```bash
   python src/aiops_pipeline.py
   ```
5. Confirm the output shows detected anomalies and consumed events.

---

