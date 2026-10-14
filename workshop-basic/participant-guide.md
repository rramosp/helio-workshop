# Heliophysics Toolkit Workshop — Participant Handouts (with SDO Flare Prediction Examples)

> [!IMPORTANT]
> **About the worked examples (`sdoflare`):** Throughout these handouts, all concrete prompts, follow-up questions, bug-hunt scenarios, skill-creation examples, and error-log entries use an **imaginary Python package called `sdoflare`** — a hypothetical toolkit for preprocessing full-disk **SDO/AIA** (EUV) and **SDO/HMI** (magnetogram) images and running **operational solar flare prediction models** against the **GOES XRS** flare catalog.
>
> Everything here is meant to be given to participants verbatim (print **H0**; put the rest in `prompts/` and `templates/` in the workshop repo) after replacing `sdoflare` and the file/function names with your actual toolkit.

---

## Reference Toolkit & Worked Science Cases Used in the Examples

To make every prompt concrete, the examples below refer to this package layout and three realistic participant science cases:

### Reference package (`sdoflare`) and notebooks
- `src/sdoflare/io.py` — `load_aia_map()`, `load_hmi_los()`, `load_goes_xrs()`, `load_flare_catalog()`, `SDOTimeIndex`
- `src/sdoflare/prep.py` — `aia_l1_to_l15()` (pointing, solar-north roll, $0.6''/\text{pix}$ plate-scale registration), `correct_aia_degradation()` (time-dependent sensitivity table + exposure normalization to $\text{DN}\,\text{s}^{-1}\,\text{pix}^{-1}$), `correct_hmi_projection()` ($\mu = \cos\theta$ line-of-sight correction + NaN mask beyond $R/R_\odot > 0.95$), `coalign_full_disk()` (reproject/resample multi-channel AIA + HMI to a common $512\times 512$ helioprojective grid at synchronized cadence), `scale_channels()` (`asinh`/$\log_{10}$ for EUV, signed clip $\pm 2000\,\text{G}$ for HMI $B_{\text{los}}$)
- `src/sdoflare/labels.py` — `label_flare_window()` (binary $\ge\text{M1.0}$ or peak GOES 1–8 Å flux in $[t_0, t_0+\Delta t]$, flagging SDO eclipse gaps), `carrington_split()` (chronological split with a 27-day solar-rotation buffer so the same active region never straddles train and test)
- `src/sdoflare/models.py` — `FullDiskFlareNet` (pretrained multi-channel SDO encoder + forecast head), `extract_disk_embeddings()`, `LinearProbeForecaster`, `PersistenceBaseline`, `ClimatologyBaseline`, `attribute_disk_regions()` (integrated gradients projected to heliographic coordinates & NOAA AR boxes)
- `src/sdoflare/eval.py` — `compute_tss_hss()`, `find_optimal_tss_threshold()`, `brier_skill_score()`, `block_bootstrap_metric()`, `reliability_diagram()`, `plot_event_timeline()`
- **Notebooks**:
  - `01_sdo_ingest_and_l15_prep.ipynb` — Level 1 → 1.5 registration, AIA degradation, HMI $\mu$-correction
  - `02_multichannel_coalignment_and_labels.ipynb` — building synchronized AIA+HMI full-disk cubes & 24h flare labels
  - `03_pretrained_fulldisk_flare_forecast.ipynb` — running `FullDiskFlareNet` for 24h $\ge\text{M1.0}$ probabilities
  - `04_embedding_finetune_and_probing.ipynb` — freezing the SDO encoder and fitting custom heads/horizons
  - `05_evaluation_attribution_and_events.ipynb` — TSS/HSS, reliability diagrams, and AR attribution on X-flare events
- **Reference Pre-Installed Skill**:
  - `skills/sdo-ar-event-attribution/SKILL.md` — Gold-standard example skill showing how a validated `sdoflare` workflow is packaged for other scientists' coding agents.

### Three worked scientist personas
- **Case α (Elena — EUV channel ablation & degradation):** *"Does adding hot flaring channels (AIA 94 Å, 131 Å) to HMI $B_{\text{los}}$ + AIA 171 Å improve 24h $\ge\text{M1.0}$ flare skill across 2014–2017, or does uncorrected AIA sensitivity degradation create a spurious solar-cycle trend?"* → Distilled in M6 into skill **`sdo-euv-channel-ablation`**.
- **Case β (Marco — Limb foreshortening & $\mu$-angle bias):** *"How badly does full-disk flare forecast skill drop when the flaring active region is near the solar limb ($|\text{lon}| > 60^\circ$, $\mu < 0.5$) where HMI $B_{\text{los}}$ foreshortens, and does off-limb AIA 193/211 Å brightness recover that skill?"* → Distilled in M6 into skill **`sdo-limb-foreshortening-audit`**.
- **Case γ (Priya — Short-horizon peak X-ray flux around Sep 2017 AR 12673):** *"Can I use precomputed `FullDiskFlareNet` embeddings to predict continuous 6-hour peak $\log_{10}(\text{GOES 1–8 Å flux})$ and test whether it anticipates the X2.2 / X9.3 flares of 6 Sep 2017?"* → Distilled in M6 into skill **`sdo-preflare-flux-probe`**.

---

## H0 · The Agent Operating Contract (one page — keep it next to your keyboard)

**You are the scientist. The agent is a fast, well-read, overconfident assistant.**

1. **Ground every ask.** Point the agent at files: `@notebooks/01_sdo_ingest_and_l15_prep.ipynb`, `@src/sdoflare/prep.py`. Ask for citations (`file:line` or notebook cell). Open at least one citation per answer.
2. **Plan before code.** First ask for a plan. Edit or approve it. Only then say "implement".
3. **One step at a time.** Small change → run → look at the output → next prompt. Never "do steps 1–6".
4. **Toolkit first.** The agent must use `sdoflare` functions. If it writes its own FITS reader, `sunpy` reprojection loop, or TSS metric, stop it: *"Search `src/sdoflare` for an existing function that does this."* If the toolkit genuinely lacks it, write it down — that is feedback we want.
5. **Verify before trust.** Any number or plot that reaches a conclusion gets a sanity check you understand: units/range, a known event (e.g. AR 12192 or AR 12673), a baseline. Ask the agent to write the check; *you* decide whether the check is meaningful.
6. **You own the physics.** Data selection, metrics, interpretation, conclusions: yours. The agent does mechanics and critique.
7. **Log the misses — then bake them into a reusable Skill.** Every time the agent is wrong, add a line to `ERROR_LOG.md`: what it claimed, how you noticed, what it would have cost. In Module 6, you will distill your validated workflow **and** those hard-won guardrails into a reusable **Agent Skill (`SKILL.md` + CLI script)** so other scientists can run your methodology on new events or periods in a single prompt without hitting the same traps.

