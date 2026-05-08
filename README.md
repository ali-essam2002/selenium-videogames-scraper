# 🕹️ Oxylabs Sandbox Product Scraper

A production-grade Python web scraper that extracts **3,000 video game products** from the [Oxylabs sandbox store](https://sandbox.oxylabs.io/products) across 94 paginated pages, persisting results to both a **MySQL database** and a **CSV file**.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Configuration](#configuration)
- [Usage](#usage)
- [Output](#output)
- [Data Schema](#data-schema)
- [How It Works](#how-it-works)


---

## Overview

This project scrapes product listings from the Oxylabs e-commerce sandbox — a realistic practice target designed for web scraping experimentation. The scraper navigates all 94 pages, extracts structured product data from each card, and stores everything persistently for downstream analysis.

| Detail | Value |
|---|---|
| **Source URL** | `https://sandbox.oxylabs.io/products` |
| **Total Records** | ~3,000 products |
| **Pages Scraped** | 94 (32 items/page) |
| **Storage Targets** | MySQL database + CSV file |
| **Browser Driver** | Selenium + Headless Chrome |

---

## Features

- ✅ **Full pagination** — automatically crawls all pages from start to finish
- ✅ **Headless Chrome** — runs silently in the background without a browser window
- ✅ **Dual persistence** — saves data to both MySQL and a flat CSV file simultaneously
- ✅ **Resilient extraction** — gracefully skips malformed cards with warning logs
- ✅ **Polite crawling** — configurable delay between page requests
- ✅ **Structured config** — all parameters in a single `Config` dataclass
- ✅ **Summary report** — prints stats (total records, unique categories, in-stock rate) after completion

---

## Tech Stack

| Layer | Technology |
|---|---|
| Language | Python 3.10+ |
| Browser Automation | Selenium 4, ChromeDriver (via `webdriver-manager`) |
| Database | MySQL + `mysql-connector-python` |
| Data Analysis | pandas |
| Notebook | Jupyter |

---

## 📁Project Structure

```
🎮selenium-videogames-scraper/
│
├── oxylabs_videogames_scraper.ipynb   # Main scraper notebook
├── oxylabs_products.csv               # Exported data (generated on run)
└── README.md
```

---

## Getting Started

### Prerequisites

- Python 3.10 or higher
- Google Chrome browser installed
- A running MySQL server (local or remote)

### Installation

1. **Clone the repository**

```bash
git clone https://github.com/your-username/oxylabs-sandbox-scraper.git
cd oxylabs-sandbox-scraper
```

2. **Install dependencies**

```bash
pip install selenium webdriver-manager mysql-connector-python pandas jupyter
```

3. **Start Jupyter**

```bash
jupyter notebook
```

Then open `oxylabs_videogames_scraper.ipynb`.

### Configuration

All settings are centralized in the `Config` dataclass at the top of the notebook. Edit these before running:

```python
@dataclass
class Config:
    base_url: str = "https://sandbox.oxylabs.io/products"
    total_results: int = 3_000
    items_per_page: int = 32
    page_delay_s: float = 1.0       # delay between page requests (seconds)
    page_timeout_s: int = 10        # max wait for product cards to load
    csv_path: Path = Path("oxylabs_products.csv")

    # MySQL credentials
    db_host: str = "localhost"
    db_user: str = "root"
    db_password: str = "your_password"
    db_name: str = "oxylabs_db"
```

> ⚠️ **Security note:** Avoid committing real credentials to version control. Use environment variables or a `.env` file in production.

---

## Usage

Run all cells in `oxylabs_videogames_scraper.ipynb` from top to bottom. The notebook will:

1. Connect to MySQL and create the database/table if they don't exist
2. Launch a headless Chrome browser
3. Iterate through all 94 pages, extracting product cards
4. Save all records to MySQL and `oxylabs_products.csv`
5. Print a summary report

**Sample console output:**
```
10:14:22  INFO      Config loaded — 3000 products across 94 pages.
10:14:23  INFO      Database 'oxylabs_db' and table 'products' are ready.
10:14:28  INFO      Page   1 / 94 — 32 cards found.
10:14:30  INFO      Page   2 / 94 — 32 cards found.
...
10:27:11  INFO      Scraping complete — 3000 products collected.
10:27:14  INFO      3000 rows inserted into MySQL.
10:27:14  INFO      CSV saved → /path/to/oxylabs_products.csv (3000 rows).
```

---

## Output

### CSV — `oxylabs_products.csv`

A UTF-8 encoded flat file with one row per product:

```
name,price,rating,stock,category
The Legend of Zelda,€59.99,5 / 5,In stock,Nintendo, Action-Adventure
...
```

### MySQL — `oxylabs_db.products`

```sql
SELECT * FROM products LIMIT 5;
```

| id | name | price | rating | stock | category | scraped_at |
|----|------|-------|--------|-------|----------|------------|
| 1 | The Legend of Zelda | €59.99 | 5 / 5 | In stock | Nintendo, Action-Adventure | 2025-01-01 10:27:14 |

---

## Data Schema

| Field | Type | Description |
|---|---|---|
| `name` | `VARCHAR(255)` | Product title |
| `price` | `VARCHAR(50)` | Displayed price (with currency symbol) |
| `rating` | `VARCHAR(10)` | Star rating in `X / 5` format |
| `stock` | `VARCHAR(50)` | `In stock` or `Out of Stock` |
| `category` | `VARCHAR(255)` | Comma-separated category tags |
| `scraped_at` | `TIMESTAMP` | Auto-set insertion timestamp |

---

## How It Works

```
┌─────────────────────────────────────────────────────────┐
│                    Scraping Pipeline                    │
│                                                         │
│  Config ──▶ MySQL Setup ──▶ build_driver()             │
│                                  │                      │
│                           Headless Chrome               │
│                                  │                      │
│              ┌───────────────────▼──────────────────┐   │
│              │  for page in range(1, total_pages+1) │   │
│              │  scrape_page() ──▶ extract_product()│   │
│              │  time.sleep(page_delay_s)            │   │
│              └───────────────────┬──────────────────┘   │
│                                  │                      │
│               ┌──────────────────┴─────────────────┐    │
│               ▼                                    ▼    │
│         save_to_mysql()                   save_to_csv() │
└─────────────────────────────────────────────────────────┘
```

Each product card is parsed using CSS selectors and XPath into a typed `Product` `NamedTuple`, ensuring data integrity throughout the pipeline.

---



> Built as a web scraping practice project using the [Oxylabs Sandbox](https://sandbox.oxylabs.io/products).
