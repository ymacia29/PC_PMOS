# PC_PMOS Analyses

**Authors:** Yilda Macias, Katie O'Brien

**Status:** Under review / In preparation

### Data Access
These analyses use data from the Sister Study (NIEHS), the Mexican Teachers Cohort (INSP), the Generations Study (ICR), and the Cancer Prevention Study-3 (ACS). 
Data are not publicly available but can be requested.

Cohort abbreviations:
SIS: Sister Study
CPS-3: Cancer Prevention Study-3
MTC: Mexican Teachers Cohort
GS: Generations Study

### Reproducibility
All analytic code is provided. To reproduce:
1. Obtain data access
2. Edit the data_path variable in A1_COH_00_DataMan.Rmd
3. Run A1_COH_00_DataMan.Rmd to generate derived datasets (replace COH with cohort abbreviations above)
4. Run A1_COH_01_Tab1.Rmd for table 1
5. Run A1_COH_02_Analysis.Rmd for table 2 (main analysis)


# Analytic Decisions

This document records key methodological decisions made during data cleaning and analysis for the PMOS and pancreatic cancer pooled analysis. Decisions are organized by cohort and updated as each cohort is added. Where decisions differ across cohorts, rationale is provided. This document serves as the methods supplement for the manuscript and as documentation for the dissertation committee.

**Study:** PMOS (polycystic ovary syndrome/polycystic ovary morphology syndrome), menstrual cycle irregularity, and pancreatic cancer risk: a pooled prospective analysis

**Analysis method:** Pooled logistic regression with yearly person-time intervals

**Last updated:** June 2026


---

## General Decisions (apply to all cohorts)

### Exposure definition
PMOS is operationalized as self-reported PCOS diagnosis. The term PMOS (polycystic ovary morphology syndrome) is used in the manuscript to align with AE-PMOS guidelines, but variables retain PCOS naming in code for consistency with source data. Once a participant is diagnosed with PMOS, it is irreversible.

Menstrual cycle irregularity (MCI) is defined as cycle length outside the 21-31 day range, including cycles too irregular to estimate and never having had a period. MCI is fixed at baseline and not treated as time-varying, as it reflects a set reproductive characteristic instead of a time-varying condition.

Combined exposure variables:
- `pcos_or_mci`: PMOS or MCI (either condition present)
- `pcos_and_mci`: PMOS and MCI (both conditions present)

### Time scale
Study time (years since baseline) is used as the primary time scale. Age at baseline entry (`AgeExact_T0` in SIS, reconstructed from person-years in CPS-3) and interval index (within-person time counter) are included in all models as continuous terms to control for age.

### Pooled logistic regression
PLR with yearly person-time intervals is used in place of Cox proportional hazards regression. OR from PLR approximates the HR under the rare disease assumption, which holds for pancreatic cancer. The interval index and age at entry together account for the age time scale.

### Time-varying covariate assignment
For each yearly interval, the covariate value assigned is from the most recent survey wave at or before the start of that interval (using last observation carried forward, LOCF). _Where a participant has no observed value at earlier waves but has an observed value at a later wave, the value is carried backward (downup fill)._ This applies to: BMI, smoking, alcohol, physical activity, menopause status, parity, hormonal birth control, diabetes, and cholesterol.

Pending committee input on whether backward fill is acceptable or whether intervals with no prior observed value should be left as missing and handled by complete-case exclusion within the model.

### Diabetes
Time-varying binary. Flips from 0 to 1 at diagnosis. Pre-diabetes and borderline diabetes collapse to 0 due to lack of wave-level data on these conditions. Type 1 diabetes handling is cohort-specific (documented below). Once diabetic, always diabetic (irreversible).

### Cholesterol
Time-varying binary. Flips from 0 to 1 at or after diagnosis. Borderline cholesterol collapses to 0 due to lack of wave-level data on condition. Once diagnosed, always 1 (irreversible).

### Model sequence
All exposures follow the same five-model sequence:
- Model 1: demographics (education, race/ethnicity) + smoking + age adjustment
- Model 2: Model 1 + reproductive factors (menarche, parity, hormonal birth control)
- Model 3: Model 1 + metabolic factors (BMI, diabetes, cholesterol)
- Model 4: Model 1 + lifestyle factors (physical activity, alcohol)
- Model 5: fully adjusted (all covariates from Models 1-4)