**3-strikes rule:** after three failed fix attempts, stop prompting. Run prompt **S5**, read the source yourself, ask a facilitator, or cut scope.

### Weak vs. directed prompts (quick contrast)

| Weak prompt (undirected) | Directed prompt (what to type instead) |
|---|---|
| *"Preprocess the AIA 171 and HMI images for 2014 and make a training tensor."* | *"Read `@notebooks/01_sdo_ingest_and_l15_prep.ipynb` and `@src/sdoflare/prep.py`. Which `sdoflare` functions take AIA L1 and HMI LOS files in `data/sdo_2014/` to a co-aligned $512\times 512$ cube? List the functions in order with `file:line`, state whether `correct_aia_degradation` divides by `EXPTIME` internally, and wait for my approval before writing code."* |
| *"Train a model to predict M flares and print the accuracy."* | *"Using `sdoflare.models.LinearProbeForecaster` and `sdoflare.labels.carrington_split` (cite `@notebooks/04_embedding_finetune_and_probing.ipynb`), propose a plan to evaluate 24h $\ge\text{M1.0}$ forecasting on `data/embeddings/sdo_fulldisk_embeddings_6h_2011_2018.zarr` using TSS, HSS, and Brier Skill Score against `PersistenceBaseline`. Do not report raw accuracy. Wait for my approval."* |
| *"Turn my notebook into a skill."* | *"Read `@skills/sdo-ar-event-attribution/SKILL.md`, `@templates/SKILL_TEMPLATE.md`, `@work/elena/src/channel_variants.py`, and `@work/elena/ERROR_LOG.md`. First brainstorm with me the inputs, CLI subcommands, mandatory `sdoflare` guardrails, and failure modes for a reusable `sdo-euv-channel-ablation` skill so another scientist can run channel ablation on any date range or flare threshold. Do not write files until I approve the design."* |

---

## H1 · Module 0 — Setup & first contact (30 min)

1. `cd sdoflare-workshop && git checkout -b <name>` then `python env/check_setup.py` — everything green? Post ✅ in the channel.
2. Start the agent **in the repo root** (so it loads `GEMINI.md` / `AGENTS.md`). First prompt:

   ```text
   Summarize the rules you loaded from the project context file in 5 bullets, especially the SDO preprocessing and flare-evaluation rules. Then list the notebooks in @notebooks/README.md and the example skill in @skills/ in one line each. Do not run anything.
   ```

3. Verify the test suite:
   ```text
   Run `pytest -q tests/` and report the result. Do not modify any files.
   ```
4. Create `work/<name>/` with `notes/`, `src/`, `tests/`, `figures/`, and copy `templates/ERROR_LOG.md` into it.

---

## H2 · Module 1 — GROUND (60 min) · agent = tutor, read-only

Goal: build a *correct* mental model of `sdoflare` and calibrate how much to trust the agent's explanations.

### G1 · Map the toolkit (20 min)

**Template prompt:**
```text
Read the package source under @src/sdoflare/ and the notebook index @notebooks/README.md. Do not run anything.
Write work/<name>/notes/toolkit_map.md with:
1. A module-by-module summary (one line each).
2. The 10–15 public functions/classes a new user needs most: signature, inputs/outputs with physical units and coordinate frames, and the notebook that demonstrates each.
3. A table: science question → notebook → key functions.
4. Five mistakes a new user is likely to make, inferred from docstrings, warnings, asserts and TODOs in the code.
Cite file:line for every claim. If something is undocumented, write "undocumented" instead of guessing.
```

**Concrete follow-up questions a scientist asks right after G1:**
- **Elena (AIA channels & degradation):**
  ```text
  Look closely at @src/sdoflare/prep.py. When I call `aia_l1_to_l15()` followed by `correct_aia_degradation()`, does `aia_l1_to_l15()` already normalize by exposure time (`EXPTIME`), or does `correct_aia_degradation()` do it? Cite the exact lines in both functions so I don't divide by `EXPTIME` twice.
  ```
- **Marco (HMI full-disk geometry & limb mask):**
  ```text
  In @src/sdoflare/prep.py, how does `correct_hmi_projection()` compute $\mu = \cos(\theta)$ across the full solar disk, what happens to pixels with $\mu < 0.2$ or $R > R_\odot$, and what coordinate frame (`Helioprojective` vs `HeliographicStonyhurst`) does it expect in the FITS header? Cite file:line.
  ```
- **Priya (Model inputs & channel order):**
  ```text
  In @src/sdoflare/models.py, what exact channel order, image resolution, and intensity scaling does `FullDiskFlareNet.forward()` expect? Where in @notebooks/03_pretrained_fulldisk_flare_forecast.ipynb is that input tensor constructed?
  ```

→ **Your job:** open **three** of the `file:line` citations. Mark each ✅/❌ in `toolkit_map.md`. Log any ❌ in `ERROR_LOG.md`.

### G2 · Teach me one notebook (15 min)

**Template prompt:**
```text
Walk me through @notebooks/<nb>.ipynb cell by cell, for a solar physicist who is an intermediate Python programmer. For each cell: (a) what it does scientifically, (b) which `sdoflare` function it calls and what that function assumes (units, data level, coordinate frame, cadence, disk coverage), (c) one thing that would silently go wrong if I changed the input. Under 600 words. Do not execute anything.
```

**Concrete conversational follow-ups after G2:**
- *"In Cell 6 of `@notebooks/02_multichannel_coalignment_and_labels.ipynb`, why does `coalign_full_disk()` resample HMI to the AIA time grid instead of AIA to the HMI 720s grid? What is the maximum allowed timestamp mismatch (`max_dt`) before it drops a frame?"*
- *"In Cell 9 of `@notebooks/03_pretrained_fulldisk_flare_forecast.ipynb`, how are timestamps during SDO spring/autumn eclipse seasons handled when creating 24-hour flare labels? Does `label_flare_window()` return `NaN`, `0`, or drop the timestamp?"*

### G3 · Quiz me (10 min)

**Concrete prompt:**
```text
Quiz me on `sdoflare`'s full-disk preprocessing pipeline and @notebooks/01_sdo_ingest_and_l15_prep.ipynb + @notebooks/03_pretrained_fulldisk_flare_forecast.ipynb. Ask ONE question at a time (5 total), wait for my answer, then grade it and cite the exact code (`file:line`) or notebook cell. Include questions on: (1) AIA L1 vs L1.5 registration, (2) HMI LOS $\mu$-correction near the limb, (3) channel scaling before the neural network, (4) how 24h flare labels handle data gaps, and (5) why `carrington_split` is used instead of a random split.
```

