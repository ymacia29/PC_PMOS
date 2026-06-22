# PC_PMOS: Pooled Logistic Regression Analyses

**Authors:** Yilda Macias
**Status:** Under review / In preparation

### Data Access
These analyses use data from the Sister Study (NIEHS), the Mexican Teachers Cohort (INSP), the Generations Study (ICR), and the Cancer Prevention Study-3 (ACS). 
Data are not publicly available but can be requested.

### Reproducibility
All analytic code is provided. To reproduce:
1. Obtain Sister Study data access
2. Edit the data_path variable in 01_data_cleaning.Rmd
3. Run 01_data_cleaning.Rmd to generate derived datasets
4. Run 02_analysis.Rmd for all models and tables

### Software
R version 4.5.2
Key packages: haven, tidyverse, Hmisc, survival
Full session info in docs/session_info.txt

### Citation
