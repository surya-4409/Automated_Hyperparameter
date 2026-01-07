
# Automated Hyperparameter Optimization with Optuna and MLflow

## Author Information

* **Name:** Billakurti Venkata Suryanarayana
* **Roll Number:** 23MH1A4409

---

## Project Overview

This project implements a **production-grade automated hyperparameter optimization pipeline** using **Optuna** and **MLflow**. The core objective is to move beyond manual trial-and-error tuning by implementing an intelligent, reproducible system to improve the performance of an **XGBoost regressor** on the **California Housing dataset**.

The system is designed to demonstrate modern **MLOps practices**, including:

* **Automated Search:** Utilizing Bayesian optimization via Optuna to find the best model configuration.
* **Experiment Tracking:** Using MLflow to log every trial, parameter, and metric for full audibility.
* **Containerization:** Ensuring the entire environment is portable and reproducible via Docker.

---

## Detailed Project Explanation

### 1. Data Pipeline

The system uses the **California Housing dataset**, which is split into an 80/20 train/test ratio. Due to potential network restrictions when fetching the data online, a verified local version (`data/california_housing.csv`) is included to ensure the pipeline remains robust.

### 2. Hyperparameter Search Space

The pipeline tunes **exactly 7 hyperparameters** for the XGBoost model to balance performance and complexity:

* `n_estimators`: [50, 300]
* `max_depth`: [3, 10]
* `learning_rate`: [0.001, 0.3] (Logarithmic scale)
* `subsample`: [0.6, 1.0]
* `colsample_bytree`: [0.6, 1.0]
* `min_child_weight`: [1, 10]
* `gamma`: [0, 0.5]

### 3. Optimization Strategy (Optuna)

* **Objective Function:** Each trial calculates a 5-fold cross-validation score based on Negative Mean Squared Error (MSE).
* **Pruning:** To save computational resources, a `MedianPruner` stops underperforming trials early if they don't show potential after an initial "warm-up" period.
* **Parallelism:** The study uses a SQLite backend (`optuna_study.db`), allowing multiple workers (`n_jobs=2`) to run trials concurrently without race conditions.

### 4. Experiment Tracking (MLflow)

Every trial is logged to an MLflow experiment named `optuna-xgboost-optimization`.

* **Per Trial:** Logs parameters, `cv_rmse`, `trial_number`, and a state tag (COMPLETE, PRUNED, or FAIL).
* **Best Model:** After 100 trials, the pipeline retrains the model using the best parameters and creates a final MLflow run tagged `best_model=true`.
* **Artifacts:** The final model, optimization history plots, and parameter importance charts are saved as MLflow artifacts.

### 5. Containerization (Docker)

The `Dockerfile` creates a Python 3.9 environment that installs all necessary system and Python dependencies. It is configured to:

* Install `chromium` and `kaleido` for generating static visualization images.
* Map an internal volume (`/app/outputs`) to the host machine so that results, the database, and MLflow logs persist after the container exits.

---

## Repository Structure

```text
.
├── Dockerfile                  # Builds the reproducible environment
├── requirements.txt            # Python package dependencies
├── data/                       # Dataset storage
│   └── california_housing.csv  # California Housing data
├── src/                        # Modular source code
│   ├── data_loader.py          # Data loading and 80/20 split logic
│   ├── objective.py            # Definition of search space and CV logic
│   ├── optimize.py             # Main pipeline orchestration
│   └── evaluate.py             # Logic for saving results.json
├── notebooks/                  # Post-optimization analysis
│   └── analysis.ipynb          # Visual insights and baseline comparison
└── outputs/                    # Runtime generated (MLflow, DB, and Plots)

```

---

## How to Run

1. **Build the Image:**
```bash
docker build -t optuna-mlflow-pipeline .

```


2. **Run the Pipeline:**
```bash
docker run -v $(pwd)/outputs:/app/outputs optuna-mlflow-pipeline

```


*All outputs (results.json, plots, and logs) will appear in your local `outputs/` folder.*