### Menarche
Used as continuous in cohorts where continuous data are available (SIS). Used as binary (under 13 / 13 or older) in cohorts where only categorical data are available (CPS-3). For the pooled analysis, menarche will be harmonized to the binary version across all cohorts.



---

## Sister Study (SIS)

### Data source
NCI Sister Study (dr00328_03_01). Access via approved data request.

### Baseline
Enrollment baseline (AgeExact_T0). Six survey waves: T0 through T5.

### Sample size
- After PanCa outcome exclusions (withdrew, pre-baseline, uncertain, unknown): n = 50,859
- After education exclusion: n = 50,847
- After race/ethnicity exclusion: n = 50,840
- After menstrual dysfunction exclusion: n = 44,433
- After zero follow-up exclusion (EOF age <= baseline age): n = 44,186
- Final analytic sample: n = 44,186
- Incident pancreatic cancer cases: 216

### PMOS operationalization
- Prevalent PMOS: `NA_FU_PCOS_Event == "b"` (pre-baseline diagnosis flag from SAS format)
- Incident PMOS: `FU_PCOS_Event == "1"` with `FU_PCOS_DxAgeExact <= AgeExact_Tn` at each wave
- Time-varying PMOS updates at each of T1 through T5
- Total PMOS cases (prevalent + incident across all waves): 2,784 (5.5% of sample)

### Menstrual cycle irregularity
Defined using HR57 (cycle length at baseline):
- No: codes 1, 2, 3 (less than 35 days between periods)
- Yes: codes 4, 5, 6, 7, 96 (35+ days, too irregular, never had a period)
- Excludes 6,407 participants missing HR57

### Outcome
`FU_PanCa_EOFAgeExact`: exact age at end of follow-up. Age at diagnosis for cases, age at censoring (death, LTFU, end of study) for non-cases. Participants censored at other cancer diagnoses are handled implicitly through this variable.

### Age reconstruction
Age variables directly observed at each wave (AgeExact_T0 through AgeExact_T5). No reconstruction needed.

### Diabetes
Event-based time-varying variable. `diab_dx_age` derived from `FU_Diab_DxAgeExact`. Pre-baseline diabetes flagged using `NA_FU_Diab_Event == "b"` and assigned `diab_dx_age = AgeExact_T0`. Pre-diabetes (`FU_Diab_Flag_PreDiab_BaselineRpt`) collapses to 0.

### Cholesterol
Event-based time-varying variable. `chol_dx_age` derived from `FU_Chol_DxAgeExact`. Pre-baseline cholesterol flagged using `FU_Chol_DxPreBaseline == "1"` and assigned `chol_dx_age = AgeExact_T0`. Borderline cholesterol collapses to 0.

### Hormonal birth control
Three-category time-varying variable (Never / Current / Former) constructed from `HR_BCAny_Ever`, `HR_BCAnyCurrent`, and `HR_BCAny_MaxAge` at each wave. Current defined as use within 5 years of wave age; Former defined as use more than 5 years before wave age.

### Exclusion criteria for time-varying covariates
Complete-case exclusion not applied for time-varying covariates (BMI, smoking, alcohol, physical activity, menopause, parity, HBC, diabetes, cholesterol). Missing intervals are left as NA and handled by complete-case exclusion within the model. _Pending committee guidance on backward fill._



---

## Cancer Prevention Study-3 (CPS-3)

**Data source:** American Cancer Society CPS-3. Access via approved data request.  
**Data freeze:** 2020-12-31 (next registry update expected 2027)  
**File:** `COHORTFINAL_SA_061626.Rda`  
**Participant identifier:** `altID` (not documented in data dictionary; confirmed present in dataset)

### Survey wave structure
- REF (enrollment, 2006-2013): analytic baseline; age = `AGEREF`
- 2015 follow-up survey: first time-varying covariate update; age = `AGEREF + FAILPYRS_REF_15`
- 2018 follow-up survey: second time-varying covariate update; age = `AGEREF + FAILPYRS_REF_15 + FAILPYRS_15_18`
- Survey return is not required for cohort inclusion; proxy dates used for non-returnees in FAILPYRS variables

