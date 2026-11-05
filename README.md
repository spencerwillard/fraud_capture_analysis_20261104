# Fraud Capture Opportunity Analysis

Evaluate how fraud score cutoffs change manual review volume, precision, and the share of labeled fraud identified. This Python mini-project uses pandas to validate application data, build a score performance table, and compare review strategies with Matplotlib.

## Key results

The notebook contains **10,000 applications** across **January–June 2025**, including **202 labeled fraudulent applications**: a **2.02% baseline fraud rate**.

| Review rule | Applications flagged | Flag rate | Labeled fraud flagged | Precision | Fraud recall | False positive rate |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Score ≥ 800 | 93 | 0.93% | 18 | 19.35% | 8.91% | 0.77% |
| Score ≥ 700 | 395 | 3.95% | 59 | 14.94% | 29.21% | 3.43% |
| Score ≥ 600 | 997 | 9.97% | 108 | 10.83% | 53.47% | 9.07% |

**All counts above cover the full six-month dataset, not one month.** Results are taken from the notebook’s saved cumulative performance table, rounded here to two decimal places. Cutoffs are inclusive because the score bins use `right=False` and cumulative totals run from the highest bin downward.

At a cutoff of 700, the review queue contains 59 labeled fraud cases and 336 non-fraud cases. Precision is about **7.39 times** the overall fraud rate (14.94% versus 2.02%). This is retrospective identification opportunity; it does not establish that those cases or their losses would have been prevented.

## Decision implications

A cutoff of **700 is a candidate for evaluation**, with higher precision and fewer reviews than 600, while identifying more labeled fraud than 800. Lowering the cutoff from 700 to 600 adds **602 reviews** and identifies **49 additional labeled fraud cases** across the observed period.

The notebook’s original recommendation describes 395 and 997 as monthly review counts. Those are six-month totals, so the claim that a 600 cutoff exceeds a capacity of 500 reviews per month is unsupported. The notebook does not show how those 997 flags are distributed by month. At 700, even all 395 flags in one month would remain below that capacity. Before selecting a cutoff, calculate flagged applications by month for each rule and evaluate review costs, customer friction, and the value of additional fraud detection. If prioritizing the greatest fraud recall under capacity, also evaluate the other available thresholds rather than limiting the decision to 600, 700, and 800.

## Data

The notebook reads `../data/fraud_capture_mini_project.csv` relative to its `notebooks/` directory.

| Field | Purpose |
| --- | --- |
| `application_id` | Application identifier |
| `application_date` | Application date, parsed with `%m/%d/%y` |
| `fraud_score` | Score used to prioritize review |
| `fraud_label` | Observed binary label: 1 = fraud, 0 = non-fraud |
| `application_amount` | Application amount; not used to estimate financial savings |

Saved data checks show no missing values, unique application IDs, no exact duplicate rows, and fraud scores from 4 to 965. Labels contain 9,798 non-fraud cases and 202 fraud cases. Observed application amounts range from 300 to 45,000; repeated amounts alone are not duplicate applications.

## Analysis workflow

1. Load and inspect the input data.
2. Check missing values, uniqueness, duplicates, score bounds, labels, and amounts.
3. Copy the data, parse dates, and create an `is_fraud` indicator.
4. Summarize monthly application counts, fraud counts, and fraud rates.
5. Assign 50-point score bins from `[0, 50)` through `[950, 1000)`.
6. Aggregate applications and fraud by bin; accumulate from the highest scores downward.
7. Calculate review flag rate, precision, recall, and false positive rate.
8. Compare cutoffs of 800, 700, and 600 in a table and grouped bar chart. The chart uses **steelblue** for application flag rate and **seagreen** for fraud recall.

## Metric definitions

- **Flag rate:** flagged applications ÷ all applications.
- **Precision:** flagged labeled fraud ÷ flagged applications.
- **Fraud recall:** flagged labeled fraud ÷ all labeled fraud.
- **False positive rate:** flagged non-fraud applications ÷ all non-fraud applications.

Notebook performance-table percentages are stored on a 0–100 scale. Baseline and monthly fraud rates are stored as fractions on a 0–1 scale. Format them accordingly when displaying results.

## Run locally

Use this layout:

```text
fraud-capture-opportunity/
├── README.md
├── data/
│   └── fraud_capture_mini_project.csv
└── notebooks/
    └── fraud_capture_analysis.ipynb
```

Install Python 3 and the required packages:

```bash
python3 -m pip install pandas numpy matplotlib jupyterlab
```

Start JupyterLab, open the notebook, and run all cells in order with the working directory set to `notebooks/`:

```bash
cd notebooks
python3 -m jupyterlab
```

No package versions are pinned in the supplied notebook. For more predictable categorical grouping across pandas versions, explicitly set `observed=False` in the score-bin aggregation to retain all defined bins.

## Limitations and next steps

- Findings describe the observed January–June 2025 sample; future application volumes and fraud patterns may differ.
- The notebook analyzes an existing score. It does not train a model or validate performance on a separate holdout period.
- Review success, intervention timing, and prevented losses are not measured. Application amounts are not equivalent to fraud losses.
- Monthly threshold review counts and review economics are needed to substantiate an operational cutoff recommendation.
- Sparse high-score bins can produce unstable precision estimates; for example, the highest bin contains one application.

## Evidence and reproducibility

This README was prepared from the actual code and saved outputs of `fraud_capture_analysis.ipynb`. The input CSV was not provided with the referenced notebook attachment, so the analysis was not rerun against its source data. The accompanying cleanup notes identify redundant cells and the corrections needed before presenting the notebook as a final analysis.
