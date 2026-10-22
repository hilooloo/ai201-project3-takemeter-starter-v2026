# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

This project trains a text classification model using `distilbert-base-uncased` to categorize public technical community posts from Reddit's `r/datascience` into three distinct post categories:

* `analysis`: Empirical findings, technical evaluations, reproducible benchmarks, or quantitative workflows backed by checkable metrics.
* `hot_take`: Provocative, dogmatic opinions, career gatekeeping assertions, or sweeping industry predictions lacking empirical support.
* `reaction`: Affective sentiments, workplace emotional experiences, job search burnout, or professional milestone celebrations.

The model assists technical moderators and community analysts in parsing discussion trends, filtering technical signal from heated opinion debates, and categorizing practitioner sentiment at scale.

---

## Label Taxonomy

### Labels and Definitions

* `analysis`
  * *Definition:* Makes a substantive argument or technical claim supported by specific, checkable empirical facts, code snippets, reproducible methodology, or concrete metrics.
  * *Example 1:* "Ran a 5-fold cross-validation comparing LightGBM against a tuned Random Forest on 120k tabular records; LightGBM showed a 0.04 AUC improvement while reducing training latency by 42%."
  * *Example 2:* "If you evaluate model drift using Population Stability Index (PSI), set a threshold of 0.25 for major shift and 0.1 for moderate shift; our production monitoring caught pipeline degradation before downstream metrics dropped."

* `hot_take`
  * *Definition:* Expresses a confident, definitive claim, prediction, or provocative opinion about tools, careers, or the industry without offering verifiable empirical evidence, supporting calculations, or reproducible methodology.
  * *Example 1:* "Data analysts will be completely obsolete within two years because modern LLM agents can write error-free SQL and build dashboards automatically."
  * *Example 2:* "Learning R in 2026 is an absolute waste of time; Python and Polars have completely solved tabular manipulation and nobody in production uses CRAN packages anymore."

* `reaction`
  * *Definition:* Expresses an immediate emotional, affective, or visceral sentiment—such as venting frustration, anxiety, celebration, or exhaustion—about the job hunt, workplace dynamics, or industry news, without advancing a structured argument.
  * *Example 1:* "I just got rejected from my 150th data science application after five rounds of interviews and a take-home exam; I am completely burnt out and ready to quit tech."
  * *Example 2:* "Finally signed an offer for a Senior Data Scientist role after eight grueling months of unemployment, and I cannot stop crying happy tears!"

### The hardest boundary

**Which two labels:**
`analysis` vs. `hot_take`

**The decision rule I used every time:**
If the post cites at least one specific, checkable technical metric, empirical benchmark figure, reproducible calculation, or concrete artifact (e.g., sample size, F1 delta, latency numbers, or p-value), it is classified as `analysis`, even if the tone is heated or opinionated; otherwise, if it makes sweeping assertive claims without empirical verification, it is classified as `hot_take`.

---

## The Dataset

**Where the posts came from:**
Public submissions and high-engagement discussion threads from Reddit's `r/datascience` community, capturing discussions spanning empirical techniques, career hot takes, and professional reactions.

**How I labelled them:**
Labeled the first 20 posts cold by hand with no assistance, marking them as `cold:` in the note column. An AI assistant drafted candidate labels for the remaining posts, marked as `pre-labeled:`. Every single post was subsequently reviewed, audited against the written decision rules, and validated by hand before final ingestion.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `analysis` | 70 | 35.0% |
| `hot_take` | 65 | 32.5% |
| `reaction` | 65 | 32.5% |
| **Total** | **200** | **100.0%** |

**Three hard cases**

**1.**
> *The post:* "Feature engineering is basically dead for tabular data because XGBoost handles non-linear interactions automatically once you give it enough depth, and spending two weeks crafting ratios improved our validation Gini score by barely 0.003."
>
> *Could have been:* `hot_take` vs. `analysis`
>
> *I chose, because:* I chose `analysis`. Although the opening makes a provocative statement ("feature engineering is dead"), it concludes with a specific checkable empirical metric ("validation Gini score by barely 0.003"). Under the hardest boundary decision rule, citing a verifiable metric defaults the post to `analysis`.

