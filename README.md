# How a non-programmer reached silver, and benched the gold

Companion code for the working note *"How I Vibe-Coded my way to almost Top 10 position but then fell to 36th"* (Kaggle **AI Agent Security: Multi-Step Tool Attacks**, ID 134815; OpenAI / Google / IEEE). Final result: **36th of 4,187, silver**, directed entirely by a non-programmer.

**Full write-up:** [How I Vibe-Coded my way to almost Top 10 position but then fell to 36th](https://www.kaggle.com/competitions/ai-agent-security-multi-step-tool-attacks/writeups/how-i-vibe-coded-my-way-to-almost-top-10-position)

This is not another winning method. It offers two things the leaderboard repositories do not:

1. **A reproducibility audit of the field's published solutions** (below). I cannot read the code; I directed an agent. After the deadline I pasted the top teams' shared solutions back through the grader to answer a question no other write-up measured: *how far do the published solutions actually carry a copier who cannot read them, and where does that stop?*
2. **The one defect that shaped the entire public board, shown live and paired with its one-line fix**: a short program you can run right now with no model, no GPU, and no key.

## Scope and safety

Everything here concerns the competition's **offline sandbox** and the SDK shipped with it. Every "secret" is a **synthetic sentinel** (`SECRET_MARKER`); every destination is a **non-routable** placeholder (`corp.invalid`, `://a`). Nothing reads a real secret or contacts a real system, and **every attack is paired with its fix**. The mutable-replay-config harness bug from the note (§1) was disclosed privately to the organisers and is **deliberately not included here in runnable form**; there is no config-mutation exploit in any file. The two reconstructions are of *publicly published* solutions and credit their authors.

## Run this first (no model, no GPU, no key)

```bash
python3 findings/guardrail_extract_target_demo.py
```

Ten lines that reproduce the defect at the heart of the public leaderboard: the reference guardrail's `_extract_target` returns on the first argument key it finds, so for an `http.post` it inspects the `url` and never the `data`, the exact field the exfiltration scorer reads. The synthetic sentinel sails through. The same file shows the one-line fix (inspect every argument) and asserts it now blocks. This is the note's §3, made runnable.

## The reproducibility audit (our distinct contribution)

After the competition I rebuilt the top teams' solutions from their public write-ups and code and re-scored them. The point was to measure, empirically, how reproducible the published field actually is for someone who cannot read the code:

| what was published | reproducible by pasting? | result |
|---|---|---|
| top public **exfiltration** solution (fully shared) | **yes** | 130.5 public (a few points under the author's original) |
| 7th-place **confused-deputy** solution (fully shared) | **yes** | **35.0 private, gold-caliber**, with no GPU of my own and nothing else added (this is the one I actually rebuilt) |
| 4th-place **confused-deputy** (published complete + self-contained) | public; **not run by me** | **41.325 private, gold**; recipient bank embedded in the code, so runnable by anyone; I read it, I did not re-run it |
| 1st-place **confused-deputy + GCG-optimized prompts** | **partly** | the winning notebook publishes the exact prompts, but the ordered screened-recipient dataset it feeds on is marked *private*; matching its 46 means re-deriving that set on a GPU, not pasting a file |
| last stretch of public score (Gemma multi-post + resubmission volume) | not pursued by me | forms and best-of-many volume I did not chase |

**The finding, split by what I verified versus read:** what I *verified* is that two published builds, pasted and re-scored with no GPU of my own, reproduce to **35.0 private** (gold-caliber) and **130.5 public**. What I *read but did not run*: the 4th-place team's build is public and self-contained (recipient bank embedded in the code) and scored 41.325, so a gold-tier build is fully in the open; the 1st-place winner published his prompts but kept the recipient dataset private, so matching 46 would need re-deriving it on a GPU. The observation, not a score I am claiming, is that the published field is remarkably open up to a point: a complete gold build is copyable, and the private board protected against novel *discovery*, not against copying published winners. See §5 of the note.

## What is here

| file | what it is | public | private | note section |
|---|---|---|---|---|
| <a href="solutions/01_public_best_ours__v19__public_120.9.ipynb"><code>solutions/01_public_best_ours</code><br><code>__v19__public_120.9.ipynb</code></a> | our best public build (submitted): exfiltration, K9 over-ask | **120.9** | 0.000 | Sec. 7 |
| <a href="solutions/02_public_reconstruction__exfil__public_130.5.py"><code>solutions/02_public_</code><br><code>reconstruction__exfil</code><br><code>__public_130.5.py</code></a> | reconstruction of a top public exfiltration solution | **130.5** | 0.000 | Sec. 5 |
| <a href="solutions/03_private_best_ours__v12b__private_30.7_BENCHED.ipynb"><code>solutions/03_private_best_ours</code><br><code>__v12b__private_30.7</code><br><code>_BENCHED.ipynb</code></a> | our best private build. **Benched. Would have been gold (~11th).** | 30.7 | **30.735** | Sec. 4.3 |
| <a href="solutions/04_private_reconstruction__deputy__private_35.0.py"><code>solutions/04_private_</code><br><code>reconstruction__deputy</code><br><code>__private_35.0.py</code></a> | reconstruction of a 7th-place confused-deputy solution | 35.2 | **35.040** | Sec. 5 |
| <a href="solutions/05_private_SUBMITTED__v17__private_27.3.ipynb"><code>solutions/05_private_SUBMITTED</code><br><code>__v17__private_27.3.ipynb</code></a> | the private build **we actually submitted**, lower than the one we benched | 27.3 | **27.300** | Sec. 4.3 |
| <a href="findings/guardrail_extract_target_demo.py"><code>findings/guardrail_</code><br><code>extract_target_demo.py</code></a> | the guardrail defect, live, **paired with its fix** | n/a | n/a | Sec. 3 |
| <a href="WORKING_NOTE.md"><code>WORKING_NOTE.md</code></a> | the full working note | | | all |

Every score above is the value that build actually received from Kaggle's own scoring. Builds `01/03/05` are our submissions; `02/04` are the reconstructions from the audit.

## How to reproduce

| What | How |
|---|---|
| The guardrail defect and its one-line fix | `python3 findings/guardrail_extract_target_demo.py`. No model, no GPU, no key, no install. It prints the defect, applies the fix, and asserts the fix blocks it. |
| Our best public build (120.850) | Open `solutions/01_…ipynb` as a Kaggle notebook for this competition, attach a T4 x2 accelerator, and run. |
| Our benched private gold (30.735) | `solutions/03_…ipynb`, same procedure. This is the build we did not select. |
| The build we actually submitted (27.300) | `solutions/05_…ipynb`, same procedure. |
| The exfiltration reconstruction (130.490 public) | `solutions/02_…py` is a self-contained `attack.py`. Drop it into a competition notebook in place of the default attack and run. |
| The confused-deputy reconstruction (35.040 private) | `solutions/04_…py`, same procedure. |
| Any leaderboard number quoted in the note | Kaggle's own public and private leaderboards for this competition. |

The scored rows need a Kaggle notebook with a GPU, because the harness replays two local models under a fixed clock. Only `findings/` runs anywhere, and it is the one that demonstrates the actual defect.

The SDK facts the note quotes (severity weights, the novelty bonus, the `/200` normalisation, and the authorisation heuristic at `core/predicates.py` lines 221 to 224) are checkable directly in the competition SDK, without running anything.

## Attribution for the reconstructions

Files `02` and `04` are **our reconstructions** of other competitors' **publicly published** solutions, rebuilt from their write-ups and code and keeping their exact prompt strings, included only for the audit above. All credit for those techniques belongs to their original authors:

- `02` (exfiltration) follows the public **`agi-exfil-160`** solution.
- `04` (confused-deputy) follows the public **7th-place `plr40`** solution.

These are not those teams' original notebooks. If you are one of the authors and would like the attribution changed or the file removed, please open an issue.

## The one finding that matters most

The public board is a mirage. Across all 4,187 teams the two boards' scores are almost uncorrelated (Pearson ≈ +0.23, a measure from -1 to +1 of how the two move together), and **two thirds of the field (2,802 of 4,187) scored exactly 0.000 private**. You can see the inversion in one place here: our proudest public number (120.9) scored **zero** on the board that decided the competition, while the quiet 30.735 build we left on the bench would have been **gold**. Whatever a scorer reads, the guardrail must read too; `findings/` shows why in ten lines.
