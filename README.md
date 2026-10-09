# MSBA_CAPSTONE_2

PBS Utah donor-prediction capstone (IS 6813, Fall 2026).

## Business problem
Congress eliminated federal funding for public broadcasting in 2025, leaving PBS Utah with an estimated recurring shortfall of about \$2 million per year. Major donors, who give **\$1,200 or more annually**, generate an outsized share of donor revenue. PBS Utah has no reliable, data-driven way to identify which current non-major donors are most likely to become major donors, or at what giving level.

**Benefit.** A ranked prospect list lets development staff spend limited stewardship time on donors with the strongest evidence of major-giving potential, instead of searching the full donor file. The project will also show which giving, engagement, and viewing behaviors are most associated with moving into major giving.

**Success metrics.** The model is judged by backtesting on held-out historical periods, not by revenue.
- **Primary target:** recall of future \$1,200+ donors near the top of the ranked list, over a 3–5 year window, with a 12-month result reported alongside.
- **Comparisons:** random selection and a recency-frequency-value baseline.
- **Giving tiers:** predicted tiers are checked against the tier each donor actually reached.
- **Other measures:** precision, lift, and calibration, including at a list size staff can realistically work.

**Analytics approach.**
- **Model:** a rare-outcome classifier at the individual donor level. It estimates each non-major donor's probability of reaching \$1,200+ within 12 months (immediate priority) and within 3–5 years (long-term nurture), plus the most likely giving tier.
- **Validation:** time-based, using only information available at each prediction date.
- **Risks addressed:** data leakage, incomplete Passport coverage, and the FY2026 donation influx.

**Scope.**
- **Delivered:**
  - a documented major-donor definition and eligible population
  - a cleaned donor-level dataset
  - a validated model benchmarked against baselines
  - a working-queue dashboard with *Immediate priority* and *Long-term nurture* segments, plus a backtest view
  - reproducible code and documentation
- **Out of scope:** contacting donors, assigning prospects to fundraisers, guaranteeing gifts, measuring actual revenue, and automated decisions without staff review.

**Team:** Erin Bednarik, Benjamin Hogan, Samantha Huang, and Kate Klinger, with support from Jason Hoggan and Natalie Benoy at PBS Utah.
**Milestones:** initial results November 11, 2026; final project December 2, 2026.

## What the EDA found about the problem statement
Section 13 of the report tests the statement against the data:
- **Major-donor share:** major donors are **0.25–0.38% of donors and 5–11% of revenue**, not the 1% and 19% in the statement.
- **Eligible population:** active **and lapsed** donors (ever gave, under \$1,200 in the snapshot year). Lapsed donors are 75% of eligible donor-years but 8% of converters (24 of 293), mostly returning former majors, so results are reported by segment.
- **The 25% target:** recall in the top 10% is already exceeded by simple rules. Ranking by prior-year giving reaches 84% (12-month) and 73% (3-year), and the RFV baseline reaches 83% and 74%. The target should be to beat the prior-year-giving baseline, measured at a list of about 1,000 and for never-major donors.
- **Tiers:** 95% of converters reach \$1,200–4,999, so three tiers (\$1,200 / \$2,000 / \$5,000+) are more realistic than five.
- **Time windows:** the data support testing 12-month and 3-year windows. The 5-year window can only be a projection.
- **Inputs:** University of Utah engagement and capacity scores are not in the data, and the UofU dollar fields leak future PBS giving.
- **Passport:** kept as a candidate input with a "no record" level. Its raw link to conversion is mostly a status effect and fades once giving is held fixed, so it should be tested with and without in the model.

## Contents
- `notebooks/EDA_PBS_Utah_Donors.qmd`: exploratory data analysis (Quarto source, Python)
- `notebooks/EDA_PBS_Utah_Donors.html`: rendered, self-contained report with all outputs
- `docs/data_dictionary_combined.md`: every variable in all nine tables, with definitions, profile, and EDA status (use / drop / leaks); CSV copy alongside

## Data
The course data (synthetic) lives at https://github.com/jefftwebb/donor_prediction_capstone_project and is not committed here. Large tables use Git LFS:

```bash
git clone https://github.com/jefftwebb/donor_prediction_capstone_project.git
cd donor_prediction_capstone_project && git lfs pull
```

Copy or symlink its `tables/` folder to `data/tables/` in this repo, or edit `DATA_DIR` in the report.

## Rendering
Requires [Quarto](https://quarto.org) and Python with `pandas numpy matplotlib seaborn scipy jupyter` (about 10 GB of RAM):

```bash
cd notebooks
quarto render EDA_PBS_Utah_Donors.qmd
```
