# Employee Engagement Analysis – US Federal Workforce (FEVS 2024)

**What drives employee engagement and plans to leave, and where should leaders act first?**

I analysed 674,207 responses to the US Office of Personnel Management's Federal Employee Viewpoint Survey (FEVS) 2024. I rebuilt OPM's official engagement index, designed my own well-being index, found the workplace factors most linked to engagement, and built a 5-page Power BI dashboard for agency leaders.

**Tools:** Python (pandas, NumPy, matplotlib) · Jupyter · Power BI (Power Query, DAX)

---

## Key findings

1. **Senior leadership is the weak spot.** Engagement is 73.0 across the federal workforce, but employees rate senior leaders (62.7) almost 19 points lower than their own supervisors (81.3).
2. **Management communication and involvement matter most and score lowest.** Of the 16 high-impact, low-scoring factors, 9 are about communication, recognition and involvement. "Management involves employees in decisions" is only 47.8% positive.
3. **Growth keeps people.** 68.8% of employees who don't feel they can improve their skills are considering leaving, against 28.4% of those who do. That 40-point gap is the largest of any factor.
4. **Workload is uneven.** Well-being ranges from 91.9 (Pension Benefit Guaranty Corporation) to 59.0 (Social Security Administration). Within SSA, the Operations unit, with 69% of its respondents, scores just 54.3.

## Recommendations

1. **Involve employees in decisions before changes are made.** This is the lowest-scoring of the top drivers.
2. **Make skills growth visible** through development plans and internal moves. It has the biggest link to plans to leave.
3. **Fix workload where it is worst first,** starting with SSA Operations, where 27,637 respondents report the lowest well-being.


## Dashboard

The report is built for an agency leader: headline findings, a government benchmark next to every number, and detail on demand.

| Page | Question it answers |
|---|---|
| 1. Summary | How engaged is the federal workforce, and what should leaders do? |
| 2. Drivers | Which workplace factors matter most, and which score lowest? (priority matrix of 52 questions) |
| 3. Agency scorecard | How does one agency compare with government, and where are its biggest gaps? |
| 4. Work units | Where inside the agency is the problem? |
| 5. About the data | How were the numbers built, and can they be trusted? |


The full report is also available as a [PDF](outputs/fevs-engagement-analysis-dashboard.pdf) and as the [Power BI file](power%20bi/fevs-engagement-analysis.pbix).

---

## Data

- **Source:** OPM FEVS 2024 Public Release Data File ([opm.gov/fevs](https://www.opm.gov/fevs/)), 674,207 respondents, 36 agencies, 163 work units, over 80 survey questions
- **Weights:** every result uses OPM's survey weight (`POSTWT`), so numbers represent the federal workforce, not just the people who answered
- **Privacy:** I left out race, ethnicity, sex and disability. They aren't needed for the business question.

The raw data, the cleaned files and the Power BI model tables are too large for GitHub. They are stored on Google Drive:

**📁 [Data and model files (Google Drive)](https://drive.google.com/drive/folders/12dSHPyDHoW7vAfsaD4vgfqEnFeyGLpI-)**

---

## Method

The analysis runs in four notebooks, each saving its output for the next.

| Notebook | What it does |
|---|---|
| [`01_data_cleaning`](notebook/01_data_cleaning.ipynb) | Loads the raw file and codebook, turns 891,565 "Don't know" answers into blanks, checks every value is in range, adds agency and work-unit names, removes sensitive demographics |
| [`02_build_indices`](notebook/02_build_indices.ipynb) | Rebuilds OPM's indices (weighted % positive, OPM method), designs and tests my Well-being & Workload index, scores every agency and work unit |
| [`03_driver_analysis`](notebook/03_driver_analysis.ipynb) | Weighted correlation of 52 workplace factors with each person's engagement score, a standardised regression on the top 12, the "leave gap" for each factor, and a priority matrix |
| [`04_build_model`](notebook/04_build_model.ipynb) | Builds the star-schema tables for Power BI and checks them against notebook 02 |

### Checking my numbers against OPM

| Index | My result | OPM published |
|---|---|---|
| Employee Engagement Index | 73.0 | 73 |
| Employee Experience Index | 74.4 | 74 |

### My own index: Well-being & Workload

The index uses four questions on workload, work-life balance and leadership support (Q5, Q34, Q49, Q63). They hang together well as one measure (Cronbach's α = 0.78). Across agencies, well-being moves closely with engagement (r = 0.93).

### Driver analysis

- **Importance** = weighted correlation between each question and a person's engagement score
- **Leave gap** = % considering leaving among negative answerers minus % among positive answerers
- **Priority matrix** = importance vs % positive, split at the medians (r 0.65, 71% positive), giving four groups: **Fix first**, Protect, Monitor, Maintain
- A regression on the top 12 drivers explains most of the variation in engagement (R² = 0.89), but the drivers overlap strongly, so I rank them by theme rather than claiming one single top driver

---

## Power BI data model

A person × question table would have about 49 million rows. % positive only needs two sums (the weight of people who answered, and the weight of people who answered 4–5), and sums can be added up in any combination. So I pre-aggregated to work unit × respondent group × question. That gives **540,767 rows with exactly the same results for every filter.**

| Table | Type | One row per |
|---|---|---|
| `fact_responses` | Fact | work unit × respondent group × question |
| `fact_respondents` | Fact | work unit × respondent group |
| `dim_workunit` | Dimension | work unit (with its agency) |
| `dim_group` | Dimension | supervisory status × tenure × leaving plan |
| `dim_question` | Dimension | question (text, index, theme, driver results) |

- 1-to-many relationships, single-direction filtering
- 50+ DAX measures, grouped into folders: core, indices, benchmarks ("vs government" and ranks), formatting (colour rules, dynamic titles) and drivers
- A disconnected agency table lets the strip chart show all 36 agencies while highlighting the selected one
- Every DAX result was checked against the Python results (government-wide and for all 36 agencies)

---

## Repository structure

```
├── notebook/          Python notebooks 01–04
├── power bi/          Power BI report (.pbix), theme and Power Query scripts
├── outputs/           Dashboard exported to PDF
├── images/            Dashboard screenshots for this README
└── docs/              OPM FEVS 2024 technical report
```

## How to reproduce

1. Download the data from the [Google Drive folder](https://drive.google.com/drive/folders/12dSHPyDHoW7vAfsaD4vgfqEnFeyGLpI-) and place it in the same structure: `data/raw/2024/`, `data/clean/` and `model/`.
2. Run the notebooks in order (01 → 04) from the `notebook/` folder. They use relative paths such as `../data/clean`.
3. Open `power bi/fevs-engagement-analysis.pbix`. Go to **Transform data → Manage parameters** and set `ModelFolder` to your `model/` folder, then click **Refresh**.

Requires Python 3 with pandas and NumPy (openpyxl to read the codebook), plus Power BI Desktop.

---

## Limits

- **One year only (2024),** so there are no trends yet. Adding 2023 data is the next step.
- **Driver links are correlations, not causes.** All questions come from the same survey, which can inflate correlations.
- **"Considering leaving" may include retirement.** The public file has no separate retirement option.
- **Work units with fewer than 30 respondents are hidden** in the dashboard.