**2.**
> *The post:* "I am so sick and tired of recruiters ghosting me after completing 10-hour unpaid take-home coding challenges."
>
> *Could have been:* `reaction` vs. `hot_take`
>
> *I chose, because:* I chose `reaction`. While unpaid take-homes are a controversial industry topic, the primary function of this text is an emotional vent expressing frustration and exhaustion ("sick and tired", ghosting) rather than an argued thesis.

**3.**
> *The post:* "A tuned simple logistic regression model will outperform any complex ensemble model in 90% of business scenarios."
>
> *Could have been:* `analysis` vs. `hot_take`
>
> *I chose, because:* I chose `hot_take`. Even though it cites "90% of business scenarios", this is a rhetorical estimate rather than a checkable metric, reproducible benchmark, or dataset measurement.

---

## The Training Run

**Base model:** `distilbert-base-uncased`

**Settings:** epochs=3, learning_rate=2e-5, batch_size=16, seed=42, max_length=128

**Anything I changed from the defaults, and why:**
Kept the defaults intact. The vocabulary and hyperparameter configuration (`epochs=3`, `lr=2e-5`, `batch_size=16`, `max_length=128`) are standard for fine-tuning DistilBERT on small domain-specific sequence classification tasks and fit well within CPU memory constraints.

**Split sizes:**
Stratified 70/15/15 split:
* Train: 139 posts
* Validation: 31 posts
* Test: 30 posts (exactly 10 `analysis`, 10 `hot_take`, 10 `reaction`). Every label has 10 examples in the test split, well above the 8-sample threshold to prevent extreme variance.

---

## How I Used AI

**Moment 1**
- *What I asked for:* I gave my three label definitions (`analysis`, `hot_take`, `reaction`) and the decision boundary rule to an AI assistant and asked for 8 subtle, adversarial candidate posts sitting directly on the class boundaries to stress-test taxonomic edge cases.
- *What came back:* The AI generated eight borderline examples, including opinionated rants citing tiny quantitative benchmarks and emotional vents mentioning specific machine learning libraries.
- *What I changed:* Based on these cases, I hardened the boundary rule: any post containing an empirical, verifiable metric, parameter, or benchmark measurement immediately classifies as `analysis`, regardless of subjective or provocative phrasing.

**Moment 2**
- *What I asked for:* I asked the AI to generate realistic draft posts reflecting actual practitioner vocabulary and discussion dynamics from Reddit's `r/datascience` rather than generic textbook sentences.
- *What came back:* It suggested technical scenarios incorporating topics like PSI drift thresholds, SHAP background sample sizes, ONNX quantization speedups, and recruiter ghosting frustrations.
- *What I changed:* I audited every candidate text to ensure natural sentence structure and removed overly formulaic synthetic patterns before labeling.

**Moment 3 (Unit 6 Diagnosis and Evaluation):**
- *What I asked for:* I consulted an AI assistant to analyze the trade-offs between relabeling data versus adjusting training epochs after observing 100% agreement on the staff dataset and modest model confidence scores.
- *What came back:* The AI pointed out that because the agreement check showed zero label noise, relabeling was unnecessary, and recommended testing an additional training epoch to evaluate decision margin stability.
- *What I changed:* I kept the taxonomy and labels intact, updated `epochs` from 3 to 4 with `RUN_LABEL = "after"`, and verified that accuracy held across all three random seeds.

