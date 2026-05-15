## Documentation — What this code does

This document summarizes the purpose, inputs, processing steps, and outputs for the analysis code in the notebook. The code is organized by research questions (Q1–Q5). Each section below explains the data operations, weighting, and visualizations produced.

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

### General outputs and reproducibility
- **CSV exports**: Each major intermediate table is saved to `/content/` (or project `data/` path) for reproducibility and downstream use.  
- **Visualizations**: Interactive Plotly charts and static Matplotlib/Seaborn figures are displayed inline and can be exported as PNG/HTML from the notebook.  
- **Weighting and group size rules**: Weighted statistics use `PWGTP`; group thresholds (e.g., `len(g) > 10` or `> 20`) are applied to avoid unstable estimates.

---

### Implementation notes and suggestions
- Ensure `wtd_median()` is defined and tested for edge cases (small groups, ties).  
- Keep raw wage values for regressions; only clip for plotting.  
- Document any occupation or field code mappings (e.g., `fod_map`) in `docs/mappings.md`.  
- Save figures and CSVs to a consistent `data/` or `outputs/` folder for easier sharing.

---
