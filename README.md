# Who should be sent for angiography?

**A decision tree on the UCI Heart Disease data.** Final project for *Augmented Decision: Statistical Learning, Risk and Optimization* (2026).

**Presentation:** [open the slides](https://claude.ai/artifact/Qsz1obyqwfsjxnZFMKY6LM)

**Team:** Andrea, Walid, Ghali & Paul

---

## The question

A patient arrives with chest symptoms. Angiography is the invasive test that confirms heart disease: it is costly and carries some risk. Can a simple decision tree, built from basic measurements, help a doctor decide who really needs it?

| | |
|---|---|
| **Prediction** | Does the patient have heart disease? (yes / no) |
| **Decision** | Send the patient for angiography, or not |
| **Metrics** | Accuracy, recall and specificity for the classifier. Average cost per patient for the decision |
| **Approach** | Option 1 of the brief: train and evaluate a decision-tree classifier |

## Data

Cleveland Clinic heart disease data from the UCI Machine Learning Repository:
Janosi, A., Steinbrunn, W., Pfisterer, M., and Detrano, R. (1989). *Heart Disease* [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C52P4X

- 303 patients and 13 inputs (age, sex, chest pain type, blood pressure, cholesterol, exercise ECG, and others).
- Target: the original 0–4 diagnosis turned into yes / no. 46% of patients have disease.
- 6 patients with a missing value were dropped, which leaves 297 (68% men, aged 29–77).
- Every patient in the data had an angiography (that is where the diagnosis comes from), so they were all already judged worrying enough to be tested.

## Method

1. **Preprocessing.** One-hot encode the 4 category variables (chest pain, resting ECG, ST slope, thal), so every split reads as a yes / no question. Trees need no scaling.
2. **Split.** 75% training (222 patients), 25% test (75 patients), stratified. The test set is used once, at the end.
3. **Baseline.** Always predict "no disease".
4. **Tree.** Splits are chosen by Gini impurity, G(q) = 2q(1 − q). The notebook checks the first split by hand.
5. **Tuning.** 5-fold cross-validation on the training set, over depth 1–10 and minimum leaf size {1, 5, 10, 20}.
6. **Decision.** A missed sick patient costs c angiographies, so the Bayes rule is to send the patient when p > 1/c. We do not know c, so we compare several values.
7. **Robustness.** Tuning for cost instead of accuracy, a tree without the two extra imaging tests, the whole procedure on 30 random splits, and a Hoeffding bound on the test score.

## Results

| Model (75 test patients) | Accuracy | Recall | Specificity |
|---|---|---|---|
| Always "no disease" | 53.3% | 0% | 100% |
| Stump (1 question, reference) | 74.7% | 71.4% | 77.5% |
| Decision tree (depth 4, ≥ 10 patients per leaf) | **77.3%** | **68.6%** | **85.0%** |

- The most important inputs are asymptomatic chest pain, the number of blocked vessels, and the thallium test.
- Average cost per test patient (1 = one angiography):

| c (cost of a miss) | Send nobody | Send everybody | Tree + cost rule |
|---|---|---|---|
| 2 | 0.93 | 1.00 | **0.69** |
| 3 | 1.40 | 1.00 | **0.85** |
| 4 | 1.87 | 1.00 | **0.93** |
| 5 | 2.33 | **1.00** | 1.04 |
| 10 | 4.67 | **1.00** | 1.23 |

The tree with the cost rule is cheaper than sending everybody only while a miss costs less than about **4.7 angiographies**.

## Robustness checks (notebook section 9)

| Question | Check | Answer |
|---|---|---|
| Does tuning for cost instead of accuracy change things? | Same grid, CV scored by the average cost at c = 3 | Different tree (depth 3, 7 leaves), almost the same test cost (0.88 vs 0.85). Same conclusion. |
| Are all inputs known before the decision? | Tree without `n_vessels` and `thal` (extra imaging tests) | 74.7% test accuracy instead of 77.3%, same cost at c = 3. Still clearly useful. |
| Is our split lucky? | Whole procedure repeated on 30 random splits | Mean test accuracy 76.3% (range 67–88%), beats the baseline on 30/30 splits. Depth and root question change between splits. |
| How precise is one test score? | Hoeffding bound from Lecture 2, 75 patients | About ±16 points at 95%. Small differences between models are noise. |

## Recommendation

The tree is **not safe enough to decide alone** who skips angiography: for a realistic cost of a missed disease (more than about 4.7 angiographies), sending everybody is cheaper and safer. It is useful as a **triage aid**, to flag high-risk patients first, while the doctor keeps the final decision. Using it as a filter would require the hospital to state its cost c explicitly.

**Limitations:**
- The data is small and old: 297 patients from one hospital, collected in the 1980s, 68% men.
- Every patient was already sent for angiography, so the tree estimates the risk among patients a doctor already found worrying, not among all patients with chest symptoms.
- With only 75 test patients, one patient is worth 1.3 accuracy points (±16 points at 95%).
- A single tree is unstable: a different random split can give a different depth and a different root question.
- `n_vessels` and `thal` are extra tests; without them the tree loses a few points.
- The cost c is assumed, not measured, and the loss table ignores the risk of the angiography itself.

## Repository layout

```
├── heart_disease_tree.ipynb   # the full analysis, already run, with explanations
├── slide_notes.md             # what to say on each slide, plus likely questions
├── figures/                   # every plot used in the slides
├── data/
│   └── processed.cleveland.data   # UCI file, used as downloaded
└── requirements.txt
```

## How to run

Tested with Python 3.11, scikit-learn 1.8.0, pandas 2.3.3, NumPy 2.4.6, matplotlib 3.10.9. The 30-split stability check takes under a minute. From the repository folder, install the packages:

```bash
pip install -r requirements.txt
```

Then open the notebook and run all cells:

```bash
jupyter lab heart_disease_tree.ipynb
```

The notebook reads from `data/` and writes the plots to `figures/`.

## Use of AI tools

An LLM (Claude) helped us choose the dataset and scope, write the notebook, and build the slides. As the brief requires, the full prompts and answers will be included in the appendix of the report.
