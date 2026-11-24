# Customer Churn Analytics & Retention Intelligence

An end-to-end customer churn analytics project that combines **SQL and Python** to transform customer, subscription, and support data into actionable retention insights. The workflow covers data ingestion, cleaning, relational joins, feature engineering, KPI analysis, and visual exploration.

> **Business objective:** Understand which customers are more likely to churn, quantify the recurring revenue exposed to churn, and identify where retention teams should focus their efforts.

---

## Executive Summary

The analysis is designed to answer four business questions:

1. How many customers are leaving, and how is churn changing over time?
2. Which subscription and plan segments have the highest churn?
3. How much recurring monthly revenue is associated with churned customers?
4. Do customer-support signals, such as escalations and complaints, relate to churn?

### Key business findings

- **Monthly contracts showed a 55.6% churn rate**, nearly **7× the annual-contract churn rate**, identifying contract commitment as an important retention segment to investigate.
- The analysis quantified approximately **$74 in monthly recurring charges at risk** among churned accounts.
- The notebook also calculates churn, retention, ARPU, customer tenure, escalation rate, complaint frequency, and the correlation between escalations and churn.

*Metric validation note: the supplied notebook's saved output currently shows a 21-customer working dataset, an overall churn rate of 28.57%, and 73.94 as the sum of monthly charges for churned customers. The 55.6% contract-level result and the currency symbol should be rechecked against the complete source dataset before presenting them as final business-wide figures. The notebook currently calculates churn by `plan_type`; add a `contract_type` group-by to reproduce the monthly-versus-annual comparison.*

---

## Project Workflow

```text
Raw customer / subscription / support data
                  |
                  v
          SQL database (SQLite)
                  |
                  v
       Import tables into Pandas
                  |
                  v
     Data cleaning and standardisation
                  |
                  v
    Join customer + subscription + support
                  |
                  v
        Feature engineering
                  |
                  v
       Business KPI calculations
                  |
                  v
      Visualisation and insights
                  |
                  v
       Retention recommendations
```

## Technical Stack

| Tool | How it is used |
|---|---|
| **SQL / SQLite** | Database creation, table inspection, and structured data storage |
| **Python** | Analysis workflow and business logic |
| **Pandas** | Data cleaning, joins, aggregations, and KPI calculations |
| **NumPy** | Churn-flag and risk-segment creation |
| **Matplotlib** | Churn trend and segment visualisations |
| **Seaborn** | Correlation heatmaps and multi-dimensional analysis |
| **Jupyter Notebook** | Reproducible, step-by-step analysis |

---

## Data Model

The notebook works with three logical tables:

- **Customer:** customer ID, name, country, state, gender, and date of birth.
- **Subscription:** subscription dates, subscription source/type, plan, contract, cancellation details, monthly charges, customer lifetime value (CLTV), and churn score.
- **Support:** complaint date, escalation indicator, and customer satisfaction score.

The tables are connected using `customerid`. Support records are deduplicated by customer after calculating complaint counts, so the final customer-level dataset does not multiply rows because of multiple support interactions.

## What the Analysis Covers

### 1. Data ingestion and preparation

- Reads SQLite tables into Pandas DataFrames.
- Includes an optional Excel-to-SQLite loading step.
- Inspects table schemas and column types.
- Renames columns for clarity and removes unused fields.
- Converts date fields to datetime.
- Standardises gender labels.
- Fills missing country values using the corresponding state-to-country mapping.

### 2. Data integration and feature engineering

- Creates `churn_flag` from `cancellation_date`:
  - `1` = customer has a cancellation date.
  - `0` = customer has not cancelled.
- Calculates complaint counts before deduplicating support records.
- Uses left joins to retain the customer/subscription population while adding customer and support attributes.
- Calculates customer tenure using cancellation date for churned customers and the current date for active customers.
- Segments customers into low-, medium-, and high-risk groups using the available `churn_score`:
  - Low: score below 50
  - Medium: score from 50 to below 70
  - High: score of 70 or above

