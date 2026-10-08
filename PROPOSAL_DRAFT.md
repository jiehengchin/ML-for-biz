# Which Diabetes Indicators Should Make Insurers Charge a Higher Premium?

## 1. Problem Statement

**Business problem.** A health or life insurer must price each policyholder's expected claims cost. Diabetes is one of the largest drivers of that cost, but most underwriting questions can't confirm it, because many cases are undiagnosed. We ask which self-reported health and lifestyle indicators best flag diabetes or prediabetes, and so which indicators justify a risk-based premium loading.

**Why it matters**
- Diabetes costs the US about $327B a year diagnosed, and close to $400B including undiagnosed diabetes and prediabetes (CDC).
- About 1 in 5 diabetics and 8 in 10 prediabetics don't know they have it. An insurer who can't see this risk underprices it, a form of adverse selection.
- Diabetes drives expensive downstream claims: heart disease, kidney disease, vision loss and amputation.
- Early identification also lets an insurer offer wellness programmes, which lowers claims and helps the customer.

**Why it's interesting**
- It is a cheap-screening question. With a short questionnaire instead of a medical exam, how well can we separate high-risk from low-risk applicants?
- It is a fairness and compliance question. Some strong predictors (age, income, education, sex) are protected or socially sensitive, so we must separate *actionable, medically relevant* indicators from *proxies* we shouldn't price on.

**Analytical framing.** We predict diabetes status (Y) from 21 survey indicators (X). The model's important features and their effect sizes act as the **risk-factor ranking**. A higher predicted probability maps to a higher expected claims cost, so a higher premium loading.

**Limitation to state up front.** The data has no premiums or claims. Diabetes risk is a proxy for cost, and the premium loading itself would need actuarial cost data per risk tier, which is out of scope here.

---

## 2. Data

### 2.1 What is the data?
- **Source:** CDC Behavioral Risk Factor Surveillance System (BRFSS), 2015, a national telephone health survey. We use the cleaned Kaggle version (see `DATASET.md`).
- **Size:** 253,680 respondents, 21 features, 1 target, no missing values.
- **Target (Y):** `Diabetes_012` (0 = none, 1 = prediabetes, 2 = diabetes), or the binary `Diabetes_binary` (0 = none, 1 = prediabetes or diabetes).
- **Features (X):**

| Group | Variables |
|---|---|
| Medical conditions | HighBP, HighChol, Stroke, HeartDiseaseorAttack, DiffWalk |
| Self-rated health | GenHlth (1 excellent to 5 poor), MentHlth and PhysHlth (bad days in last 30) |
| Body | BMI |
| Lifestyle | Smoker, PhysActivity, Fruits, Veggies, HvyAlcoholConsump |
| Healthcare access | CholCheck, AnyHealthcare, NoDocbcCost |
| Demographics | Age (13 bands), Sex, Education (1 to 6), Income (1 to 8) |

- **Data quality notes:**
  - There are **23,899 exact duplicate rows (9.4%)**. Most are probably distinct people with identical answers, because the features are mostly binary or ordinal, so we keep them but should check sensitivity.
  - BMI has an extreme maximum (98), so we should check outliers.
  - All features are self-reported, so expect reporting bias.

### 2.2 Distribution of Y

| Class | Count | Share |
|---|---|---|
| 0: No diabetes | 213,703 | 84.2% |
| 1: Prediabetes | 4,631 | 1.8% |
| 2: Diabetes | 35,346 | 13.9% |

