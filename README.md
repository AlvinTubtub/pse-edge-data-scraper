# PSE Edge Data Scraper

A lightweight Python package and CLI for downloading historical stock price data from PSE Edge and exporting it into clean CSV files for analysis.

## Features

- Download the PSE company list
- Fetch historical stock price data per company
- Save one CSV file per company
- Combine all company files into a single dataset
- Deduplicate repeated trading dates returned by the API
- Provide both a CLI and a Python API
- Support caching and refresh options

## Project Structure

```text
pse-edge-data-scraper/
├── main.py
├── pyproject.toml
├── requirements.txt
├── requirements-dev.txt
├── pse_data_scraper/
│   ├── __init__.py
│   ├── cli.py
│   ├── client.py
│   ├── combiner.py
│   ├── downloader.py
│   ├── models.py
│   ├── pipeline.py
│   ├── scraper.py
│   ├── status.py
│   └── utils.py
├── tests/
└── docs/

## Requirements

- Python 3.10 or later
- `requests`
- `beautifulsoup4`

---

## Installation

Clone the repository:

```bash
git clone https://github.com/AlvinTubtub/pse-edge-data-scraper.git
cd pse-edge-data-scraper
```

Create a virtual environment:

```bash
python3 -m venv .venv
```

Activate the virtual environment:

**macOS / Linux**

```bash
source .venv/bin/activate
```

**Windows (PowerShell)**

```powershell
.venv\Scripts\Activate.ps1
```

Upgrade pip:

```bash
python -m pip install --upgrade pip
```

Install the project:

```bash
pip install -r requirements.txt
pip install -e .
```

---

## Quick Start

Run the complete data pipeline:

```bash
pse sync
```

or run the main entry point directly:

```bash
python main.py
```

---

# CLI Usage

## Download the company list

```bash
pse companies
```

Refresh the company list:

```bash
pse companies --refresh
```

---

## Download historical stock prices

Example:

```bash
pse prices --symbols BDO,EURO,GLO,GMA7,MEG --from 2026-01-01 --to 2026-07-15
```

Download all companies:

```bash
pse prices
```

Limit the number of companies:

```bash
pse prices --max-companies 10
```

---

## Export all CSV files into a single dataset

```bash
pse export --format csv
```

---

## View download status

```bash
pse status
```

---

# Output Files

After a successful run, the project generates:

| File | Description |
|------|-------------|
| `data/companies.csv` | Master list of PSE companies |
| `data/history/` | Individual historical CSV files (one per company) |
| `data/combined.csv` | Combined historical dataset |
| `.cache/` | Cached API responses for faster downloads |

---

# Automatic Data Cleaning

To improve data quality, the downloader automatically performs the following checks before saving each CSV:

- Removes duplicate trading dates returned by the PSE Edge API.
- Preserves only the first occurrence of identical duplicate records.
- Detects conflicting duplicate records and logs a warning.
- Sorts all historical records by trading date.

This guarantees that every exported company CSV contains **one record per trading day**.

---

# Python API

The package can also be used directly in Python.

```python
from pse_data_scraper.client import PSEClient
from pse_data_scraper.pipeline import (
    ensure_companies_csv,
    download_prices,
    export_prices,
)

client = PSEClient(rate_limit_seconds=0.6)

companies = ensure_companies_csv(
    client,
    "data/companies.csv",
)

download_prices(
    client=client,
    companies=companies,
    history_dir="data/history",
    symbols=["BDO", "EURO", "GLO", "GMA7", "MEG"],
    start_date="2026-01-01",
    end_date="2026-07-15",
)

export_prices(
    "data/history",
    "data/combined.csv",
)
```

---

# Testing

Install the development dependencies:

```bash
pip install -r requirements-dev.txt
```

Run all tests:

```bash
pytest
```

---

# Notes

- Historical price data is downloaded directly from the PSE Edge web service.
- Company information is retrieved from the official PSE Edge listings.
- The package supports both CLI and Python API usage.
- Duplicate historical records returned by the API are automatically removed before export.
- The project intentionally keeps external dependencies to a minimum.

---

# License

This project is licensed under the **MIT License**.

See the `LICENSE` file for more information.
