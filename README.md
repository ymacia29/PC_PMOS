# PC_PMOS: Sister Study Analysis

**Authors:** Yilda Macias
**Status:** Under review / In preparation

### Data Access
This analysis uses Sister Study data (NIEHS). 
Data are not publicly available but can be requested through:
https://sisterstudy.niehs.nih.gov/English/data-requests.htm

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
