# Oral script: Who should be sent for angiography?

15 minutes, 4 speakers, then 5 minutes of questions.
All numbers come from `heart_disease_tree.ipynb`.

| Part | Speaker | Topic | Time | Slides |
|---|---|---|---|---|
| 1 | Speaker 1 | The problem and the data | 3:30 | 1–4 |
| 2 | Speaker 2 | Method: how we built and tuned the tree | 3:45 | 5–8 |
| 3 | Speaker 3 | Results: from prediction to decision | 4:00 | 9–12 |
| 4 | Speaker 4 | Can we trust it? Checks, recommendation, limits | 3:45 | 13–16 |

`[Slide N]` = change slide. *Italics in brackets* = stage directions, not to be read.
"Why this and not that?" boxes are optional: say them if time allows, otherwise keep them for the questions.

---

## Part 1: The problem and the data (Speaker 1, about 3:30)

**[Slide 1: Title]**

Good morning everyone. We are Andrea, Walid, Ghali and Paul.
Our project asks a simple question: **who should be sent for angiography?**

**[Slide 2: The situation]**

Imagine a patient comes to the hospital with chest symptoms. The doctor wonders: does this patient have heart disease?
The test that gives the real answer is called **angiography**. The doctor puts a catheter into the arteries of the heart and looks at them with an X-ray. It is very reliable, but it is invasive, it is expensive, and it carries a small risk.

So the doctor has a real decision to make: send the patient for angiography, or not.

**[Slide 3: Prediction, decision, metric]**

In this course, we learned to separate three things, and we built our whole project around them.

- First, the **prediction**: what is the probability that this patient is sick?
- Second, the **decision**: do we send the patient for angiography or not?
- Third, how we **measure** success. For the prediction, we use accuracy, and also recall: the share of sick patients we catch. For the decision, we use the **average cost per patient**.

These are not the same thing. Two patients can have the same probability of disease, but the right action depends on how costly each mistake is. Keep this in mind: it will come back in the second half.

**[Slide 4: The data]**

Our data comes from the **UCI Machine Learning Repository**: the Heart Disease dataset, collected at the Cleveland Clinic.

- It has **303 patients** and **13 inputs**: age, sex, type of chest pain, blood pressure, cholesterol, and the results of an exercise ECG, among others.
- The original diagnosis goes from 0 to 4. We turned it into yes or no: sick or not sick.
- **46%** of patients are sick, so the two classes are almost balanced.
- Only **6 patients** had a missing value. We removed them, which leaves **297 patients**.

> **Why remove them, and not fill them in?** It is only 2% of the data. And removing rows does not learn anything from the data, so it cannot leak information from the test set.

One important detail about this data: the diagnosis comes from angiography itself. So **every patient in this dataset was already sent for angiography** by a doctor. We will come back to what this means at the end.

*[Transition]* Now that we know the question and the data, Walid will explain how we built our model.

---

## Part 2: Method (Speaker 2, about 3:45)

**[Slide 5: Why a decision tree?]**

Thank you. We chose **option 1 of the brief: a decision-tree classifier**.

> **Why a tree, and not another model?** Because a doctor can read it. A tree is just a list of yes-or-no questions, like a medical checklist. It also works directly with our mix of numbers and categories, with no scaling. And each leaf of the tree gives a probability, which we need for the decision later.

**[Slide 6: Preparing the data and the baseline]**

Before training, we did three things.

First, **preprocessing**. Four inputs are categories written as numbers, like the type of chest pain. A tree would treat them as ordered numbers, which makes no sense. So we turned each category into its own yes/no column. Nothing is learned from the data in this step, so there is no leakage.

Second, the **split**. We kept **25% of the patients, so 75 patients, as a test set**. We locked it away and used it only once, at the very end. The other **222 patients** are for training. Both parts keep the same share of sick patients.

Third, a **baseline**. The simplest possible model always says "not sick". It is right **53%** of the time. Any useful model must beat this.

**[Slide 7: How the tree chooses its questions]**

How does the tree choose its questions? It uses the **Gini impurity**, from the lecture: G(q) = 2q(1 − q), where q is the share of sick patients in a group. It is zero for a pure group, and 0.5 for a 50/50 mix.

