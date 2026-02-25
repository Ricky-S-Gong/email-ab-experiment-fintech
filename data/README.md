# Data Directory

This directory contains the datasets used in the email A/B experiment analysis.

## Included Files (committed to repo)

| File | Size | Description |
|------|------|-------------|
| `sample_segment_groups.csv` | ~2 KB | 12 user segment definitions |
| `sample_uuid_email_order.csv` | ~29 MB | 480K users × randomized email delivery schedule |
| `control_groups_rate.csv` | ~1 KB | Pre-computed control group baseline rates |

## Large Files — Download from GitHub Release

The following two files exceed GitHub's 100 MB file size limit and are distributed
via [**GitHub Releases → v1.0.0**](https://github.com/Ricky-S-Gong/email-ab-experiment-fintech/releases/tag/v1.0.0).

| File | Size | Direct Download |
|------|------|-----------------|
| `email_events.csv` | 871 MB | [⬇ Download](https://github.com/Ricky-S-Gong/email-ab-experiment-fintech/releases/download/v1.0.0/email_events.csv) |
| `user_events.csv` | 81 MB | [⬇ Download](https://github.com/Ricky-S-Gong/email-ab-experiment-fintech/releases/download/v1.0.0/user_events.csv) |

### One-command setup

```bash
# From the repo root — downloads both files directly into data/
gh release download v1.0.0 \
  --repo Ricky-S-Gong/email-ab-experiment-fintech \
  --dir data/
```

Or with `wget`:

```bash
wget -P data/ https://github.com/Ricky-S-Gong/email-ab-experiment-fintech/releases/download/v1.0.0/email_events.csv
wget -P data/ https://github.com/Ricky-S-Gong/email-ab-experiment-fintech/releases/download/v1.0.0/user_events.csv
```

> **Note:** `user_events.csv` is the primary analysis table used in almost every notebook section. `email_events.csv` is only required for the Negative Effects event-volume chart (Section 3). If you only want to run the core analyses, downloading `user_events.csv` alone is sufficient.

---

## Column Reference

### `user_events.csv`

| Column | Type | Description |
|--------|------|-------------|
| `user_uuid` | string | Unique user identifier |
| `group_name` | string | Treatment group (e.g., `20D_Act_D`, `6M_App_W`) |
| `ml_funding_enables_investing` | string | Email status: `delivered` / `open` / `unsubscribe` / `spamreport` / NaN |
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
| `first_funded_at` | datetime | First funding event timestamp (**primary outcome**; NaN = not yet funded) |
| `first_linked_bank_account_at` | datetime | Bank account linkage timestamp |
| `5d_trading_avg_event_count` | float | Avg trading events per day over past 5 days |
| `2d_non_trading_avg_event_count` | float | Avg non-trading app events per day over past 2 days |
| `20d_trading_avg_event_count` | float | Avg trading events per day over past 20 days |
| `8d_non_trading_avg_event_count` | float | Avg non-trading events per day over past 8 days |
| `1d_trading_avg_event_count` | float | Avg trading events on the most recent day |
| `1d_non_trading_avg_event_count` | float | Avg non-trading events on the most recent day |
| `num_received_email` | int | Total emails delivered to this user (0–10) |

### `email_events.csv`

| Column | Type | Description |
|--------|------|-------------|
| `stitch_email_events.category` | string | JSON-like string with template name and sender, e.g. `["ml_funding_faq","post-office"]` |
| `stitch_email_events.dt_date` | string | Event date (`YYYY-MM-DD`) |
| `user_uuid` | string | User identifier |
| `event` | string | Event type: `processed`, `delivered`, `open`, `dropped`, `bounce`, `deferred`, `unsubscribe`, `spamreport` |
| `reason` | string | ESP error string for bounce/drop events; NaN otherwise |
| `stitch_email_events.count_events` | int | Event count for this user-email-date (usually 1) |
