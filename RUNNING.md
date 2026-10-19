# Running TakeMeter

Everything about how the starter works.

This pair is different from the last two: **you train a model.** It runs on
your own machine, in a notebook, from inside this repo. No accounts, no keys,
nothing hosted.

---

## Before your first class

One setup, one virtual environment, everything in this repo. The
[environment setup page](../pages/ide_setup) has the per-operating-system
commands.

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\Activate.ps1
pip install -r requirements.txt
python test.py
```

**`python test.py` passing is what you need before your first session.** It
checks your Python version against the pins, your disk, your memory, and
whether the packages import.

**Your disk needs about 9 GB free, up from 2 in the last two pairs.** The baseline model is roughly
1.6 GB and PyTorch is not small.

**Your machine needs about 6 GB of RAM.** Two things here are memory-hungry:
`baseline.py` peaks around 1.8 GB loading the zero-shot model, and the training
run peaks around 2.1 GB. Neither is comfortable beside an editor and a browser
on a 4 GB machine. `test.py` warns you.

You'll see one `[WARN] Baseline model — not downloaded yet`. That's expected in
unit 5. **Clear it before the unit 6 session** by running `python baseline.py`
once — downloading 1.6 GB during class is not how you want to spend a breakout.

> **No API key and no accounts for this pair.** Nothing here calls a hosted
> model. The baseline and the training both run on your own machine.

### What you'll be training on

`test.py` tells you, and so does section 1 of the notebook. There are three
possibilities and all of them work:

| Device | What it means | Speed |
|---|---|---|
| `cuda` | An NVIDIA GPU | Under a minute per seed |
| `mps` | Apple Silicon — M1 and later | Fast; much quicker than the CPU path |
| `cpu` | Everything else | A few minutes per seed. Three seeds is a coffee, not an afternoon |

⚠️ **Write down which one you get.** It is part of your result, not a detail
about your laptop. Two people running the same notebook on the same data with
the same seed can get different numbers, because different hardware does the
arithmetic in a slightly different order. Unit 6 asks you to report it beside
your three seeds, and `results.json` records it for you.

---

## The notebook

`takemeter.ipynb` runs **on your own machine**, from inside this repo.

1. Open it in VS Code (or `jupyter notebook`, if you prefer).
2. **Pick the kernel: the `.venv` inside this project.** In VS Code that's the
   selector in the top right. Getting this wrong is the most common way the
   first cell fails — it will say a package isn't installed when it is,
   because it's looking at a different Python.
3. Fill in the two lines at the top of section 1's second cell — your name and
   email. They only label your results.

> ⚠️ **Open it from your repo, not from a copy.** The notebook writes next to
> your code and expects `labels.csv` to be there. If you open a copy from your
> Downloads folder, it will tell you so rather than failing strangely.

### It reads and writes this folder

There is nothing to upload and nothing to download. `labels.csv` is already
here; `results.json`, `test_split.csv` and `results_three_seeds_*.json` land
here when the notebook writes them.

**Commit them yourself**, the way you commit anything else. Nothing is pushed
for you.

- Section 5 writes `results.json` and `test_split.csv` — **unit 5**
- Section 6 writes `results_three_seeds_<label>.json` — **unit 6**.
  Set `RUN_LABEL = "after"` in that cell before the Milestone 5 re-run, or the
  improvement overwrites the numbers you're comparing against.

> **Add rows to `labels.csv` mid-session?** Re-run section 2, which is what
> reads the file. The notebook holds the data in memory once it has read it.

> **Your edits to the notebook itself are yours to commit too.** `LABELS`, the
> settings and the seed live in the file — if you change them, that's a change
> to your repo like any other.

### What's in it

| Section | When | What it does |
|---|---|---|
| 1. Setup | unit 5 | Reports your device, and confirms it can see your files |
| 2. Your labels | unit 5 | `LABELS`, the settings, and **the seed** |
| 3. Split | unit 5 | 70/15/15, stratified |
| 4. Train | unit 5 | Minutes on a laptop CPU, under one on a GPU |
| 5. Results | unit 5 | Writes `results.json` and `test_split.csv` |
| 6. Three seeds | **unit 6** | Retrains three times in one cell |
| 7. Confusion matrix | **unit 6** | Prints a markdown table to paste |

Sections 6 and 7 are next unit. Nothing in unit 5 needs them.

---

## Your data

`labels.csv` in your repo. Three columns:

| Column | What's in it |
|---|---|
| `text` | The whole post. Don't trim it to fit |
| `label` | One of your labels, spelled **exactly** as in `LABELS` |
| `note` | `cold` for the 20 you labelled unaided, and a note on any you pre-labelled |

`labels_template.csv` shows the shape. `data/practice_labels.csv` is the set
used in class — 60 posts from a fictional running forum, already labelled, so
you can run the whole notebook before you have data of your own.

**One file, unsplit.** Don't split it yourself — the notebook does the 70/15/15
split, and splitting twice is how examples leak between training and test.

### The check before training

Section 2's last cell looks for the things that break a run or quietly ruin it:

| It says | What it means |
|---|---|
| `[FIX ME] Labels in your CSV that aren't in LABELS` | A spelling or capital mismatch. `Hot_take` and `hot_take` are different |
| `[FIX ME] 'x' is 78% of your data` | Over the brief's 70% cap. The model will learn to guess it |
| `[look at] N duplicate posts` | They'll land in different splits and inflate your score. That's leakage |
| `[look at] Only 12 examples of 'x'` | Its test split will be tiny, so its F1 will jump between seeds |
| `[look at] 60 rows` | Under the submission floor. Training still runs — fine for the practice data |

