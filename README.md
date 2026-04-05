# 📈 Azure MLOps Trading Pipeline: Regime Clustering & Agentic AI

This repository contains the backend machine learning operations (MLOps) pipeline for a quantitative trading system. It utilizes a decoupled, two-step architecture deployed on Azure Machine Learning, featuring an auto-scaling compute cluster and secure identity management.

## 🏗️ Enterprise Architecture

This pipeline is designed with strict separation of concerns, ensuring high fault tolerance and modularity. It is orchestrated via an Azure ML Job Schedule.

### 1. Data Engineering & Clustering (`etl_pipeline.py`)
* **Extraction:** Securely fetches daily market CSVs and macroeconomic JSON data from Azure Blob Storage.
* **Transformation:** Calculates 20-day SPY volatility and daily returns. Merges macroeconomic indicators (CPI) using forward-fill techniques.
* **Machine Learning:** Executes a K-Means Clustering algorithm (k=3) to classify the current market environment into distinct Regimes.
* **Loading:** Pushes the cleaned, classified data into an Azure SQL Database (`ProcessedMarketData`).

### 2. Agentic Thesis Generation (`ai_agent.py`)
* **Trigger Mechanism:** Waits for a successful "dummy signal" from the ETL step (preventing AI execution on dirty/stale data).
* **Context Gathering:** Reads the single most recent row from Azure SQL.
* **LLM Integration:** Prompts Google's Gemini 2.5 Flash model with the current market regime, volatility, and macro data to generate a structured JSON investment thesis (Macro, Sector Signals, Risk Protocol).
* **Alerting:** Broadcasts the AI thesis via Discord Webhooks and automated Email, linking to the decoupled frontend Streamlit dashboard.

## 🔐 Security & Infrastructure

This project adheres to strict cloud security standards:
* **Zero-Trust Secrets:** No API keys or passwords are hardcoded. All secrets (Gemini API, SQL Passwords, Webhooks) are fetched dynamically at runtime from **Azure Key Vault**.
* **Identity Pass-Through:** The pipeline uses a System Assigned Managed Identity, completely eliminating the need for local `.env` files or hardcoded service principals.
* **Auto-Scaling Compute:** Runs on a 0-node minimum Azure ML Compute Cluster (`trading-cluster`), ensuring infrastructure costs scale to zero immediately after the daily job completes.

## 🚀 Deployment

The pipeline graph and recurrence schedule are defined in `03_scheduler.ipynb`. 

To deploy or update the schedule:
1. Ensure your Azure ML Workspace environment contains the dependencies listed in `conda.yml`.
2. Run the `03_scheduler.ipynb` notebook to bind the decoupled components into an Azure ML Pipeline Job.
3. The cluster will automatically boot up at the scheduled time, execute Step 1, pass the signal to Step 2, and shut down.

---
*Part of a microservice architecture. See the frontend dashboard repository for the user interface.*