# CDC Diabetes Health Indicators (BRFSS 2015)

## Overview
Survey-based dataset for predicting diabetes status from health and lifestyle indicators. Derived from the CDC's Behavioral Risk Factor Surveillance System (BRFSS), 2015 edition. Cleaned and consolidated from the larger BRFSS 2015 Kaggle dataset (441,455 respondents, 330 features); the cleaning was done by the Kaggle uploader, not the CDC.

## Background
- Diabetes impairs the body's ability to regulate blood glucose (too little insulin, or ineffective use of it). Type II is the most common form.
- Complications: heart disease, vision loss, lower-limb amputation, kidney disease.
- CDC (2018): 34.2M Americans have diabetes, 88M have prediabetes. About 1 in 5 diabetics and roughly 8 in 10 prediabetics are unaware of their condition.
- Cost: about $327B/year diagnosed; close to $400B/year including undiagnosed diabetes and prediabetes.
- Prevalence varies by age, education, income, location and race. The burden falls more heavily on lower socioeconomic groups.
- Early diagnosis enables lifestyle change and more effective treatment, so risk-prediction models are useful for public health.

## Data source
**BRFSS**: annual CDC telephone survey, running since 1984, with over 400,000 responses per year on risk behaviours, chronic conditions and use of preventive services.

## Files

| File | Rows | Target | Balance |
|---|---|---|---|
| `diabetes_012_health_indicators_BRFSS2015.csv` | 253,680 | `Diabetes_012`: 0 = none/pregnancy only, 1 = prediabetes, 2 = diabetes | Imbalanced |
| `diabetes_binary_health_indicators_BRFSS2015.csv` | 253,680 | `Diabetes_binary`: 0 = no diabetes, 1 = prediabetes or diabetes | Imbalanced |
| `diabetes_binary_5050split_health_indicators_BRFSS2015.csv` | 70,692 | `Diabetes_binary` (same as above) | Balanced 50/50 |

All three files have 21 feature variables.

## Research questions
1. Can BRFSS survey questions accurately predict whether an individual has diabetes?
2. Which risk factors are most predictive?
3. Can a subset of risk factors predict diabetes accurately?
4. Can feature selection produce a short-form questionnaire that flags people with or at high risk of diabetes?

## Modelling notes
- Use the **balanced** file for quick model comparison; use the **imbalanced** files to reflect real prevalence (then use class weights/resampling and judge with recall, precision, F1 or AUC rather than accuracy).
- Class imbalance makes plain accuracy misleading in the 253,680-row files.
- This is self-reported survey data, so expect reporting bias; it supports risk screening, not diagnosis.

## References
- Xie Z. et al. *Building Risk Prediction Models for Type 2 Diabetes Using Machine Learning Techniques*, Preventing Chronic Disease 2019 (based on 2014 BRFSS), the inspiration for this dataset: https://www.cdc.gov/pcd/issues/2019/19_0109.htm
- Kaggle: *Diabetes Health Indicators Dataset* (BRFSS 2015).
