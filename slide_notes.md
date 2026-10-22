# Slide notes: Heart Disease decision tree (15 min)

All numbers come from `heart_disease_tree.ipynb`. All figures are in `figures/`.
About 1.5 minutes per slide.

---

## 1. Title and question
**Who should be sent for angiography? A decision tree on the UCI Heart Disease data**

- A patient arrives with chest symptoms.
- Angiography is the invasive test that confirms heart disease. It is costly and carries some risk.
- Can a simple tree, built from basic measurements, help decide who really needs it?

## 2. The dataset
Figure: `01_target_distribution.png`

- Cleveland Clinic data, UCI Machine Learning Repository (Janosi et al., 1989, doi:10.24432/C52P4X).
- 303 patients, 13 variables: age, sex, chest pain type, blood pressure, cholesterol, max heart rate, exercise ECG results, and others.
- Target: disease (1) or not (0). The original 0–4 scale is turned into yes/no.
- 137 sick (46%) and 160 healthy (54%), so the classes are fairly balanced.
- Missing values: 6 patients (4 in `n_vessels`, 2 in `thal`). We dropped them, which leaves 297.

## 3. Problem setup: target, decision, metric
- **Prediction:** probability that the patient has disease.
- **Decision:** send for angiography or not.
- **Metrics:**
  - Accuracy and recall (the share of sick patients we catch).
  - Average cost per patient, for the decision.

## 4. Method
- One-hot encoding of 4 category variables (chest pain, ECG, ST slope, thal), so splits read as yes/no questions.
- Train/test split of 75% / 25% (222 / 75 patients), stratified. The test set is used only once, at the end.
- Tuning by 5-fold cross-validation on the training set.
- Baseline: always predict "no disease", which gives **53.3%** test accuracy.

## 5. How a tree picks a split: Gini impurity
Formula: G(q) = 2q(1 − q)

| Group | sick / total | q | G(q) |
|---|---|---|---|
| All training | 102 / 222 | 0.459 | 0.497 |
| Chest pain NOT asymptomatic | 24 / 114 | 0.211 | 0.332 |
| Chest pain asymptomatic | 78 / 108 | 0.722 | 0.401 |

- Weighted impurity of the children: 114/222 × 0.332 + 108/222 × 0.401 = **0.366**
- Decrease: 0.497 − 0.366 = **0.131**. This was the best split, so it became the root.
- Say: "The tree tries every variable and cut point and keeps the biggest drop. This is the same calculation as in the lecture's Gini example."

## 6. Choosing tree depth
Figure: `02_depth_tuning.png`

- We tried depths 1–10 and minimum leaf sizes {1, 5, 10, 20}.
- Best setting: **depth 4, min 10 patients per leaf**, with CV accuracy **82.4%**.
- Training accuracy keeps rising, but CV accuracy stops improving after depth 3–4. Going deeper would only memorise the training data.

## 7. The final tree
Figures: `03_final_tree.png`, `04_feature_importance.png`

- The first question is: is the chest pain "asymptomatic"? (This means no typical chest pain. In this dataset it is the strongest warning sign.)
- Next come the number of blocked vessels on fluoroscopy, the thallium test result, and ST depression.
- Read out one path: "Asymptomatic pain and at least 1 vessel coloured: 55 of 58 training patients were sick."
- Most important variables: chest pain asymptomatic (43%), n_vessels (19%), thal reversible defect (15%).

## 8. Test results
Figure: `05_confusion_matrix.png`

| Model | Test accuracy | Recall |
|---|---|---|
| Baseline (always "no disease") | 0.533 | 0.000 |
| Decision tree (depth 4) | **0.773** | **0.686** |

- That is 24 points above the baseline.
- At the usual 0.5 threshold, the tree still misses 11 of 35 sick patients. That leads to the next slide.

## 9. From probability to decision
Figure: `06_cost_sensitivity.png`

Loss table, with costs in units of one angiography:

| | healthy | sick |
|---|---|---|
| don't send | 0 | c |
| send | 1 | 1 |

- Expected loss: c·p if we don't send, and 1 if we send. So we **send if p > 1/c** (Bayes rule, Lecture 1).
- Average cost per test patient for each value of c:

| c | send nobody | send everybody | tree + cost rule |
|---|---|---|---|
| 2 | 0.93 | 1.00 | **0.69** |
| 3 | 1.40 | 1.00 | **0.85** |
| 5 | 2.33 | **1.00** | 1.04 |
| 10 | 4.67 | **1.00** | 1.23 |

- Say: "If missing a sick patient costs less than about 4–5 angiographies, the tree saves cost. If a miss is worse than that, the tree is not safe enough, and you should just test everyone."

## 10. Recommendation and limitations
**Recommendation:** use the depth-4 tree as a **triage aid** (a first sort of patients), not as a replacement for the doctor. Only use it if the hospital judges a missed case to cost less than about 4–5 angiographies.

**Limitations:**
- Small, old data: 297 patients, one hospital, 1988, mostly men.
- Only 75 test patients, so each patient is about 1.3 accuracy points. Small differences are noise.
- The cost c is assumed, not measured.
- `n_vessels` and `thal` are themselves medical tests, so the tree assumes they were already done.

---

## Likely questions from the audience
- **Why a tree and not LASSO?**
  - It is easy to read and explain to a doctor.
  - It handles yes/no and numeric variables with no scaling.
- **Why drop the missing rows instead of imputing?**
  - Only 6 rows (2%) were missing.
  - Dropping them learns nothing from the data, so no test information can leak.
- **Why cross-validation instead of a validation set?**
  - With 222 training patients, a separate validation set would be too small and noisy.
- **At c = 3, threshold 0.5 does slightly better than 1/3 on the test set (0.84 vs 0.85). Why?**
  - The difference is a single patient out of 75, so it is noise.
  - The 1/c rule is the right rule in expectation. One small test set won't always show it.
- **Is the tree stable?**
  - Not fully. A different random split can give a somewhat different tree. This is a known weakness of single trees.
- **Did you tune for cost instead of accuracy?**
  - No. We tuned for accuracy and then applied the cost threshold.
  - The lecture suggests selecting with the decision loss. We checked, and it gives a similar depth here.