### G4 · Personal cheat sheet (10 min)

**Concrete example (Elena — Case α):**
```text
My science case: I want to test whether adding AIA 94 Å and 131 Å to HMI B_los + AIA 171 Å improves 24h >=M1.0 flare forecasting across 2014–2017, and quantify how much skill changes if `correct_aia_degradation()` is omitted.
Using @work/elena/notes/toolkit_map.md, write @work/elena/notes/my_cheatsheet.md: the 6–8 `sdoflare` functions I will need, each with a minimal working snippet copied from `@notebooks/01_...` through `05_...` (cite notebook and cell). If you adapt a snippet instead of copying it verbatim, mark it "ADAPTED — verify".
```

**Exit ticket (no agent, 2 min):** (1) Which function loads and registers your main SDO product, and what units/frame does it return? (2) Name one physical or statistical assumption `sdoflare` makes that matters for your case.

---

## H3 · Module 2 — REPRODUCE (60 min) · agent = navigator

Goal: run the notebooks confidently; learn where the agent helps with debugging and where it misses domain bugs.

### R1 · Run it headless (10 min)
```text
Execute @notebooks/03_pretrained_fulldisk_flare_forecast.ipynb without modifying it:
papermill notebooks/03_pretrained_fulldisk_flare_forecast.ipynb work/<name>/runs/03_run.ipynb
Report: total runtime, every warning (including FITS header or sunpy warnings), and the final TSS, HSS, and Brier Skill Score printed at the end. Do not fix anything — just report.
```

### R2 · Perturb one thing (25 min)

**Concrete worked prompts (pick one style that matches your case):**

- **Example 1 — Changing the target event window (Sep 2017 X-flares vs Oct 2014 AR 12192):**
  ```text
  I want a variant of @notebooks/05_evaluation_attribution_and_events.ipynb that evaluates full-disk flare probability and active-region attribution on the October 2014 AR 12192 super-active region (`2014-10-18` to `2014-10-28`, available in `data/sdo_events/ar12192_201410/`) instead of the September 2017 AR 12673 event.
  Step 1 — before editing anything: list every cell that must change (file paths, date strings, Carrington rotation or NOAA AR number, threshold) and flag any `sdoflare` assumptions that could break (e.g., AIA degradation table epoch, missing channels, cadence). Wait for my approval.
  Step 2 — copy to `work/<name>/05_ar12192_variant.ipynb`, apply the changes one cell at a time, run with `papermill`, and summarize how the peak predicted probability and attribution map changed.
  ```
- **Example 2 — Changing forecast horizon or flare class threshold:**
  ```text
  I want a variant of @notebooks/04_embedding_finetune_and_probing.ipynb that predicts >=C5.0 flares in a 12-hour window (`horizon_hours=12, min_class="C5.0"`) instead of >=M1.0 in 24 hours.
  Step 1 — list every cell and `sdoflare` call that must change, and tell me how the positive-class base rate and decision threshold calibration in `compute_tss_hss()` will be affected. Wait for my approval.
  Step 2 — create `work/<name>/04_c5_12h_variant.ipynb`, execute with `papermill`, and report old vs new base rate, TSS, HSS, and BSS.
  ```

→ **Your job:** write 3 sentences in `notes/`: what you changed, how the output changed, and why that makes physical sense.

### R3 · Bug hunt (20 min)

The repo contains three broken notebooks in `bughunt/` that **run without throwing any Python exception** but produce scientifically invalid flare predictions:
- `bughunt/bug_A_high_skill.ipynb` — reports $\text{TSS} = 0.91$ on 24h $\ge\text{M1.0}$ forecasting.
- `bughunt/bug_B_cycle_drift.ipynb` — shows flare probability systematically dropping from 2011 to 2017 even for identical active regions.
- `bughunt/bug_C_quiet_limb.ipynb` — misses almost every East-limb M/X flare and mis-aligns AIA and HMI features by ~6 hours.

**Prompt to run on each:**
```text
@bughunt/bug_A_high_skill.ipynb runs without errors but produces a scientifically wrong solar flare prediction result. Compare it against @src/sdoflare/ and the reference notebooks in @notebooks/. Find the bug(s). For each candidate:
1. The physical or statistical consequence on the flare forecast.
2. Exact evidence in the notebook (cell and line) vs how `sdoflare` is supposed to be called.
3. A one-line fix.
Rank by confidence. Do not edit the file.
```

**Follow-up when the agent only spots superficial code issues:**
```text
You only flagged code style / minor issues. Look specifically for domain errors in SDO preprocessing and operational flare evaluation:
- How were train and test split relative to the 24h flare label window and 27-day solar rotation?
- Were AIA intensities corrected for exposure time and multi-year sensitivity degradation?
- Are timestamps all UTC observation times (`T_OBS` / `DATE-OBS`), or did file-download or local timestamps sneak in?
- What happens to HMI `B_los` values beyond `R/R_sun > 0.95` or when divided by `mu = cos(theta)` near the limb?
Re-check the notebook against that list.
```

**Exit ticket:** (1) What did you change in R2, how did the result change, and why physically? (2) Which bug in R3 did the agent catch immediately, and which one required your domain nudge?

---

## H4 · Module 3 — ARTICULATE (60 min) · agent = skeptical collaborator · **no code allowed**

Goal: turn your science case into a falsifiable, 5-hour plan mapped to `sdoflare` functions, with baseline and validation defined *before* writing code.

### A1 · Scope with a skeptic (25 min)

**Template prompt:**
```text
I am a <role> working on: <science question>. Full context: <paste your intake answers>.
Constraints: about 5 hours of hands-on time; only the data already in `data/` (see @data/README.md); I must use `sdoflare` wherever possible (see @work/<name>/notes/my_cheatsheet.md and @notebooks/README.md).
Act as a skeptical senior heliophysics collaborator. Do NOT write code.
1. Restate my question as a precise, falsifiable computational question: inputs → outputs → metric.
2. Propose a Minimum Viable Science (MVP) version achievable in 5 hours, and a stretch version.
3. Map each MVP step to `sdoflare` functions or notebooks (cite them). Flag steps `sdoflare` does not support.
4. List the validation checks: physical sanity checks, a baseline (persistence/climatology/reference model), the metric and why it fits the class balance, and every data-leakage or selection-bias risk.
5. Top 3 risks with a fallback each, and a cut list (what I drop first if I run late).
Write it to `work/<name>/PLAN.md` following @templates/PLAN_TEMPLATE.md. Ask me up to 3 clarifying questions first if my case is ambiguous.
```

