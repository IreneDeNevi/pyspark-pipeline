# PySpark Data Engineering Task 

## Overview

This project is a notebook-driven PySpark data pipeline for customer and order analytics. It loads raw CSV files, performs data cleaning and transformation, computes customer and city-level metrics, and writes final outputs to Parquet and CSV.

The main artifact is the notebook:

- `assignment_pyspark.ipynb`

## Quick Start

### 1. Activate your environment

```bash
source /home/$(whoami)/.venvs/assignment/bin/activate
```

### 2. Install dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

### 3. Run Jupyter Lab

```bash
jupyter lab --ip=127.0.0.1 --port=8888
```

Open `assignment_pyspark.ipynb` and run cells top to bottom.

---

## System Requirements

- OS: Windows with WSL2 (recommended), Linux, or macOS
- Python: 3.10+
- Java: OpenJDK 17+
- PySpark: from `requirements.txt`

### Tested Environment

- Host OS: Windows 11 + WSL2 (Ubuntu 26.04)
- Python: 3.14.4
- Spark: 4.1.2
- JDK: OpenJDK 17

---

## Installation (WSL2)

### Step 1: Install Java

```bash
sudo apt-get update
sudo apt-get install -y openjdk-17-jre-headless
java -version
```

### Step 2: Create virtual environment

```bash
cd /your-directory/data-engineer-assignment
python3 -m venv /home/$(whoami)/.venvs/assignment
source /home/$(whoami)/.venvs/assignment/bin/activate
```

### Step 3: Install Python packages

```bash
pip install -r requirements.txt
```

### Step 4: Verify

```bash
python -c "import pyspark; print(f'PySpark {pyspark.__version__} OK')"
java -version
python --version
```

---

## Project Structure

```text
data-engineer-assignment/
|-- assignment_pyspark.ipynb
|-- Dockerfile
|-- README.md
|-- requirements.txt
|-- data/
|   |-- customers.csv
|   `-- orders.csv
|-- outputs/
|   |-- final_orders_with_customers.parquet/
|   |-- top3_customers.csv
|   `-- top3_customers_csv_temp/
`-- doc/
```

### File Roles

- `assignment_pyspark.ipynb`:  pipeline (load, clean, transform, aggregate, join, visualize, export, verify)
- `requirements.txt`: Python dependencies
- `Dockerfile`: containerized environment for Jupyter + PySpark
- `data/`: source CSV input files
- `outputs/`: generated artifacts

---

## Notebook Pipeline Documentation

The notebook implements the following logical stages:

1. Initialize Spark session and imports
2. Load customers and orders CSV files in permissive mode
3. Explore schemas, counts, and initial data quality
4. Clean and cast fields (`customer_id`, `order_id`, `order_date`, `order_amount`)
5. Enrich orders with `order_year`, `order_month`, `order_day`
6. Filter valid 2024 orders
7. Compute customer metrics (sum, count, average)
8. Join with customer dimension data
9. Build top-3 customers by revenue
10. Build city-level totals
11. Visualize top customers and top cities
12. Write outputs to Parquet and CSV
13. Reload Parquet to validate persisted output

---

## Outputs

### 1) `outputs/final_orders_with_customers.parquet/`

- Type: Parquet
- Content: cleaned and enriched 2024 orders joined with customer attributes
- Typical columns:
  - `order_id`
  - `customer_id`
  - `order_date`
  - `order_amount`
  - `order_year`
  - `order_month`
  - `order_day`
  - `customer_name`
  - `customer_city`

### 2) `outputs/top3_customers.csv`

- Type: CSV
- Content: top three customers by total order amount

### 3) `outputs/top3_customers_csv_temp/`

- Spark temporary output folder created during CSV write operations

---

## Running in Container (Docker/Podman)

### Build image

```bash
podman build -t pyspark-assignment:latest .
```

### Run Jupyter Lab

```bash
podman run --rm -it -p 8888:8888 -v "${PWD}:/workspace" pyspark-assignment:latest jupyter lab --ip=0.0.0.0 --allow-root
```

---

## Troubleshooting

### Podman + WSL2 networking issue (resolved)

Symptom:

- Jupyter starts successfully in container, but Windows browser shows `ERR_CONNECTION_REFUSED` when opening `127.0.0.1:8888`.

Root cause:

- Podman is running inside WSL2, so localhost on Windows is not always the same network path as localhost inside WSL2/container.

Resolution:

1. Get WSL2 IP address from terminal:

```bash
hostname -I
```

2. Open Jupyter from Windows browser using the first WSL2 IP:

```text
http://<wsl2-ip>:8888/lab?token=<your-token>
```


3. If WSL restarts and IP changes, rerun `hostname -I` and use the new value.

Validation command inside WSL2:

```bash
curl "http://127.0.0.1:8888/lab?token=<your-token>"
```

If this returns HTML, Jupyter is healthy and the issue is only host-to-WSL networking.
