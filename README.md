# Email Delivery A/B Experiment: Optimizing Funding Rate for Fintech Users

**A production-grade multi-arm A/B experiment analyzing the causal impact of personalized email campaigns on user funding behavior for a retail investing platform.**

---

## Table of Contents
- [Project Overview](#project-overview)
- [Motivation & Business Context](#motivation--business-context)
- [Experiment Design](#experiment-design)
- [Repository Structure](#repository-structure)
- [Key Findings](#key-findings)
- [Analysis Workflow](#analysis-workflow)
- [Tech Stack](#tech-stack)
- [How to Run](#how-to-run)

---

## Project Overview

This project simulates a real-world growth experiment at a fintech company (analogous to Webull, Robinhood, or Square). Our team designed and executed a **5-week multi-arm email A/B experiment** targeting 480,000 approved-but-unfunded users to answer a core business question:

> **Can a targeted email campaign causally increase the probability that an approved user will fund their investment account?**

The experiment tested 10 ML-generated email templates across 12 behavioral user segments, delivered at two frequencies (daily and twice-a-week), yielding **24 treatment arms** with matched control groups.

---

## Motivation & Business Context

In retail investing platforms, the **funding rate** (percentage of approved users who deposit and begin trading) is a critical growth and revenue metric. A significant portion of users complete account approval but never take the final step of funding — a gap commonly caused by:

- **Knowledge barriers**: Uncertainty about how the funding process works
- **Trust concerns**: Hesitation about linking bank accounts or transferring money
- **Engagement decay**: Users approve accounts but lose momentum before funding

Email marketing is a high-ROI, low-cost intervention for re-engaging these users. However, not all email content works equally well across different user segments, and poorly targeted campaigns can backfire through increased unsubscribes and spam reports — damaging long-term deliverability.

**This experiment provides a rigorous, statistically-grounded answer** to which combination of email content, user segment, and delivery frequency maximizes incremental funding lift.

---

## Experiment Design

| Parameter | Details |
|-----------|---------|
| **Audience** | 480,000 approved-but-unfunded users (as of 2020-11-27) |
| **Experiment Period** | 2020-11-30 (Day 0) through Day 32 |
| **Email Templates** | 10 ML-generated templates covering: trust/safety, knowledge gaps, simplicity of use |
| **User Segments** | 12 segments defined by 4 binary behavioral flags (approved within 6M, bank linked, 20-day activity, 5-day trading) |
| **Delivery Frequencies** | Daily (_D): Days 0–9 · Twice-a-week (_W): Mon/Fri schedule, Days 0/4/7/11/14/18/21/25/28/32 |
| **Treatment Arms** | 24 (12 segments × 2 frequencies) |
| **Email Ordering** | Randomized without replacement per user (eliminates order-specific confounding) |
| **Control Groups** | Matched users with same segmentation, zero email exposure |
| **Statistical Test** | One-sided proportions z-test (H₁: treatment rate > control rate, α = 0.05) |

### User Segmentation Matrix

| Segment | Approved <6M | Bank Linked | Active 20d | Trade 5d |
|---------|:---:|:---:|:---:|:---:|
| ML_unfund_exp_control | ✗ | ✗ | ✗ | ✗ |
| 20D_Act | ✗ | ✗ | ✓ | ✗ |
| 20D_Act-5D_Act | ✗ | ✗ | ✓ | ✓ |
| link | ✗ | ✓ | ✗ | ✗ |
| link-20D_Act | ✗ | ✓ | ✓ | ✗ |
| link-20D_Act-5D_Act | ✗ | ✓ | ✓ | ✓ |
| 6M_App | ✓ | ✗ | ✗ | ✗ |
| 6M_App-20D_Act | ✓ | ✗ | ✓ | ✗ |
| 6M_App-20D_Act-5D_Act | ✓ | ✗ | ✓ | ✓ |
| 6M_App-link | ✓ | ✓ | ✗ | ✗ |
| 6M_App-link-20D_Act | ✓ | ✓ | ✓ | ✗ |
| 6M_App-link-20D_Act-5D_Act | ✓ | ✓ | ✓ | ✓ |

---

## Repository Structure

```
email-ab-experiment-fintech/
│
├── notebooks/
│   └── Project-3-AB-testing.ipynb     # Complete analysis notebook (main deliverable)
│
├── data/
│   ├── sample_segment_groups.csv       # 12 user segment definitions with user counts
│   ├── sample_uuid_email_order.csv     # 480K users × randomized email delivery schedule
│   ├── email_events.csv                # Raw email events (9.5M rows) — NOT in git (871 MB)
│   ├── user_events.csv                 # User-level email status + outcome metrics — NOT in git (81 MB)
│   ├── control_groups_rate.csv         # Pre-computed funding/link rates for control groups
│   └── README.md                       # Column reference for large excluded files
│
├── assets/
│   ├── open_rate_heatmap.png           # Email open rate: group × template heatmap
│   ├── avg_open_rate_by_template.png   # Average open rate per email template
│   ├── negative_effects.png            # Unsubscribe/spam rate analysis
│   ├── correlation_analysis.png        # User traits vs. open rate correlation
│   ├── funding_link_rates.png          # Funding & link rates by treatment group
│   ├── ab_test_results.png             # A/B test: treatment vs. control comparison
│   ├── time_series_open_rate.png       # Open rate heatmap by delivery order
│   ├── open_rate_trend.png             # Open rate trend curves across email sequence
│   └── funnel_analysis.png             # Email open → link → fund conversion funnel
│
├── reports/
│   ├── Report.tex                      # Executive report (LaTeX source)
│   └── Report.pdf                      # Compiled PDF report (29 pages)
│
└── README.md
```

---

## Data Dictionary

The experiment uses five structured datasets that form a clear pipeline from design to outcome measurement. Below is a complete reference for every file and every column.

---

### File 1: `sample_segment_groups.csv` — Experiment Segmentation Design
**Shape:** 12 rows × 8 columns · **Role:** Defines the 12 behavioral user segments and their population sizes.

| Column | Type | Description |
|--------|------|-------------|
| `group_id` | int | Integer ID 0–11 identifying each segment |
| `approved_within6M_flag` | bool | `True` if user's account was approved within the past 6 months — proxy for recency of onboarding intent |
| `link_flag` | bool | `True` if user has linked at least one bank account — prerequisite step before funding |
| `recent_activity_flag(20days)` | bool | `True` if user had any platform activity (login, browse, watch) in the past 20 days |
| `5day_trade_flag` | bool | `True` if user executed at least one trade in the past 5 days — the strongest behavioral engagement signal |
| `user_uuid` | int | **Note:** Stores the *count* of users in the full population pool for that segment, **not** an individual user ID |
| `group_name` | str | Human-readable label built by concatenating active flags, e.g. `6M_App-link-20D_Act` |

**Interpretation:** The four binary flags create a behavioral engagement hierarchy. Segments at the top (e.g., `6M_App-link-20D_Act-5D_Act`) are intent-rich users who are functionally one step from funding. The baseline `ML_unfund_exp_control` segment has no distinguishing behavioral signals.

---

### File 2: `sample_uuid_email_order.csv` — Per-User Delivery Schedule
**Shape:** 480,000 rows × 13 columns · **Role:** The randomization manifest — records exactly which email was assigned to each delivery position for each user.

| Column | Type | Description |
|--------|------|-------------|
| `user_uuid` | str | Unique user identifier in the format `id_{integer}` — primary key for joining with `user_events.csv` |
| `group_id` | int | Integer group ID 0–23 (24 groups = 12 segments × 2 frequencies) |
| `group_name` | str | Full treatment arm label, e.g. `20D_Act_D` (daily) or `6M_App_W` (twice-a-week) |
| `order_0` | int | Index (0–9) into `PO_number_list` of the email assigned to delivery position 0 (first send) |
| `order_1` | int | Email index for delivery position 1 (second send) |
| … | … | … |
| `order_9` | int | Email index for delivery position 9 (final send) |

**Email index → template name mapping (`PO_number_list`):**

| Index | Template Name | Content Focus |
|-------|--------------|---------------|
| 0 | `ml_funding_enables_investing` | How funding unlocks investing features |
| 1 | `ml_investing_starts_here` | Getting-started / onboarding |
| 2 | `ml_explore_the_app_investing` | App exploration (investment features) |
| 3 | `ml_funding_faq` | FAQ about the funding process (trust) |
| 4 | `ml_user_clustering_emails_fracs` | ML-personalized cluster-based content |
| 5 | `ml_funding_is_safe` | Trust & safety / security assurances |
| 6 | `ml_picking_an_investment` | Investment selection guidance |
| 7 | `ml_investing_101` | Investing education / basics |
| 8 | `ml_diversified_portfolio` | Portfolio diversification concepts |
| 9 | `ml_explore_the_app_list` | App exploration (browsing/list features) |

**Randomization:** Each user's `order_0`–`order_9` is a permutation of {0–9} drawn without replacement, ensuring every user receives all 10 templates exactly once and no template is systematically favored in any delivery position.

**Timing:** For `_D` groups: position 0 = Day 0, position 1 = Day 1, …, position 9 = Day 9. For `_W` groups: positions map to Mon/Fri delivery days (0=Day 0, 1=Day 4, 2=Day 7, … 9=Day 32).

---

### File 3: `user_events.csv` — Primary Analysis Table
**Shape:** 480,000 rows × 22 columns · **Role:** One row per experiment user with summarized email status, account outcome timestamps, and platform activity metrics. This is the central table for all analyses.
**Download:** [⬇ user_events.csv (81 MB)](https://github.com/Ricky-S-Gong/email-ab-experiment-fintech/releases/download/v1.0.0/user_events.csv)

#### Identifiers & Group Assignment
| Column | Type | Description |
|--------|------|-------------|
| `user_uuid` | str | Unique user ID — primary join key |
| `group_name` | str | Treatment arm (24 possible values) |

#### Email Interaction Status (10 columns, one per template)
Each `ml_*` column stores the **highest-intent action** that user took on that specific email template:

| Value | Meaning |
|-------|---------|
| `NaN` | Email not yet sent to this user, or was dropped before delivery |
| `delivered` | Email reached inbox; user did not open it |
| `open` | User opened the email |
| `unsubscribe` | User clicked the unsubscribe link (counter-metric) |
| `spamreport` | User reported the email as spam (critical counter-metric) |

> **Key encoding rule:** `NaN` means "not delivered" for purposes of computing open rate — use `.notnull()` to identify the delivered population, then `== 'open'` to identify openers.

#### Account Lifecycle Outcomes
| Column | Type | Description |
|--------|------|-------------|
| `approved_at` | datetime | Timestamp when the user passed identity verification (KYC) and account was approved |
| `first_funded_at` | datetime | **Primary outcome.** Timestamp of user's first qualifying deposit. `NaN` = not yet funded during the observation window |
| `first_linked_bank_account_at` | datetime | **Secondary outcome.** Timestamp of first bank account linkage. `NaN` = not yet linked |

#### Platform Activity Metrics (behavioral covariates)
| Column | Type | Description |
|--------|------|-------------|
| `5d_trading_avg_event_count` | float | Average daily trading events over past 5 days; `NaN` = no recent trading activity |
| `2d_non_trading_avg_event_count` | float | Average daily non-trading app events (browse, login) over past 2 days |
| `20d_trading_avg_event_count` | float | Average daily trading events over past 20 days (broader window) |
| `8d_non_trading_avg_event_count` | float | Average daily non-trading events over past 8 days |
| `1d_trading_avg_event_count` | float | Average trading events on the most recent day |
| `1d_non_trading_avg_event_count` | float | Average non-trading events on the most recent day |
| `num_received_email` | int | Total emails successfully delivered to this user (0–10); filter `> 0` to isolate active treatment participants |

---

### File 4: `email_events.csv` — Raw Post-Office Event Log
**Shape:** 9,553,330 rows × 6 columns · **Role:** Raw event stream from the email delivery system. One row per event (not per user), enabling time-series delivery monitoring.
**Download:** [⬇ email_events.csv (871 MB)](https://github.com/Ricky-S-Gong/email-ab-experiment-fintech/releases/download/v1.0.0/email_events.csv)

| Column | Type | Description |
|--------|------|-------------|
| `stitch_email_events.category` | str | JSON-like string encoding template and sender, e.g. `["ml_funding_faq","post-office"]` — template name must be parsed out |
| `stitch_email_events.dt_date` | str | Event date in `YYYY-MM-DD` format |
| `user_uuid` | str | User identifier — joins to `user_events.csv` |
| `event` | str | Event type: `processed`, `delivered`, `open`, `dropped`, `bounce`, `deferred`, `unsubscribe`, `spamreport` |
| `reason` | str | For bounce/drop events: ESP error string (e.g., `"550 5.1.1 The email account does not exist"`); `NaN` for normal events |
| `stitch_email_events.count_events` | int | Event count for this user-email-date combination (usually 1; may be >1 for repeat opens) |

**Event type reference:**

| Event | Count | Rate | Meaning |
|-------|------:|------|---------|
| `processed` | 4,208,119 | — | Accepted by ESP for delivery attempt |
| `delivered` | 4,191,336 | 100% (base) | Successfully reached recipient inbox |
| `open` | 870,651 | 20.8% | Recipient opened (pixel-tracked) |
| `dropped` | 241,805 | 5.8% | Suppressed before send (prior unsubscribe, invalid address) |
| `bounce` | 18,166 | 0.43% | Hard or soft delivery failure |
| `deferred` | 12,059 | 0.29% | Temporarily delayed; usually auto-retried |
| `unsubscribe` | 10,894 | 0.26% | Opted out — **counter-metric** |
| `spamreport` | 300 | 0.007% | Marked as spam — **critical counter-metric** (threshold: <0.10%) |

---

### File 5: `control_groups_rate.csv` — Control Group Baselines
**Shape:** 12 rows × 6 columns · **Role:** Pre-computed baseline funding and link rates for matched control groups (no email exposure). Used as the null hypothesis value $p_0$ in A/B z-tests.

| Column | Type | Description |
|--------|------|-------------|
| `group_name` | str | Segment label with `_D` suffix (one row per segment; both `_D` and `_W` treatment arms reference the same control row) |
| `num_users_in_control` | int | Total users in the control group for this segment |
| `num_funded_in_control` | int | Users who funded during the observation window — numerator for funding rate |
| `funding_rate_in_control` | float | `= num_funded / num_users` — the baseline funding rate used as $p_0$ in the proportions z-test |
| `num_link_in_control` | int | Users who linked a bank account during the window |
| `link_rate_in_control` | float | Baseline bank-link rate; `= 1.0` for segments where all users already had a linked account by design |

> **Join logic:** Treatment arms labeled `_W` are matched to their `_D` control counterpart via `group_name.str.replace('_W$', '_D')`. Control groups are maintained at the segment level, not the frequency level.

---

### How the Files Connect

```
sample_segment_groups.csv
  └─ group_id / group_name
        ├─► sample_uuid_email_order.csv  (group_id, group_name per user)
        │       └─ user_uuid
        │             ├─► user_events.csv        (one row per user)
        │             └─► email_events.csv       (many rows per user)
        └─► control_groups_rate.csv      (one row per segment, via group_name)
```

---

## Key Findings

### 1. Email Template Performance
- **`ml_funding_faq`** achieves the highest open rate (~31.5%), roughly **60% above the average** of other templates. Users' primary barrier to funding is informational — they want factual answers about the funding process, not motivational content.
- **`ml_diversified_portfolio`** performs worst — portfolio diversification messaging is premature for users who haven't yet funded their accounts.

### 2. User Segment Insights
- The highest-engagement segment is **`6M_App-link-20D_Act-5D_Act`** (recently approved, bank linked, actively trading). These users are deep in the funnel and respond strongly to investment-focused content.
- **Correlation analysis** confirms: bank account linkage and recent 6-month approval are the strongest predictors of email engagement.

### 3. A/B Test Results
- **7 out of 24 treatment arms** show statistically significant improvement in funding rate (p ≤ 0.05).
- **Daily delivery (_D) significantly outperforms twice-a-week (_W)**: 6 of the 7 significant groups are daily-delivery arms.
- Highly-engaged segments (`6M_App-link-20D_Act-5D_Act`) do **not** show significant email lift — they convert organically, suggesting **diminishing returns** of email intervention for already-engaged users.

### 4. Email Cadence Discovery
- **Day 1 (order_1) consistently outperforms Day 0 (order_0)** in open rate — contradicting the "Day 0 novelty" hypothesis. This suggests a **priming effect**: users exposed to the first email are more likely to open the second.
- **Recommendation**: Front-load 2–3 emails in the first week, then taper frequency.

### 5. Experiment Completion Status
- **Daily groups: Complete** — all 10 emails delivered, results are reliable.
- **Weekly groups: Incomplete** — order_9 (final email at Day_32) has not yet been delivered. Final readout for weekly groups requires waiting for Day_32 + 2-week observation window.

---

## Analysis Workflow

The main analysis is in `notebooks/Project-3-AB-testing.ipynb`, structured as:

1. **Data Loading & Sanity Check** — Validate dataset integrity, explore distributions
2. **Email Open Rate Analysis** — Heatmap of open rates by group × template
3. **Negative Effects Analysis** — Unsubscribe and spam report monitoring
4. **Correlation Analysis** — User traits vs. open rate feature importance
5. **Link & Funding Rates** — Treatment group outcome metrics
6. **A/B Testing** — One-sided proportions z-test, causal impact estimation
7. **Time Series Analysis** — Open rate dynamics across delivery sequence
8. **Funnel Analysis** — End-to-end conversion: email open → bank link → account fund
9. **Business Recommendations** — Actionable next steps for product and growth teams

---

## Tech Stack

| Library | Version | Usage |
|---------|---------|-------|
| `pandas` | 2.2.2 | Data manipulation, aggregation |
| `numpy` | 1.26.4 | Numerical operations |
| `matplotlib` | 3.9.2 | Visualization |
| `seaborn` | 0.14.2 | Statistical visualization, heatmaps |
| `statsmodels` | 0.14.2 | Proportions z-test for A/B significance |

---

## How to Run

```bash
# 1. Clone the repository
git clone https://github.com/Ricky-S-Gong/email-ab-experiment-fintech.git
cd email-ab-experiment-fintech

# 2. Download the large data files from GitHub Releases
gh release download v1.0.0 --dir data/
# Or with wget (no GitHub CLI needed):
# wget -P data/ https://github.com/Ricky-S-Gong/email-ab-experiment-fintech/releases/download/v1.0.0/user_events.csv
# wget -P data/ https://github.com/Ricky-S-Gong/email-ab-experiment-fintech/releases/download/v1.0.0/email_events.csv

# 3. Install dependencies
pip install pandas==2.2.2 numpy==1.26.4 matplotlib==3.9.2 seaborn==0.14.2 statsmodels==0.14.2

# 4. Launch Jupyter and open the main notebook
jupyter notebook notebooks/Project-3-AB-testing.ipynb
```

> **Minimum setup:** Only `user_events.csv` is needed for all core analyses. `email_events.csv` (871 MB) is only used for the event-volume breakdown chart in Section 3.

---

---

# 邮件投递 A/B 实验：面向金融科技用户的出资率优化

**一个工业级多臂 A/B 实验，分析个性化邮件营销对零售投资平台用户出资行为的因果影响。**

---

## 目录
- [项目概述](#项目概述)
- [业务背景与动机](#业务背景与动机)
- [实验设计](#实验设计)
- [仓库结构](#仓库结构)
- [核心发现](#核心发现)
- [分析流程](#分析流程)
- [技术栈](#技术栈)
- [运行方式](#运行方式)

---

## 项目概述

本项目模拟了金融科技公司（类似 Webull、Robinhood 或 Square）的真实增长实验。我们团队设计并执行了一个**为期 5 周的多臂邮件 A/B 实验**，目标群体为 48 万名已注册但尚未出资的用户，旨在回答核心业务问题：

> **个性化邮件营销能否在因果意义上提升已注册用户完成账户出资的概率？**

实验对 10 种机器学习生成的邮件模板、12 个行为用户分群进行了测试，分别按两种频率（每日和每周两次）投递，形成 **24 个实验组**，并匹配对应的对照组。

---

## 业务背景与动机

在零售投资平台中，**出资率**（已注册用户完成首次存款并开始交易的比例）是核心增长和营收指标。大量用户完成注册审批后迟迟不进行出资，常见原因包括：

- **知识壁垒**：不清楚出资流程
- **信任顾虑**：对绑定银行账户或资金转账存有疑虑
- **参与度衰减**：注册后失去动力，未完成出资动作

邮件营销是一种高 ROI、低成本的用户再召回手段。然而，并非所有邮件内容对不同分群都同样有效；定向不精准的营销活动反而可能增加退订率和垃圾邮件举报，损害长期投递能力。

**本实验提供了严格的统计学答案**——哪种邮件内容、用户分群和投递频率的组合能最大化出资转化的增量提升。

---

## 实验设计

| 参数 | 详情 |
|------|------|
| **受众** | 48 万名已注册未出资用户（截至 2020-11-27） |
| **实验周期** | 2020-11-30（第 0 天）至第 32 天 |
| **邮件模板** | 10 种 ML 生成模板，涵盖：信任与安全、知识差距、操作简便性 |
| **用户分群** | 12 个分群，由 4 个二元行为标签定义（6 个月内获批、已绑定银行、20 天内活跃、5 天内有交易） |
| **投递频率** | 每日（_D）: 第 0–9 天 · 每周两次（_W）: 周一/周五，第 0/4/7/11/14/18/21/25/28/32 天 |
| **实验组数** | 24 组（12 分群 × 2 频率） |
| **邮件排序** | 每位用户无放回随机排列（消除排序偏差） |
| **对照组** | 相同分群规则的用户，不接收任何邮件 |
| **统计检验** | 单侧比例 Z 检验（H₁：实验组出资率 > 对照组出资率，α = 0.05） |

---

## 仓库结构

```
email-ab-experiment-fintech/
│
├── notebooks/
│   └── Project-3-AB-testing.ipynb     # 完整分析笔记本（主要交付物）
│
├── data/
│   ├── sample_segment_groups.csv       # 12 个用户分群定义及用户数量
│   ├── sample_uuid_email_order.csv     # 48 万用户 × 随机邮件投递计划
│   ├── email_events.csv                # 原始邮件事件日志（950 万行）— 未上传（871 MB）
│   ├── user_events.csv                 # 用户级邮件状态 + 结果指标 — 未上传（81 MB）
│   ├── control_groups_rate.csv         # 对照组预计算出资/绑定率
│   └── README.md                       # 大文件的字段说明文档
│
├── assets/                             # 所有可视化输出图表（9 张）
├── reports/
│   ├── Report.tex                      # 执行摘要报告（LaTeX 源文件）
│   └── Report.pdf                      # 编译后的 PDF 报告（29 页）
│
└── README.md
```

---

## 核心发现

### 1. 邮件模板效果
- **`ml_funding_faq`** 取得最高开信率（约 31.5%），比其他模板平均水平高约 **60%**。用户出资的核心障碍是信息获取——他们想要关于出资流程的具体解答，而非激励性内容。
- **`ml_diversified_portfolio`** 效果最差——对于尚未出资的用户，投资组合多样化的话题时机不成熟。

### 2. 用户分群洞察
- 参与度最高的分群是 **`6M_App-link-20D_Act-5D_Act`**（近期获批、已绑定银行、近期活跃交易）。这类用户处于转化漏斗深处，对投资内容反应强烈。
- 相关性分析证实：**银行账户绑定**和**近 6 个月获批**是预测邮件参与度的最强信号。

### 3. A/B 检验结果
- **24 个实验组中有 7 组**在出资率上呈现统计显著提升（p ≤ 0.05）。
- **每日投递（_D）显著优于每周两次（_W）**：7 个显著组中有 6 个为每日投递组。
- 高度活跃的分群（如 `6M_App-link-20D_Act-5D_Act`）未显现显著邮件提升效果——这类用户会自然转化，表明对高参与度用户的邮件干预存在**边际收益递减**。

### 4. 邮件节奏发现
- **第 1 封邮件（order_1）的开信率持续高于第 0 封（order_0）**，与"首日新鲜感"假设相反。这暗示存在**预热效应**：接触第一封邮件的用户更可能打开第二封。
- **建议**：前期集中发送 2–3 封邮件，此后逐步降低频率。

### 5. 实验完成状态
- **每日组：已完成**——全部 10 封邮件已投递，结果可信。
- **每周组：未完成**——order_9（第 32 天最后一封邮件）尚未投递。每周组的最终结论需等待第 32 天 + 2 周观察窗口后方可得出。

---

## 分析流程

主要分析在 `notebooks/Project-3-AB-testing.ipynb` 中，结构如下：

1. **数据加载与质量检验** — 验证数据完整性，探索分布特征
2. **邮件开信率分析** — 分群 × 模板开信率热力图
3. **负面效果分析** — 退订和垃圾邮件举报监控
4. **相关性分析** — 用户特征与开信率的特征重要性分析
5. **绑定率与出资率** — 各实验组业务结果指标
6. **A/B 检验** — 单侧比例 Z 检验，因果影响量化
7. **时间序列分析** — 邮件投递序列中开信率的动态变化
8. **漏斗分析** — 端到端转化：开信 → 银行绑定 → 账户出资
9. **业务建议** — 面向产品和增长团队的可落地行动方案

---

## 技术栈

| 库 | 版本 | 用途 |
|----|------|------|
| `pandas` | 2.2.2 | 数据处理与聚合 |
| `numpy` | 1.26.4 | 数值计算 |
| `matplotlib` | 3.9.2 | 数据可视化 |
| `seaborn` | 0.14.2 | 统计可视化、热力图 |
| `statsmodels` | 0.14.2 | 比例 Z 检验（A/B 显著性） |

---

## 运行方式

```bash
# 1. 克隆仓库
git clone https://github.com/Ricky-S-Gong/email-ab-experiment-fintech.git
cd email-ab-experiment-fintech

# 2. 从 GitHub Releases 下载大型数据文件
gh release download v1.0.0 --dir data/
# 或用 wget（无需 GitHub CLI）：
# wget -P data/ https://github.com/Ricky-S-Gong/email-ab-experiment-fintech/releases/download/v1.0.0/user_events.csv
# wget -P data/ https://github.com/Ricky-S-Gong/email-ab-experiment-fintech/releases/download/v1.0.0/email_events.csv

# 3. 安装依赖
pip install pandas==2.2.2 numpy==1.26.4 matplotlib==3.9.2 seaborn==0.14.2 statsmodels==0.14.2

# 4. 启动 Jupyter 并打开主分析笔记本
jupyter notebook notebooks/Project-3-AB-testing.ipynb
```

> **最小配置：** 核心分析仅需 `user_events.csv`。`email_events.csv`（871 MB）仅用于第 3 节的邮件事件量图表。