**Concrete example (Marco — Case β, limb foreshortening):**
```text
I study eruptive active regions near the solar limb. I want to know how badly `FullDiskFlareNet`'s 24h >=M1.0 flare forecast skill degrades when the dominant active region is near the limb (|heliographic longitude| > 60 deg, mu < 0.5), where HMI B_los suffers projection foreshortening and apparent polarity reversal, and whether training a linear probe that upweights off-limb EUV corona channels (AIA 193, 211 Å between 0.95 and 1.15 R_sun) recovers limb skill.
Constraints: 5 hours hands-on; only the pre-staged data in `data/` (check @data/README.md to see what raw images vs precomputed embeddings exist for 2012–2017); use `sdoflare` functions.
Act as a skeptical senior collaborator. Do NOT write code. First ask me up to 3 clarifying questions (e.g., how we assign a full-disk prediction to a single longitude bin when multiple ARs are on disk), then draft `work/marco/PLAN.md`.
```

### A2 · Peer review (15 min)
Swap `PLAN.md` with a pod-mate. Optional referee prompt:
```text
Review @work/<peer>/PLAN.md as a solar physics journal referee.
1. Can the MVP actually be executed in 5 hours using only `data/` (check @data/README.md) and existing `sdoflare` functions?
2. Is there any hidden temporal leakage (e.g., 24h label autocorrelation, active regions rotating from East limb into disk center across splits)?
3. Is the baseline fair?
Be blunt and under 250 words.
```

### A3 · Plan gate
A facilitator signs off on `PLAN.md` (MVP ≤ 5 h; data in `data/` now; metric + baseline named; ≥ 3 validation checks; `sdoflare` functions per step; cut list).

### Template · `PLAN.md`
```markdown
# <name> — <short title>
## Science question
## Computational question (inputs → outputs → metric)
## Data (path in data/, instrument, level, time range, cadence, known gaps)
## MVP (≤ 5 h) — steps, each with the `sdoflare` function(s) and the notebook that shows it
1. …
## Stretch
## Baseline
## Validation checks (≥ 3: sanity/units/range, known event, baseline comparison, leakage audit, uncertainty)
## Leakage / circularity risks and how the protocol avoids them
## Risks & fallbacks (top 3)
## Cut list (first thing to drop, second, third)
## Candidate reusable skill name & what another scientist would pass as inputs
## Definition of done (the figure/table I will show tomorrow)
```

---

## H5 · Module 4 — SOLVE, sprint 1 (120 min) · agent = pair programmer

Goal: data loaded through `sdoflare`, a **first-light plot** by minute 90, and the first method step implemented in `work/<name>/src/` with sanity tests.

**The loop (repeat per `PLAN.md` step):** S1 → run → inspect → S3 → S4 → `git commit` → log misses.

### S1 · Implement one step (plan first, then code)

**Concrete example (Elena — building the channel-ablation preprocessing & feature step):**
```text
We are implementing Step 1 of @work/elena/PLAN.md: "Load co-aligned 6-hour cadence full-disk cubes for 2014–2017 from `data/cubes_512/` and construct three input variants: (A) HMI B_los + AIA 171, (B) HMI B_los + AIA 171/94/131 with `correct_aia_degradation`, and (C) HMI B_los + AIA 171/94/131 WITHOUT `correct_aia_degradation`."
Constraints:
- Use `sdoflare.io` and `sdoflare.prep` functions (see @work/elena/notes/my_cheatsheet.md). Search `src/sdoflare/` before writing anything new.
- Put reusable functions in `work/elena/src/channel_variants.py` with docstrings stating physical units (`G`, `DN s^-1 pix^-1`), coordinate frame, and channel order.
- Add a fast unit test in `work/elena/tests/test_channel_variants.py` on a 3-timestamp sample.
- The notebook only imports `channel_variants` and plots.
First show me a 6-line plan (exact `sdoflare` functions you will call, with `file:line` citations) and wait for my approval. Do not edit any files yet.
```

### S2 · First-light plot (must be done by minute 90)

**Concrete example (Elena — first-light multi-channel panel + degradation ratio):**
```text
Create my first-light diagnostic plot in `work/elena/figures/first_light.png`:
1. Top row: a 4-panel full-disk view at `2014-10-24T21:00:00Z` (just before the X3.1 flare in AR 12192) showing HMI `B_los` (clipped to +-1500 G, grayscale) and AIA 171, 94, 131 Å (asinh-scaled, in `DN s^-1 pix^-1`) using `sdoflare`'s plotting helpers if available. Draw the solar limb circle and label axes in arcseconds (`Helioprojective`).
2. Bottom row: disk-integrated median AIA 94 Å and 131 Å intensity from 2014 to 2017 BEFORE vs AFTER `correct_aia_degradation()`, with the GOES 1–8 Å daily background overplotted on the right axis.
Show me the code plan first, then run it and display the saved figure.
```

### S3 · Sanity check

**Concrete example:**
```text
Write and run `work/elena/tests/test_sanity.py` with three physical sanity checks on our prepared dataset:
1. Units & range assertion: after `correct_aia_degradation()` and before `scale_channels()`, all on-disk AIA 94/131/171 pixels are finite, non-negative, and 99th-percentile intensity is within `[1, 5e4] DN s^-1 pix^-1`; HMI on-disk `B_los` has `|median| < 5 G` (net flux balance check) and `max(|B_los|) in [500, 6000] G`.
2. Known-event check: on `2017-09-06T06:00:00Z` (6 hours before the X9.3 flare), the prepared 24h label `y_m10 == 1` and `peak_goes_flux >= 1e-4 W m^-2`.
3. Eclipse/gap check: timestamps flagged with `is_eclipse_gap == True` in `SDOTimeIndex` are excluded from evaluation rather than labeled `y_m10 == 0`.
Explain what physical or pipeline bug each check would catch, then run `pytest work/elena/tests/test_sanity.py`.
```

### S4 · Explain back (before committing any non-trivial step)
```text
Explain `work/<name>/src/<module>.py` as if I have to defend it at an SHINE or AGU session tomorrow:
1. Every physical and statistical assumption it makes.
2. Anything you wrote that is NOT directly calling a `sdoflare` function.
3. One subtle way this code could still be wrong even though `pytest` passed.
```

### S5 · Stuck protocol (after 3 failed fix attempts)
```text
Stop changing code. Summarize in 10 lines:
1. What we are trying to accomplish.
2. The three attempts we made and the exact error message or unexpected plot each produced.
3. Three hypotheses ranked by likelihood (check FITS header WCS keys, `sunpy` map metadata, array shape `[C,H,W]` vs `[H,W,C]`, and `NaN` propagation from off-limb pixels), each with a 1-minute command to test it.
```

