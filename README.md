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
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

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

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



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