### 3. Business KPIs

| KPI | Definition / purpose |
|---|---|
| Churn rate | Share of customers with `churn_flag = 1` |
| Retention rate | 100% minus churn rate |
| Churn by plan / contract | Compares churn across subscription segments |
| ARPU | Average monthly charges per customer in the analysed dataset |
| Average tenure | Average number of days customers remain subscribed |
| Revenue at risk | Sum of monthly charges associated with churned customers |
| Escalation rate | Share of records marked as escalated |
| Average complaints per user | Complaint count divided by unique customers |
| Escalation–churn correlation | Measures the linear association between escalation and churn flags |

### 4. Visual analysis

The notebook includes:

- Monthly churn trend line chart.
- Churn rate by plan type.
- Churn rate by state.
- Correlation heatmap for selected customer and subscription features.
- Pairwise feature plots.
- Plan, monthly-charge, gender, and churn-risk comparisons.
- Pivot tables for segment-level churn and revenue summaries.

---

## Running the Project

### Prerequisites

- Python 3.9+
- Jupyter Notebook or JupyterLab
- The source data files referenced by the notebook:
  - `customer_churn.db`, or
  - `customer_churn_data_raw.xlsx` if creating the SQLite database locally.

Install the required packages:

```bash
pip install numpy pandas matplotlib seaborn jupyter openpyxl
```

### Steps

1. Clone or download this repository.
2. Place the source database (`customer_churn.db`) in the notebook's working directory. Alternatively, place `customer_churn_data_raw.xlsx` there and run the notebook's Excel-to-SQLite setup cell.
3. Open `churn_analysis.ipynb` in Jupyter.
4. Run the cells from top to bottom.
5. Review the KPI outputs, charts, and exported `exported_churn_data.csv`.

```bash
jupyter notebook churn_analysis.ipynb
```

---

## Example: Reproducing Contract-Level Churn

The following aggregation can be run after the final merged DataFrame (`df`) has been created. It calculates the churn rate separately for monthly and annual contracts:

```python
contract_churn = (
    df.groupby("contract_type")["churn_flag"]
      .mean()
      .mul(100)
      .round(2)
      .reset_index(name="churn_rate_pct")
)

print(contract_churn)
```

To calculate the ratio between monthly and annual churn rates:

```python
rates = contract_churn.set_index("contract_type")["churn_rate_pct"]

if "Monthly" in rates.index and "Annual" in rates.index and rates["Annual"] != 0:
    print("Monthly-to-annual churn ratio:", round(rates["Monthly"] / rates["Annual"], 2))
```

This makes the contract-level comparison reproducible rather than relying on a manually reported figure.

---

## Retention Opportunities

The analysis can help a business prioritise follow-up actions:

- **Review monthly-contract journeys:** investigate whether shorter commitment, pricing concerns, or renewal experience contribute to higher churn.
- **Prioritise high-risk accounts:** use the existing churn score to create targeted retention lists.
- **Investigate support escalations:** examine escalated cases alongside churn outcomes to identify service issues that may need intervention.
- **Monitor recurring revenue exposure:** track monthly charges associated with churned accounts to quantify the financial impact.
- **Track segment KPIs over time:** compare churn across contract types, plans, and geographies to measure whether retention actions are working.

These are analysis-led opportunities, not proof of causation. The correlation between escalations and churn should be treated as an association and investigated further.


## Future Scope

- The notebook's saved outputs represent a small working dataset; validate all headline metrics on the complete dataset before making operational decisions.
- Correlation does not establish causation.
- The current workflow performs descriptive analytics and risk segmentation using an existing churn score; it does **not** train or evaluate a new predictive machine-learning model.
- Next steps could include cohort analysis, a validated churn-prediction model, model evaluation, and measurement of retention-campaign lift.

---

## Project Takeaway

This project demonstrates a practical analytics workflow: integrating relational data with Python, preparing a customer-level dataset, engineering churn-related features, and translating the results into business KPIs that can inform customer-retention decisions.
