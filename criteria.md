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

1. Overall accuracy

Criterion: Overall accuracy must be at least 0.65 on the held-out test set for each of the three random seeds.

**Why this target:**
It sets a useful standard above random guessing without assuming the model will classify every post correctly.

How to check it: For each seed, divide the number of correct predictions by the total number of predictions on the held-out test set. The criterion is MET only if accuracy is at least 0.65 for all three seeds.

How to check it: For each seed, divide the number of correct predictions by the total number of predictions on the held-out test set. The criterion is MET only if accuracy is at least 0.65 for all three seeds.

---

## 2.

<!-- Your criterion. -->
Criterion: The F1 score for each label (analysis, personal_response, and brief_reference) must be at least 0.60 on the held-out test set for each of the three random seeds.



**Why this target:**

I chose this target because the model needs to recognize all three labels, but the smallest label may have fewer examples and a less stable score.

How to check it: Calculate the F1 score separately for each label on the held-out test set for all three seeds. The criterion is MET only if every label achieves an F1 score of at least 0.60 for every seed.

> **Revised in unit 6:** The F1 score for each label — `analysis`, `hot_take`
> and `reaction` — must be at least 0.60 on the held-out test set for each of
> the three random seeds.
>
> **Why revised:** `personal_response` and `brief_reference` were real labels
> when I filed this criterion — they were my taxonomy at the time, and the
> first committed version of `labels.csv` used them. Later in unit 5 I reworked
> the taxonomy to `analysis` / `hot_take` / `reaction` and re-labelled all 216
> posts, and I never came back to update this file. Both of those commits are
> from 18 Oct, before anything was trained; the taxonomy has not moved in
> unit 6.
>
> That leaves the criterion naming two labels that no longer exist, so there is
> no F1 to compute for them and nothing to check. It is unmeasurable as filed,
> not merely unmet — which is the case this file says I may revise.
>
> Only the label names change. The 0.60 threshold, the "every label, every
> seed" rule and the method are exactly as filed, and the verdict is unchanged:
> **MISSED**, because `reaction` scored 0.000 on all three seeds.

---

## 3.

<!-- Your criterion. -->
Criterion: No single label (analysis, personal_response, or brief_reference) may account for more than 50% of the full labeled dataset.

**Why this target:**
Why this target: It helps prevent the dataset from being dominated by one label, while allowing some differences in the number of examples for each label.

How to check it: Count the examples assigned to each label, divide each count by the total number of labeled examples, and convert each result to a percentage. The criterion is MET only if every label accounts for 50% or less of the full dataset.

> **Revised in unit 6:** No single label — `analysis`, `hot_take` or
> `reaction` — may account for more than 50% of the full labeled dataset.
>
> **Why revised:** The same defect as criterion 2, from the same cause — the
> criterion was filed under my original taxonomy and never updated when I
> re-labelled the dataset later in unit 5. Two of the three labels named are no
> longer in `labels.csv`, so their share of the dataset is not a number that
> can be computed, and the criterion could not be checked as written.
>
> Only the label names change. The 50% threshold and the method are as filed
> in unit 5, and the verdict is unchanged: **MET** — `hot_take` 101/216
> (46.8%), `analysis` 73/216 (33.8%), `reaction` 42/216 (19.4%).



---

## 4.

<!-- Your criterion. -->
Criterion: The difference between the highest and lowest overall accuracy scores across the three random seeds must be no greater than 0.10 on the held-out test set.



**Why this target:**
I chose this target because the model should perform consistently across different training runs, while allowing some variation caused by random initialization and data splitting.

How to check it: Record the overall accuracy for each of the three random seeds. Subtract the lowest accuracy from the highest accuracy. The criterion is MET if the difference is 0.10 or less.


---

## 5.

<!-- Your criterion. -->
Criterion: At least 70% of the model's predictions on the held-out test set must have a maximum predicted class probability of 0.70 or higher for each of the three random seeds.

**Why this target:**
 I chose this target because the model should be reasonably confident when making predictions, but it should not treat every prediction as certain.

How to check it: For each seed, calculate the predicted probability for each of the three labels and identify the highest probability for each test example. Count the predictions whose highest probability is at least 0.70, then divide by the total number of test examples. The criterion is MET only if at least 70% of predictions meet the confidence threshold for all three seeds.


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