---

## The baseline — unit 6

```bash
python baseline.py
python baseline.py --trained results.json
```

This runs a general model over your posts using **only your label
definitions**, with no training. It's the fair opponent your fine-tuned model
has to beat.

**It scores `test_split.csv`** — the held-out posts the notebook saved in unit 5,
the same ones your trained model was measured on. Two numbers from two different
sets of posts aren't a comparison, which is why it reads that file rather than
picking its own sample. If the file isn't there it says so, falls back to a
random sample of `labels.csv`, and tells you to disclose that in your README.

Runs **on your machine**. No account, no key. A few seconds per post, so 30
posts takes a couple of minutes — that slowness is itself part of the finding.

Put your definitions in `label_definitions.txt`, one per line as
`label: definition`, copied from your README. **The definitions are the prompt** —
they're the entire input the model gets.

Without the file it falls back to bare label names, which is a much weaker
prompt. Worth doing on purpose once: run it both ways and the gap between the
two accuracies is what your definitions were worth, measured instead of assumed.
If you report the bare-names run as your baseline, say so in your README.

The second command adds a Trained column and prints the **Baseline vs.
Trained** table ready to paste.

---

## The agreement check — unit 6

Staff labelled 30 posts under the taxonomy in `data/staff_taxonomy.md`. You
label the same 30 under **those** definitions — not the ones you designed in
unit 5 — and then compare.

```bash
python agreement.py
```

Four files. The posts and the rules ship with the starter; only the answers
come from your TF, and only after yours are done. Posts and labels join on `id`:

| File | Where it comes from |
|---|---|
| `data/staff_taxonomy.md` | **Ships in the starter.** The three definitions and the decision rules staff used. Read it first — you label under *these*, not your own |
| `data/staff_posts.csv` | **Ships in the starter.** The 30 posts, no labels |
| `data/staff_labels.csv` | **Your TF gives you this**, once your own labels are done |
| `my_staff_labels.csv` | You make it: the same posts, labelled by you, with `text` and `label` columns |

**You label the staff set under the staff taxonomy, not the one you designed in
unit 5.** Your own taxonomy was yours to invent; this one is somebody else's to
apply, and whether you can apply a written rule the way its authors did is the
only evidence you have about whether your own 200 labels were consistent.

Put both staff files in `data/` under those names and the bare command finds
them. Anywhere else, point at them:

```bash
python agreement.py --staff-posts posts.csv --staff-labels labels.csv
```

Posts are matched on their text, so **paste it unchanged**. Retype or trim a
post and it won't match — the script tells you which ones didn't.

It works out the rate and lists every disagreement. **It stops there on
purpose.** The rate is arithmetic; the adjudication is the graded part, and your
README has a block to fill in for each one.

> This is the only instrument you have that can tell a labelling problem from a
> model problem. Milestone 4 asks you to use it as exactly that — a label pair
> you disagreed with staff about, and that your matrix also confuses, is a
> finding about your labels rather than about the model.

---

## Every command

| Command | What it does |
|---|---|
| `python test.py` | Checks your environment |
| `python baseline.py` | The zero-shot baseline on your held-out posts — **unit 6** |
| `python agreement.py` | Your labels vs. the staff set — **unit 6** |
| `python baseline.py --help` | All the options |
| Notebook: sections 1–5 | Train once, write `results.json` — **unit 5** |
| Notebook: sections 6–7 | Three seeds and the matrix — **unit 6** |