### S6 · End-of-day handoff (Day 1, 17:15)
```text
Write `work/<name>/HANDOFF.md`: status vs `PLAN.md`, what works (with the exact command to reproduce our plots/numbers), what is broken, open questions for tomorrow morning, the next 3 steps, and any code in `work/<name>/src/` that is agent-written and not yet verified by me.
```

### Template · `ERROR_LOG.md` (with realistic SDO examples pre-filled)

| time | task | what the agent claimed/did | how I noticed | cost if unnoticed | fix | category |
|---|---|---|---|---|---|---|
| 10:55 | G1 map | Claimed `aia_l1_to_l15()` corrects time-dependent channel degradation | Opened `src/sdoflare/prep.py:42` — it only does pointing/roll/plate-scale; degradation is in `correct_aia_degradation()` at line 118 | Secular downward drift in EUV intensity from 2011→2017 | Called `correct_aia_degradation()` explicitly | hallucinated API / doc misread |
| 14:20 | S1 prep | Divided AIA image by `header['EXPTIME']` *after* `correct_aia_degradation(..., normalize_exptime=True)` | First-light plot showed AIA 171 values ~0.5 instead of ~150 $\text{DN}\,\text{s}^{-1}\,\text{pix}^{-1}$ (`EXPTIME` ≈ 2 s) | All EUV channels scaled wrong by $1/\text{EXPTIME}$ before feeding pretrained net | Removed manual `/ exptime` | wrong units/time |
| 15:35 | S1 split | Used `sklearn.model_selection.train_test_split(shuffle=True)` on 6-hour SDO cubes | Asked in S4 explain-back how splits were made | AR 12192 frames 6 hours apart in both train and test → fake TSS = 0.89 | Replaced with `sdoflare.labels.carrington_split(buffer_days=27)` | wrong statistics / leakage |

---

## H6 · Module 5 — SOLVE sprint 2 + PROVE (90 min) · agent = hostile referee

Finish your MVP in the first 45 minutes, then **stop building and switch to proving**.

### P1 · Baseline comparison
```text
Using the exact same Carrington-rotation test split, 24h >=M1.0 labels, and timestamps as my method, run:
1. `sdoflare.models.PersistenceBaseline` (24h persistence of prior 24h max GOES flare class / probability),
2. `sdoflare.models.ClimatologyBaseline`, and
3. The unmodified pretrained `FullDiskFlareNet` baseline from @notebooks/03_pretrained_fulldisk_flare_forecast.ipynb.
Produce a comparison table (`work/<name>/results_table.md`) with TSS (at the threshold chosen on the validation set ONLY via `find_optimal_tss_threshold`, never tuned on the test set!), HSS, Brier Skill Score (relative to climatology), and false-alarm ratio. Do not tune the baselines.
```

### P2 · Leakage & solar-physics circularity audit
```text
Audit `@work/<name>/src/` and my notebook for data leakage and solar-physics circularity. Check every item below and cite `file:line`:
1. Temporal / AR persistence leakage: can images of the SAME solar active region appear in both train and test (i.e., is there at least a 27-day / 1-Carrington-rotation buffer at every split boundary)?
2. Label-window overlap: for an issue time `t0` with forecast window `[t0, t0+24h]`, do any input channels, rolling statistics, or GOES background features use data after `t0`?
3. Flare-in-progress contamination: if an M/X flare is ALREADY peaking in AIA 94/131 Å at `t0`, is the model merely detecting an ongoing flare rather than forecasting a future one? How does skill change if we exclude timestamps where GOES 1–8 Å flux at `t0` is already above C5.0?
4. Normalization & threshold leakage: were channel scalers and the probability threshold that maximizes TSS fitted strictly on the training/validation split and frozen for the test split?
5. Eclipse / missing-data leakage: how were SDO eclipse gaps handled?
Do not modify code; output a severity-ranked table.
```

### P3 · Known-event & quiet-Sun plausibility check
```text
Run my complete pipeline on two benchmark intervals in `data/sdo_events/` and plot `plot_event_timeline()` for both:
1. Active benchmark: September 4–10, 2017 (NOAA AR 12673, including the X2.2 and X9.3 flares on Sep 6 and the X8.2 limb flare on Sep 10). Also run `attribute_disk_regions()` at `2017-09-06T06:00Z` — does the attribution peak on AR 12673 rather than on quiet-Sun network or the solar limb artifact?
2. Quiet-Sun negative control: a 7-day solar-minimum / spotless interval in `data/sdo_events/quiet_201905/`. Does the predicted >=M1.0 probability stay low (< 0.05), or does my model produce chronic false alarms?
Report where the model behaves physically and where it fails.
```

### P4 · Hostile referee
```text
You are a hostile Space Weather / ApJ referee reviewing `@work/<name>/PLAN.md`, my code in `@work/<name>/src/`, and `@work/<name>/results_table.md`.
Give the 5 strongest scientific or statistical objections to my claim that "<state your main finding in 1 sentence>".
Consider: sample size of independent flaring active regions (not just number of 6-hour frames!), flare-in-progress contamination in EUV 94/131 Å, center-to-limb projection bias, threshold tuning, and calibration/reliability.
For each objection, propose one cheap test (< 15 minutes with the data in `data/`) that could refute or confirm the objection. Be blunt.
```
→ **Your job:** pick at least **one** of those cheap tests and run it now.

### P5 · Uncertainty (block-bootstrapped by active region / Carrington rotation)
```text
Add 95% confidence intervals to my TSS, HSS, and Brier Skill Score in `results_table.md` and on my reliability diagram (`work/<name>/figures/main_result.png`) using `sdoflare.eval.block_bootstrap_metric` over Carrington rotations (NOT i.i.d. resampling of individual 6-hour timestamps). State clearly in the figure caption what this uncertainty captures and what it does not.
```

---

## H7 · Module 6 — PACKAGE, SKILLIFY & PRESENT (90 min) · agent = skill architect & editor

Goal: leave behind (1) a clean, reproducible notebook and findings summary, and (2) a **reusable Agent Skill (`skills/<skill-name>/SKILL.md` + CLI script)** encapsulating the science workflow you just validated so other scientists (and their coding agents) can run it on new solar events, date ranges, or thresholds in a single prompt.

### X1 · Clean & reproduce (10 min)
```text
Restructure my notebook into 6 clean sections: Setup → SDO Data & Preprocessing → Method → Results vs Baselines → Physical & Statistical Validation → Next Steps. Remove dead scratch cells, ensure all heavy logic is imported from `work/<name>/src/`, and verify a clean headless run:
papermill work/<name>/analysis.ipynb work/<name>/runs/final_verified.ipynb
Then write `work/<name>/README.md` showing how to reproduce the final figure and table in 3 commands.
```

