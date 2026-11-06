# Telecom-Customer-Churn-Prediction-Retention-Analytics
Telecom churn prediction and retention analytics using Python, SQL, and scikit-learn, with an interactive dashboard for scoring customer CSV and Excel files and estimating campaign ROI.

Predict 30-day customer churn and use individual risk scores to estimate which customers are worth contacting under a given retention budget and set of campaign assumptions.

**[Try the live dashboard](https://telecom-churn-analytics.scenicmaple.chatgpt.site/)** · **[Data dictionary](data/DATA_DICTIONARY.md)** · **[Test results](reports/test_metrics.csv)**

## Live Dashboard

Explore the model results and risk scores for 1,000 synthetic test customers, or score your own customer file.

1. Click **Download Excel template** and replace the example rows with your customer data.
2. Import an `.xlsx`, `.xls`, or UTF-8 `.csv` file. Keep the template headers. A churn label is optional.
3. Review customer-level churn scores, adjust campaign assumptions, and download the filtered results as CSV.

The dashboard supports English and Chinese. It processes imported files in the browser and scores them with the existing trained model.

## Data and Modeling

The project generates 5,000 reproducible synthetic telecom customer records and stores them in CSV and SQLite. The six input features are tenure, monthly charges, contract type, recent support calls, internet service, and payment method. The target is whether the customer churns in the following 30 days.

- **Exploratory analysis** examines churn prevalence and differences by contract type, monthly charges, and support calls.
- **Preprocessing** uses a scikit-learn Pipeline for missing-value imputation, numerical scaling, and one-hot encoding.
- **Model selection** compares Logistic Regression and Random Forest using a stratified 60/20/20 train, validation, and test split. Validation ROC-AUC determines the winner. The selected pipeline is refit on the combined training and validation data, then evaluated on the held-out test set.

## Results

| Model | Validation ROC-AUC |
| --- | ---: |
| Logistic Regression | 0.765 |
| Random Forest | 0.748 |

Logistic Regression achieved a **test ROC-AUC of 0.741**. At a classification threshold of 0.50, it correctly flagged **111 of 307** customers who churned.

| Accuracy | Precision | Recall | F1 |
| ---: | ---: | ---: | ---: |
| 72.5% | 58.4% | 36.2% | 0.447 |

![Held-out test confusion matrix and ROC curve](reports/test_evaluation.png)

## Retention Analysis

The example campaign assumes a **$20 cost per contact**, **$600 in future profit per successfully retained customer**, and a **30% chance that outreach retains a contacted customer who would otherwise churn**.

```text
Expected net value per contact = churn probability × 0.30 × $600 − $20
```

Under these assumptions, contacting customers with a churn probability above **11.11%** has positive expected net value. On the 1,000-customer synthetic test set, this policy selects **844 customers**, with **$16,880** in outreach costs and approximately **$38,088** in expected net profit. See the [campaign comparison](reports/retention_roi.csv) for the no-contact and contact-all scenarios.

The model was trained on synthetic data, and campaign returns are scenario estimates. Before using the model for real customer decisions, validate its scores against real historical outcomes and estimate the incremental effect of outreach with a randomized experiment.

## Run Locally

Open the repository folder in VS Code or Codex. In Windows PowerShell, with Python 3.12 installed:

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe churn_prediction.py
.\.venv\Scripts\python.exe -m unittest discover -s tests -v
```

On macOS or Linux, create the environment with `python3.12 -m venv .venv` and use `.venv/bin/python` in the remaining commands.

| Path | Contents |
| --- | --- |
| `churn_prediction.py` | End-to-end training and evaluation |
| `src/` | Data generation, visualization, scoring, and retention calculations |
| `data/` | Synthetic CSV, SQLite database, and data dictionary |
| `sql/` | Example queries for exploratory analysis |
| `reports/` | EDA charts, test metrics, customer risk scores, and campaign results |
| `tests/` | Automated checks for data, preprocessing, and campaign calculations |
