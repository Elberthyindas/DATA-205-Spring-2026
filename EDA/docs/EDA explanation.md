## Documentation — What this code does

This document summarizes the purpose, inputs, processing steps, and outputs for the analysis code in the notebook. The code is organized by research questions (Q1–Q10). Each section below explains the data operations, weighting, and visualizations produced.

---

### Common setup and conventions
- **Input datasets**: `bls_md` (BLS OEWS Maryland occupational table) and `pums2` / `pums` (ACS PUMS person‑level data).  
- **Weighting**: All ACS aggregates use the person weight **`PWGTP`**. Weighted medians are computed with a helper `wtd_median()`; weighted means use `np.average(..., weights=PWGTP)`.  
- **Filtering**: Most analyses restrict to positive wages (`WAGP > 0`) and apply simple sample filters (e.g., minimum group sizes).  
- **Outputs**: Several CSV files are written for downstream use (see each section). Interactive and static figures are shown with Plotly and Matplotlib/Seaborn.

---

### Q1 — Occupational structure (treemap, location quotients, PUMS wage by occupation)
**Purpose**
- Describe Maryland’s occupational composition and compare occupational concentration to the U.S.

**Key steps**
- Filter `bls_md` to major occupation groups, drop the “All Occupations” row, and remove rows missing `TOT_EMP` or `A_MEDIAN`. Save as `occ_major_for_treemap.csv`.
- **Treemap**: `px.treemap()` with `values='TOT_EMP'` (size = workers) and `color='A_MEDIAN'` (color = median wage).
- **Location quotient bar**: Filter major groups with `LOC_QUOTIENT`, plot horizontal bar chart sorted by `LOC_QUOTIENT`, and add a vertical line at `x=1` (US baseline).
- From ACS (`pums2`), compute **weighted median** and **weighted mean** wages and total workers by `OCC_TITLE` using `PWGTP`. Save as `occ_wage_pums.csv`.
- **Bar chart**: Plot weighted median wage by occupation using Plotly.

**Files produced**
- `/content/occ_major_for_treemap.csv`  
- `/content/occ_wage_pums.csv`

---

### Q2 — Gender wage gap by occupation (dumbbell chart)
**Purpose**
- Measure and visualize the male–female median wage gap within occupations.

**Key steps**
- Group `pums2` by `OCC_TITLE` and `sex_label`, compute weighted medians.
- Pivot to wide format so each occupation has `Male` and `Female` median columns; compute `gap = Male − Female`.
- Save results to `sex_wage_gap_by_occupation.csv`.
- **Dumbbell chart**: Custom Plotly figure that draws a horizontal line between female and male medians for each occupation and plots markers for each sex.

**Files produced**
- `/content/sex_wage_gap_by_occupation.csv`

**Notes**
- Occupations with missing medians for either sex are dropped before plotting.

---

### Q3 — Education × Occupation wage premium (heatmap) and top fields of degree
**Purpose**
- Show how mean wages vary across education tiers and occupations; identify top-paying bachelor’s fields.

**Key steps**
- Compute weighted mean wage by `edu_tier` × `OCC_TITLE` (only if group size > 10), pivot to create a matrix for the heatmap, and save `edu_pivot.csv`.
- Plot a Seaborn heatmap of mean wages (in thousands) with annotations.
- For bachelor’s holders (`SCHL == 21`), map `FOD1P` codes to readable `fod_name` using `fod_map`, compute weighted mean wages by field (require > 20 observations), save `fod_wage.csv`, and plot top fields with Plotly.

**Files produced**
- `/content/edu_pivot.csv`  
- `/content/ba_df.csv`  
- `/content/fod_wage.csv`

**Notes**
- The heatmap uses `edu_order` to control education tier ordering.

---

### Q4 — STEM vs Non‑STEM wage premium (violins, race interactions, regression)
**Purpose**
- Compare wage distributions for STEM vs non‑STEM degree holders by sex and race; estimate interaction effects.

**Key steps**
- Build `stem_df` filtering for reasonable wages and non‑missing STEM indicator; save `stem_df.csv`.
- Create side‑by‑side violin plots (Male / Female) comparing STEM and Non‑STEM wage distributions (wages clipped for visualization).
- Compute mean wages by `race_eth` × `stem_label` and save `stem_race.csv`; plot grouped bar chart.
- **Interaction regression**: Fit a weighted WLS model on `log_wage` with `stem * C(sex_label)` plus controls `C(race_eth)`, `C(edu_tier)`, and `AGEP`, using `PWGTP` weights and HC3 robust SEs. Print coefficient table.

