# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a data analysis project for an **email delivery A/B experiment** aimed at improving user funding rates for a fintech/investing app. The experiment evaluates 10 ML-generated email templates sent to segmented user groups, comparing treatment (email-receiving) groups against control groups.

## Running the Notebooks

Notebooks are designed for **Jupyter / Google Colab**. To run locally:

```bash
# Install dependencies
pip install pandas numpy matplotlib statsmodels seaborn

# Launch Jupyter
jupyter notebook
```

The solution notebook (`Email_Experiment_Project_solution.ipynb`) uses Google Drive paths — switch these to local `data/` paths as done in `Project-3-AB-testing.ipynb`:

```python
# Colab path (in solution notebook)
pd.read_csv('/content/drive/MyDrive/Email_project/sample_segment_groups.csv')

# Local path (use this instead)
pd.read_csv('data/sample_segment_groups.csv')
```

## Key Libraries

```
pandas==2.2.2
numpy==1.26.4
matplotlib==3.9.2
statsmodels==0.14.2
seaborn==0.14.2
```

Statistical testing uses `statsmodels.stats.proportion.proportions_ztest` (one-sided, `alternative='larger'`).

## Data Files (`data/`)

| File | Description |
|------|-------------|
| `sample_segment_groups.csv` | 12 user segments defined by 4 boolean flags (approved_within6M, link, recent_activity_20d, 5d_trade) with user counts per segment |
| `sample_uuid_email_order.csv` | 480,000 users × randomized email delivery order (order_0 through order_9 are email IDs 0–9) |
| `email_events.csv` | Raw email events per user/email: open, delivered, bounce, unsubscribe, spamreport, deferred, dropped, processed |
| `user_events.csv` | One row per user with email status per template, user activity metrics, and outcome columns (approved_at, first_funded_at, first_linked_bank_account_at) |
| `control_groups_rate.csv` | Pre-computed funding and link rates for control groups (users who received no emails) |

## Experiment Design

- **10 email templates** (`PO_number_list`): `ml_funding_enables_investing`, `ml_investing_starts_here`, `ml_explore_the_app_investing`, `ml_funding_faq`, `ml_user_clustering_emails_fracs`, `ml_funding_is_safe`, `ml_picking_an_investment`, `ml_investing_101`, `ml_diversified_portfolio`, `ml_explore_the_app_list`
- **12 user segments** (groups 0–11) defined by combinations of: approved within 6 months, linked bank account, recent 20-day activity, recent 5-day trading activity
- **Two send frequencies**: `_D` suffix = daily, `_W` suffix = twice-a-week
- **Control groups**: same segmentation, no emails sent; rates stored in `control_groups_rate.csv`

## Analysis Workflow (Solution Notebook)

1. **Load data**: segment groups, user-email order assignments, email events, user status, control rates
2. **Email open rate**: compute delivered counts and open counts per group × template, derive open rate heatmap
3. **Negative effects**: check unsubscribe + spamreport rates
4. **Correlation analysis**: flatten open rates with segment flags to find which user traits correlate with opening emails
5. **Funding/link rates for treatment**: filter users who received ≥1 email, compute funding and link conversion rates per group
6. **A/B test**: merge treatment rates with control rates, run one-sided proportions z-test (`H0`: treatment rate ≤ control rate), flag `significant` where `p_val ≤ 0.05`
7. **Time-series analysis**: pivot email events by delivery order (order_0…order_9) to track open rates across send sequence; check experiment completion (weekly groups may not have received order_9 yet)
8. **Funnel analysis**: combine link rate, activity rate, and funding rate into a conversion funnel; compute `delta_rate_funnel` and `extra_conversion_rate`

## Working Notebook vs. Template

- `Project-3-AB-testing.ipynb` — active working notebook with local data paths; use this for development
- `Email_Experiment_Project_template.ipynb` — starter template (Colab paths, minimal code)
- `Email_Experiment_Project_solution.ipynb` — reference solution (Colab paths)