- Binary view: **15.8%** have prediabetes or diabetes (84.2% don't).
- The data is **strongly imbalanced**, and the prediabetes class is tiny, so modelling it separately is unreliable. We recommend the binary target.
- A model that always predicts "no diabetes" scores 84% accuracy and is useless for insurance. We therefore evaluate with recall, precision, AUC and cost-weighted metrics, not accuracy.
- The roughly 16% positive rate includes prediabetes (1.8%) as well as diabetes (13.9%).

### 2.3 Relationship between Y and X

Share with diabetes or prediabetes, by indicator (overall base rate 15.8%):

**Binary indicators**

| Indicator | Without | With | Ratio |
|---|---|---|---|
| HighBP | 7.2% | 27.1% | 3.8x |
| HighChol | 9.2% | 24.7% | 2.7x |
| DiffWalk | 12.1% | 33.8% | 2.8x |
| HeartDiseaseorAttack | 13.7% | 35.8% | 2.6x |
| Stroke | 15.0% | 34.3% | 2.3x |
| PhysActivity | 23.6% | 13.2% | 0.56x (protective) |
| Smoker | 13.7% | 18.3% | 1.3x |
| NoDocbcCost | 15.3% | 20.3% | 1.3x |
| Sex (male) | 14.8% | 17.0% | 1.1x |
| HvyAlcoholConsump | 16.3% | 7.3% | 0.45x (counter-intuitive) |

**BMI band**

| BMI | Diabetes rate |
|---|---|
| Under 18.5 | 6.3% |
| 18.5 to 25 | 6.7% |
| 25 to 30 | 13.0% |
| 30 to 35 | 21.7% |
| 35 and over | 33.0% |

**Ordinal indicators**
- **GenHlth:** rises steadily from 3.2% (excellent) to 40.8% (poor), a 13x spread.
- **Age:** rises from 1.7% (age 18 to 24) to a peak of about 24% at ages 70 to 79, then eases slightly in the oldest band.
- **Income:** 27.5% in the lowest bracket, peaks at 29.2% in the second, then falls steadily to 9.1% (highest).
- **Education:** peaks at 33.2% (elementary), then falls to 11.1% (college graduate). The lowest group (28.2%) has only 174 respondents.

**Correlation with binary Y (Pearson)**

| Strongest positive | r | Strongest negative | r |
|---|---|---|---|
| GenHlth | 0.30 | Income | -0.17 |
| HighBP | 0.27 | Education | -0.13 |
| BMI | 0.22 | PhysActivity | -0.12 |
| DiffWalk | 0.22 | Veggies | -0.06 |
| HighChol | 0.21 | HvyAlcoholConsump | -0.06 |
| Age | 0.19 | Fruits | -0.04 |

### 2.4 What the exploration tells us
1. **Clinical conditions dominate.** High blood pressure, high cholesterol, obesity and prior cardiac events show the biggest jumps. These are objective, medically relevant and the strongest candidates for premium loading.
2. **BMI is non-linear.** Risk is low and flat below 25, then climbs about 6 to 11 points per band above. A banded or tree-based treatment will fit better than a straight line.
3. **Self-rated general health is the single best predictor.** It likely also reflects existing disease, so it is partly a consequence of diabetes rather than a cause, which matters for how we interpret it.
4. **Income and education matter, but they are proxies.** They are the likely vehicle for social-determinant effects and may be unsuitable for direct pricing (regulatory and fairness concerns).
5. **Some effects look wrong and need care.** Heavy drinkers show *lower* diabetes rates, in every age and health group (the 18-39 gap is small). They report better health (12.0% vs 17.5% fair/poor), but that does not close the gap. The cause is unresolved: confounding or a "sick quitter" effect (people with diabetes avoid alcohol) is possible but untested, and it is not a causal benefit. We should not reward drinking in pricing.
6. **Weak features.** AnyHealthcare and Fruits have little signal. Sex is weak on its own (1.1x) and adds little in the model (about the same as income), so all are candidates to drop from a short-form questionnaire.
7. **Correlation is not causation.** The survey is a single cross-section, so we show association with diabetes status, not that an indicator *causes* diabetes or future claims.

### 2.5 Where the rest is
Charts are in `EDA.ipynb` and `FINAL.ipynb`; models are in `MODEL.ipynb`, `MODEL2.ipynb` (SMOTE, XGBoost, ensemble) and `FINAL.ipynb` (clean XGBoost-only version). Still **not done**: translating each risk tier into a premium loading, which needs claims cost data per tier.

---

## 3. Modelling Approach and Results

- **Imbalance:** about 16% positive, so we used stratified splits, class weights, PR-AUC and ROC-AUC (not accuracy), threshold tuning and calibration.
- **Train/test integrity:** the file has 23,899 duplicate rows and 1,804 identical answer profiles with conflicting yes/no labels, so we split by answer profile. No profile appears in more than one split.
- **SMOTE:** applied to training data only. It did not help: class weights scored equal or better for every algorithm (XGBoost PR-AUC 0.455 vs 0.441).
- **Models:** logistic regression, random forest, HistGradientBoosting and XGBoost, plus rank-averaged ensembles. All score ROC-AUC 0.814 to 0.824. Ensembles do not beat the best single model.
- **Best model:** XGBoost with class weights, isotonic-calibrated (mean predicted risk 0.155 vs actual 0.156).
- **Short form:** 5 questions (BMI, general health, age, cholesterol, blood pressure) reach AUC 0.815, vs 0.824 for all 21 features (AUC 0.786 without general health).
- **Operating point (assumed 5:1 cost of a missed diabetic vs a false alarm):** catches 77% to 80% of diabetics at about 33% precision, cutting the assumed cost from 49k to about 26k.

## 4. Conclusion and Recommendation

**Which indicators justify a higher premium?**

| Indicator | Evidence | Pricing view |
|---|---|---|
| High blood pressure | 27% vs 7% (3.8x) | Strongest actionable factor |
| BMI 30 or more | 26% vs 10% for BMI under 30 (22% at BMI 30-35 and 33% at 35+, vs about 7% at 18.5-25) | Risk is flat below 25, then climbs |
| High cholesterol | 25% vs 9% (2.7x) | Medical, objective |
| Heart disease, stroke, difficulty walking | 2.3x to 2.8x on their own; add little to the model once BMI, age, BP and general health are known | Medical, often already priced |
| Age | about 2% at 18 to 24, about 24% at 70 to 79 | Strong, but age-based pricing may be regulated |

Risk compounds: diabetes rate rises from 2.7% (0 risk flags) to 62% (6 flags).

**Strong predictors not to price on directly**
- **Income and education:** the income gap persists within every BMI, blood-pressure and age group, so it is not only a medical or age proxy, and pricing on it raises fairness and regulatory risk. (Education shows the same gradient but was not tested separately. In the model, income and education add little once the five short-form questions are known.)
- **General health:** partly a consequence of illness already present.
- **Heavy drinking:** associated with lower diabetes rates in every age and health group, unexplained (heavy drinkers report better health, but the gap holds within each health level). Do not reward it.

**Recommendation**
1. Use a 5-question short form (blood pressure, cholesterol, BMI, age, general health). Dropping general health lowers AUC from 0.815 to 0.786, so it is worth keeping even though it partly reflects existing illness.
2. Assign low/medium/high tiers from the calibrated model score, not indicator by indicator.
3. Set the premium loading per tier from real claims cost data.
4. Pair the high-risk tier with a screening or wellness programme, since early diagnosis lowers claims.

**Limitations**
- No premium or claims data: diabetes risk is a proxy for cost, and the 5:1 cost ratio is an assumption.
- Association only, from self-reported 2015 survey data. The target mixes prediabetes and diabetes.
- Only about a third of people flagged as high-risk actually have diabetes, so the model suits risk-tiering, not diagnosis or a flat loading on everyone flagged.