**Files produced**
- `/content/stem_df.csv`  
- `/content/stem_race.csv`

**Notes**
- Visualizations clip wages (e.g., at \$300k) for readability; regressions use `log_wage` and robust inference.

---

### Q5 — Class of Worker (government vs private vs self‑employed)
**Purpose**
- Compare mean wages and racial composition across sectors (class of worker).

**Key steps**
- Filter `pums` for positive wages and non‑missing `COW`, save `cow_df.csv`.
- Compute weighted mean wage by `cow_label` × `race_eth` and save `cow_race.csv`; plot grouped bar chart of mean wages.
- Compute racial composition shares within each `cow_label` (weighted sums of `PWGTP`), save `cow_comp.csv`, and plot stacked bar chart showing shares by race/ethnicity.

**Files produced**
- `/content/cow_df.csv`  
- `/content/cow_race.csv`  
- `/content/cow_comp.csv`

---


### Q6 — Racial Wage Gap & Blinder–Oaxaca Decomposition

**Purpose**
- Describe unconditional and conditional racial wage differences and decompose the White–Black log‑wage gap into explained (endowments) and unexplained (coefficients/residual) components.

**Inputs**
- `pums` / `pums2` (ACS person‑level data with `WAGP`, `log_wage`, `PWGTP`, `race_eth`, `full_time`, `edu_tier`, `AGEP`, `soc2`).

**Key steps**
- **Unconditional gap table**: compute weighted mean, weighted median (via `wtd_median()`), and total weighted workers by `race_eth`. Save as `race_wages.csv`. Compute each group's gap relative to White NH.
- **Distribution plot**: KDE of annual wages for **full‑time** workers (clip at \$250k for plotting) to visualize distributional differences by race.
- **Subsamples for Oaxaca**: create and save `white_subdataset.csv` and `black_subdataset.csv` containing observations with positive wages and non‑missing `log_wage`.
- **Weighted regressions**: estimate WLS models separately for White NH and Black NH:
  - Formula: `log_wage ~ AGEP + full_time + C(edu_tier) + C(soc2)`
  - Weights: `PWGTP`; robust HC3 standard errors for inference.
- **Decomposition calculation**:
  - Compute weighted group means of model regressors (`Xbar_w`, `Xbar_b`) using the model's aligned weights.
  - Use White coefficients as the reference (`b_w`) to compute the **explained** component: \((\bar{X}_w - \bar{X}_b) \cdot \beta_w\).
  - **Unexplained** = raw weighted mean log‑wage gap − explained.
  - Convert log differences to percent changes for interpretation where helpful.
- **Visualization**: horizontal bar chart showing explained vs unexplained contributions to the log‑wage gap.

**Outputs**
- `/content/race_wages.csv`  
- `/content/white_subdataset.csv`  
- `/content/black_subdataset.csv`  
- Decomposition bar chart displayed inline.

**Notes**
- The decomposition uses the White coefficients as the reference; alternative weighting or coefficient choices change the split. Document this choice in `docs/methods.md`.
- Ensure `wtd_median()` and the model design matrices align with the rows used in the regression (statsmodels may drop rows during formula expansion).

---

### Q7 — Nativity & Occupational Sorting

**Purpose**
- Compare occupational distributions of native‑born and foreign‑born workers and estimate conditional nativity wage differences.

**Inputs**
- `pums2` with `nativity`, `OCC_TITLE`, `WAGP`, `PWGTP`, `edu_tier`, `AGEP`, `full_time`, `soc2`.

**Key steps**
- Save `nat_df.csv` with observations having non‑missing `nativity` and `WAGP`.
- **Occupation shares**: aggregate weighted counts by `nativity` × `OCC_TITLE`, compute within‑group shares, and save `nat_occ.csv`.
- **Bar chart**: grouped horizontal bar chart of occupation shares by nativity.
- **Regression**: WLS model estimating `log_wage` on `C(nativity)` (reference = Native‑born), `C(edu_tier)`, `AGEP`, `full_time`, and `C(soc2)` with `PWGTP` weights and HC3 SEs. Extract and report the foreign‑born coefficient and percent effect.

**Outputs**
- `/content/nat_df.csv`  
- `/content/nat_occ.csv`  
- Occupation distribution bar chart and regression coefficient printed.