### Age reconstruction
All wave ages reconstructed from person-years segments. Approach confirmed correct by CPS-3 data team (Ellen Mitchell, June 2026):
- `age_ref  = AGEREF`
- `age_2015 = AGEREF + FAILPYRS_REF_15`
- `age_2018 = AGEREF + FAILPYRS_REF_15 + FAILPYRS_15_18` (2018 returnees only)
- `age_stop = AGEREF + FAILPYRS_TOTAL`

Note: `FAILPYRS_15_18` and `FAILPYRS_18_ENDFU` are coded 0 (not NA) for participants who failed before those intervals -- reset to NA before age reconstruction (per Ellen Mitchell, June 2026). `SUR_2018` similarly reset to NA for participants who failed before 2018.

### Sample size
- Update after data cleaning run

### PMOS operationalization
PCOS was not collected at REF enrollment (confirmed by Caroline Um, June 2026). Prevalent PMOS at REF baseline is derived retrospectively from reported diagnosis age at the 2015 or 2018 follow-up survey:
- Prevalent at REF: `PCOS_AGE15 <= AGEREF` OR `PCOS_AGE18 <= AGEREF`, with plausibility check (diagnosis age > 10)
- Incident at 2015: `PCOS_AGE15 > AGEREF` AND `PCOS_AGE15 <= age_2015`
- Incident at 2018: `PCOS_AGE18 > age_2015` AND `PCOS_AGE18 <= age_2018`
- PCOS coded 1=No, 2=Yes in source data (reverse of Sister Study)
- Limitation: prevalent PMOS relies on recalled diagnosis age reported 5-15 years after the fact; documented as limitation in manuscript

### Menstrual cycle irregularity
Defined using `CYCLENGTH` at REF enrollment (fixed at baseline):
- No: codes 1, 2, 3 (less than 32 days)
- Yes: codes 0, 4, 5, 6 (too irregular to estimate, 32+ days)
- Missing: code 99

### Outcome
`PANCCAN_FINAL`: final binary pancreatic cancer outcome including registry-verified cases and cancer death cases.

**Censoring:** Participants are censored at death, loss to follow-up, or end of study (2020-12-31). Non-pancreatic cancer diagnosis is NOT a censoring event -- participants continue contributing person-time after other cancer diagnoses. Note: current `FAILPYRS_TOTAL` variable truncates at other cancer diagnoses; corrected follow-up time variable requested from data team (June 2026).

### Key analytic decisions
- Analytic baseline: REF enrollment survey (`AGEREF`)
- `age_start = AGEREF` for all participants
- `FAILPYRS_15_18` and `FAILPYRS_18_ENDFU` reset from 0 to NA for pre-interval failures
- All 2015/2018 covariate updates gated on valid reconstructed wave age to prevent spurious covariate values for participants who failed before those waves
- Diabetes: time-varying T2D binary; gestational and Type 1 excluded; once diabetic always diabetic
- Cholesterol: time-varying direct binary per wave; once Yes always Yes
- Physical activity: PA_REF comparability with PA15/PA18 pending data team confirmation (June 2026); all three waves included but flagged
- Alcohol: formats incompatible across waves; REF uses never/former/current status; 2015 uses continuous FFQ drinks/day; 2018 uses ordinal 5-category. Harmonized to never/former/current/<1 drink/day/1+ drink/day where possible. For pooled analysis will simplify to never/former/current. Documented in analytic_decisions.md
- Menarche: categorical only (7 categories); harmonized to binary (under 13 / 13 or older) for consistency with pooled analysis
- Education: CPS-3 collapses HS grad with less than HS (codes 1-2 map to 0); 4-year and graduate degrees combined (code 4 maps to 3); documented in analytic_decisions.md
- Race/ethnicity: Asian (4) and Other/Missing (5) collapsed to Other for harmonization with Sister Study
- LOCF chains: REF -> 2015 -> 2018 for all time-varying covariates

Exposure-specific exclusion decisions (June 2026)
PCOS was not captured at REF enrollment in CPS-3, meaning PMOS classification relied on follow-up survey return at 2015 or 2018. An initial analytic approach excluded participants who returned neither PCOS survey (n = 78,545) to ensure exposure classifiability. This exclusion was found to remove 116 of 237 total pancreatic cancer cases (49%), with outcome-related missingness likely driving the pattern (cases more likely to die or become too ill to return follow-up surveys -- fatal illness).
Upon further review, all 78,545 excluded participants had complete CYCLENGTH data and could be fully classified on MCI without PCOS survey data. Additionally, no exposed pancreatic cancer cases were observed for PMOS or the combined PMOS-and-MCI exposure in CPS-3, making PMOS-specific models non-estimable regardless.
Decision: The PCOS survey response exclusion and the pre-2015 case exclusion are not applied. All participants with valid AGEREF and follow-up time are retained. Exclusion criteria are exposure-specific:

