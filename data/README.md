# Data Directory

This directory contains the datasets used in the email A/B experiment analysis.

## Included Files (committed to repo)

| File | Size | Description |
|------|------|-------------|
| `sample_segment_groups.csv` | ~2 KB | 12 user segment definitions |
| `sample_uuid_email_order.csv` | ~18 MB | 480K users × randomized email delivery schedule |
| `control_groups_rate.csv` | ~1 KB | Pre-computed control group baseline rates |

## Large Files (not committed — exceed GitHub 100MB limit)

The following two files are excluded from version control due to GitHub's file size constraints. They are required to run the full analysis notebook.

| File | Size | Description |
|------|------|-------------|
| `email_events.csv` | ~871 MB | Raw email events (9.5M rows) from the Post-Office delivery system |
| `user_events.csv` | ~81 MB | User-level email status and outcome metrics (480K users) |

### How to Obtain

These files were part of the original experiment data extract. If you have access to the source dataset, place them in this `data/` directory before running the notebook.

The analysis in `notebooks/Project-3-AB-testing.ipynb` primarily uses `user_events.csv` for all core analyses. The `email_events.csv` file is used only for the raw event volume breakdown chart in the Negative Effects section (Section 3).

### Column Reference: `user_events.csv`

| Column | Type | Description |
|--------|------|-------------|
| `user_uuid` | string | Unique user identifier |
| `group_name` | string | Treatment group (e.g., `20D_Act_D`, `6M_App_W`) |
| `ml_funding_enables_investing` | string | Email status: `delivered`/`open`/`unsubscribe`/`spamreport`/NaN |
| `ml_investing_starts_here` | string | Email status |
| `ml_explore_the_app_investing` | string | Email status |
| `ml_funding_faq` | string | Email status |
| `ml_user_clustering_emails_fracs` | string | Email status |
| `ml_funding_is_safe` | string | Email status |
| `ml_picking_an_investment` | string | Email status |
| `ml_investing_101` | string | Email status |
| `ml_diversified_portfolio` | string | Email status |
| `ml_explore_the_app_list` | string | Email status |
| `approved_at` | datetime | Account approval timestamp |
| `first_funded_at` | datetime | First funding event timestamp (primary outcome) |
| `first_linked_bank_account_at` | datetime | Bank account linkage timestamp |
| `5d_trading_avg_event_count` | float | Avg trading events in past 5 days |
| `20d_trading_avg_event_count` | float | Avg trading events in past 20 days |
| `num_received_email` | int | Total emails received (0 = not yet sent) |