### X2 · One-page findings (10 min)
```text
Write `work/<name>/FINDINGS.md` (max 1 page):
- Science question & computational formulation
- SDO datasets, channels, time ranges, and `sdoflare` functions used
- Main result table (my method vs Persistence, Climatology, and unmodified `FullDiskFlareNet`, with Carrington-block bootstrap 95% CIs)
- Main figure (`figures/main_result.png`) + known-event check on AR 12673 / AR 12192
- Validation summary: leakage audit outcome + which hostile-referee test we ran and what happened
- Limitations & next steps after the workshop
```

### X3 · Distill your workflow into a reusable Agent Skill (25 min)

Why create a skill? A notebook shows *one* run; an **Agent Skill** (`skills/<skill-name>/SKILL.md` + `scripts/<skill_cli>.py`) teaches any coding agent **when** to trigger your methodology, **how** to run your parameterized pipeline via CLI commands, and **which physical/statistical traps to avoid** (taken straight from your `ERROR_LOG.md` and `P2` audit).

#### Step 1 — Brainstorm & design the skill (no files written yet)

**Template prompt:**
```text
We are going to distill the science workflow we just built and validated in `@work/<name>/src/`, `@work/<name>/PLAN.md`, `@work/<name>/FINDINGS.md`, and `@work/<name>/ERROR_LOG.md` into a reusable agent skill in `skills/<skill-name>/`, following the gold-standard reference `@skills/sdo-ar-event-attribution/SKILL.md` and `@templates/SKILL_TEMPLATE.md`.

Phase 1 (Brainstorm & Design — DO NOT write any skill files yet):
1. Propose a hyphenated skill name (`<= 64` chars) and a sharp YAML `description` stating both WHEN to trigger this skill and WHEN NOT to use it.
2. List the configurable inputs another scientist would want to change (e.g., date range, SDO channels, forecast horizon, GOES flare class threshold, active-region longitude bin, output path) vs. the steps that MUST remain strictly fixed (e.g., `correct_aia_degradation`, `carrington_split(buffer_days=27)`, validation-only threshold selection, baselines).
3. Propose a self-contained CLI script `skills/<skill-name>/scripts/<skill_cli>.py` (wrapping our functions in `work/<name>/src/`) with 2–3 `argparse` subcommands that write JSON/markdown/figure outputs to `--output-dir` (never dumping large tables to stdout).
4. Extract the top 3–4 domain pitfalls from `@work/<name>/ERROR_LOG.md` and our `P2` leakage audit to put in the skill's `## Common Mistakes & Domain Guardrails` section.
5. Propose one natural-language test prompt and expected output that a pod-mate can use to verify the skill in a fresh agent session.
Ask me up to 3 clarifying questions and wait for my approval of this design.
```

**Concrete worked examples of Step 1 for our three scientist personas:**

- **Elena (Case α → `sdo-euv-channel-ablation` skill):**
  ```text
  Read `@skills/sdo-ar-event-attribution/SKILL.md`, `@templates/SKILL_TEMPLATE.md`, `@work/elena/src/channel_variants.py`, and `@work/elena/ERROR_LOG.md`.
  I want to package my EUV channel-ablation and AIA degradation-sensitivity workflow into a reusable skill named `sdo-euv-channel-ablation` so any scientist can test adding/removing AIA EUV channels (94, 131, 171, 193, 211, 304, 335 Å) on top of HMI `B_los` for any date range, horizon, and GOES flare threshold, and automatically quantify how much skill changes with vs. without `correct_aia_degradation()`.
  Propose the skill design (YAML frontmatter, CLI subcommands `prepare-variants`, `evaluate-ablation`, and `plot-degradation-impact`, mandatory `sdoflare` guardrails from my `ERROR_LOG.md`, and a verification query). Wait for my approval before creating files.
  ```
- **Marco (Case β → `sdo-limb-foreshortening-audit` skill):**
  ```text
  Read `@skills/sdo-ar-event-attribution/SKILL.md`, `@templates/SKILL_TEMPLATE.md`, `@work/marco/src/limb_stratify.py`, and `@work/marco/ERROR_LOG.md`.
  Design a reusable skill named `sdo-limb-foreshortening-audit` that stratifies any full-disk flare forecaster's skill by active-region heliographic longitude (`|lon|` or `mu = cos(theta)` bins) and evaluates whether off-limb EUV coronal ring features (`0.95 < R/R_sun < 1.15`) recover limb recall. Propose the YAML frontmatter, CLI subcommands, domain guardrails, and test prompt. Do not write files yet.
  ```
- **Priya (Case γ → `sdo-preflare-flux-probe` skill):**
  ```text
  Read `@skills/sdo-ar-event-attribution/SKILL.md`, `@templates/SKILL_TEMPLATE.md`, `@work/priya/src/flux_probe.py`, and `@work/priya/ERROR_LOG.md`.
  Design a reusable skill named `sdo-preflare-flux-probe` that trains a lightweight probe on precomputed `FullDiskFlareNet` embeddings to predict continuous peak `log10(GOES 1-8 A flux)` over a configurable horizon (`--horizon-hours`), automatically running a "quiet-background pre-flare" filter (`--max-initial-goes-class C2.0`) to separate true pre-flare forecasting from flare-in-progress detection. Propose the design and wait for my approval.
  ```

#### Step 2 — Implement & validate the skill

Once you approve the design, run:
```text
Approved. Now implement the skill in `skills/<skill-name>/`:
1. Create `skills/<skill-name>/SKILL.md` following `@templates/SKILL_TEMPLATE.md`.
2. Create `skills/<skill-name>/scripts/<skill_cli>.py` following `@templates/skill_cli_template.py` (with `argparse` subcommands, explicit required flags, `--output-dir`, and non-zero exit codes on domain violations).
3. Run `python env/validate_skill.py skills/<skill-name>` to validate the frontmatter, required sections, `--help` execution, and register the skill symlinks in `.gemini/skills/`, `.claude/skills/`, and `.agents/skills/`.
4. Report the validation output.
```

#### Concrete example of a finished `SKILL.md` (`skills/sdo-euv-channel-ablation/SKILL.md`)
```markdown
---
name: sdo-euv-channel-ablation
description: >-
  Quantifies the impact of adding or removing SDO/AIA EUV channels (94, 131, 171,
  193, 211, 304, 335 A) alongside SDO/HMI B_los on operational solar flare
  prediction skill (TSS, HSS, BSS), and audits sensitivity to time-dependent AIA
  channel degradation (`correct_aia_degradation`). Use when a scientist asks to
  compare SDO channel combinations, test whether hot flaring channels (94/131 A)
  help forecast C/M/X flares, or check if uncorrected EUV degradation biases
  multi-year flare predictions. Do not use for single-event spatial attribution
  (use `sdo-ar-event-attribution`) or raw Level 1 FITS ingestion alone.
