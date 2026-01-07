
# Automated Hyperparameter Optimization with Optuna and MLflow

## Author Information

* **Name:** Billakurti Venkata Suryanarayana
* **Roll Number:** 23MH1A4409

---

## Project Overview

This project implements a **production-grade automated hyperparameter optimization pipeline** using **Optuna** and **MLflow**. The goal is to move beyond manual trial-and-error tuning by implementing an intelligent, reproducible system to optimize an **XGBoost regressor** on the **California Housing dataset**.

The system demonstrates modern **MLOps practices**, including:

* **Automated Bayesian Search:** Using Optuna to efficiently navigate hyperparameter spaces.
* **Experiment Tracking:** Logging every trial and metric to MLflow for full auditability.
* **Containerized Execution:** Packaging the entire workflow into Docker for guaranteed reproducibility across any environment.

---

## Detailed Project Explanation

### 1. Data Pipeline

The system utilizes the **California Housing dataset**. To ensure reliability against network errors (HTTP 403), the data is stored locally in `data/california_housing.csv`. The pipeline automatically splits this data into an 80/20 train/test ratio with a fixed seed for consistency.

### 2. Search Space & Optimization

The pipeline tunes **7 critical hyperparameters**:

* `n_estimators`, `max_depth`, `learning_rate` (log scale), `subsample`, `colsample_bytree`, `min_child_weight`, and `gamma`.

It employs a **MedianPruner** to stop underperforming trials early, saving significant computational time. The optimization runs for exactly **100 trials** and uses a SQLite backend to support parallel execution with multiple workers.

### 3. Experiment Tracking (MLflow)

Every trial is logged to the `optuna-xgboost-optimization` experiment. After the search, the system:

* Identifies the best parameters.
* Retrains a final model and logs it as an MLflow artifact.
* Saves visualization plots (Optimization History and Param Importance) directly into MLflow.

---

## Repository Structure

```text
.
├── Dockerfile                  # Builds the Python 3.9 environment
├── requirements.txt            # Project dependencies (XGBoost, Optuna, MLflow, etc.)
├── .dockerignore               # Optimizes Docker build context
├── data/                       # Dataset storage
│   └── california_housing.csv  # Verified dataset
├── src/                        # Modular source code
│   ├── data_loader.py          # Data splitting logic
│   ├── objective.py            # Search space and CV logic
│   ├── optimize.py             # Main orchestration script
│   └── evaluate.py             # Results persistence
├── notebooks/                  # Post-optimization analysis
│   └── analysis.ipynb          # Detailed insights and baseline comparison
└── outputs/                    # Runtime generated artifacts (MLflow, DB, Plots)

```

---

## Setup and Usage Instructions

### 1. Clone the Repository

```bash
git clone <your-repository-url>
cd <repository-name>

```

### 2. Build the Docker Image

```bash
docker build -t optuna-mlflow-pipeline .

```

### 3. Run the Optimization Pipeline

To ensure the results are saved to your local machine, run the container with a volume mount:

**For Linux or macOS:**

```bash
docker run -v $(pwd)/outputs:/app/outputs optuna-mlflow-pipeline

```

**For Windows (PowerShell):**

```bash
docker run -v ${PWD}/outputs:/app/outputs optuna-mlflow-pipeline

```

---

## Expected Outputs

After the run completes, the `outputs/` directory will contain:

* **`results.json`**: A summary of the best parameters and test metrics.
* **`optimization_history.png`**: Plot showing the objective value over trials.
* **`param_importance.png`**: Chart showing which parameters impacted performance most.
* **`mlruns/`**: Complete MLflow tracking database and model artifacts.
* **`optuna_study.db`**: The SQLite database for the Optuna study.