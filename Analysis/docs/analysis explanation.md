#### Purpose
- **Q2**: Estimate the **conditional gender wage gap** controlling for education, age, full‑time status, and occupation fixed effects.  
- **Q11**: Estimate a **full wage determinants model** that includes sex, race, education, age, full‑time status, STEM, class of worker, nativity, disability, and occupation fixed effects.  
- **Q12**: Produce a concise **summary table** of key conditional effects (coefficients, percent wage effects, p‑values, significance).

#### Inputs
- **Primary dataset**: `pums2` (ACS PUMS cleaned person‑level table).  
- **Key variables used**:
  - `WAGP` (annual wage), `log_wage` (natural log of wage)  
  - `sex_label`, `race_eth`, `edu_tier`, `AGEP`, `full_time`  
  - `stem`, `cow_label` (class of worker), `nativity`, `disabled`  
  - `soc2` (2‑digit occupation fixed effects)  
  - `PWGTP` (person weight used as WLS weight)

#### Processing steps and models

- **Prepare regression dataset**
  - Select relevant columns and drop rows with missing values:
    ```python
    reg_df = pums2[['log_wage','sex_label','edu_tier','AGEP','full_time','soc2','PWGTP']].dropna()
    full_df = pums2[['log_wage','sex_label','race_eth','edu_tier','AGEP','full_time','stem','cow_label','nativity','disabled','soc2','PWGTP']].dropna()
    ```
  - Export cleaned regression inputs for reproducibility:
    - `/content/reg_df.csv`
    - `/content/full_df.csv`

- **Q2 Conditional gender gap model**
  - Estimate a weighted least squares model:
    ```
    log_wage ~ C(sex_label, Treatment("Male")) + C(edu_tier) + AGEP + full_time + C(soc2)
    ```
  - Use `PWGTP` as weights and HC3 robust standard errors.
  - Extract the female coefficient `C(sex_label)[T.Female]`, its p‑value, and report the percent effect as `(exp(coef)-1)*100`.

- **Q11 Full model**
  - Estimate the full WLS model:
    ```
    log_wage ~ C(sex_label, Treatment("Male"))
               + C(race_eth, Treatment("White NH"))
               + C(edu_tier, Treatment("High School/GED"))
               + AGEP + full_time + stem
               + C(cow_label, Treatment("Private for-profit"))
               + C(nativity, Treatment("Native-born"))
               + disabled
               + C(soc2)
    ```
  - Use `PWGTP` weights and HC3 robust SEs.
  - Save model results and compute confidence intervals.

- **Forest plot of coefficients**
  - Build a coefficient table excluding occupation fixed effects (`C(soc2)`), including:
    - **variable**, **coef**, **ci_lo**, **ci_hi**, **pval**
  - Color code bars:
    - Blue for positive & significant, red for negative & significant, gray for not significant.
  - Plot horizontal bar chart with error bars and a vertical zero line.

- **Q12 Summary table**
  - Define a list of key comparisons and variables (female vs male, Black vs White NH, Hispanic vs White NH, Asian vs White NH, Bachelor's vs HS/GED, Graduate vs HS/GED, STEM premium, Federal govt vs Private, Foreign‑born vs Native, Disability penalty).
  - For each variable present in the model, extract:
    - **Coefficient** (rounded), **Wage Effect (%)** = `(exp(coef)-1)*100`, **p‑value**, and a significance marker (`***`, `**`, `*`, `ns`).
  - Print the summary table and report model R² and a short conclusion.

#### Outputs
- CSVs:
  - `/content/reg_df.csv` — regression input for gender model  
  - `/content/full_df.csv` — regression input for full model
- Visuals:
  - Forest plot of key coefficients (saved/displayed inline)
- Console outputs:
  - Conditional gender gap estimate with percent interpretation and p‑value  
  - Model R² and adjusted R² for the full model  
  - Summary table of conditional wage effects with significance markers
- Final textual conclusion:
  - A short statement that **Maryland wage inequality is both structural (occupational sorting)** and **residual (within‑job gaps by race, gender, nativity, and disability persist)**.

#### Interpretation guidance
- **Coefficient interpretation**: Coefficients are on the log wage scale. Convert to percent change with `(exp(coef)-1)*100` for intuitive interpretation.  
- **Occupation fixed effects**: Including `C(soc2)` controls for the job held and isolates within‑occupation differences.  
- **Weights and inference**: Use `PWGTP` for population representativeness and HC3 robust SEs for conservative inference.  
- **Significance**: Use p‑values and confidence intervals to assess robustness; color coding in the forest plot highlights direction and significance.

#### Reproducibility notes
- Keep the exported CSVs with timestamps or in a `data/` folder to allow rerunning models without reprocessing raw PUMS.  
- Document any recoding (e.g., reference levels for categorical variables) in `docs/methods.md`.  
- Do not clip or alter wage values used in regressions; clipping is only for visualization.  
- Record the random seed and package versions in `requirements.txt` for full reproducibility.

