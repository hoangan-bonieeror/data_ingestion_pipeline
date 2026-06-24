# 🌦️ Weather ETL Pipeline

An automated data engineering pipeline that ingests hourly weather data for 5 global cities from a public API, applies data quality checks, and loads it into a database — running on a schedule, every hour.

> Built as a portfolio project to demonstrate core Data Engineering skills: API ingestion, ETL pipeline design, data quality validation, and incremental loading.

---

## 📐 Architecture

```
┌─────────────┐     ┌─────────────────────────────────────────────┐     ┌──────────────┐
│  Scheduler  │────▶│              ETL Pipeline                   │────▶│   Database   │
│ (every 1hr) │     │  Extract → Transform → Quality → Load       │     │  PostgreSQL  │
└─────────────┘     └─────────────────────────────────────────────┘     │  / SQLite    │
                           │                                             └──────────────┘
                    ┌──────▼──────┐
                    │  Open-Meteo │
                    │  Public API │
                    └─────────────┘
```

**Data flow:**
1. `scheduler.py` triggers the pipeline every hour
2. `fetch_weather.py` calls the Open-Meteo API for 5 cities
3. `clean_weather.py` flattens JSON, casts types, adds timestamps
4. Quality checks validate row count and null constraints
5. `load_to_db.py` upserts into the database (no duplicates on reruns)

---

## 🛠️ Tech Stack

| Layer | Tool |
|---|---|
| Language | Python 3.11 |
| Extract | `requests` + Open-Meteo API |
| Transform | `pandas` |
| Load | `SQLAlchemy` (SQLite / PostgreSQL) |
| Scheduling | `schedule` |
| Database | SQLite (dev) · PostgreSQL (prod) |

---

## 📁 Project Structure

```
weather-pipeline/
│
├── extract/
│   └── fetch_weather.py      # Calls API, returns raw JSON for 5 cities
│
├── transform/
│   └── clean_weather.py      # Flattens JSON, casts types, runs quality checks
│
├── load/
│   └── load_to_db.py         # Upserts clean data into the database
│
├── sql/
│   └── create_tables.sql     # Schema definition (SQLite + PostgreSQL versions)
│
├── pipeline.py               # Orchestrates Extract → Transform → Load
├── scheduler.py              # Runs pipeline.py every hour
├── requirements.txt
├── .env.example
└── README.md
```

---

## 🚀 Getting Started

### 1. Clone the repo
```bash
git clone https://github.com/your-username/weather-etl-pipeline.git
cd weather-etl-pipeline
```

### 2. Install dependencies
```bash
pip install -r requirements.txt
```

### 3. Configure environment
```bash
cp .env.example .env
# Edit .env if using PostgreSQL; SQLite works out of the box
```

### 4. Create the database table
```bash
# SQLite (default)
sqlite3 weather.db < sql/create_tables.sql

# PostgreSQL
psql -U your_user -d weather_db -f sql/create_tables.sql
```

### 5. Run the pipeline once
```bash
python pipeline.py
```

### 6. Start the hourly scheduler
```bash
python scheduler.py
```

---

## 📊 Sample Output

```
==================================================
Pipeline started at 2024-01-15 10:00:00 UTC
==================================================

[1/4] Extracting data from API...
  Fetching data for New York...
  Fetching data for London...
  Fetching data for Tokyo...
  Fetching data for Sydney...
  Fetching data for São Paulo...
  Fetched data for 5 cities.

[2/4] Transforming data...
  Transformed 120 rows.

[3/4] Running quality checks...
  Quality checks passed. 120 rows ready to load.

[4/4] Loading data into database...
  Inserted 120 new rows into 'weather_observations' (SQLite).

==================================================
Pipeline finished in 2.3s
  Rows processed : 120
  Rows inserted  : 120
==================================================
```

---

## 🔍 Key Design Decisions

**Upsert over insert** — The load step uses `INSERT OR IGNORE` (SQLite) / `ON CONFLICT DO NOTHING` (PostgreSQL) based on a `UNIQUE(city, time)` constraint. This means the pipeline can be re-run safely at any time without creating duplicate records.

**Quality checks before load** — Data is validated after transformation but before loading. If checks fail, the load is aborted. This prevents bad data from ever reaching the database.

**SQLite by default** — Keeps local development dependency-free. Switching to PostgreSQL requires only a one-line change in `.env`.

---

## 🌆 Cities Tracked

| City | Country |
|---|---|
| New York | 🇺🇸 USA |
| London | 🇬🇧 UK |
| Tokyo | 🇯🇵 Japan |
| Sydney | 🇦🇺 Australia |
| São Paulo | 🇧🇷 Brazil |

---

## 📈 Possible Improvements

- [ ] Add retry logic for failed API calls
- [ ] Connect a BI tool (Metabase / Grafana) for dashboards
- [ ] Migrate orchestration to Apache Airflow
- [ ] Deploy to cloud (AWS RDS + EC2 or Railway)
- [ ] Add dbt for SQL-based transformations
- [ ] Send pipeline failure alerts via email or Slack

---

## 📚 What I Learned

- How to design an **incremental ETL pipeline** that avoids duplicate data on reruns
- Structuring Python projects into **modular, testable layers** (extract / transform / load)
- Applying **data quality checks** as a pipeline gate before loading
- Working with **SQLAlchemy** for database-agnostic loading
- Scheduling recurring jobs with the **`schedule`** library

---

## 📄 License

MIT License — feel free to use this as a starting point for your own projects.

---

*Built by [An] · [LinkedIn](https://www.linkedin.com/in/an-nguyen-3168751a6) · [GitHub](https://github.com/hoangan-bonieeror)*