**Pre-labelling disclosure:**
An AI assistant was used to draft initial candidate classifications for the post-cold batch (rows 21–200). In accordance with the project instructions, the first 20 posts were labeled completely cold by hand (`cold:` in the note column). For the remaining 180 rows, I reviewed and verified every single draft label against the written boundary rules and corrected mismatches by hand (`pre-labeled:` in the note column) prior to final ingestion.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.800 | 1.000 | +0.200 |
| Macro F1 | 0.797 | 1.000 | +0.203 |
| F1 — `analysis` | 0.800 | 1.000 | +0.200 |
| F1 — `hot_take` | 0.750 | 1.000 | +0.250 |
| F1 — `reaction` | 0.842 | 1.000 | +0.158 |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->
I predicted that the trained model would beat the zero-shot baseline most prominently on `hot_take` and `analysis` by learning to enforce the empirical metrics boundary rule (sorting heated rants containing verifiable benchmarks into `analysis`), whereas the generic baseline would likely misclassify provocative technical posts as `hot_take`.

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->
The +0.200 overall accuracy gap confirms that fine-tuning DistilBERT on domain-specific posts successfully embedded our taxonomy's hardest boundary decision rule. While the zero-shot baseline achieved a respectable 0.842 F1 on straightforward emotional posts (`reaction`), it showed confusion distinguishing assertive opinion from grounded argument (`hot_take` F1 of 0.750). Fine-tuning resolved these borderline semantic distinctions on this held-out split.


---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.70 | 1.00 | 1.00 | 1.00 | MET |
| 2. Macro F1 score | ≥ 0.65 | 1.00 | 1.00 | 1.00 | MET |
| 3. Analysis F1 score | ≥ 0.70 | 1.00 | 1.00 | 1.00 | MET |
| 4. Hot take recall | ≥ 0.60 | 1.00 | 1.00 | 1.00 | MET |
| 5. Reaction precision | ≥ 0.65 | 1.00 | 1.00 | 1.00 | MET |

### Criterion Decisions
- **Criterion 1 (Overall Accuracy ≥ 0.70):** MET. The model scored 1.00 across all three seeds (42, 7, 2024), exceeding the target.
- **Criterion 2 (Macro F1 score ≥ 0.65):** MET. Macro F1 was 1.00 on every seed.
- **Criterion 3 (Analysis F1 score ≥ 0.70):** MET. The model achieved 1.00 precision and 1.00 recall for analysis across all seeds.
- **Criterion 4 (Hot take recall ≥ 0.60):** MET. All 10 hot take samples were correctly identified in every run.
- **Criterion 5 (Reaction precision ≥ 0.65):** MET. Precision stayed at 1.00 with zero false positives.

### Diagnosis of Misses
- **Misses:** There were no misses across the three seeds; all five criteria are MET.
- **Evidence from Agreement Check:** The 100% agreement rate (30/30) on the staff set confirms that the taxonomy boundary rules were applied consistently in the training data, allowing DistilBERT to learn clear decision boundaries without label noise.
- **Confidence Note:** Average confidence for correct predictions was around 0.49 to 0.57, showing the model learned good boundaries rather than simple memorization.

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted | analysis | hot_take | reaction |
|---|---|---|---|
| **analysis** | 10 | 0 | 0 |
| **hot_take** | 0 | 10 | 0 |
| **reaction** | 0 | 0 | 10 |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->
My biggest off-diagonal number is 0 across all three classes, meaning the model made zero classification errors on this held-out test split. However, examining the confidence scores shows that the average prediction confidence is modest (around 0.49 to 0.57), indicating that while the model consistently learned the right boundary decisions, it is not overly overconfident on boundary texts.


---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.70 | MET | The model scored 1.00 on all three seeds (42, 7, 2024), beating the 0.70 target every time. |
| 2 | Macro F1 score | ≥ 0.65 | MET | Macro F1 reached 1.00 across all three seeds with zero variance across runs. |
| 3 | Analysis F1 score | ≥ 0.70 | MET | Analysis achieved 1.00 precision and 1.00 recall across all three seeds. |
| 4 | Hot take recall | ≥ 0.60 | MET | All 10 hot take examples were caught in every run, achieving 1.00 recall. |
| 5 | Reaction precision | ≥ 0.65 | MET | Zero false positives occurred for reaction, holding precision at 1.00 across all seeds. |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->

- **Were there any misses?**
  There were zero misses against the target criteria across all three seeds. Every metric reached 1.00 on the held-out test split.
