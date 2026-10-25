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

<!-- Your community, and what your classifier sorts posts into. Three or four
     sentences. -->

TakeMeter is a text classification project that sorts public Reddit posts and comments from r/hiphopheads into three categories: analysis, hot_take, and reaction. The goal is to see whether a language model can learn the differences between posts that make fact-based arguments, confident unsupported claims, and immediate reactions. I created a dataset of 216 labeled posts and fine-tuned DistilBERT to classify them. I evaluated the trained model on a held-out test set to see which categories it learned and where it struggled.



---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `analysis`

**Definition:** The post gives a reason for its claim — a why, a comparison, or
a piece of history that supports it. The reasoning is in the post, not assumed.

**Example 1:**
> They just didn't have it. They weren't that successful as a group in the
> first place and the two things they did have going for them were Busta and
> their connection to ATCQ. So when they didn't have Busta anymore nobody was
> going to be tuning in to see what Charlie Brown does with his career.

**Example 2:**
> Apparently a lot of the tension in LotNS was between Busta and Charlie Brown,
> both of whom were convinced they were the star. Fairly apparent there was a
> clear winner

### `hot_take`

**Definition:** A claim, answer or assertion about the music or the artists,
stated flat with no reasoning attached. It may well be true, and it may well be
a fact — it just isn't argued for.

**Example 1:**
> Busta Rhymes