---

# SDO EUV Channel Ablation & Degradation Sensitivity Skill

## Overview
Evaluates how different subsets of full-disk SDO/AIA EUV channels combined with SDO/HMI `B_los` affect operational flare forecasting skill across a configurable date range, forecast horizon, and GOES flare class threshold, with a built-in ablation comparing degradation-corrected vs. uncorrected EUV intensities and filtering out flare-in-progress timestamps.

## Dependencies & `sdoflare` Functions Used
- `sdoflare.prep.correct_aia_degradation`, `sdoflare.prep.scale_channels`
- `sdoflare.labels.label_flare_window`, `sdoflare.labels.carrington_split`
- `sdoflare.models.LinearProbeForecaster`, `sdoflare.models.PersistenceBaseline`, `sdoflare.models.ClimatologyBaseline`
- `sdoflare.eval.find_optimal_tss_threshold`, `sdoflare.eval.compute_tss_hss`, `sdoflare.eval.brier_skill_score`, `sdoflare.eval.block_bootstrap_metric`

## Quick Start
```bash
python skills/sdo-euv-channel-ablation/scripts/euv_ablation_cli.py run-ablation \
  --data-path data/embeddings/sdo_fulldisk_embeddings_6h_2011_2018.zarr \
  --base-channels hmi_blos aia_171 \
  --test-channels aia_94 aia_131 \
  --start-date 2014-01-01 --end-date 2017-12-31 \
  --horizon-hours 24 --min-flare-class M1.0 \
  --exclude-ongoing-above C5.0 \
  --output-dir work/elena/runs/ablation_24h_m10
```

## Utility Scripts & Subcommands (`scripts/euv_ablation_cli.py`)
1. `run-ablation`: Trains and evaluates `LinearProbeForecaster` on (a) `--base-channels`, (b) `--base-channels + --test-channels` (degradation-corrected), and (c) `--base-channels + --test-channels` (uncorrected degradation, if cubes provided), alongside `PersistenceBaseline` and `ClimatologyBaseline`. Writes `metrics.json`, `comparison_table.md` (with Carrington-rotation block-bootstrap 95% CIs), and `ablation_figure.png` to `--output-dir`.
2. `check-ongoing-contamination`: Re-evaluates a saved `ablation` run after excluding timestamps where GOES 1–8 Å flux at issue time `t0` exceeds `--exclude-ongoing-above` (default `C5.0`).

## Domain Guardrails (Mandatory)
- **Carrington Split Buffer**: Always splits train/val/test with `carrington_split(buffer_days=27.0)`. Never uses random or unbuffered splits.
- **Validation-Only Threshold Selection**: Calls `find_optimal_tss_threshold` on the validation split only and freezes that threshold for the test split.
- **Single `EXPTIME` Normalization**: Relies on `correct_aia_degradation(..., normalize_exptime=True)` — never divides by `EXPTIME` manually.

## Common Mistakes (from Workshop Error Logs)
1. **Dividing AIA by `EXPTIME` twice**: `correct_aia_degradation()` already normalizes to `DN s^-1 pix^-1`. Dividing by `header['EXPTIME']` again scales EUV channels down by ~2–3x.
2. **Confusing flare-in-progress detection with pre-flare forecasting**: AIA 94/131 Å brighten dramatically during ongoing C/M flares at `t0`. Always check `check-ongoing-contamination --exclude-ongoing-above C5.0` before claiming hot channels improve pre-flare lead time.
3. **Resampling individual 6h frames for error bars**: Consecutive frames of the same active region are highly correlated; always use Carrington-rotation block bootstrap (`block_bootstrap_metric`).
```

### X4 · Fresh-session / pod-mate skill verification (15 min)

A skill only counts if **another** agent session (or your pod-mate) can use it without you hand-holding it.
1. Open a **brand-new coding agent session** in the repo root (or swap branches with a pod-mate).
2. Type a natural-language science request that slightly perturbs your parameters (e.g., different year range, flare threshold, or horizon):

   **Concrete verification prompt (testing `sdo-euv-channel-ablation` in a fresh session):**
   ```text
   Use the `sdo-euv-channel-ablation` skill to test whether adding AIA 94 Å and 131 Å to HMI B_los + AIA 171 Å improves 12-hour >=C5.0 flare forecasting across 2015–2016, including the check that excludes ongoing flares above C2.0 at t0. Save all outputs to `work/elena/runs/skill_verify_12h_c50/` and summarize the comparison table.
   ```

3. Verify that the fresh agent:
   - automatically discovers and reads `skills/<skill-name>/SKILL.md`,
   - runs `scripts/<skill_cli>.py` with the right flags,
   - produces `metrics.json`, `comparison_table.md`, and the figure in `--output-dir` without writing ad-hoc code!

### X5 · Toolkit feedback harvest (5 min)
```text
From this session and `@work/<name>/ERROR_LOG.md`, list every place where `sdoflare` lacked a needed function, had an ambiguous docstring (units, coordinate frames, `EXPTIME`/degradation handling), or where its API confused the coding agent. Format each as a short GitHub issue: Title, Reproduction, Expected, Actual, Suggested API/docstring improvement.
```

### Template · 5-slide lightning talk (5 min)
1. **Question** — one sentence, and why it matters physically/operationally.
2. **Toolkit pipeline** — which `sdoflare` functions and notebooks you built on (1 diagram or 4 bullets).
3. **Result + Baseline** — main figure with Carrington-block error bars + baseline comparison table.
4. **Validation + Agent Story** — one hostile-referee test you ran, **one error the agent made and how you caught it**, and one thing the agent accelerated.
5. **Reusable Skill + Toolkit Wishes** — your new skill's name (`skills/<skill-name>/`), the 1-sentence prompt any scientist in the room can now use to invoke it on their own events/dates, and one wish for `sdoflare`.

### Template · `HANDOFF.md`
```markdown
# Handoff — <name> — <date>
## Status vs PLAN.md
## Works (command to reproduce)
## Reusable Skill created (`skills/<skill-name>/SKILL.md` + verification command)
## Broken / unverified (mark agent-written code you have not checked)
## Open questions
## Next 3 steps
```

---

## H8 · Organizer templates (concrete `sdoflare` version)

### Repo context file — `GEMINI.md` (symlink `AGENTS.md`, `CLAUDE.md`)

```markdown
# sdoflare — Agent Context & Domain Rules

