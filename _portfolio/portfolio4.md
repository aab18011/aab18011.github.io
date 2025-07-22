---
title: "Scoring and Evaluation Algorithm for MD Simulation Data"
excerpt: "Overview of our MD scoring workflow for 5‑HT₂A receptor simulations.<br/><img src='/images/md_scoring_workflow.png'>"
collection: portfolio
---

## Scoring and Evaluation Algorithm for MD Simulation Data

### Overview of Algorithmic Workflow

To quantify and compare the stability of each replicate in the molecular dynamics (MD) simulation of the 5‑HT₂A receptor, we implemented a scoring algorithm that extracts and summarizes several key metrics:

- Backbone RMSD  
- Cα RMSD  
- Side‑chain RMSD  
- Radius of gyration (Rg)  
- Root mean square fluctuation (RMSF)  
- Principal component analysis (PCA) projections  

Input files are standard GROMACS `.xvg` files, organized by replicate number and parsed using custom Python utilities.

---

### Metric Extraction and Averaging

For each metric, only the **latter half** of the simulation (50 % of the total time) is used to compute average values and standard deviations. This focuses analysis on the equilibrated portion of each trajectory, reducing bias from initial relaxation artifacts. Each metric is:

1. Extracted from its `.xvg` file  
2. Validated for formatting consistency  
3. Transformed into a numerical `pandas.DataFrame`

---

### Scoring Function

Each metric is converted into a normalized score in the range \([0,100]\) using an inverse‑square function:

\[
S = \frac{100}{\,1 + \bigl(\tfrac{x}{T}\bigr)^2 + k\,\delta^2\,}
\]

Where:

- \(x\) is the observed value of the metric (in Å if applicable)  
- \(T\) is the predefined threshold  
- \(k\) is a penalty coefficient  
- \(\delta = \max(0,\,x - T)\) is the amount by which \(x\) exceeds the threshold  

> **Note:** If \(x \le T\), the penalty term \(k\,\delta^2\) is omitted, so the score decays purely by the inverse‑square law.

---

### Weighting Scheme

To ensure fair contribution of each metric to the final score, we assign weights proportional to the absolute magnitude of each normalized metric:

\[
w_i = \frac{\bigl|x_i\bigr|}{\sum_j \bigl|x_j\bigr|}
\]

Metrics with higher fluctuations are down‑weighted, while those with smaller, stable values carry more influence. This also normalizes across different scales (e.g., RMSF vs Rg).

---

### Total Score Aggregation

The final score per replicate is computed as the weighted average of all individual scores:

\[
S_{\mathrm{total}} = \sum_i w_i \, S_i
\]

- \(S_i\): normalized score for metric \(i\)  
- \(w_i\): corresponding weight  
- Missing or `NaN` values are automatically excluded

---

### Justification and Interpretation

This scoring algorithm balances interpretability with quantitative rigor:

1. **Penalty for large deviations:** Any metric exceeding physiological thresholds is penalized via \(\delta\).  
2. **Balanced weighting:** No single metric can dominate unless it exhibits truly large variation.  
3. **Equilibrated focus:** By analyzing the second half of each trajectory, we minimize artifacts from initial relaxation.

Together, these design choices enable:

- **Meaningful comparison** between replicates  
- **Rational selection** of representative structures  
- **Filtering** of unstable trajectories prior to downstream analysis  