MCI models: no PCOS survey exclusion applied; all participants with valid CYCLENGTH retained
PMOS or MCI models: same as MCI; PMOS component contributes no exposed cases in CPS-3
PMOS models: not run; no exposed pancreatic cancer cases observed
PMOS and MCI models: not run; no doubly-exposed pancreatic cancer cases observed

Decision made in June 2026 and applies to CPS-3 only. *For MTC and GS, the same logic will be applied upon data receipt: PCOS survey exclusion will only be applied if PMOS models have sufficient exposed cases to estimate.*

### Menstrual cycle irregularity
Defined using `CYCLENGTH` at REF enrollment (fixed at baseline):
- No: codes 1, 2, 3 (less than 32 days)
- Yes: codes 0, 4, 5, 6 (too irregular to estimate, 32+ days)
- Missing: code 99

### Outcome
`PANCCAN_FINAL`: final binary pancreatic cancer outcome including registry-verified cases and cancer death cases.

**Censoring:** Participants are censored at death, loss to follow-up, or end of study (2020-12-31). Non-pancreatic cancer diagnosis is NOT a censoring event -- participants continue contributing person-time after other cancer diagnoses. Note: current `FAILPYRS_TOTAL` variable truncates at other cancer diagnoses; corrected follow-up time variable requested from data team (June 2026).

### Diabetes
Wave-level T2D binary variables (`DIAB2_15`, `DIAB2_18`). Type 1 diabetes excluded. Gestational diabetes excluded. Pre-baseline T2D identified from `DIABETES_REF == 1 & DIABTYPE_REF == 2`.

### Cholesterol
Direct binary per wave (`HCHOL_B`, `HCHOL15`, `HCHOL18`). Once Yes, always Yes. Note: 2018 variable has no explicit No option -- coded 0 if survey returned and Yes not checked.

### Alcohol
Formats differ across waves and cannot be directly harmonized to a common continuous measure:
- REF: categorical status (never/former/current) plus ordinal drinks/day
- 2015: continuous drinks/day from FFQ
- 2018: ordinal 5-category drinks/day

Harmonized variable uses never/former/current/<1 drink/day/1+ drink/day where possible. For pooled analysis, may simplify further to never/former/current. Documented discrepancy will be noted in methods.

### Menarche
Categorical only (MENARCHE_AGECAT, 7 categories). Harmonized to binary (under 13 / 13 or older) for consistency with pooled analysis.

### Education
CPS-3 collapses HS grad with less than HS in a single category (codes 1-2 map to 0). 4-year and graduate degrees are combined (code 4 maps to 3). Two-level coding at top and bottom differs from SIS. Noted in harmonization documentation.




---

## Mexican Teachers Cohort (MTC)

*Data expected approximately July 2026. This section will be completed upon data receipt.*




---

## Generations Study (GS)

*Data expected September 2026. This section will be completed upon data receipt.*




---

## Pooled Analysis

*To be completed after all cohort-specific analyses are finalized.*

Planned approach:
- Cohort-specific PLR models using harmonized variable set
- Cochran Q test and I² for heterogeneity across cohorts
- If heterogeneity is non-significant: harmonized pooled PLR with cohort as fixed effect (strata)
- If heterogeneity is significant: report cohort-specific results and note heterogeneity; consider random effects meta-analysis

Harmonization decisions to be finalized:
- Menarche: binary (under 13 / 13 or older) across all cohorts
- Alcohol: simplify to never/former/current if CPS-3 and other cohorts cannot support finer categories
- Education: four-level scheme where possible; document collapses per cohort
- Race/ethnicity: four-level scheme (NH White, NH Black, Hispanic/Latina, Other); Asian collapsed to Other where cell sizes are insufficient



---

### Software
R version 4.5.2
Key packages: haven, tidyverse, Hmisc, survival
Full session info in docs/session_info.txt



---

### Citation