We checked the first question by hand. Among all training patients, Gini is **0.497**. If we split on "is the chest pain asymptomatic?", the weighted Gini of the two groups drops to **0.366**. That drop of **0.131** is the biggest of all possible questions, so it becomes the first question of the tree.

**[Slide 8: How deep should the tree grow?]**

The main choice is the **size of the tree**. A deeper tree fits the training data better, but it can start to memorise noise. That is overfitting.

So we tested **40 settings**: depths from 1 to 10, and a minimum of 1, 5, 10 or 20 patients per leaf. We compared them with **5-fold cross-validation** on the training set.

> **Why cross-validation, and not a separate validation set?** With only 222 patients, a validation set would be very small and noisy. Cross-validation uses every training patient for validation once.

*[Point at the curve]* As you can see, training accuracy keeps going up with depth, but cross-validation accuracy stops improving after depth 3 or 4.

The best setting is **depth 4, with at least 10 patients per leaf**, with a cross-validation accuracy of **82.4%**. The leaf size also matters for us: bigger leaves give more reliable probabilities.

*[Transition]* So we have our tree. Ghali will now show what it does on the test set, and, more importantly, how to turn it into a decision.

---

## Part 3: Results, from prediction to decision (Speaker 3, about 4:00)

**[Slide 9: The final tree]**

Thank you. Here is our final tree: **4 levels and 12 leaves**.

Let me read one path. *[Point]* First question: is the chest pain asymptomatic? Yes. Second question: is at least one vessel visible on fluoroscopy? Yes. In the training data, **55 of the 58 patients** who follow this path were sick.

The three most important inputs are asymptomatic chest pain, the number of vessels on fluoroscopy, and the thallium stress test.

**[Slide 10: Test results]**

Now the test set, used once.

- The baseline gets **53.3%**.
- Our tree gets **77.3%** accuracy: that is **24 points better**.
- It catches **69%** of sick patients, and correctly leaves out **85%** of healthy ones.

But look at the confusion matrix. *[Point]* At the usual threshold of 0.5, the tree still **misses 11 of the 35 sick patients**. For a heart disease, that is a lot. This is where the decision part starts.

**[Slide 11: From probability to decision]**

Accuracy counts every mistake the same. For a doctor, they are not the same.

We write a **loss table**, in units of one angiography:

- Sending a patient costs **1**, sick or not.
- Not sending a healthy patient costs **0**.
- Not sending a sick patient costs **c**: the cost of a missed disease.

Then we use the rule from Lecture 1: choose the action with the lowest expected loss.
If we don't send, the expected cost is c times p. If we send, it is 1.
So we **send the patient when p is greater than 1 over c**.

> **Why not just use 0.5?** Because 0.5 is only right when both mistakes cost the same. If missing a sick patient costs 3 angiographies, we should already send the patient when the risk is above one third.

Note that this threshold comes from the costs, not from the data, so we did not tune it on the test set.

**[Slide 12: Does the tree save cost?]**

The problem is: **we don't know c**. So we tried several values.

*[Point at the plot]* The black dashed line is "send everybody": it always costs 1 per patient. The red line is our tree with the cost rule.

- When a miss is cheap, for example c = 3, the tree is the best policy: **0.85 per patient, against 1 for sending everybody**.
- But the lines cross at about **c = 4.7**. Above that, **sending everybody is cheaper**.

Why? When c is large, the rule sends almost everyone anyway. The few patients it leaves out looked healthy in training, but some of them are actually sick. Each miss costs c, so a few misses are enough to lose.

In plain words: the tree only beats "test everyone" if missing a heart disease costs less than about 5 angiographies.

*[Transition]* These results come from one tree and one test set of 75 patients. So can we trust them? Paul will answer that.

---

## Part 4: Can we trust it? (Speaker 4, about 3:45)

**[Slide 13: Four checks]**

Thank you. We asked ourselves four questions that we expected from you, and we checked each one.

**One: what if we tune for the cost instead of accuracy?** The lecture says to choose the model with the loss of the real decision. So we did it, with c = 3. We get a smaller tree, depth 3, but almost the same cost: 0.88 instead of 0.85. That is about 2 patients. **Same conclusion.**