## What this repo is
`sdoflare` is a Python toolkit for preprocessing full-disk SDO/AIA (EUV) and SDO/HMI (photospheric magnetogram)
observations and running solar flare prediction models evaluated against GOES XRS flare catalogs.
- `notebooks/README.md` maps science questions → notebooks → `sdoflare` functions.
- `docs/CHEATSHEET.md` lists public signatures, units, coordinate frames, and expected tensor shapes.
- `skills/` contains reusable Agent Skills built on `sdoflare` (see `skills/sdo-ar-event-attribution/SKILL.md` and `templates/SKILL_TEMPLATE.md`).

## Mandatory workflow rules
- TOOLKIT FIRST: Always search `src/sdoflare/`, `notebooks/`, and `skills/` before writing new code. Never write custom FITS
  loaders, custom AIA degradation tables, custom reprojection loops, or custom TSS/HSS functions when `sdoflare`
  provides them.
- CITE SOURCES: Every explanation or plan must cite `src/sdoflare/<file>.py:<line>` and the relevant notebook cell.
  Never invent function or keyword names; verify with `python -c "import sdoflare; help(...)"` if unsure.
- PLAN BEFORE CODE: Present a short plan and wait for user approval before creating or editing files.
- CODE IN `.py`, PLOTS IN `.ipynb`: Put reusable logic in `work/<name>/src/*.py` with docstrings stating units,
  coordinate frames, and data levels, plus a test in `work/<name>/tests/`. Keep notebooks thin.
- SKILL DISTILLATION: When asked to package a workflow into a skill (`X3`), follow `templates/SKILL_TEMPLATE.md` and
  `templates/skill_cli_template.py`, wrap the validated `.py` logic in an `argparse` CLI script that writes outputs to
  `--output-dir`, include domain guardrails and `ERROR_LOG.md` pitfalls, and validate with `python env/validate_skill.py`.
- NO DOWNLOADS: Never call VSO, JSOC, Fido, or external URLs. Use only pre-staged files in `data/` (see `data/README.md`).

## SDO & solar flare domain conventions (NEVER VIOLATE)
1. Time standard: All timestamps are UTC observation times (`T_OBS` / `DATE-OBS`), timezone-aware or `astropy.time.Time`.
   Never mix file creation time with observation time. State cadence explicitly.
2. SDO/AIA preprocessing (`sdoflare.prep`):
   - Level 1 → Level 1.5 (`aia_l1_to_l15`) registers pointing, rolls solar north to the top of the array (`CROTA2 -> 0`),
     and rescales plate scale to 0.6 arcsec/pixel (at 4096x4096) or target resolution. It does NOT correct degradation.
   - `correct_aia_degradation(map, normalize_exptime=True)` divides by `EXPTIME` AND applies the time-dependent AIA
     sensitivity table, returning units of `DN s^-1 pix^-1`. Never divide by `EXPTIME` a second time.
   - EUV intensities span 4–5 orders of magnitude: use `scale_channels()` (`asinh` or `log10` scaling) before model input.
3. SDO/HMI preprocessing (`sdoflare.prep`):
   - `B_los` is in Gauss (`G`). Near the solar limb (`mu = cos(theta) -> 0`), line-of-sight projection foreshortens
     active regions and causes apparent polarity reversal. `correct_hmi_projection()` divides by `mu` down to `mu_min=0.2`
     and masks pixels with `R/R_sun > 0.95` to `NaN` (or `fill_value=0.0` after scaling).
4. Multi-channel co-alignment (`coalign_full_disk`):
   - AIA and HMI have different native cadences (12s/24s vs 45s/720s) and slightly different boresights. Always use
     `coalign_full_disk(..., max_dt=180*u.s)` rather than stacking raw arrays by index.
5. Gaps & eclipses:
   - SDO experiences daily eclipses during spring/autumn eclipse seasons and occasional calibration rolls (bad `QUALITY`
     bitmask). Never linearly interpolate images across gaps > 30 minutes, and never label an eclipse gap as a quiet (`0`)
     flare window.
6. Operational flare evaluation (`sdoflare.labels`, `sdoflare.eval`):
   - Splits MUST be chronological with a solar-rotation buffer (`carrington_split(buffer_days=27)`) so the same active
     region never appears in both train and test. Never use `train_test_split(shuffle=True)`.
   - For issue time `t0` and forecast window `[t0, t0 + horizon]`, no feature, scaler, or threshold may use data after `t0`.
   - Select probability thresholds on the validation split only (`find_optimal_tss_threshold`), never on the test split.
   - Always report True Skill Statistic (TSS), Heidke Skill Score (HSS), and Brier Skill Score (BSS) alongside
     `PersistenceBaseline` and `ClimatologyBaseline`. Never report raw accuracy (M/X flares are rare, ~1–5% base rate).
   - Compute confidence intervals via block bootstrap over Carrington rotations or active regions, not i.i.d. frames.
```

---

## H9 · Troubleshooting card — when the agent…

| …does this | do this |
|---|---|
| Invents an `sdoflare` function or kwarg | Run `python -c "import sdoflare.<mod> as m; help(m)"`; re-prompt with `@src/sdoflare/<mod>.py`; log it |
| Writes raw `astropy.io.fits` / `sunpy.map.Map` loops instead of using `sdoflare` | *"Stop. Search `src/sdoflare/` for the function that handles this and cite `file:line`."* |
| Divides AIA by `EXPTIME` twice or skips degradation | Point it at `@src/sdoflare/prep.py` and run sanity check **S3** on median disk intensity (should be ~$10^1\text{–}10^3\,\text{DN}\,\text{s}^{-1}\,\text{pix}^{-1}$) |
| Uses `train_test_split(shuffle=True)` or zero buffer between splits | Replace with `sdoflare.labels.carrington_split(buffer_days=27)` and run audit **P2** |
| Reports 97% accuracy or tunes the TSS threshold on the test set | Require `compute_tss_hss()` with the threshold fixed on the validation split, plus `PersistenceBaseline` (**P1**) |
| Writes a `SKILL.md` (`X3`) that hardcodes your specific dates/paths or has no CLI script | Require an `argparse` CLI script in `skills/<name>/scripts/` with `--start-date`, `--end-date`, `--output-dir` and run `python env/validate_skill.py skills/<name>` |
| Breaks `.ipynb` JSON | Keep functions in `work/<name>/src/*.py`; have the notebook only call them; `git checkout` the broken `.ipynb` |
| Loops on the same failed fix 3 times | Run stuck prompt **S5**; check FITS header / array shape yourself; call a facilitator |

