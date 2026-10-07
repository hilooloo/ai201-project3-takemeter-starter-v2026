# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1.

The classifier achieves an overall accuracy of at least **0.65** on the held-out test split across all seeds.

**Why this target:**

In a 3-class problem (`analysis`, `hot_take`, `reaction`), a random baseline yields ~33.3%. Achieving at least 0.65 requires the model to perform at nearly double chance, verifying genuine semantic learning from r/datascience posts while allowing headroom for seed-to-seed variance on a 30-post test set.

---

## 2.

The macro F1 score across all three classes is at least **0.62** on the held-out test set across all seeds.

**Why this target:**

Macro F1 weights each class equally regardless of support size. This target ensures the model does not lean excessively on the easiest class (`reaction`) to artificially boost overall accuracy, requiring balanced discriminative capacity across all categories.

---

## 3.

Every individual label achieves an F1 score of at least **0.55** on the held-out test split (`min(F1_analysis, F1_hot_take, F1_reaction) >= 0.55`).

**Why this target:**

Our smallest label has around 45–50 instances total, meaning only ~7 to 8 posts appear in the test set where even 1–2 false negatives cause severe metric drops. Setting a 0.55 floor prevents total classification collapse on the subtle `analysis` vs `hot_take` boundary while remaining realistic under small-sample variance.

---

## 4.

No single class represents more than **50%** (100 posts) of the dataset in `labels.csv`, and the smallest class comprises at least **20%** (at least 40 posts).

**Why this target:**

While the syllabus hard cap is 70%, maintaining each class between 20% and 50% across 200 posts prevents DistilBERT from exploiting naive majority priors and guarantees that stratified test splits retain enough samples per class to evaluate reliably.

---

## 5.

The prediction accuracy of the most confident third of test predictions is at least **0.15 higher** than the accuracy of the least confident third (`accuracy_most_confident_third - accuracy_least_confident_third >= 0.15`).

**Why this target:**

The evaluation script in the notebook outputs a calibrated confidence report using softmax output probabilities. Setting a 15-percentage-point differential verifies that when the classifier assigns high confidence to a post, it is substantially more reliable than when its output distribution is uncertain.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
           0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->