**Example 2:**
> DJ Subroc (Doom's brother) made some amazing beats

### `reaction`

**Definition:** The writer responding to the thing itself rather than making a
claim about it — surprise, amusement, or a memory of encountering it. It is
about the writer's own experience.

**Example 1:**
> Mans got kicked out mid interview. That was crazy lmao

**Example 2:**
> I've got an old graffiti mag that had a KMD ad in it. Blew my mind when I went
> back and rediscovered it.

### The hardest boundary

**Which two labels:** `analysis` and `hot_take`.

All three of my hard cases below sit on this boundary. A thread like this one is
full of posts that state something true about an artist without ever arguing
from it, and "it contains a fact" is not enough to separate them.

**The decision rule I used every time:**
If the post gives a *reason* for its claim — a why, a comparison, or a piece of
history that the claim rests on — it is `analysis`. If it only states the claim,
even a checkable factual one, it is `hot_take`. A bare fact with nothing built
on it is a `hot_take`, not analysis.

**Second rule, for `reaction` vs `hot_take`:**
If the post is about the writer's own experience of the thing — watching it,
hearing it, being surprised by it — it is `reaction`. If it is a claim about the
artist or the music, it is `hot_take`, however heated it sounds.



---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:**
I collected public posts and comments from the Reddit community r/hiphopheads. The dataset is stored in one CSV file named labels.csv, with the columns text, label, and note.

**How I labelled them:** <!-- Cold first? Pre-labelled with AI and corrected?
Say so plainly — the disclosure is required, not penalised. -->

I used three labels: `analysis`, `hot_take`, and `reaction`. I labelled the
first 20 posts independently, without AI assistance, and marked them `cold` in
the note column. I used AI assistance to draft labels for the remaining 196 and
marked those rows `not cold`. I read and corrected every AI-drafted label before
finalising the dataset.

**The taxonomy changed once, part-way through unit 5.** My first labelling pass
used `analysis` / `personal_response` / `brief_reference` (92 / 38 / 86), and
that version is in the repo at commit `48a391a`. I then reworked the taxonomy
to `analysis` / `hot_take` / `reaction` and re-labelled the same 216 posts at
commit `545fc49`. Both commits are from 18 October, before anything was
trained, and the taxonomy has not moved since.

It was not a rename. Against the obvious mapping — `brief_reference` to
`hot_take`, `personal_response` to `reaction`, `analysis` unchanged — **59 of
the 216 posts came back in a different class, 27.3% of the dataset.** The
largest flows were 23 posts from `analysis` to `hot_take` and 10 from
`analysis` to `reaction`. I am recording this because it is the most direct
measurement I have of my own labelling stability, and it is not a flattering
one. It is also why `criteria.md` criteria 2 and 3 still named the old labels
until unit 6.

**Counts per label:**

| Label | Count | Share |
|---|---:|---:|
| `analysis` | 73 | 33.8% |
| `hot_take` | 101 | 46.8% |
| `reaction` | 42 | 19.4% |
| **Total** | **216** | **100%** |

**Three hard cases**

<!-- Any post that made you pause: what it was, which two labels it could have
     been, and what you chose. These are worth more than the easy 190. -->

**1.**
> *The post:*
> "He was a dancer for them primarily though"
> *Could have been:*
> `hot_take` or `analysis`
> *I chose, because:*
> I labelled it `analysis`, and this is the one I am least sure of. It corrects
> what Tupac's role actually was, so I read the correction as doing the work of
> an argument. My own decision rule says a bare fact with nothing built on it is
> a `hot_take`, and by that rule this should have been `hot_take`. I am leaving
> the label as it is in `labels.csv` and flagging the tension instead, because
> it is direct evidence about how consistently I applied my own rule — which is
> what the diagnoses below are about.

**2.**
> *The post:*
> "He was never a member of digital underground really, but he was in a band
> called strictly dope before that"
> *Could have been:*
> `analysis` or `hot_take`
> *I chose, because:*
> I labelled it `analysis`. It corrects an earlier claim and then backs the
> correction with a further detail — the earlier band — so the fact is carrying
> the correction rather than sitting next to it.

**3.**
> *The post:*
> "I didn't like any Odd Future music, but I love all of their solo work."
> *Could have been:*
> `hot_take` or `reaction`
> *I chose, because:*
> I labelled it `reaction`. It is a claim about the music, which pushes toward
> `hot_take`, but every word of it is about what the writer personally likes
> rather than about the group, so my `reaction` vs `hot_take` rule sends it to
> `reaction`.
---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** distilbert-base-uncased

**Settings:** <!-- epochs, learning rate, batch size, seed -->
3 epochs, learning rate 2e-5, batch size 16, maximum sequence length 128 tokens, seed 42.

**Anything I changed from the defaults, and why:**
I kept the default training settings so I could establish a baseline before experimenting with changes in the next unit.

**Split sizes:** <!-- train / val / test, and per-label counts in the test
split. If a label had fewer than about 8 in test, say so — it explains a lot
of next unit's variance. -->

150 training posts, 33 validation posts, and 33 test posts.

Test examples by label:

analysis: 11

hot_take: 16

reaction: 6

Device: Apple Silicon GPU (MPS).

Test results: Accuracy was 0.636 (63.6%), and macro F1 was 0.462. F1 was 0.667 for analysis, 0.718 for hot_take, and 0.000 for reaction. The reaction test set had only six examples, which limits how confidently I can interpret that score.


---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1**

- *What I asked for:* Help proposing labels for Reddit posts using my three-category taxonomy.
- *What came back:* AI-assisted label suggestions for posts after my initial set of independently labeled examples.
- *What I changed:* I used the project’s final labels—analysis, hot_take, and reaction—and documented difficult examples where the distinction between categories required judgment.

**Moment 2**

- *What I asked for:* Help understanding the notebook’s training output, warnings, and evaluation metrics.
- *What came back:*  An explanation of the PyTorch and progress-bar warnings, along with an interpretation of accuracy, macro F1, and the per-label scores.
- *What I changed:* I kept the default training settings and recorded the weak reaction result and the small number of reaction examples in the test set as limitations.

**Moment 3 — unit 6, reading the error pattern**

- *What I asked for:* I gave it my confusion matrix and the three-seed results
  and asked what the pattern actually was, rather than whether the numbers were
  good.
- *What came back:* The observation that the whole `reaction` **column** is
  zero, not just the diagonal cell — the model never predicts that label at
  all, on any of the 99 held-out predictions. Then a breakdown showing that the
  6 `reaction` posts account for half of all errors on two of the three seeds,
  that the ceiling while the label stays inert is 0.818, and that 50%
  `reaction` recall alone would clear my accuracy target.
- *What I changed:* My diagnosis for criterion 2 had said the model "struggled
  to distinguish `reaction`", which is a restatement of the score. I rewrote it
  around the fact that the label is never emitted, and reframed criterion 1 as
  downstream of criterion 2 rather than as a separate failure.

**Moment 4 — unit 6, when the agreement check came back empty**

- *What I asked for:* My staff agreement came out 30/30, so it gave me no
  disagreements to diagnose from. I asked what else in the repo could speak to
  whether my own labelling was consistent.
- *What came back:* The suggestion to diff the committed versions of
  `labels.csv` against each other. That surfaced something I had not thought of
  as evidence: I labelled the same 216 posts twice in unit 5 under two versions
  of my taxonomy, so I already had a self-agreement measurement sitting in my
  git history. It comes out at 59 posts changing class, 27.3%, with `reaction`
  the least stable label at 38.1% moved in from elsewhere.
- *What I changed:* The criterion 2 diagnosis now rests on that number instead
  of on the uninformative 30/30, and I added the taxonomy change to **The
  Dataset** — it was a material fact about my data that I had not disclosed.

**Scope of AI assistance in unit 6**

Being plain about this, since the sections above are specific moments and the
use was broader than four moments. I used AI assistance across this unit to
audit the repo against the brief, to add the confidence measurement to
`confidence_report` in the notebook, to implement the weighted-loss trainer, to
check my reported numbers against the JSON files, and to draft and edit much of
the prose in this README. The runs, the data and the results are my own and
were produced on my machine; the analysis and the writing were done with
substantial AI help.

**Pre-labelling disclosure:**

Yes, I pre-labelled. I labelled the first 20 posts independently, with no AI
assistance, and marked them `cold` in the note column. For the remaining 196 I
had a model propose a label, then read and corrected each one myself. Those
rows are marked `not cold`, so the two groups can be told apart in
`labels.csv`. I am disclosing this rather than presenting all 216 as
independent human labelling.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

Both columns are scored on the same 33 held-out posts — `baseline.py` read
`test_split.csv`, so this is like-for-like. The baseline is
`facebook/bart-large-mnli`, zero-shot, prompted with the definitions in
`label_definitions.txt`.

| Measure | Baseline | Trained | Difference |
|---|---:|---:|---:|
| Overall accuracy | 0.333 | 0.636 | +0.303 |
| Macro F1 | 0.167 | 0.462 | +0.295 |
| F1 — `analysis` | 0.500 | 0.667 | +0.167 |
| F1 — `hot_take` | 0.000 | 0.718 | +0.718 |
| F1 — `reaction` | 0.000 | 0.000 | +0.000 |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->
I predict that the trained model will perform better overall than the baseline, particularly on the analysis and hot_take labels, while reaction may remain difficult to classify.

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->
My prediction was half right. Fine-tuning did beat the baseline on `analysis`
and `hot_take`, and by more than I expected: accuracy went from 0.333 to 0.636
and macro F1 from 0.167 to 0.462. But I predicted `reaction` would stay hard
for both, and it did — both models score 0.000 on it.

The baseline number is worse than it looks, and that matters more than the gap.
The zero-shot model predicted `analysis` for **all 33 posts**. It did not get
0.333 by being a third right across the board; it got it by guessing one label
every time and being handed the 11 posts where that happened to be correct.
0.333 on three labels is chance. So the fair reading is not "fine-tuning added
0.303" but "without training there is no classifier here at all, and with
training there is one that works on two labels out of three."

That makes the shared `reaction` failure the interesting result. If only my
trained model missed `reaction` I would suspect the training run. Both models
missing it, with my model sending 5 of 6 `reaction` posts to `hot_take`, points
instead at the boundary itself — or at how I drew it. The agreement check is
the instrument that can tell those apart, and I come back to it in the
diagnoses below.

The baseline is also far slower. It took minutes to get through 33 posts,
because the zero-shot model runs a separate entailment pass for every
label on every post, where the fine-tuned DistilBERT is one small forward pass
per post. That is the second thing skipping fine-tuning costs you, and it does
not show up anywhere in the accuracy table.

**What my definitions were worth (measured, not assumed)**

I ran the baseline twice: once with `label_definitions.txt`, once with the file
absent so it falls back to bare label names.

| Baseline run | Accuracy | Macro F1 | What it predicted |
|---|---:|---:|---|
| With my definitions | 0.333 | 0.167 | `analysis` ×33 |
| Bare label names | 0.394 | 0.303 | `hot_take` ×16, `reaction` ×17 |

My definitions made the baseline **worse** by 0.061 accuracy, which is the
opposite of what the milestone led me to expect. The cause is visible in the
prediction counts: my `analysis` definition — "gives a reason for its claim" —
is the broadest sentence of the three, and dropped into the entailment template
*"This post is …"* it beat the other two on every single post. The bare-names
run at least spread its guesses around, which is why its macro F1 is higher
even though neither run is usable.

The number I report as my baseline is the **0.333 definitions run**, because
that is the one the milestone specifies and it is the prompt I actually wrote.
Both files are committed: `baseline_results.json` and
`baseline_results_bare_names.json`.



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

Three seeds — 42, 7 and 2024 — each a fresh stratified 70/15/15 split and a
fresh fine-tune, run in one cell by section 6 of the notebook. Trained on an
Apple Silicon GPU (MPS), torch 2.14.1. Raw numbers in
`results_three_seeds_before.json`.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.65 | 0.636 | 0.636 | 0.667 | MISSED |
| 2. Every label F1 | ≥ 0.60 for every label | analysis: 0.667; hot_take: 0.718; reaction: 0.000 | analysis: 0.636; hot_take: 0.737; reaction: 0.000 | analysis: 0.700; hot_take: 0.750; reaction: 0.000 | MISSED |
| 3. No label exceeds 50% of the dataset | ≤ 50% per label | analysis: 33.8%; hot_take: 46.8%; reaction: 19.4% | Same dataset | Same dataset | MET |
| 4. Accuracy spread across seeds | ≤ 0.10 | 0.636 | 0.636 | 0.667 | MET (spread: 0.030) |
| 5. Predictions with maximum class probability ≥ 0.70 | ≥ 70% for each seed | 0.0% (0/33) | 0.0% (0/33) | 0.0% (0/33) | MISSED |

Criterion 5 said "not measured" until I went back for it. The notebook's
`confidence_report` recorded how the confident third compared with the unsure
third, which is a relative measure and cannot answer a criterion with an
absolute threshold in it. I added `share_at_or_above_threshold` and
`max_confidence` to that function and re-ran section 6. Accuracy and macro F1
came back identical on all three seeds, so the added fields measure the same
runs the rest of this table describes.

The answer is not close. Not one prediction out of 99 across the three seeds
reached 0.70. The highest top probability the model ever produced was 0.602,
on seed 2024; on seed 42 it was 0.491 and on seed 7 it was 0.505. With three
labels the floor is 0.333, so the model spent the whole test set hovering just
above a coin-flip between two of the three classes.

**The spread table, as section 6 printed it**

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---:|---:|---:|---:|
| Overall accuracy | 0.636 | 0.636 | 0.667 | 0.030 |
| Macro F1 | 0.462 | 0.458 | 0.483 | 0.026 |
| F1 — `analysis` | 0.667 | 0.636 | 0.700 | 0.064 |
| F1 — `hot_take` | 0.718 | 0.737 | 0.750 | 0.032 |
| F1 — `reaction` | 0.000 | 0.000 | 0.000 | 0.000 |
| Share with top prob ≥ 0.70 | 0.000 | 0.000 | 0.000 | 0.000 |
| Highest top prob seen | 0.491 | 0.505 | 0.602 | 0.112 |

The notebook's own verdict on the spread: *"Accuracy moved 0.030 across seeds.
Stable enough to trust."*

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->


Seed 42, the 33 held-out posts.

| true \ predicted | analysis | hot_take | reaction |
|---|---:|---:|---:|
| **analysis** | 7 | 4 | 0 |
| **hot_take** | 2 | 14 | 0 |
| **reaction** | 1 | 5 | 0 |


**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->
The biggest off-diagonal number is 5: five real `reaction` posts were called
`hot_take`, and zero real `hot_take` posts went the other way. That is a
direction, not just an error rate — the model collapses `reaction` into
`hot_take` and never the reverse.

But the column tells the bigger story. **The whole `reaction` column is zero.**
The model did not merely get `reaction` wrong, it never once predicted
`reaction` — not on seed 42, and not on seed 7 or seed 2024 either, where the
`reaction` column is also all zeros. Across 99 held-out predictions it used two
of its three labels. That is why `reaction` F1 is exactly 0.000 rather than
merely low: recall and precision are both undefined-to-zero because the label
was never emitted.

So the boundary the model did not learn is `reaction` vs `hot_take`, in one
direction. Both are short and unsupported; what separates them in my taxonomy
is whether the post is about the writer's own experience or about the music,
and that distinction is carried by tone rather than by vocabulary. `hot_take`
has 101 training examples to `reaction`'s 42, so when the model is unsure the
cheapest guess is the bigger class — and at a top probability of 0.49 it is
unsure on essentially every post.

Whether this is a model problem or a labelling problem is exactly what the
agreement check is for, and I take it up in the diagnoses below.

**On the spread:** accuracy moved 0.030 across the three seeds — 0.636, 0.636,
0.667 — well inside the 0.10 band, so these numbers are stable enough to trust
and criterion 4 is MET. Stability is not quality here, though: what is stable
is a model that is consistently blind to one of its three labels. `reaction`
F1 was 0.000 on all three seeds, and the `reaction` column was empty on all
three. Three seeds confirm the failure is structural rather than one unlucky
draw.




---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.65 for all three seeds | MISSED | Seeds 42 and 7 scored 0.636, below 0.65. Seed 2024 scored 0.667. |
| 2 | F1 for every label | ≥ 0.60 for every label across all three seeds | MISSED | The `reaction` F1 score was 0.000 for all three seeds, below the 0.60 target. |
| 3 | No single label exceeds 50% of the dataset | ≤ 50% per label | MET | `hot_take` = 101/216 (46.8%), `analysis` = 73/216 (33.8%), and `reaction` = 42/216 (19.4%). |
| 4 | Accuracy spread across seeds | ≤ 0.10 | MET | The recorded accuracy spread was 0.030, below the 0.10 target. |
| 5 | Predictions with maximum class probability ≥ 0.70 | ≥ 70% for each seed | MISSED | 0 of 33 predictions cleared 0.70 on every seed — 0.0% against a 70% target. The highest top probability the model produced anywhere was 0.602. |

Three missed, two met. Criteria 2 and 3 carry a **Revised in unit 6** note in
`criteria.md`: both were filed naming `personal_response` and
`brief_reference`, labels I replaced later in unit 5, so neither could be
checked as written. Only the label names changed — the 0.60 and 50% thresholds
are as filed, and neither verdict moves.

### Arguing the opposite

I tried to break both METs before keeping them.

**Criterion 3** holds. `hot_take` is 46.8% against a 50% ceiling, and the best
case against it is that 3.2 points is a thin margin on a dataset I built
myself. That is a fair complaint about the target, not about the verdict.

**Criterion 4 is the one that survives the arithmetic and fails on meaning,
and I am keeping the MET with that written next to it.** The spread is 0.030
against a 0.10 ceiling, so MET is the honest call. But I wrote that criterion
to mean "the model behaves consistently across training runs", and what it
actually certified is that the model fails the same way every time. `reaction`
F1 was 0.000 on all three seeds and the `reaction` column was empty on all
three; a model that never emits one of its labels cannot produce much variance
to begin with. Two further things flatter the number: the split is stratified,
so every seed gets the same per-label test counts (11 / 16 / 6) and one source
of variance is removed by construction, and three seeds is a very small sample
to estimate a spread from. **Criterion 4 is MET, and it is the criterion I
would rewrite if I were filing these again** — stability is only worth
measuring alongside a floor on quality, and mine had none.

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->

**Criterion 1 — Overall accuracy (MISSED: 0.636, 0.636, 0.667 against ≥ 0.65)**

**Cause:** Not an independent failure. This miss is criterion 2 showing up in
the accuracy column, and it is small enough that fixing `reaction` would clear
it.

**How I know:** Splitting the errors by seed:

| Seed | Correct | Errors from the 6 `reaction` posts | Errors from `analysis`/`hot_take` mixups |
|---|---:|---:|---:|
| 42 | 21/33 | 6 | 6 |
| 7 | 21/33 | 6 | 6 |
| 2024 | 22/33 | 6 | 5 |

Every `reaction` post in the test split is wrong on every seed, because the
label is never predicted. That is half of all errors on two of the three
seeds. The model's ceiling while `reaction` stays inert is 27/33 = 0.818, and
it leaves 6 of those 27 on the table on seeds 42 and 7 and 5 on seed 2024 — so
on the two labels it will actually use it runs at 0.778, 0.778 and 0.815,
comfortably above the 0.65 target.
**Lifting `reaction` recall to even 50% puts accuracy at 0.727 on seeds 42 and
7 and 0.758 on seed 2024, clearing the target on all three without the model
getting any better at the other two labels.** So the useful diagnosis is not
"accuracy is low" — it is that one dead label is costing roughly 18 accuracy
points, and criterion 1 is downstream of criterion 2.

**Criterion 2 — F1 for every label (MISSED: `reaction` 0.000 on all three seeds)**

**Cause:** Two of the four, and they compound: **too few examples for the
label**, and **a boundary I applied inconsistently in my own training data.**
The second is the one I can now measure, and it is the one that matters.

**How I know, on the examples:** `reaction` is my smallest class at 42 of 216
posts (19.4%), which leaves about 29 in train and 6 in test. `hot_take` has
101. When the model is unsure, the cheapest guess is the larger class, and the
matrix shows exactly that: 5 of the 6 real `reaction` posts went to `hot_take`
on seed 42 and 0 went the other way. But scarcity alone does not explain
`reaction` F1 being *exactly* 0.000 rather than merely low — the label is never
emitted at all, on any of the 99 held-out predictions across three seeds. A
label that is merely thin usually still gets predicted sometimes.

**How I know, on the labelling — and this is where the agreement report comes
in:** the staff check came back 30/30 with zero disagreements, so taken at face
value it reports no labelling problem. I do not think it is usable that way,
for the reasons set out in the **Agreement Report** below: the staff decision
rules are far more testable than mine, `data/staff_posts.csv` is ordered in
label blocks, and it tests a different taxonomy on different posts. A null
result from an instrument that insensitive is not evidence of consistency.

The instrument that does work is my own repo. I labelled the same 216 posts
twice in unit 5, and `reaction` is the least stable label in the dataset:

| Final label | Total | From its own predecessor | Moved in from elsewhere |
|---|---:|---:|---:|
| `analysis` | 73 | 59 | 14 (19.2%) |
| `hot_take` | 101 | 72 | 29 (28.7%) |
| **`reaction`** | **42** | **26** | **16 (38.1%)** |

**Over a third of my final `reaction` posts are ones I had first called
something else — and 10 of them I had called `analysis`, the opposite end of
the taxonomy.** On my own data, in my own hand, `reaction` is the category I
could least reliably recognise twice. So the honest reading is that `reaction`
is not only small, it is noisily defined: more than a third of the `reaction`
posts the model trains on come from a boundary I had drawn somewhere else days
earlier. A model cannot learn a distinction its training labels do not make
consistently.

It is also the weakest rule in my taxonomy, and the two facts fit together. My
`analysis` rule gives me something to check on the page — is a reason given for
the claim, yes or no. My `reaction` rule asks what the post is *about*, the
writer or the music, and nothing on the surface settles that. A rule with no
observable test is one I can apply two ways without noticing, which is
precisely what the churn table shows me doing.

**Criterion 5 — Confidence (MISSED: 0% of predictions at ≥ 0.70, on all three seeds)**

**Cause:** The fourth one — a task the model cannot reach from this much data
at these settings. This is underfitting, not miscalibration.

**How I know:** A three-class model that outputs a flat distribution has a
cross-entropy loss of ln(3) = 1.099. Here is where three epochs got each run:

| Seed | Train loss, epoch 1 → 3 | Final eval loss | Val accuracy by epoch | Highest top prob |
|---|---|---:|---|---:|
| 42 | 1.079 → 0.983 | 0.989 | 0.576 → 0.545 → 0.545 | 0.491 |
| 7 | 1.053 → 0.937 | 1.002 | 0.485 → 0.455 → 0.455 | 0.505 |
| 2024 | 1.083 → 0.923 | 0.942 | 0.545 → 0.576 → 0.576 | 0.602 |

After all three epochs the loss has moved only about 0.12 to 0.18 below the
flat-distribution value. The model has barely left its starting point, which is
why no prediction anywhere reaches 0.70 and the mean top probability sits
between 0.434 and 0.474 against a floor of 0.333. On seeds 42 and 7 validation accuracy
actually went *down* after epoch 1 and then stopped moving.

The arithmetic says why: 150 training posts at batch size 16 is about 10
optimiser steps per epoch, so the whole fine-tune is **roughly 30 gradient
steps**. That is not enough to move a 66M-parameter model far from
initialisation, and it is the same fact underneath criterion 5 and the
`reaction` failure — an underfit model defaults to the majority class and never
commits to anything.

This one is cleanly separable from the labelling problem. Criterion 2 is about
*which* label the model picks, and the evidence there points at my data.
Criterion 5 is about *how hard* it picks anything at all, and that points at
the training run. They need different fixes, which is what makes the choice in
**The Improvement** below a real choice.

---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

I labelled all 30 posts in `data/staff_posts.csv` under the definitions and the
four decision rules in `data/staff_taxonomy.md` — not my own taxonomy — and
saved them to `my_staff_labels.csv` before asking for the answer key. Raw output
in `agreement_results.json`.

**Agreement rate:** 30 / 30 = 100%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements:** none. There is nothing to adjudicate and nothing to sort
into piles (a), (b) or (c).

`agreement.py` asks, when the rate comes out at 100%, whether the set was easy
or my rule is genuinely tight. The honest answer is neither — and the reasons
matter more than the number, because Milestone 4 wants to use this check as
evidence about my own 216 labels.

**Why 100% is weaker evidence than it looks**

Two properties of the instrument, not of my labelling:

1. **The decision rules are unusually explicit.** Rule 1 gives a test I can
   actually run on a post — is the fact load-bearing, or is it decoration on a
   conclusion already reached? Rule 2 gives another — is this about the writer
   or about the world? These are far tighter than anything I wrote for my own
   taxonomy in unit 5, where my `reaction` vs `hot_take` boundary came down to
   tone. A rule that specific is hard to apply two different ways.

2. **`data/staff_posts.csv` is ordered by label.** `sp_001`–`sp_010` are all
   `analysis`, `sp_011`–`sp_020` are all `hot_take`, `sp_021`–`sp_030` are all
   `reaction`. Labelling in file order means each post sits next to nine others
   of its own class, and consecutive posts corroborate each other. That is a
   meaningfully easier task than the same 30 posts shuffled, and it is nothing
   like labelling 216 posts arriving in thread order. The rate from an ordered
   set and the rate from a shuffled one are not the same measurement.

**The seven posts staff flagged as hard**

Staff marked 7 of the 30 `borderline` in `data/staff_labels.csv`. Those are the
only ones where agreeing says anything, so here is the reasoning on three of
them, written from the staff rules:

- **`sp_014`** — *"Times are down about 3% since the shoe rules changed, so
  obviously the shoes are the whole story and the sport has a problem."* I
  labelled it `hot_take`. It contains a real, checkable number, which is what
  makes it tempting. But rule 1 asks whether the fact carries the argument, and
  here "so obviously" carries it — the 3% is attached to a conclusion the
  writer had already reached, not the reason they reached it. A statistic in a
  post is not the test.

- **`sp_019`** — *"Nobody actually enjoys the long run. They enjoy having done
  it…"* I labelled it `hot_take`. It reads reflective and personal, which pulls
  toward `reaction`, but rule 2 asks whether it is about the writer or about
  the world, and "nobody" is a claim about everyone. Tone is not the test.

- **`sp_028`** — *"Third DNF this year. I don't really want to talk about it,
  just wanted to put it somewhere."* I labelled it `reaction`. "Third this
  year" is countable, so rule 3 is in play — recency does not outrank evidence.
  But rule 3 only rescues a post into `analysis` if some fact is load-bearing,
  and nothing is argued from the count at all. The writer is reporting their
  own state.

What these have in common is that in each one the surface signal — a number, a
reflective tone, a countable detail — points the wrong way, and the written
rule overrides it. That is the thing worth claiming from a 100%: on this set I
followed the rule rather than the surface.

**What the pattern tells me — and what it does not**

There is no pattern of disagreement to read, which is itself the problem. A
100% rate gives me no instance of pile (a) and therefore no direct measurement
of my own inconsistency. It is a null result, not a clean bill of health.

And I should not carry it across to my own data, for three reasons. The staff
set is 30 short, purpose-written posts from a fictional forum; my set is 216
real Reddit comments, some one word long, many of them replies that only make
sense against a parent I did not record. The staff rules were written to be
applied; mine were written after I had already started labelling. And 196 of my
216 were AI-drafted and then corrected by me, where all 30 of these were mine
from scratch.

I also already have direct evidence pointing the other way, from my own repo
rather than from this check, and it is much stronger than anything 30 posts
could have told me. As recorded in **The Dataset** above, I labelled all 216
posts twice in unit 5 under two versions of my taxonomy, and **59 of them —
27.3% — came back in a different class** once the obvious rename is accounted
for. That is a direct, measured disagreement rate between me and myself on my
own data, against 0% disagreement with staff on theirs. The two numbers are not
in tension; they are measuring different things, and the one that bears on my
216 labels is the 27.3%.

The smaller instance points the same way. Hard case 1 in **The Dataset** —
*"He was a dancer for them primarily though"* — is labelled `analysis` in
`labels.csv`, and my own written decision rule says a bare fact with nothing
built on it is a `hot_take`. That is a confirmed instance of me applying my own
rule two different ways, and the agreement check did not and could not catch
it, because it tests a different taxonomy on different posts.

So the conclusion I take into the diagnoses is narrow and it is deliberately
not reassuring: **the agreement check failed to find a labelling problem, which
is not the same as finding there isn't one.** On the specific question of
whether `reaction` is inert because of my labelling or because of the model, 30
clean posts under somebody else's tight rule cannot answer it either way.

---

## The Improvement

**What I changed:**
One thing: the loss function. Training now uses a **weighted cross-entropy**,
where each class is weighted by inverse frequency computed from the training
split — `analysis` 0.980, `hot_take` 0.714, `reaction` 1.724. A missed
`reaction` post costs about 2.4× what a missed `hot_take` post costs. The code
is `WeightedTrainer` and `balanced_class_weights` in the notebook, switched on
by `USE_CLASS_WEIGHTS` in section 6.

Nothing else moved. Same 216 posts, same labels, same taxonomy, same stratified
splits, same three seeds, same 3 epochs / 2e-5 / batch 16, same criteria. Set
`USE_CLASS_WEIGHTS = False` and the cell reproduces the before numbers exactly.

**Which diagnosis pointed at it:**
Criterion 2. The model never predicted `reaction` once across 99 held-out
predictions, and `hot_take` outnumbers `reaction` 101 to 42, so while the model
was unsure the cheapest available strategy was to ignore the small class
entirely. Plain cross-entropy prices every mistake the same and therefore
rewards exactly that. Criterion 1 is downstream of criterion 2, so this was the
change with the most reach — and unlike relabelling, it is allowed this unit.

I chose it over the other candidate, training longer, even though criterion 5's
diagnosis says the model is badly underfit. A dead label is a worse failure
than a timid one, and one change was the budget.

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.65 | 0.636 | 0.576 | 0.727 | MISSED |
| 2. Every label F1 | ≥ 0.60 for every label | analysis: 0.621; hot_take: 0.615; reaction: 0.727 — **all met** | analysis: 0.581; hot_take: 0.643; reaction: 0.286 | analysis: 0.769; hot_take: 0.720; reaction: 0.667 — **all met** | MISSED |
| 3. No label exceeds 50% of the dataset | ≤ 50% per label | analysis: 33.8%; hot_take: 46.8%; reaction: 19.4% | Same dataset | Same dataset | MET |
| 4. Accuracy spread across seeds | ≤ 0.10 | 0.636 | 0.576 | 0.727 | **MISSED (spread: 0.152)** |
| 5. Predictions with maximum class probability ≥ 0.70 | ≥ 70% for each seed | 0.0% (0/33) | 0.0% (0/33) | 0.0% (0/33) | MISSED |

**Before and after, side by side**

| Measure | Seed 42 before → after | Seed 7 before → after | Seed 2024 before → after |
|---|---|---|---|
| Overall accuracy | 0.636 → 0.636 (+0.000) | 0.636 → 0.576 (−0.061) | 0.667 → 0.727 (+0.061) |
| **Macro F1** | 0.462 → **0.654** (+0.193) | 0.458 → **0.503** (+0.045) | 0.483 → **0.719** (+0.235) |
| F1 — `analysis` | 0.667 → 0.621 (−0.046) | 0.636 → 0.581 (−0.056) | 0.700 → 0.769 (+0.069) |
| F1 — `hot_take` | 0.718 → 0.615 (−0.103) | 0.737 → 0.643 (−0.094) | 0.750 → 0.720 (−0.030) |
| **F1 — `reaction`** | 0.000 → **0.727** (+0.727) | 0.000 → **0.286** (+0.286) | 0.000 → **0.667** (+0.667) |
| `reaction` predictions emitted | 0 → 5 | 0 → 1 | 0 → 9 |
| Highest top probability | 0.491 → 0.458 | 0.505 → 0.460 | 0.602 → 0.467 |

Criteria met: **2 before, 1 after.** Macro F1: **up on all three seeds.** Those
two sentences are the whole result, and they disagree with each other.

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->

**It fixed the thing it was aimed at, did nothing for anything else, and cost
me a criterion I had passed.**

*What it fixed.* `reaction` is no longer inert. It went from 0 predictions
across 99 to 5, 1 and 9 on the three seeds, and its F1 went from exactly 0.000
to 0.727, 0.286 and 0.667. Macro F1 rose on every seed, by +0.193, +0.045 and
+0.235. On seeds 42 and 2024 **every label now clears the 0.60 bar** — criterion
2 is satisfied on two of three seeds where before it was satisfied on none.
The diagnosis was right: the label was not unlearnable, it was unprofitable.

*What it did not fix.* Overall accuracy is flat — +0.000, −0.061, +0.061 — so
criterion 1 is still missed. Criterion 5 did not move at all: still 0 of 33
predictions above 0.70 on every seed, and the highest top probability actually
*fell*, from 0.491/0.505/0.602 to 0.458/0.460/0.467. That is what I should have
expected. Weighting the loss changes which way probability mass leans, not how
concentrated it is, and the underfitting diagnosed under criterion 5 is
untouched by it. Roughly 30 gradient steps is still roughly 30 gradient steps.

*What it cost.* `hot_take` F1 fell on all three seeds and `analysis` fell on
two. The matrices say where it went: `analysis`↔`hot_take` confusions across
the three seeds went from 17 to 22, almost all in one direction — `hot_take`
posts called `analysis` rose from 2, 2 and 1 to 7, 7 and 4. Up-weighting the
rare class also shifted `analysis` (0.980) above `hot_take` (0.714), so the
model traded one bias for another. Accuracy stayed flat because the `reaction`
posts it started getting right were paid for almost exactly by `hot_take` posts
it started getting wrong.

*And criterion 4 flipped from MET to MISSED* — spread 0.030 → 0.152. **This is
the result I am most pleased with, and it is a failure.** In Milestone 4 I
argued against my own MET on criterion 4: the model looked stable because it
was degenerate, and a model that never emits a label cannot vary much. The
improvement tested that claim without my intending it to. The moment the model
started using all three labels, the variance appeared — `reaction` recall is
0.667, 0.167 and 0.833 across the seeds, which is 4, 1 and 5 posts out of 6.
A label with 6 test examples cannot produce a stable score, and the before-run
hid that behind a constant zero. Criterion 4 did not get worse because the
model got worse. It got worse because the model started doing the thing the
criterion was too crude to measure.

**Verdict:** I would keep this change. The scoreboard says 2 criteria met
before and 1 after; macro F1, the measure that does not let a dead label hide,
is up on all three seeds. I trust the second reading, and the first one is an
argument about my criteria rather than about my model.

---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->

**Criterion 1 — overall accuracy (0.636, 0.576, 0.727 against ≥ 0.65).**
What I would do: train longer. Criterion 5's diagnosis says the model is still
near its initialisation after about 30 gradient steps, and the obvious next
change is more epochs, a higher learning rate, or both, measured the same way.
Why I stopped: one improvement was the budget for this unit, and I spent it on
the dead label. I think that was the right order — accuracy that averages over
a label the model refuses to predict is not a number worth lifting first.

**Criterion 2 — every label F1 ≥ 0.60 (met on seeds 42 and 2024, failed on
seed 7 at `reaction` 0.286).**
What I would do: more `reaction` examples, but not before rewriting the rule.
`reaction` recall across the three seeds is 4/6, 1/6 and 5/6 — a label measured
on 6 posts cannot give a stable F1, so part of this is sample size and more
data would genuinely help. The other part is not fixable by volume: the churn
table shows 38.1% of my `reaction` posts were first labelled something else,
and my `reaction` rule asks what a post is *about*, with no observable test.
Labelling more posts under a rule I apply inconsistently would add noise, not
signal. Why I stopped: labels, taxonomy and criteria were frozen this unit.

**Criterion 4 — accuracy spread ≤ 0.10 (0.152).**
What I would do: change the criterion, not the model, which is the one thing I
am not allowed to do here — and it would not count anyway, because it would be
loosening a target I missed. The honest version pairs stability with a quality
floor, something like "spread ≤ 0.10 *and* every label F1 ≥ 0.60", so that it
cannot be satisfied by failing consistently. Why I stopped: lowering or
reshaping a missed target is the change that earns nothing, so it stays missed
and diagnosed.

**Criterion 5 — confidence (0% of predictions at ≥ 0.70, all three seeds).**
What I would do: this is the same fix as criterion 1 — train longer — and I
would measure it with the `share_at_or_above_threshold` field I added to
`confidence_report`, which is now in the notebook for exactly this. Why I
stopped: one change. It is also worth saying that this criterion may be wrong
rather than merely unmet: I asked for confident predictions without asking
whether confidence tracked correctness, and `accuracy_most_confident_third`
versus `accuracy_least_confident_third` was already answering the better
question. I am leaving it as filed and missed.

**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->

My taxonomy is built on one idea — whether a post gives a reason for its claim
— and that is the one thing neither model learned: the `analysis`↔`hot_take`
confusions, the boundary I wrote an explicit decision rule for, number 17
across three seeds before the change and 22 after, getting worse even as
everything else improved. What the model actually learned is the easier,
shallower split my labels also happen to encode, between posts about the writer
and posts about the music, which is why weighting the loss could switch
`reaction` on almost immediately while the distinction I actually care about
stayed broken in both runs.

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

       [x] Baseline vs. Trained table, with your prediction written beforehand
       [x] Run Log — Before, five criteria across three seeds
       [x] Confusion matrix TYPED AS A MARKDOWN TABLE
       [x] A verdict on every criterion
       [x] A diagnosis for every miss, using the agreement report as evidence
       [x] Agreement Report with every disagreement adjudicated
           (30/30, so no disagreements to adjudicate — the section says why
           that is a null result rather than a clean bill of health)
       [x] One improvement, with Run Log — After
       [x] What's Still Broken
       [x] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [x] At least four new commits
       [ ] The SAME repository URL as last unit  <-- SUBMIT THIS YOURSELF

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**