**Notes**
- The occupation share plot uses weighted sums; small‑cell occupations may be noisy—consider grouping small occupations or showing top N occupations.

---

### Q8 — Disability & Commute Patterns

**Purpose**
- Measure employment rates by disability status and examine commute patterns across wage quartiles.

**Inputs**
- `pums_raw` (for working‑age population), `pums` (for commute variables `JWMNP`, `commute_mode`, `WAGP`).

**Key steps**
- **Working‑age sample**: filter ages 18–64, create `disabled` and `employed` indicators, save `working_age.csv`.
- **Employment rate**: compute weighted employment rate by disability status and save `emp_rate.csv`.
- **Commute vs wage quartile**:
  - Create wage quartiles with `pd.qcut` (handle ties with `duplicates='drop'`).
  - Save `commute_df.csv` for observations with non‑missing commute time.
  - Boxplot of commute time (`JWMNP`) by wage quartile.
  - Compute commute mode shares by wage quartile, save `mode_q.csv`, and plot stacked/grouped bar chart.

**Outputs**
- `/content/working_age.csv`  
- `/content/emp_rate.csv`  
- `/content/commute_df.csv`  
- `/content/mode_q.csv`  
- Boxplots and bar charts displayed inline.

**Notes**
- Interpret employment rates with weights; document how `DIS` is coded and any recoding performed.
- Commute time distributions can be skewed—use medians or robust summaries where appropriate.

---

### Q9 & Q10 — Hours Worked, Underemployment, and Part‑time Rates

**Purpose**
- Describe part‑time employment patterns by education and race, and explore the relationship between annual hours and wages.

**Inputs**
- `pums` with `part_time` indicator, `edu_tier`, `race_eth`, `PWGTP`, `annual_hours`, `WAGP`.

**Key steps**
- **Part‑time rates**:
  - Compute weighted part‑time rate by `edu_tier` (ordered by `edu_order`) and by `race_eth`. Save `pt_edu.csv` and `pt_race.csv`.
  - Plot horizontal bar charts for part‑time rates by education and race.
- **Annual hours vs wage scatter**:
  - Sample 5,000 observations (seeded for reproducibility) with positive wages and annual hours; save `scatter_sample.csv`.
  - Scatter plot of `annual_hours` vs `WAGP` colored by `edu_tier` with an OLS trendline.

**Outputs**
- `/content/pt_edu.csv`  
- `/content/pt_race.csv`  
- `/content/scatter_sample.csv`  
- Bar charts and scatter plot displayed inline.

**Notes**
- Use `PWGTP` for weighted part‑time rates; the scatter uses an unweighted sample for visualization but regressions (if added) should be weighted.
- Document sampling seed and clipping rules used for plotting.

---

### General implementation and reproducibility notes
- **CSV exports**: Each major intermediate table is saved to `/content/` (or project `data/` path) for reproducibility and downstream use.  
- **Visualizations**: Interactive Plotly charts and static Matplotlib/Seaborn figures are displayed inline and can be exported as PNG/HTML from the notebook.  
- **Weighting and group size rules**: Weighted statistics use `PWGTP`; group thresholds (e.g., `len(g) > 10` or `> 20`) are applied to avoid unstable estimates.
- **CSV exports**: Each intermediate table is saved to `/content/` (or project `data/`) to allow reloading without rerunning heavy computations.  
- **Weighting**: All descriptive statistics use `PWGTP`. Regression models use `PWGTP` as WLS weights and HC3 robust SEs.  
- **Group thresholds**: The code applies minimum group sizes (e.g., `len(g) > 10` or `> 20`) before reporting group means to avoid unstable estimates—document these thresholds in `docs/analysis_notes.md`.  
- **Mappings**: Keep `fod_map`, occupation mappings, and any crosswalks in `docs/mappings.md`.  
- **Visualization**: Clip extreme wage values for plotting only; do not clip values used in regressions. Save static exports of key figures to a consistent `figures/` folder for the report.  
- **Reproducibility**: Seed random sampling (`random_state=42`) and record package versions in `requirements.txt`.

### Implementation notes and suggestions
- Ensure `wtd_median()` is defined and tested for edge cases (small groups, ties).  
- Keep raw wage values for regressions; only clip for plotting.  
- Document any occupation or field code mappings (e.g., `fod_map`) in `docs/mappings.md`.  
- Save figures and CSVs to a consistent `data/` or `outputs/` folder for easier sharing.

---
