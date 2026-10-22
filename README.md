# Who should be sent for angiography?

**A decision tree on the UCI Heart Disease data.** Final project for *Augmented Decision: Statistical Learning, Risk and Optimization* (2026).

**Presentation:** [open the slides](https://claude.ai/artifact/Qsz1obyqwfsjxnZFMKY6LM)

**Team:** [Team member names]

---

## The question

A patient arrives with chest symptoms. Angiography is the invasive test that confirms heart disease: it is costly and carries some risk. Can a simple decision tree, built from basic measurements, help a doctor decide who really needs it?

| | |
|---|---|
| **Prediction** | Does the patient have heart disease? (yes / no) |
| **Decision** | Send the patient for angiography, or not |
| **Metrics** | Accuracy and recall for the classifier. Average cost per patient for the decision |
| **Approach** | Option 1 of the brief: train and evaluate a decision-tree classifier |

## Data

Cleveland Clinic heart disease data from the UCI Machine Learning Repository:
Janosi, A., Steinbrunn, W., Pfisterer, M., and Detrano, R. (1989). *Heart Disease* [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C52P4X

- 303 patients and 13 inputs (age, sex, chest pain type, blood pressure, cholesterol, exercise ECG, and others).
- Target: the original 0–4 diagnosis turned into yes / no. 46% of patients have disease.
- 6 patients with a missing value were dropped, which leaves 297.

## Method

1. **Preprocessing.** One-hot encode the 4 category variables (chest pain, resting ECG, ST slope, thal), so every split reads as a yes / no question. Trees need no scaling.
2. **Split.** 75% training (222 patients), 25% test (75 patients), stratified. The test set is used once, at the end.
3. **Baseline.** Always predict "no disease".
4. **Tree.** Splits are chosen by Gini impurity, G(q) = 2q(1 − q). The notebook checks the first split by hand.
5. **Tuning.** 5-fold cross-validation on the training set, over depth 1–10 and minimum leaf size {1, 5, 10, 20}.
6. **Decision.** A missed sick patient costs c angiographies, so the Bayes rule is to send the patient when p > 1/c. We do not know c, so we compare several values.

## Results

| Model (75 test patients) | Accuracy | Recall |
|---|---|---|
| Always "no disease" | 53.3% | 0% |
| Decision tree (depth 4, ≥ 10 patients per leaf) | **77.3%** | **68.6%** |

- The most important inputs are asymptomatic chest pain, the number of blocked vessels, and the thallium test.
- Average cost per test patient (1 = one angiography):

| c (cost of a miss) | Send nobody | Send everybody | Tree + cost rule |
|---|---|---|---|
| 2 | 0.93 | 1.00 | **0.69** |
| 3 | 1.40 | 1.00 | **0.85** |
| 5 | 2.33 | **1.00** | 1.04 |
| 10 | 4.67 | **1.00** | 1.23 |

**Recommendation:** use the tree as a triage aid, not as a replacement for the doctor. Only use it if a missed case is judged to cost less than about 4–5 angiographies. Above that, sending everybody is as good or better.

**Limitations:**
- The data is small and old: 297 patients from one hospital in 1988, mostly men.
- With only 75 test patients, one patient is worth 1.3 accuracy points.
- The cost c is assumed, not measured.
- A single tree is unstable: a different random split can give a different tree.

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

Uses Python 3.12. From the repository folder, install the packages:

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
