# Honeypot Risk Scoring Model

**Project:** Honeypot Risk Scoring Model (2025)

**Short description**
A telemetry analysis pipeline that ingests honeypot logs, extracts behavioral features, computes a quantized risk score per event, and visualizes attack frequency and risk levels in Kibana to help SOC teams prioritize alerts.

---

## Table of contents

* [Motivation](#motivation)
* [Features](#features)
* [Architecture](#architecture)
* [Data pipeline](#data-pipeline)
* [Risk scoring methodology](#risk-scoring-methodology)
* [Visualization & Dashboards](#visualization--dashboards)
* [Prerequisites](#prerequisites)
* [Example workflow](#example-workflow)
* [Testing & Validation](#testing--validation)
* [Tuning & Reducing False Positives](#tuning--reducing-false-positives)
* [Acknowledgements & Tools used](#acknowledgements--tools-used)
* [Contact](#contact)

---

## Motivation

Honeypots attract malicious activity and produce rich telemetry about attacker behavior. However, raw honeypot logs are noisy and voluminous. This project converts raw telemetry into prioritized, quantized risk scores so SOC analysts can quickly focus on the most dangerous activity.

---

## Features

* Ingests honeypot telemetry (connection attempts, commands, payloads, timestamps, source metadata).
* Extracts behavioral features such as frequency, repetition, payload type, and severity indicators.
* Computes a weighted, quantized risk score (e.g., Low/Medium/High or levels 1–5) for each event.
* Enriches logs with contextual metadata (geolocation, ASN lookup, known-malicious lists).
* Pushes enriched events to Elasticsearch and visualizes via Kibana dashboards.
* Provides configurable scoring weights and thresholding for tuning.

---

## Architecture

```
Honeypot(s)  -->  Log Collector  -->  Feature Extractor  -->  Risk Scorer  -->  Elasticsearch  -->  Kibana Dashboards
```

* **Honeypot(s):** Low-interaction honeypots (e.g., Cowrie, Dionaea) that log incoming connections and interaction attempts.
* **Log Collector:** Aggregates raw logs (Filebeat / Logstash / custom collector).
* **Feature Extractor:** Parses events and computes features (frequency, payload tags, command types, etc.).
* **Risk Scorer:** Applies weighted rules / models to compute a numeric score and quantizes it into levels.
* **Elasticsearch:** Stores indexed enriched events for search and analytics.
* **Kibana:** Dashboards and visualizations used by SOC for triage and trend analysis.

---

## Data pipeline

1. **Collection:** Honeypot writes raw JSON/text logs to a file or forwards to Logstash/Filebeat.
2. **Parsing:** Logstash (or a Python parser) normalizes fields and extracts base attributes (src_ip, ts, event_type, payload).
3. **Enrichment:** Add geolocation, ASN, RBL checks, and a lookup against threat intelligence lists.
4. **Feature extraction:** Compute per-event features like `hit_count`, `interarrival_time`, `payload_risk`, `command_type_score`.
5. **Scoring:** Apply the scoring function to calculate `risk_score` and `risk_level`.
6. **Indexing:** Send enriched documents to Elasticsearch index (e.g., `honeypot-events-*`).
7. **Visualization:** Kibana visualizations & dashboards for SOC consumption.

---

## Risk scoring methodology

The scoring system is intentionally configurable. A typical formula used during development:

```
risk_score = w1 * normalized_frequency + w2 * severity + w3 * payload_risk + w4 * reputation_score
```

* **normalized_frequency:** Number of events from the same source in a time window (0–1 after normalization).
* **severity:** Baseline severity for event type (e.g., exploit > brute-force > scan).
* **payload_risk:** Heuristic score based on payload content (e.g., presence of known exploit strings, shell commands).
* **reputation_score:** External signals such as blacklists, ASN risk, or previous malicious history.

**Quantization:** Map `risk_score` ranges to categories, for example:

* 0.0–0.2 => Low (1)
* 0.2–0.45 => Medium (2–3)
* 0.45–1.0 => High (4–5)

Weights (w1..w4) are stored in a configuration file so analysts can tune scoring.

---

## Visualization & Dashboards

Suggested Kibana dashboards & visualizations:

* **Overview:** Total events, distribution of `risk_level`, events per minute
* **Top Attackers:** Top source IPs with highest cumulative risk
* **Time-series:** Risk level over time and peak attack windows
* **Heatmap:** Attack frequency across geo-locations
* **Event Details:** Drill-down table with raw payload and computed score for forensic analysis

Export and include the Kibana dashboard JSON in `/dashboards/` to import in your environment.

---

## Prerequisites

* Python 3.8+
* Elasticsearch 7.x or 8.x (compatible client versions)
* Kibana corresponding to Elasticsearch version
* Logstash / Filebeat (optional if using Python collector)
* Pip packages: `elastic-transport`, `elasticsearch`, `geoip2`, `pandas`, `scikit-learn` (optional), `pyyaml`

---


---

## Example workflow

1. Honeypot receives a connection; log entry appended to file.
2. Filebeat forwards the log to Logstash.
3. Logstash parses and applies grok rules; output sent to the scorer service.
4. Scorer computes `risk_score` and writes to Elasticsearch index.
5. Kibana dashboard updates, highlighting new high-risk events for analysts.

---

## Testing & Validation

* **Unit tests:** Add tests for parsers, feature extraction, and scoring logic in `tests/`.
* **Replay testing:** Use PCAPs or saved honeypot logs and replay them through the collector to validate detection and scoring behavior.
* **Evaluation:** Compare scoring output against labeled incidents to compute precision/recall and tune weights.

---

## Tuning & Reducing False Positives

* Apply adaptive thresholds (time-window normalization) to avoid flagging noisy scanners.
* Whitelist known benign scanning sources (e.g., internal scanners) via config.
* Incorporate contextual signals (user agent, repeated IP behavior across different ports) rather than single-event heuristics.
* Periodically review highest false-positive sources and adjust weights.

---



---

## Acknowledgements & Tools used

* Honeypot engines: Cowrie / Dionaea (example)
* Log pipeline: Filebeat, Logstash (or custom Python collector)
* Storage & visualization: Elasticsearch + Kibana
* Utilities: geoip2, pandas, scikit-learn (optional), pyyaml

---

## Contact

If you have questions about the pipeline or want to reproduce results, contact:

Swastik Ami — [swastik362004@gmail.com](mailto:swastik362004@gmail.com)

---

*README generated and tailored for the Honeypot Risk Scoring Model (2025). Edit configuration sections and paths as needed for your environment.*
