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
```

## Requirements

- Python 3.10 or later
- `requests`
- `beautifulsoup4`
- `pandas` (installed automatically via `requirements.txt`)

---

## Installation

Follow these steps **in order**. The most common setup mistake is running `pse` before the package has actually been installed with `pip install -e .` — if you skip that step, your terminal will not recognize the `pse` command.

### macOS / Linux — copy and paste

```bash
git clone https://github.com/AlvinTubtub/pse-edge-data-scraper.git
cd pse-edge-data-scraper

python3 -m venv .venv
source .venv/bin/activate

python -m pip install --upgrade pip
pip install -r requirements.txt
pip install -e .

pse --help
```

### Windows (PowerShell) — copy and paste

```powershell
git clone https://github.com/AlvinTubtub/pse-edge-data-scraper.git
cd pse-edge-data-scraper

python -m venv .venv
.venv\Scripts\Activate.ps1

python -m pip install --upgrade pip
pip install -r requirements.txt
pip install -e .

pse --help
```

> **Windows note:** If `Activate.ps1` fails with a message about execution policies (a script cannot be loaded because running scripts is disabled), run this once in an elevated PowerShell session and then retry activation:
> ```powershell
> Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
> ```

If `pse --help` prints the usage/help text, installation succeeded.

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

# Troubleshooting

### `zsh: command not found: pse` / `'pse' is not recognized...` (Windows)

This means the package hasn't been installed into your active virtual environment yet, or the virtual environment isn't activated. Fix:

```bash
# make sure you're inside the activated .venv, then:
pip install -e .
```

### `520 Server Error` for `DisclosureCht.ax` when running `pse prices`

This is a server-side error returned by the PSE Edge website itself (`edge.pse.com.ph`), not a bug in the scraper. It usually means PSE Edge is temporarily overloaded, rate-limiting requests, or briefly down. Things to try:

- Wait a few minutes and re-run the same `pse prices` command.
- Reduce request volume with `--max-companies` or fewer `--symbols` per run.
- Increase the client's `rate_limit_seconds` (see the Python API example above) if you're scripting downloads.
- Check whether `https://edge.pse.com.ph` loads normally in your browser — if it doesn't, the issue is on PSE's end and not fixable locally.

### `WARNING: No CSV files found in data/history` when running `pse export`

This happens when `pse prices` didn't successfully download any data (for example, because of the 520 errors above). Resolve the prices download issue first, confirm `data/history/` contains `.csv` files, then re-run `pse export`.

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