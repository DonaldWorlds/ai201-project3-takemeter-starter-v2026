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



**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->
**Criterion 1 — Overall accuracy**

Cause: The model did not meet the accuracy target consistently across seeds.

Evidence: Seeds 42 and 7 scored 0.636, below 0.65, while Seed 2024 scored 0.667. The accuracy spread was only 0.030, suggesting relatively stable results, but the target was not met across all three seeds.

**Criterion 2 — F1 for every label**

Cause: The model struggled to distinguish `reaction` from the other labels.

Evidence: `reaction` F1 was 0.000 for all three seeds. In the Seed 42 confusion matrix, all six actual `reaction` posts were misclassified: five as `hot_take` and one as `analysis`. The dataset contains 42 `reaction` examples compared with 101 `hot_take` examples, so the smaller number of `reaction` examples may contribute to the problem.


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
I changed the training setup to give the reaction class more weight in the loss function. I kept the dataset labels and held-out test set unchanged so I could test whether giving more importance to reaction during training improved its predictions.

**Which diagnosis pointed at it:**
The original model had an F1 score of 0.000 for reaction across all three seeds. In Seed 42, five of the six actual reaction posts were classified as hot_take. The staff agreement check had 30 agreements and zero disagreements, so it did not reveal inconsistent labeling on that comparison set. These results motivated testing a training change before making unsupported changes to the labels.

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