---

## Which piece goes with which milestone

### Unit 5 — the build

| Milestone | What you're doing | Where |
|---|---|---|
| 1 | Pick a community and read it | Nothing to run. Read for 20 minutes |
| 2 | Design your labels | README, **Label Taxonomy** |
| 3 | Write your criteria | `criteria.md` — all five are yours this time |
| 4 | Collect and label 200 | `labels.csv` |
| 5 | Run the training | Notebook, sections 1–5 |
| 6 | Write it up | README |

### Unit 6 — the test

| Milestone | What you're doing | Where |
|---|---|---|
| 1 | Run the baseline | `baseline.py` |
| 2 | Retrain three times, read the matrix | Notebook, sections 6–7 |
| 3 | Check yourself against the staff labels | `agreement.py` |
| 4 | Call each criterion, diagnose the misses | README |
| 5 | Fix one thing and re-run | Notebook, section 6 with `RUN_LABEL = "after"` |
| 6 | Say what's still broken | README |

---

## Where everything lives

| File | What it does |
|---|---|
| `takemeter.ipynb` | The training notebook. Runs here, on your machine |
| `baseline.py` | The zero-shot baseline. Runs on your machine |
| `agreement.py` | Your labels against the staff set. **Unit 6** |
| `criteria.md` | Your five criteria. **You write all five** |
| `labels.csv` | Your labelled data. **You create this** |
| `labels_template.csv` | The shape it needs |
| `label_definitions.txt` | Your definitions, for the baseline |
| `data/practice_labels.csv` | The 60-post practice set used in class |
| `data/staff_taxonomy.md` | The staff set's definitions and decision rules. Ships here |
| `data/staff_posts.csv` | The staff set's 30 posts. Ships here |
| `data/staff_labels.csv` | The staff set's labels. **Your TF gives you this** |
| `README.md` | Your submission |
| `results.json` | Written and committed by the notebook |
| `test_split.csv` | Your held-out posts, committed by the notebook. `baseline.py` reads it |
| `results_three_seeds_before.json` | Written and committed in unit 6, section 6 |
| `results_three_seeds_after.json` | The same cell re-run with `RUN_LABEL = "after"`, in Milestone 5 |
| `baseline_results.json` | Written by `baseline.py`. **Commit it** |
| `agreement_results.json` | Written by `agreement.py`. **Commit it** |
| `my_staff_labels.csv` | Your labels on the staff set. **You create this** |

---

## When something goes wrong

| What you see | What it means |
|---|---|
| Section 1 says `cpu` | Correct on most laptops, and fine. A three-seed run is minutes. Nothing to fix |
| `ModuleNotFoundError` in the notebook | Wrong kernel. Pick the `.venv` inside this project, top right in VS Code |
| `FileNotFoundError: labels.csv` | Either you haven't created it yet, or the notebook is open from a copy rather than from your repo. Section 1's second cell says which |
| The machine crawls during training | Close the browser and anything heavy. Training peaks around 2.1 GB |
| `KeyError` during training | A label in your CSV isn't in `LABELS`. Check capitals and spaces |
| Training takes an hour | You're on CPU. See the first row |
| Accuracy swings 15 points between seeds | Your dataset is too small or too lopsided for a stable measure. **That's a diagnosis, not a mistake** — write it down |
| A label's F1 is 0.00 | It probably has almost nothing in the test split. Check the split table |
| `baseline.py` says no `test_split.csv` | Unit 5's results section never finished. Re-run it, or note in your README that your comparison isn't like-for-like |
| `baseline.py` is downloading forever | First run only, ~1.6 GB. Do it before class |
| The baseline is very slow | Expected. A few seconds per post is what no fine-tuning costs at runtime |

The three usual causes when the notebook fails, in order: the kernel is the
system Python rather than this project's `.venv`, the notebook was opened from
a copy outside the repo, or a label in your CSV doesn't match `LABELS` exactly.

If none of those fix it, commit your CSV and criteria and write down the error.
**You have a labelled dataset and a filed standard, which is most of the
unit's grade.**

---

## A note on committing

At least four new commits in unit 5, four more in unit 6. Your commit history is
what shows your criteria existed before your results did.

The `results*.json` files are deliberately **not** in `.gitignore`. They're
evidence the run happened, and so are `test_split.csv` and `labels.csv`. The
notebook writes them; you commit them, along with `criteria.md`,
`baseline_results.json` and your README.

**One machine, one repo.** Everything happens here, so there's nothing to pull
before you push.

