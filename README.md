# TracePulse

![CI Status](https://github.com/16lv/tracepulse/actions/workflows/ci.yml/badge.svg)

**TracePulse** is a telemetry data ingestion, SQLite analytical query engine, cohort aggregation, and statistical anomaly detection workbench built in **Python 3.12** with **pandas**, **NumPy**, and **matplotlib**.

## Why I Built It

Raw CSV service logs often contain missing identifiers, negative latencies, or out-of-range error percentages that silently skew averages and percentile charts if loaded straight into a notebook. I built TracePulse to separate row-level validation from statistical aggregation, storing verified records in an indexed SQLite table before computing per-service percentiles (`p50`, `p95`), Z-score latency outliers, and headless SVG comparison charts.

## Key Capabilities

- **Row-Level Domain Validation (`ingest_csv`)**: Streams CSV telemetry files, rejects rows with blank primary keys, `NaN`/negative latencies, or error rates outside `[0.0, 100.0]`, and returns exact `inserted` vs `rejected` counts.
- **SQLite + Pandas Aggregation (`summarize_by_service`)**: Queries clean observations into a pandas `DataFrame` and computes sample counts, mean, median (`p50`), 95th percentile (`p95`), population standard deviation ($\sigma$), mean error rate, and total requests/sec per service.
- **Z-Score Outlier Detection (`detect_latency_anomalies`)**: Flags individual observations whose latency deviates by $\ge 2.0\sigma$ from their service baseline while guarding against zero-variance division errors (`std == 0.0`).
- **Headless Chart Export (`export_latency_chart`)**: Uses `matplotlib`'s non-interactive `Agg` backend to render side-by-side median vs 95th-percentile latency bar charts in headless CI environments.
- **Bivariate Correlation (`pearson_correlation`)**: Calculates Pearson's $r \in [-1.0, 1.0]$ between request throughput and response latency.

## Project Structure

```text
tracepulse/
├── pyproject.toml
├── src/
│   └── data_workbench/
│       ├── __init__.py
│       └── pipeline.py      # AnalyticsWorkbench, Z-score detector & chart exporter
└── tests/
    └── test_pipeline.py     # Pytest suite verifying ingestion, stats & SVG output
```

## Usage Example

```python
from pathlib import Path
from src.data_workbench.pipeline import AnalyticsWorkbench

with AnalyticsWorkbench(":memory:") as wb:
    result = wb.ingest_csv(Path("telemetry.csv"))
    print(f"Inserted: {result['inserted']}, Rejected: {result['rejected']}")

    for stats in wb.summarize_by_service():
        print(f"{stats.service}: p50={stats.median_latency_ms}ms, p95={stats.p95_latency_ms}ms")

    anomalies = wb.detect_latency_anomalies("checkout-api", z_threshold=2.0)
    wb.export_latency_chart(Path("artifacts/latency_chart.svg"))
```

## Running Tests & Linter

```bash
pip install -e ".[dev]"
ruff check .
pytest -v
```

## Known Limitations & Future Improvements

- Z-score outlier detection assumes an approximately normal latency distribution; adding Median Absolute Deviation (MAD) or Interquartile Range (IQR) fencing would reduce false positives on heavily right-skewed latency tails.
