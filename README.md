# Which Diabetes Indicators Should Make Insurers Charge a Higher Premium?

A business-analytics project using the CDC's 2015 BRFSS health survey (253,680 respondents, 21 survey questions). We predict whether a person has **diabetes or prediabetes** (15.8% of respondents) and use the results to decide which indicators justify a higher premium.

## The business problem
Diabetes drives large claims costs, but many cases are undiagnosed (about 1 in 5 diabetics and 8 in 10 prediabetics don't know). An insurer that cannot see this risk underprices it. We ask:
1. Which self-reported indicators best flag diabetes risk?
2. Which of them can *defensibly* be priced on, and which are fairness or regulatory risks?
3. Can a short questionnaire tier applicants into low, medium and high risk?

## Answer in brief
- **Indicators that most justify a higher premium:** high blood pressure (3.8x as likely), high cholesterol (2.7x), BMI 30+ (26% vs 10%), heart disease/stroke/difficulty walking (2.3x to 2.8x), and age (strong, but age-based pricing is often regulated).
- **Risk compounds:** diabetes rate goes from 2.7% with 0 risk flags to 62% with all 6.
- **Strong predictors not to price on directly:** income and education (the income gap persists after controlling for BMI, blood pressure and age, so pricing on it penalises poorer people; education shows the same gradient but was not tested separately), general health (partly a symptom of existing illness), heavy drinking (lower rate in every age group, unexplained, so no discount), and sex (adds little, about the same as income). Note that in the model, once BMI, age, blood pressure and general health are known, income, sex, heart disease, stroke and difficulty walking all add little.
- **The model works moderately well:** XGBoost reaches AUC 0.82. A 5-question form (BMI, general health, age, cholesterol, blood pressure) reaches 0.815 against 0.824 for all 21 questions. Without general health it drops to 0.786.
- **It tiers people, it does not diagnose them:** at the assumed cost it catches about 80% of diabetics, but only about a third of the people it flags have diabetes. Don't apply a flat loading to everyone flagged.

## Files

| File | What it does |
|---|---|
| `FINAL.ipynb` | **Start here.** Clean XGBoost-only notebook with one chart per claim in the business answer. Written for a non-technical audience. |
| `EDA.ipynb` | Exploratory analysis: distribution of Y, how each indicator relates to diabetes, the alcohol puzzle, risk flags, income effect, quick feature ranking. |
| `MODEL.ipynb` | First modelling pass: baselines, class weights, threshold tuning, calibration, feature importance, short form. |
| `MODEL2.ipynb` | Model comparison: SMOTE vs class weights, XGBoost, and an ensemble of all models. Includes a strict no-overlap split. |
| `PROPOSAL_DRAFT.md` | Written draft: problem statement, data, exploration, modelling summary, conclusion. |
| `DATASET.md` | Description of the dataset and its three files. |
| `diabetes_*_BRFSS2015.csv` | The data. All notebooks use `diabetes_012_health_indicators_BRFSS2015.csv`. |

Each chart has a plain-language "How to read the chart below" box, and each notebook has a glossary near the top.

## What each notebook does

**`EDA.ipynb`**: shows that Y is imbalanced; compares diabetes rates with vs without each indicator; plots rate by BMI, age, health, income and education; tests whether age or health explain the lower rate among heavy drinkers (they don't; heavy drinkers report better health, but the gap persists within each health level); shows risk stacking by number of flags; and checks whether income matters after controlling for BMI.

**`MODEL.ipynb`**: compares logistic regression, random forest and gradient boosting against a dummy model. Handles imbalance with class weights, tunes the cut-off from an assumed cost, recalibrates the probabilities, and ranks indicators. Best model: gradient boosting, test AUC 0.828 (slightly optimistic: this notebook uses a random split, so duplicate answer profiles can sit in both train and test; the profile-based split in `MODEL2`/`FINAL` gives 0.824).

**`MODEL2.ipynb`**: applies SMOTE (synthetic extra diabetic examples, training data only) and compares it with class weights for four algorithms, then ensembles all of them. **Result:** SMOTE did not help; class weights tie or win for every algorithm, and the ensembles do not beat the best single model (XGBoost with class weights).

**`FINAL.ipynb`**: the cleaned-up story using XGBoost with class weights and calibration: indicator charts, the "don't price on these" charts, model performance, short form, operating point, and low/medium/high tiers (observed diabetes rate 3%, 17%, 43%).

## Method notes
- **Target:** Y = prediabetes or diabetes (binary), because prediabetes alone is only 1.8% of the data.
- **Imbalance (about 16% positive):** judged on ROC-AUC and PR-AUC, not accuracy (a model saying "nobody has diabetes" is 84% accurate and useless). Handled with stratified splits, class weights and threshold tuning.
- **Train/test integrity:** the file has 23,899 duplicate rows, and 1,834 identical answer profiles carry conflicting labels. `MODEL2.ipynb` and `FINAL.ipynb` split by answer profile so identical respondents never appear in both train and test. Feature rankings are computed on training or validation data, never on the test set.
- **Calibration:** class-weighted scores over-predict risk (mean 0.39 vs actual 0.16), so we recalibrate with isotonic regression before turning a score into a tier.
- **Cost assumption:** a missed diabetic costs 5x a false alarm. **This is an assumption**, to be replaced with real claims data.

## Limitations
- There is **no premium or claims data**. Diabetes risk is a proxy for cost, so the size of any premium loading is not estimated here.
- The data is **association only**, from self-reported 2015 survey answers. An indicator is not proven to cause diabetes.
- The target mixes prediabetes and diabetes.
- Results come from a single test split with no confidence intervals, so small gaps between top models (for example 0.455 vs 0.450 PR-AUC) are effectively ties.
- Plain SMOTE is a poor fit for yes/no survey answers (it creates fractional values); `SMOTENC` would be the categorical-aware alternative.
- `CholCheck` ("had a cholesterol check") looks predictive only as an artefact and is excluded from the short form.

## How to run
You need Python 3.10 or newer and the three CSV files in the same folder as the notebooks (download them from the Kaggle "Diabetes Health Indicators Dataset"; see `DATASET.md`).

```bash
# 1. (optional) create an isolated environment, using venv or conda
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate

# 2. install the libraries the notebooks import
pip install pandas numpy scipy matplotlib seaborn "scikit-learn>=1.2" imbalanced-learn xgboost jupyterlab

# 3. start Jupyter from this folder and open FINAL.ipynb first
jupyter lab
```
- The notebooks read the CSV by relative path, so launch Jupyter from the folder that contains the data.
- `scikit-learn` 1.2 or newer is needed because `HistGradientBoostingClassifier` uses `class_weight`. The results here were produced with scikit-learn 1.7, XGBoost 3.2 and imbalanced-learn 0.14, so exact numbers may differ slightly on other versions.
- `EDA.ipynb` and `MODEL.ipynb` don't need `imbalanced-learn` or `xgboost`.
- `MODEL2.ipynb` and `FINAL.ipynb` each take about a minute to run.

## Suggested next steps
1. Attach real claims cost to each risk tier to set premium loadings.
2. Add bootstrap confidence intervals to the model comparison.
3. Try `SMOTENC` and a short form that excludes pricing-sensitive variables.
4. Review the fairness and regulatory position on age, income and education before using any of them.