**Do not delete and recreate this repository.** You submit the same URL both
units.




device: mps  (Apple Silicon GPU (MPS))
torch:  2.14.1

⚠️  WRITE THIS DOWN. Your results depend on it.

Two people running this notebook on the same data with the same seed can
get different numbers, because different hardware does the arithmetic in
a slightly different order. That is not a bug and it is not one of you
being wrong — it is a real property of the thing you are building, and
unit 6 asks you to report it alongside your three seeds.


### Milestone 1: Candidate Distinctions

**Community:** r/hiphopheads on Reddit

1. **Some posts explain their opinions, while others just state them.** For example, one commenter says Busta Rhymes became successful because of his group and then became a star solo, explaining why he fits the question. Other comments simply say “Legendary” or “Lil Wayne and the hot boys.”

2. **Some posts include specific examples, while others make general statements.** One commenter names several groups and artists, including Nelly and the St. Lunatics, Eminem and D12, and 50 Cent and G-Unit. Other commenters only name one artist, such as “KOOL KEITH.”

3. **Some posts add personal experiences, while others focus on facts or opinions.** One commenter describes growing up in New Orleans and explains how people viewed the Hot Boyz as a group. Another recalls discovering MF DOOM through an old graffiti magazine.

4. **Some posts build on or correct other comments, while others stand alone.** One commenter adds that MF DOOM was in KMD, while another responds by distinguishing KMD from MIC (Monsta Island Czars). Other comments contribute a name or short reaction without developing the discussion further.

















# Milestone 2: Design My Labels

**Community:** r/hiphopheads

## Label 1: `analysis`

**Definition:** A post that explains a claim or opinion using specific details, comparisons, or reasoning.

**Example 1:**
> They just didn't have it. They weren't that successful as a group in the first place and the two things they did have going for them were Busta and their connection to ATCQ. So when they didn't have Busta anymore nobody was going to be tuning in to see what Charlie Brown does with his career.

**Example 2:**
> This is the best answer. Based on group success, solo success, and then never forming the group again. I'll accept Lil Wayne but only if you count the bg'z as a group. Hot boys were always a super group and you met them after juvenile solidified his spot.

**Why these fit:** Both comments explain their points with reasoning and details about an artist's career or group history.

## Label 2: `personal_response`

**Definition:** A post that mainly expresses a personal opinion, feeling, preference, or memory without developing a reasoned argument.

**Example 1:**
> That whole La Roux tape goes hard. Always bang it in the gym.

**Example 2:**
> This shit was sensational freshman year in college.

**Why these fit:** Both comments communicate a personal reaction or experience rather than explaining a claim with supporting details.

## Label 3: `brief_reference`

**Definition:** A post that mainly names an artist, group, song, or factual connection without explaining or supporting a larger point.

**Example 1:**
> MF DOOM was originally in KMD (as Zev Love X)

**Example 2:**
> Dr. Dre was in World Class Wreckin Cru

**Why these fit:** Both comments contribute a specific music-related reference but do not develop an explanation around it.

## Borderline post

**Post:**
> I remember Big Sean version. It was honestly dope even though he's not the best rapper.

**Labels it could fit:** `personal_response` or `analysis`

**Decision rule:** If a post mainly expresses a personal opinion or reaction and does not give a specific supporting reason, classify it as `personal_response`, even if it includes a qualification such as “even though he's not the best rapper.” Use `analysis` when the writer develops an explanation with specific details, comparisons, or reasoning.

## Stress test

I will test these definitions with eight fictional comments that sit near the boundaries between the labels. These are generated examples, not actual Reddit posts.

1. “Busta was better solo because his delivery stood out more.” — `analysis`
2. “Busta is still one of my favorites.” — `personal_response`
3. “Busta Rhymes was part of Leaders of the New School.” — `brief_reference`
4. “I loved that album in high school because I played it every day.” — `personal_response`
5. “The album was successful because it produced several hit singles.” — `analysis`
6. “KMD had MF DOOM before his solo career.” — `brief_reference`
7. “That song is fire, especially compared with his newer music.” — Borderline: `personal_response` or `analysis`
8. “I prefer the original because the vocals are clearer and the beat has more space.” — `analysis`

**Stress-test finding:** Posts 7 and 8 show that a preference can include a comparison or a reason. The decision rule should distinguish a simple preference from a developed explanation. If the writer gives a specific reason for the preference, classify it as `analysis`; otherwise, use `personal_response`.

**Limitation:** These are initial definitions. I should apply them to more real comments and check whether the labels remain clear before labeling the full dataset.