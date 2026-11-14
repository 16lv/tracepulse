# telemetry-latency-analyser

Telemetry ingestion, anomaly detection, and visualisation pipeline written in **Python 3.12** using **SQLite**, **pandas**, **NumPy**, and **Matplotlib**.

## Pipeline Stages

1. **CSV ingestion & validation**: `AnalyticsWorkbench.ingest_csv()` checks required headers (`record_id`, `service`, `region`, `latency_ms`, `error_rate_pct`, `requests_per_sec`), filters invalid or non-finite rows, and batch-loads valid records via `executemany()`.
2. **Per-service aggregation**: `summarize_by_service()` computes sample counts, mean, median (`p50`), tail (`p95`), standard deviation, mean error rate, and total RPS across services.
3. **Anomaly detection**: `detect_latency_anomalies()` flags records exceeding a configurable population z-score threshold (`|z| >= z_threshold`), handling zero-variance datasets cleanly.
4. **Correlation & plotting**: `pearson_correlation()` computes `r` across paired series, and `export_latency_chart()` renders a non-interactive SVG/PNG p50-vs-p95 comparison chart using Matplotlib's `Agg` backend.

## Structure

```text
telemetry-latency-analyser/
├── src/data_workbench/
│   └── pipeline.py     # AnalyticsWorkbench, validation, summary stats, SVG chart export
├── tests/
│   └── test_pipeline.py
├── pyproject.toml
└── requirements.txt
```

## Development

```bash
pip install -e ".[dev]"
ruff check .
pytest -v
```