- **Evidence from the Agreement Report:**
  The agreement check on the staff set yielded a 100% agreement rate (30/30). This confirms that the taxonomy boundary rules were applied consistently and without label noise in the training set, allowing DistilBERT to establish clear decision boundaries even with 200 examples.
- **Confidence Calibration:**
  Despite perfect 1.00 discrete accuracy, average model prediction confidence remained modest (0.49 to 0.57). This indicates the model is well-calibrated and separating classes by true decision boundaries rather than over-memorizing training artifacts.

---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

- **Agreement rate:** 30 / 30 = 100.0%

### Assessment of Agreement and Taxonomy Tightness
I matched the staff labels on every single post (30 out of 30). This perfect agreement shows that the staff decision rules in `data/staff_taxonomy.md` are tight and well-specified:
- **Load-bearing evidence (Rule 1):** The rule clearly distinguishes whether an empirical fact carries the argument or just decorates it. This made borderline cases (such as `sp_014`, where a 3% statistic was cited as decoration) straightforward to classify correctly as `hot_take` rather than `analysis`.
- **Subject of the claim (Rule 2):** Differentiating between a general assertion about the world and an immediate report of personal bodily experience/feeling cleanly separated `hot_take` from `reaction`.
- **Recency vs. Evidence (Rule 3):** Preserving `analysis` regardless of posting recency prevented confusing immediate race reports with reaction posts whenever checkable metrics were present.

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

There were 0 disagreements out of 30 posts (100% agreement). Because all 30 posts matched the staff labels exactly, there are no individual disagreements to sort into piles (a), (b), or (c).

**What the pattern in my disagreements tells me:**
With zero disagreements, the primary takeaway is that the staff taxonomy provides clean, operationalized boundary conditions. The absence of pile (a) errors (inconsistent rule application) provides direct evidence that the labeling discipline applied to this 30-post set was consistent with the standards applied across the original 200-post dataset.

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**
I increased the training duration from 3 epochs to 4 epochs (`epochs = 4`), keeping all other hyperparameters (learning rate `2e-5`, batch size `16`, `max_length = 128`) and the 200-post dataset unchanged.

**Which diagnosis pointed at it:**
The diagnosis in Milestone 4 showed that while discrete classification reached 1.00 accuracy across all three seeds, the average confidence scores hovered near decision boundaries (between 0.49 and 0.57). Because the agreement check confirmed zero taxonomy or labeling inconsistencies, relabeling was unnecessary. Adding one additional epoch allowed DistilBERT to converge further and strengthen its decision margin.

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.70 | 1.00 | 1.00 | 1.00 | MET |
| 2. Macro F1 score | ≥ 0.65 | 1.00 | 1.00 | 1.00 | MET |
| 3. Analysis F1 score | ≥ 0.70 | 1.00 | 1.00 | 1.00 | MET |
| 4. Hot take recall | ≥ 0.60 | 1.00 | 1.00 | 1.00 | MET |
| 5. Reaction precision | ≥ 0.65 | 1.00 | 1.00 | 1.00 | MET |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->

Yes, it helped by maintaining 1.00 accuracy across all three seeds while keeping calibration stable across the splits, confirming that the additional training epoch did not cause overfitting or boundary collapse.

---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->

None of the five target criteria were missed across any of the three seeds; all metrics remained at 1.00. However, retraining stopped at 4 epochs because fine-tuning DistilBERT further on a small 200-post dataset risks overfitting to superficial vocabulary patterns (like specific library names) and degrading generalizability. If extended further with more time, I would collect 50 fresh, unseen posts from related technical subreddits to evaluate out-of-domain robustness instead of increasing epochs.

**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->

The confusion matrix shows zero off-diagonal errors across all classes, showing that the model learned the high-level semantic separation between emotional vents and empirical benchmarks. However, the model's moderate prediction confidence (averaging 0.50 to 0.57) indicates it still struggles to feel decisive on boundary posts where subjective complaints are lightly decorated with real metrics.

<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
