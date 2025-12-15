**2025 Brown University Shooting🕯️ almost fit with our research conclusion.**
=============================================================
README — Supplemental Materials (Reproducibility Package)
=============================================================

This folder contains the code and data needed to reproduce our project analyses.
The main entry point is the R Markdown file `supplemental_code.Rmd`.
Running (knitting) this file will reproduce all key numerical outputs used in the paper.

------------------------------------------------------------
1) Files in this folder (exact names)
------------------------------------------------------------
1. supplemental_code.Rmd
   - The full reproducibility code for all three analyses.
   - Organized into three sections:
       (1) Weekday effects on total casualties (Bayesian Gibbs sampling)
       (2) Student vs. non-student shooter effects on fatalities (bootstrap + permutation)
       (3) Victim age group vs. number of victims (negative binomial regression + permutation)

2. school_shooting_db_20200316.csv
   - The dataset used by all code in supplemental_code.Rmd.
   - Must be in the same folder as the Rmd file.

3. README.txt
   - This document. Explains how to run and interpret the code output.

Note:
- Other files like .Rhistory are not required for reproduction.

------------------------------------------------------------
2) Software Requirements
------------------------------------------------------------
Recommended: RStudio + a recent R version.

Packages used in the code:
- readr
- dplyr
- MASS
- ggplot2
- rmarkdown, knitr (for knitting)

If any are missing, install with:
install.packages(c("readr","dplyr","MASS","ggplot2","rmarkdown","knitr"))

------------------------------------------------------------
3) How to Reproduce Results (Step-by-Step)
------------------------------------------------------------
Step 1 — Put all files in one folder
- Ensure these files are in the same directory:
  supplemental_code.Rmd
  school_shooting_db_20200316.csv
  README.txt

Step 2 — Open in RStudio
- Open `supplemental_code.Rmd` in RStudio.
- Set working directory to the file’s folder:
  Session > Set Working Directory > To Source File Location

Step 3 — Knit to reproduce
- Click “Knit” (HTML is recommended).
- The knitted output prints:
  - Posterior credible intervals and posterior probabilities (weekday analysis)
  - Observed mean difference, bootstrap 95% CI, and permutation p-values (shooter analysis)
  - Regression summary, IRRs with confidence intervals, AIC comparison, and permutation p-value (age-group analysis)

If PDF knitting fails due to missing LaTeX, knit to HTML instead.

------------------------------------------------------------
4) Data Cleaning and Variable Construction (What the code does)
------------------------------------------------------------
General rule:
- Each analysis filters out rows with missing values for the variables required by that analysis,
  because resampling, randomization tests, and model fitting require defined outcomes and labels.

(1) Weekday analysis: total casualties by weekday
- Outcome:
    T = `Total Injured/Killed Victims` (converted to numeric)
- Weekday:
    `Day of week (formula)` converted to a factor with levels Mon..Sun
- Filtering:
    keep rows with non-missing T and weekday
- Transformation:
    y = log1p(T) = log(1 + T), used to reduce the influence of extreme outliers

(2) Student vs. non-student shooter analysis: fatalities
- Shooter type (two groups) is derived from:
    `Shooter's Affiliation with School`
  Student shooter categories include:
    Student, Former Student, Visiting Student, Students from Rival School
  All other non-missing affiliations are classified as non-student.
- Outcome:
    killed = `Killed (includes shooter)`
- Filtering:
    keep rows with non-missing killed and non-missing shooter_type

(3) Victim age-group analysis: number of victims per event
- Outcome:
    victim_count = `Total.Injured.Killed.Victims`
- Victim age:
    victim_age = `Victim.s.age.s.`
- Age groups:
    Elementary: 0–11
    Middle: 12–14
    High: 15–18
    Adult: 19+
- Missing-age indicator is created:
    age_missing = 1 if age is missing else 0
- Regression subset:
    the regression model shown uses observed age groups only
    (rows with missing age_group are excluded for that model fit)

------------------------------------------------------------
5) Methods and Outputs (How to interpret key printed results)
------------------------------------------------------------

Analysis 1 — Weekday Effects (Bayesian Gibbs sampling)
- The code fits a hierarchical model on y = log1p(T) and runs Gibbs sampling.
- Key printed outputs:
  (a) Posterior 2.5% / 50% / 97.5% quantiles for each weekday mean (log1p scale)
  (b) Posterior probability example: P(mu_Fri > mu_Tue | data)
  (c) Posterior median tau^2 (between-weekday variability)
- Interpretation:
  Overlapping weekday credible intervals and small tau^2 suggest weak weekday differences.

Analysis 2 — Student vs. Non-Student Shooter Fatalities (Bootstrap + Permutation)
- Statistic:
  Delta_obs = mean(killed_student) − mean(killed_nonstudent)
- Bootstrap:
  prints Delta_obs and a bootstrap 95% percentile confidence interval.
- Permutation test:
  prints two-sided and one-sided p-values.
- Interpretation:
  If the CI excludes 0 and permutation p-value is small, evidence supports a real mean difference.

Analysis 3 — Victim Age Group vs. Victim Count (Negative Binomial Regression + Permutation)
- Negative binomial regression:
  fits a count regression model victim_count ~ age_group using only incidents with observed age_group.
  The output includes regression coefficients, p-values, and incident rate ratios (IRRs = exp(coef)),
  along with confidence intervals for IRRs.
- Model comparison:
  AIC(fit_pois, fit_nb) is printed to compare Poisson vs. negative binomial fit.
- Permutation test:
  prints a randomization-based p-value obtained by permuting age_group labels (keeping victim_count fixed).
- Interpretation:
  IRRs close to 1 with confidence intervals containing 1, and a large permutation p-value, suggest little evidence
  that victim counts differ systematically across age groups in the observed-age subset.