**Two: does the doctor really know all the inputs when deciding?** Two of our inputs, the number of vessels and the thallium test, come from extra imaging tests. So we removed them. The tree becomes a single question: is the chest pain asymptomatic? It still reaches **75%** accuracy, with the same cost. **The tree stays useful with basic information only.**

> **Why keep the full tree as our main model, then?** The full tree is a bit more accurate and is a real decision tree, which is the goal of the project. The smaller model shows that our conclusion does not depend on the two extra tests.

**[Slide 14: Is our split lucky?]**

**Three: what if the random split had been different?** We repeated the whole procedure on **30 different random splits**. *[Point at the histogram]* On average, the test accuracy is **76%**, between 67% and 88%. Our 77% is a typical result, and the tree **beats the baseline on all 30 splits**.
But the exact tree changes: the depth and even the first question change from one split to another. So the overall message is reliable, but we should not over-read every branch of the tree.

**Four: how precise is a score on 75 patients?** With the Hoeffding bound from Lecture 2, a test score on 75 patients is precise to about **plus or minus 16 points**. So small differences between models are noise. Only big gaps, like the one with the baseline, are real.

**[Slide 15: Our recommendation]**

So, what do we recommend?

The tree is **not safe enough to decide alone** who can skip angiography. For a disease that can kill, missing a patient surely costs more than 5 angiographies, and then **sending everybody is cheaper and safer**.

But the tree is useful as a **triage aid**: to flag the high-risk patients first, while the doctor keeps the final decision. And if a hospital wants to use it as a filter, it must first say clearly how much a missed disease costs.

**[Slide 16: Limitations and conclusion]**

Our work has limits:

- The data is **small and old**: 297 patients, one hospital, in the 1980s, and 68% men.
- **These patients were already selected**: a doctor had already sent all of them for angiography. So the tree learned the risk among worrying patients, not among everyone with chest pain.
- The cost c is **assumed**, not measured. And our table ignores the risk of the angiography itself.

To conclude: a good prediction is not yet a good decision. Our tree predicts well, much better than the baseline. But once we put a cost on each mistake, the best decision may be to **test everyone**. That is exactly the difference between prediction and decision that this course is about.

Thank you for your attention. We are happy to take your questions.

---

## Questions: who answers what

Each speaker takes the questions on their part. Full answers are in `slide_notes.md`.

| Likely question | Speaker | Short answer |
|---|---|---|
| Why are "asymptomatic" patients the most often sick? | 1 | All patients were already sent for angiography. Without chest pain, a patient was only sent if something else looked worrying, so this group is high-risk. |
| Why remove missing values instead of filling them? | 1 | Only 2% of rows, and removing rows learns nothing, so there is no leakage. |
| Why a tree and not LASSO or logistic regression? | 2 | Readable as yes/no questions, no scaling, gives a probability per leaf. Option 1 of the brief. |
| Why cross-validation and not a validation set? | 2 | 222 patients is too few to sacrifice a validation set. |
| Why is the CV score (82%) higher than the test score (77%)? | 2 | We chose the best of 40 settings, so the winning CV score is optimistic (Lecture 2). Over 30 splits the gap is smaller (78% vs 76%). |
| Why tune for accuracy and not for cost? | 2 or 4 | Classes are balanced, so accuracy is a fair first choice. We also tuned for cost: same conclusion. |
| Why the threshold 1/c and not 0.5? | 3 | Expected loss: c·p if not sent, 1 if sent. 0.5 is only right if both mistakes cost the same. |
| At c = 3, the 0.5 threshold is slightly better than 1/3 on the test set. Why? | 3 | 0.84 vs 0.85: one patient out of 75. The 1/c rule is best on average, not on every small sample. |
| Did you tune the threshold on the test set? | 3 | No. It comes from the loss table. The tree is frozen; only c changes. |
| Is the tree stable? | 4 | The performance is (30/30 splits beat the baseline); the exact branches are not. |
| Are all inputs known when the doctor decides? | 4 | Two come from extra tests. Without them: 75%, same cost. |
| So should a hospital use it? | 4 | Not alone. As a triage aid, and only if the hospital states its cost c. |
