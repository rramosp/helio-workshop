# Heliophysics Toolkit Workshop — Toolkit Developer Guide

> [!IMPORTANT]
> **About the worked examples (`sdoflare`):** Throughout this Developer Guide (as in the companion **Facilitator Guide** and **Participant Handouts**), all concrete code snippets, docstrings, file trees, data schemas, skill templates, and tests use an **imaginary Python package called `sdoflare`** — a hypothetical toolkit for preprocessing full-disk **SDO/AIA** (EUV) and **SDO/HMI** (magnetogram) images and running **operational solar flare prediction models** evaluated against the **GOES XRS** flare catalog.
>
> Use `sdoflare` as a concrete reference implementation: replace its module names, physical units, coordinate frames, and domain checks with those of your actual heliophysics toolkit.

---

## 1. Purpose of this guide

The **Facilitator Guide** and **Participant Handouts** assume that your repository is **agent-ready**: when a scientist points their coding agent at `@src/<toolkit>/`, `@notebooks/README.md`, `@data/README.md`, or `@skills/` and runs prompt cards `G1`–`X5`, the agent can discover the right functions, cite exact `file:line` locations, execute notebooks headlessly, hit loud runtime errors whenever it violates a domain convention, and **distill the participant's validated workflow into a reusable Agent Skill (`skills/<skill-name>/`)** for other science users.

If a toolkit has implicit units, monolithic 300-line notebook cells, undocumented functions, no skill templates, or requires live JSOC/VSO downloads, **the coding agent will bypass your toolkit and hallucinate broken boilerplate.**

This document specifies the **mandatory (`MUST`)** and **recommended (`SHOULD`)** engineering requirements a toolkit developer must satisfy before Day 1 of the workshop.

### Traceability: how workshop prompts depend on repo deliverables

| Workshop module & prompt cards | What the participant / agent does | What the developer **MUST** provide in the repo |
|---|---|---|
| **M0 · Setup** | Runs `check_setup.py`, loads context rules, runs `pytest` | `env/check_setup.py`, `GEMINI.md` (+ `AGENTS.md`/`CLAUDE.md` symlinks), fast `tests/` ($< 30\,\text{s}$) |
| **M1 · GROUND** (`G1`–`G4`) | Maps `@src/sdoflare/`, reads `@notebooks/README.md`, checks `file:line` citations, builds cheat sheet | Clean public API (`__all__`), structured docstrings with units/frames/assumptions, `notebooks/README.md` index table, `docs/CHEATSHEET.md` |
| **M2 · REPRODUCE** (`R1`–`R3`) | Runs notebooks via `papermill`, perturbs 1 input variable, diagnoses 3 broken notebooks | `papermill`-ready `parameters` cell in every tutorial notebook; 3 pre-built broken notebooks in `bughunt/` + facilitator answer key |
| **M3 · ARTICULATE** (`A1`–`A3`) | Scopes 5-hour MVP & target skill name using `@data/README.md` and `@templates/PLAN_TEMPLATE.md` | 3-tier offline `data/` + `data/README.md`; `templates/PLAN_TEMPLATE.md`; 3 ready-to-run `backup_cases/` |
| **M4 · SOLVE** (`S1`–`S6`) | Writes `.py` step + unit test, generates First-Light plot by min 90, writes `HANDOFF.md` | Plotting helpers with units/frames; runtime guardrails that fail fast on bad units/time/splits; `templates/ERROR_LOG.md` & `HANDOFF.md` |
| **M5 · PROVE** (`P1`–`P5`) | Runs baselines, leakage audit, known-event & quiet-Sun checks, block bootstrap | First-class `PersistenceBaseline`, `ClimatologyBaseline`, `carrington_split`, threshold-safe `compute_tss_hss`, and `block_bootstrap_metric` in the package |
| **M6 · PACKAGE & SKILLIFY** (`X1`–`X5`) | Clean `papermill` run, `FINDINGS.md`, **distills workflow into `skills/<skill-name>/` (`X3`)**, **verifies skill in fresh session (`X4`)**, toolkit issues | Reference skill `skills/sdo-ar-event-attribution/`; `templates/SKILL_TEMPLATE.md` & `templates/skill_cli_template.py`; `env/validate_skill.py` (validator + `.gemini/skills/`, `.claude/skills/`, `.agents/skills/` registrar) |

---

## 2. R1 · Mandatory repository layout

The developer **MUST** structure the workshop repository to match the exact relative paths referenced in the Participant Handouts:

```text
sdoflare-workshop/
├── GEMINI.md                        # Primary agent context & domain rules (MUST be <= 150 lines)
├── AGENTS.md -> GEMINI.md           # Symlink for Codex / Cursor / Windsurf / generic agents
├── CLAUDE.md -> GEMINI.md           # Symlink for Claude Code
├── README.md                        # Quickstart, workshop schedule, and link to handouts
├── pyproject.toml                   # Editable install (`pip install -e .`) + pinned dependencies
├── env/
│   ├── environment.yml              # Conda/mamba lockfile (or uv.lock / requirements.txt)
│   ├── check_setup.py               # Mandatory 10-second pre-flight verification script (§9)
│   └── validate_skill.py            # Mandatory validator & multi-agent registrar for participant skills (§8)
├── src/
│   └── sdoflare/                    # The toolkit package (MUST be importable as `import sdoflare`)
│       ├── __init__.py              # Exposes top-level version and core submodules
│       ├── io.py                    # Data & catalog loaders (`load_aia_map`, `load_hmi_los`, ...)
│       ├── prep.py                  # Calibration, registration, co-alignment, scaling
│       ├── labels.py                # Operational window labeling & `carrington_split`
│       ├── models.py                # Pretrained models, embedding extractors, probes, baselines
│       └── eval.py                  # `compute_tss_hss`, `brier_skill_score`, block bootstrap, plots
├── notebooks/
│   ├── README.md                    # MANDATORY index: Science Question -> Notebook -> Functions -> Tier
│   ├── 01_sdo_ingest_and_l15_prep.ipynb
│   ├── 02_multichannel_coalignment_and_labels.ipynb
│   ├── 03_pretrained_fulldisk_flare_forecast.ipynb
│   ├── 04_embedding_finetune_and_probing.ipynb
│   └── 05_evaluation_attribution_and_events.ipynb
├── skills/                          # Community Agent Skills catalog (pre-installed + participant-built in M6)
│   ├── README.md                    # Index of available skills and how agents discover them
│   └── sdo-ar-event-attribution/    # Gold-standard reference skill shipped by the developer (§8)
│       ├── SKILL.md
│       └── scripts/
│           └── ar_attribution_cli.py
├── .gemini/
│   ├── commands/                    # Optional: prompt cards packaged as slash commands
│   └── skills/                      # Symlinks to `../../skills/*` (auto-managed by `env/validate_skill.py`)
├── .claude/
│   └── skills/                      # Symlinks to `../../skills/*` for Claude Code
├── .agents/
│   └── skills/                      # Symlinks to `../../skills/*` for Codex / Antigravity
├── docs/
│   └── CHEATSHEET.md                # 15–20 public functions with signatures, units, 3-line examples
├── data/                            # Pre-staged offline dataset (3 tiers; see §5)
│   ├── README.md                    # Exact file paths, schemas, units, time ranges, eclipse gaps
│   ├── catalogs/
│   ├── embeddings/                  # Tier 1: fast CPU embeddings (< 250 MB)
│   ├── cubes_512/                   # Tier 2: downsampled co-aligned cubes
│   └── sdo_events/                  # Tier 3: benchmark active & quiet-Sun event slices
├── bughunt/                         # 3 scientifically broken notebooks for Module 2 (R3)
│   ├── bug_A_high_skill.ipynb
│   ├── bug_B_cycle_drift.ipynb
│   └── bug_C_quiet_limb.ipynb
├── backup_cases/                    # 3 complete fallback cases for Module 3 (A3)
│   ├── case_alpha_euv_degradation/
│   ├── case_beta_limb_foreshortening/
│   └── case_gamma_preflare_contamination/
├── prompts/                         # Individual markdown files for G1–G4, R1–R3, A1–A2, S1–S6, P1–P5, X1–X5
├── templates/
│   ├── PLAN_TEMPLATE.md
│   ├── ERROR_LOG.md
│   ├── HANDOFF.md
│   ├── FINDINGS_TEMPLATE.md
│   ├── SKILL_TEMPLATE.md            # Mandatory template for X3 skill distillation (§8)
│   ├── skill_cli_template.py        # Mandatory argparse CLI template for X3 skills (§8)
│   ├── REVIEW_CHECKLIST.md
│   └── TALK_TEMPLATE.md
├── tests/                           # Fast sanity tests (`pytest -q tests/` MUST finish in < 30s)
└── work/                            # Participant sandboxes (`work/<name>/src/`, `tests/`, `notes/`, `figures/`)
    └── .gitkeep
```

### `.gitignore` requirements
- **MUST ignore**: `work/*/runs/`, `work/*/*.zarr`, `work/*/*.fits`, `data/cubes_512/`, `data/sdo_events/` (if distributed via shared mount or data tarball).
- **MUST track**: `skills/**` (so participants can commit and open PRs with their newly created skills!), `work/*/*.md` (`PLAN.md`, `ERROR_LOG.md`, `HANDOFF.md`, `FINDINGS.md`), `work/*/src/*.py`, `work/*/tests/*.py`, and `work/*/figures/*.png`.

---

## 3. R2 · Agent-ready API, docstrings, and runtime guardrails

Coding agents do not "understand" solar physics; they pattern-match function names, type hints, docstrings, and exceptions. Follow these four rules so the agent uses your toolkit properly.

### Rule 2.1 — Every public function MUST have a structured domain docstring
Every function listed in `docs/CHEATSHEET.md` **MUST** include type annotations and a docstring that states:
1. **What it does and what it explicitly does NOT do** (prevents the agent from assuming a single function does everything).
2. **Physical units, coordinate frame, and data level** for every input and output.
3. **Metadata mutations / invariants** (e.g., header keys added or checked).
4. **Raises / Warns** conditions.
5. **A 3-line canonical example** citing the tutorial notebook where it is demonstrated.

#### Concrete example (`src/sdoflare/prep.py`)
```python
def correct_aia_degradation(
    aia_map: sunpy.map.GenericMap,
    *,
    normalize_exptime: bool = True,
    calibration_version: int = 10,
) -> sunpy.map.GenericMap:
  """Applies time-dependent SDO/AIA sensitivity degradation and exposure normalization.

  IMPORTANT FOR AGENTS & USERS:
  - `aia_l1_to_l15()` does NOT correct channel degradation or divide by EXPTIME;
    you MUST call `correct_aia_degradation()` after `aia_l1_to_l15()`.
  - When `normalize_exptime=True` (default), this function divides pixel values
    by `aia_map.meta['EXPTIME']`. Do NOT divide by `EXPTIME` again downstream.

  Args:
    aia_map: Registered Level 1.5 AIA map (`Helioprojective`, solar north up).
      Expected input units: raw `DN / pix` (un-normalized by exposure time).
    normalize_exptime: If True, divides by `EXPTIME` (seconds) so output is in
      `DN s^-1 pix^-1`.
    calibration_version: AIA degradation table version in `data/catalogs/`.

  Returns:
    Degradation-corrected `sunpy.map.GenericMap` with physical units
    `DN s^-1 pix^-1` (if `normalize_exptime=True`) and metadata keys
    `sdoflare_degradation_corrected=True` and `sdoflare_exptime_normalized=True`.

  Raises:
    ValueError: If `aia_map.meta.get('sdoflare_degradation_corrected')` is
      already True (prevents double degradation correction), or if
      `normalize_exptime=True` and `aia_map.meta.get('sdoflare_exptime_normalized')`
      is already True (prevents double EXPTIME division), or if `EXPTIME <= 0`.

  Example (see `notebooks/01_sdo_ingest_and_l15_prep.ipynb`, Cell 5):
    >>> m_l15 = sdoflare.prep.aia_l1_to_l15(sdoflare.io.load_aia_map(path))
    >>> m_cal = sdoflare.prep.correct_aia_degradation(m_l15)
    >>> assert m_cal.meta["BUNIT"] == "DN / (pix s)"
  """
```

### Rule 2.2 — Loud runtime guardrails (fail fast on domain errors)
Agents treat "code ran without throwing an exception" as "code is correct." Therefore, your toolkit **MUST** raise explicit `ValueError`s or emit loud `UserWarning`s for the domain mistakes scientists and agents most frequently make:

| Common heliophysics / agent mistake | Mandatory runtime guardrail in `sdoflare` |
|---|---|
| Dividing by `EXPTIME` twice or running degradation twice | Set `meta["sdoflare_exptime_normalized"] = True` and `meta["sdoflare_degradation_corrected"] = True`; raise `ValueError` if already `True`. |
| Passing Level 1 (unregistered/unrolled) AIA maps into multi-channel co-alignment or model inference | Check `meta.get("LVL_NUM") == 1.5` or `abs(meta.get("CROTA2", 0.0)) < 1e-3`; raise `ValueError("Input map is Level 1; call aia_l1_to_l15() first.")`. |
| Passing naive or non-UTC Python `datetime` objects | In `SDOTimeIndex` and `label_flare_window()`, reject naive datetimes (`dt.tzinfo is None`) with `ValueError("All timestamps must be timezone-aware UTC.")`. |
| Unnormalized EUV intensities fed into `FullDiskFlareNet` | In `FullDiskFlareNet.forward(x)`, assert input tensor values are within `[-10.0, 20.0]` and not raw `DN` ($> 10^3$) or containing `NaN`s (`torch.isnan(x).any()`), raising a descriptive `ValueError("Input contains NaNs or unscaled raw DN; call scale_channels() and mask_off_limb() first.")`. |
| Shuffled or zero-buffer train/test splits | In `carrington_split(index, buffer_days=27.0)`, raise `ValueError` if `buffer_days < 13.5` (half a solar rotation, the disk transit time of an active region) unless `allow_leakage_for_testing=True` is explicitly passed. |
| Tuning the classification threshold on the test set | In `compute_tss_hss(y_true, y_prob, *, threshold: float)`, make `threshold` a **required keyword-only float** (not `threshold=None` with auto-optimization on `y_true`!). Provide a separate function `find_optimal_tss_threshold(y_val, y_prob_val) -> float` so the API itself enforces threshold selection on the validation split. |
| Labeling SDO eclipse or bad-quality gaps as quiet (`0`) flare windows | In `label_flare_window()`, return `np.nan` (or a boolean mask `valid_window=False`) for any window overlapping an eclipse/quality gap $> 30\,\text{min}$, and make `compute_tss_hss()` raise `ValueError` if masked timestamps are passed as `0`. |

### Rule 2.3 — Ship baselines, splitters, and block bootstrap inside the package
Prompts `P1`, `P2`, `P5`, and `X3` ask the participant to compare against persistence/climatology, audit splits, and compute block-bootstrapped error bars. Do **not** force the agent to invent these from scratch during the workshop. The toolkit **MUST** include:
1. `PersistenceBaseline` and `ClimatologyBaseline` in `sdoflare.models` sharing the exact `.fit(X_train, y_train)` / `.predict_proba(X_test)` interface as `LinearProbeForecaster` and `FullDiskFlareNet`.
2. `carrington_split(time_index, val_start, test_start, buffer_days=27.0)` in `sdoflare.labels`.
3. `block_bootstrap_metric(y_true, y_prob, block_ids, metric_fn, n_boot=1000, ci=0.95)` in `sdoflare.eval`, where `block_ids` defaults to the Carrington rotation number (`carrington_rotation_number(t)`).

### Rule 2.4 — Small, discoverable public surface (`docs/CHEATSHEET.md`)
Keep the public API to **15–20 core functions/classes** documented in `docs/CHEATSHEET.md`. Every entry in `docs/CHEATSHEET.md` **MUST** list:
- Exact import path and call signature,
- Input and output types + physical units + tensor shapes,
- One 3-line copy-pasteable example.

---

## 4. R3 · Tutorial notebooks (`notebooks/`) & headless execution standards

### Rule 3.1 — `notebooks/README.md` MUST be a lookup table
Prompts `G1`, `G4`, and `A1` direct the agent to read `@notebooks/README.md` first. Format it as an explicit mapping table:

```markdown
# sdoflare Tutorial Notebooks Index

| Notebook | Science / Engineering Question Answered | Key `sdoflare` Functions Demonstrated | Data Tier Used | Headless Runtime |
|---|---|---|---|---|
| `01_sdo_ingest_and_l15_prep.ipynb` | How do I load AIA L1 & HMI LOS FITS, register to L1.5, correct AIA degradation, and apply HMI $\mu$-projection + limb masking? | `io.load_aia_map`, `io.load_hmi_los`, `prep.aia_l1_to_l15`, `prep.correct_aia_degradation`, `prep.correct_hmi_projection` | Tier 3 (`sdo_events/ar12673_201709`) | 18 s |
| `02_multichannel_coalignment_and_labels.ipynb` | How do I co-align multi-channel AIA + HMI onto a common $512\times 512$ grid, scale channels, and build 24h GOES flare labels with Carrington splits? | `prep.coalign_full_disk`, `prep.scale_channels`, `labels.label_flare_window`, `labels.carrington_split` | Tier 2 (`cubes_512`) + `catalogs` | 25 s |
| `03_pretrained_fulldisk_flare_forecast.ipynb` | How do I run pretrained `FullDiskFlareNet` to predict 24h $\ge\text{M1.0}$ probabilities and compare against Persistence & Climatology? | `models.FullDiskFlareNet`, `models.PersistenceBaseline`, `models.ClimatologyBaseline`, `eval.find_optimal_tss_threshold`, `eval.compute_tss_hss` | Tier 2 (`cubes_512`) | 35 s |
| `04_embedding_finetune_and_probing.ipynb` | How do I use precomputed `FullDiskFlareNet` disk embeddings to train a fast linear/MLP probe for a custom flare class, horizon, or channel subset? | `models.extract_disk_embeddings`, `models.LinearProbeForecaster`, `eval.brier_skill_score`, `eval.block_bootstrap_metric` | Tier 1 (`embeddings`) | 8 s |
| `05_evaluation_attribution_and_events.ipynb` | How do I generate reliability diagrams, event timelines, and active-region attribution maps for X-flare sequences (AR 12673, AR 12192)? | `models.attribute_disk_regions`, `eval.reliability_diagram`, `eval.plot_event_timeline` | Tier 3 (`sdo_events`) | 30 s |
```

### Rule 3.2 — Every notebook MUST support `papermill` headless execution
In Module 2 (`R1`, `R2`) and Module 6 (`X1`), the agent executes notebooks via `papermill notebooks/<nb>.ipynb work/<name>/runs/<nb>_out.ipynb`.
1. Cell 1 of every notebook **MUST** carry the Jupyter cell tag `"parameters"` and define all configurable inputs with sensible defaults.
2. Every tutorial notebook **MUST** execute top-to-bottom via `papermill` in **$< 60$ seconds** with zero network access and committed outputs saved in git.
3. No cell in a tutorial notebook **SHOULD** exceed ~25 lines of code. Move reusable helpers into `src/sdoflare/`.

---

## 5. R4 · Three-tier offline data architecture (`data/`)

The developer **MUST** pre-stage an offline `data/` directory organized into **three compute tiers**:

| Tier | Purpose in the workshop | Example contents (`sdoflare`) | Size & target runtime |
|---|---|---|---|
| **Tier 1 · Embeddings & Catalogs** (`data/embeddings/`, `data/catalogs/`) | Fast iteration in M4/M5; transfer learning (`04`); fast CLI verification of participant skills in M6 (`X4`); instant fallback at the 90-min First-Light checkpoint | `sdo_fulldisk_embeddings_6h_2011_2018.zarr`, `goes_xrs_1min_2011_2019.parquet`, `goes_flare_catalog_with_ar_locations.parquet`, `sdo_eclipse_and_quality_flags.parquet`, `aia_degradation_v10.csv` | $< 250\,\text{MB}$; loads & trains a probe in **$< 5\,\text{s}$ on CPU** |
| **Tier 2 · Co-aligned Downsampled Cubes** (`data/cubes_512/`) | Multi-channel preprocessing, channel ablation, limb-masking, and full-disk CNN inference (`02`, `03`) | `sdo_aia94_131_171_193_211_hmi_2014_2017.zarr` ($512\times 512$ L1.5 co-aligned cubes at 6h cadence) | $1\text{–}4\,\text{GB}$; loads a 1-year slice in **$< 15\,\text{s}$** |
| **Tier 3 · Full-Disk Event Slices** (`data/sdo_events/`) | L1 $\to$ L1.5 registration (`01`), `R2` event perturbation, `P3` known-event & quiet-Sun plausibility checks, and AR attribution (`05`) | Raw L1 + registered L1.5 FITS slices for `ar12192_201410`, `ar12673_201709`, and `quiet_201905` | $< 2\,\text{GB}$ total |

### Enforcing offline mode
In `src/sdoflare/io.py`, any function that could otherwise reach out to the network **MUST** check `os.environ.get("SDOFLARE_OFFLINE", "1") == "1"` and raise a `RuntimeError` pointing to `data/README.md`.

---

## 6. R5 · Agent context files & slash commands

### Rule 5.1 — Root context file (`GEMINI.md` + symlinks)
Place the concrete context file (from Participant Handouts `H8`) at `GEMINI.md` in the repository root and create relative symlinks so every major coding agent loads the identical rules automatically:
```bash
ln -s GEMINI.md AGENTS.md
ln -s GEMINI.md CLAUDE.md
```
Keep `GEMINI.md` **under 150 lines**.

### Rule 5.2 — Prompt cards in `prompts/` and optional slash commands
Save each prompt card (`G1`–`G4`, `R1`–`R3`, `A1`–`A2`, `S1`–`S6`, `P1`–`P5`, `X1`–`X5`) in `prompts/` and optionally expose them in `.gemini/commands/*.toml` (e.g., `/ground-map`, `/prove-leakage`, `/skillify-workflow`).

---

## 7. R6 · Engineering the 3 Bug-Hunt notebooks (`bughunt/`) and 3 Backup Cases (`backup_cases/`)

1. **Three Bug-Hunt Notebooks (`bughunt/`)**:
   - `bughunt/bug_A_high_skill.ipynb` (Statistical / Rotation Leakage: `train_test_split(shuffle=True)` on 6h frames + fitting scaler/threshold on full dataset $\to$ $\text{TSS} = 0.91$).
   - `bughunt/bug_B_cycle_drift.ipynb` (Instrument Calibration: calls `aia_l1_to_l15()`, skips `correct_aia_degradation()`, divides by `EXPTIME` manually $\to$ 40% downward probability drift across Solar Cycle 24).
   - `bughunt/bug_C_quiet_limb.ipynb` (Multi-Instrument Misalignment & Limb Blow-up: stacks AIA and HMI by file index across a 6h gap, parses local file mtime as UTC, divides by $\mu = \cos\theta$ out to $\mu = 0.01$ without limb mask).
   - All three **MUST** run via `papermill` in $< 30\,\text{s}$ with zero exceptions and have pre-executed outputs committed in git. Keep the answer key **out of the repo** (in Facilitator Guide §6.2 only).
2. **Three Backup Cases (`backup_cases/case_{alpha,beta,gamma}/`)**: Each folder **MUST** include a pre-approved `PLAN.md` (with its target skill name for M6) and a 15-line `starter_check.py`.

---

## 8. R7 · Skill-creation scaffolding, reference skill & validator (`skills/`, `templates/`, `env/validate_skill.py`)

In Module 6 (`X3`–`X4`), every participant uses their coding agent to distill their validated workflow in `work/<name>/src/` into a reusable **Agent Skill** under `skills/<skill-name>/` so other science users can invoke it on new solar events or periods.

To make `X3` and `X4` reliable across 20 participants, the toolkit developer **MUST** provide four things in the repository:

### Rule 7.1 — Ship one gold-standard reference skill (`skills/sdo-ar-event-attribution/`)
Coding agents produce dramatically better skills when given a working, in-repo exemplar to imitate (`@skills/sdo-ar-event-attribution/SKILL.md`).
- Build one complete reference skill wrapping notebook `05` (e.g., `skills/sdo-ar-event-attribution/SKILL.md` + `skills/sdo-ar-event-attribution/scripts/ar_attribution_cli.py`).
- Register it in `.gemini/skills/`, `.claude/skills/`, and `.agents/skills/` by running `python env/validate_skill.py skills/sdo-ar-event-attribution`.

### Rule 7.2 — Ship `templates/SKILL_TEMPLATE.md` and `templates/skill_cli_template.py`
Prompts `X3` Step 1 and Step 2 instruct the agent to follow `@templates/SKILL_TEMPLATE.md` and `@templates/skill_cli_template.py`.

#### Mandatory `templates/SKILL_TEMPLATE.md`
```markdown
---
name: <lowercase-hyphenated-skill-name-max-64-chars>
description: >-
  <1-3 sentences (max 1024 chars) stating what scientific workflow this skill
  executes using sdoflare, physical outputs/metrics produced, WHEN to trigger
  it ("Use when..."), and WHEN NOT to use it ("Do not use for...").>
---

# <Skill Title>

## Overview
<2-4 sentences explaining the scientific problem solved, inputs, outputs, and how it builds on `sdoflare`.>

## Dependencies & `sdoflare` Functions Used
- `<sdoflare.module.func1>` — <purpose>
- `<sdoflare.module.func2>` — <purpose>

## Quick Start
```bash
python skills/<skill-name>/scripts/<skill_cli>.py <subcommand> \
  --data-path <path-in-data/> \
  --start-date <YYYY-MM-DD> --end-date <YYYY-MM-DD> \
  --output-dir <output-folder>
```

## Utility Scripts & Subcommands (`scripts/<skill_cli>.py`)
- `<subcommand-1>`: <what it computes, required flags, and exact files written to `--output-dir` (`metrics.json`, `comparison_table.md`, `<plot>.png`).>

## Domain Guardrails (Mandatory)
- **Split & Buffer Rule**: <e.g., Always uses `sdoflare.labels.carrington_split(buffer_days=27.0)`>
- **Calibration / Units Rule**: <e.g., Uses `correct_aia_degradation(..., normalize_exptime=True)` in `DN s^-1 pix^-1`>
- **Evaluation & Baselines Rule**: <e.g., Threshold selected via `find_optimal_tss_threshold` on validation split only; always reports `PersistenceBaseline` and `ClimatologyBaseline` with Carrington-block bootstrap 95% CIs>

## Common Mistakes (from Workshop Error Logs)
1. **<Mistake 1>**: <What goes wrong physically/statistically and how this skill prevents it.>
2. **<Mistake 2>**: <What goes wrong physically/statistically and how this skill prevents it.>
```

#### Mandatory `templates/skill_cli_template.py`
All skill scripts **MUST** write results to files inside `--output-dir` (never dumping large tables or arrays to stdout, which bloats the calling agent's context window) and require explicit parameters (no hardcoded participant paths):
```python
#!/usr/bin/env python3
"""CLI wrapper template for a reusable sdoflare Agent Skill."""

import argparse
import json
from pathlib import Path
import sys


def cmd_run(args: argparse.Namespace) -> int:
  out_dir = Path(args.output_dir)
  out_dir.mkdir(parents=True, exist_ok=True)

  # Call validated functions from sdoflare / skill module here.
  metrics = {
      "skill_name": "<skill-name>",
      "start_date": args.start_date,
      "end_date": args.end_date,
      "horizon_hours": args.horizon_hours,
      "min_flare_class": args.min_flare_class,
  }
  metrics_path = out_dir / "metrics.json"
  metrics_path.write_text(json.dumps(metrics, indent=2) + "\n")
  print(f"Success! Wrote metrics to: {metrics_path}")
  return 0


def build_parser() -> argparse.ArgumentParser:
  parser = argparse.ArgumentParser(
      description="Reusable sdoflare workflow CLI."
  )
  sub = parser.add_subparsers(dest="subcommand", required=True)

  run_p = sub.add_parser("run", help="Execute workflow and write outputs.")
  run_p.add_argument("--data-path", required=True, help="Input dataset path.")
  run_p.add_argument("--start-date", required=True, help="UTC start date.")
  run_p.add_argument("--end-date", required=True, help="UTC end date.")
  run_p.add_argument("--horizon-hours", type=int, required=True)
  run_p.add_argument("--min-flare-class", required=True, help="e.g. C5.0, M1.0")
  run_p.add_argument("--output-dir", required=True, help="Output directory.")
  run_p.set_defaults(func=cmd_run)
  return parser


def main() -> int:
  parser = build_parser()
  args = parser.parse_args()
  try:
    return args.func(args)
  except Exception as exc:  # pylint: disable=broad-except
    print(f"ERROR: {exc}", file=sys.stderr)
    return 1


if __name__ == "__main__":
  sys.exit(main())
```

### Rule 7.3 — Ship the automated skill validator & multi-agent registrar (`env/validate_skill.py`)
In `X3` Step 2, the participant's agent runs `python env/validate_skill.py skills/<skill-name>`. The developer **MUST** include this script so broken or hardcoded skills are caught immediately and valid skills are automatically symlinked into `.gemini/skills/`, `.claude/skills/`, and `.agents/skills/`:

```python
#!/usr/bin/env python3
"""Validates a skill folder in skills/<name> and registers symlinks for agent CLIs."""

import os
from pathlib import Path
import re
import subprocess
import sys

ROOT = Path(__file__).resolve().parent.parent
REQUIRED_SECTIONS = [
    "## Overview",
    "## Quick Start",
    "## Domain Guardrails",
    "## Common Mistakes",
]
AGENT_SKILL_DIRS = [
    ROOT / ".gemini" / "skills",
    ROOT / ".claude" / "skills",
    ROOT / ".agents" / "skills",
]


def fail(msg: str) -> None:
  print(f"[SKILL INVALID] {msg}", file=sys.stderr)
  sys.exit(1)


def validate_and_register(skill_dir: Path) -> None:
  skill_dir = skill_dir.resolve()
  skill_md = skill_dir / "SKILL.md"
  if not skill_md.exists():
    fail(f"Missing {skill_md}")

  text = skill_md.read_text()
  m = re.match(r"^---\n(.*?)\n---\n", text, flags=re.DOTALL)
  if not m:
    fail("SKILL.md must start with YAML frontmatter delimited by '---'")
  frontmatter = m.group(1)

  name_m = re.search(r"^name:\s*([a-z0-9-]+)\s*$", frontmatter, re.MULTILINE)
  if not name_m or len(name_m.group(1)) > 64:
    fail("Frontmatter 'name:' must be lowercase alphanumeric + hyphens (<= 64 chars)")
  skill_name = name_m.group(1)
  if skill_name != skill_dir.name:
    fail(f"Frontmatter name '{skill_name}' must match folder name '{skill_dir.name}'")

  if "description:" not in frontmatter or "Use when" not in frontmatter:
    fail("Frontmatter 'description:' must be present and include 'Use when...' trigger guidance")

  for sec in REQUIRED_SECTIONS:
    if sec not in text:
      fail(f"SKILL.md is missing required section: '{sec}'")

  scripts_dir = skill_dir / "scripts"
  scripts = list(scripts_dir.glob("*.py")) if scripts_dir.exists() else []
  if not scripts:
    fail(f"No CLI script found in {scripts_dir}/ (workflows that process data must include a CLI script)")

  for script in scripts:
    src = script.read_text()
    if re.search(r"work/[a-zA-Z0-9_-]+/", src):
      fail(f"{script.name} contains a hardcoded 'work/<user>/' path; pass paths via --output-dir instead")
    res = subprocess.run(
        [sys.executable, str(script), "--help"],
        capture_output=True,
        text=True,
        check=False,
    )
    if res.returncode != 0:
      fail(f"`python {script} --help` failed:\n{res.stderr}")
    if "--output-dir" not in res.stdout and "--output" not in res.stdout:
      fail(f"{script.name} --help must expose an --output-dir or --output flag")

  for agent_dir in AGENT_SKILL_DIRS:
    agent_dir.mkdir(parents=True, exist_ok=True)
    link = agent_dir / skill_name
    if link.is_symlink() or link.exists():
      link.unlink()
    rel_target = os.path.relpath(skill_dir, agent_dir)
    link.symlink_to(rel_target)

  print(f"[OK] Validated '{skill_name}' ({len(scripts)} CLI script(s))")
  print(f"[OK] Registered symlinks in .gemini/skills/, .claude/skills/, and .agents/skills/")


if __name__ == "__main__":
  if len(sys.argv) != 2:
    fail("Usage: python env/validate_skill.py skills/<skill-name>")
  validate_and_register(Path(sys.argv[1]))
```

---

## 9. R8 · Setup verification script (`env/check_setup.py`) and fast test suite

Participants run `python env/check_setup.py` one week before the workshop and again in Module 0. The developer **MUST** ship a self-contained script that checks imports, data files, templates, the reference skill, and `pytest`:

```python
#!/usr/bin/env python3
"""Pre-flight environment, data, skill-scaffolding, and test check for sdoflare-workshop."""

from pathlib import Path
import subprocess
import sys

ROOT = Path(__file__).resolve().parent.parent

REQUIRED_FILES = [
    "GEMINI.md",
    "AGENTS.md",
    "CLAUDE.md",
    "notebooks/README.md",
    "docs/CHEATSHEET.md",
    "data/README.md",
    "data/catalogs/goes_flare_catalog_with_ar_locations.parquet",
    "data/embeddings/sdo_fulldisk_embeddings_6h_2011_2018.zarr",
    "bughunt/bug_A_high_skill.ipynb",
    "bughunt/bug_B_cycle_drift.ipynb",
    "bughunt/bug_C_quiet_limb.ipynb",
    "skills/sdo-ar-event-attribution/SKILL.md",
    "templates/PLAN_TEMPLATE.md",
    "templates/ERROR_LOG.md",
    "templates/HANDOFF.md",
    "templates/SKILL_TEMPLATE.md",
    "templates/skill_cli_template.py",
    "env/validate_skill.py",
]

REQUIRED_IMPORTS = [
    "numpy",
    "scipy",
    "pandas",
    "astropy",
    "sunpy",
    "zarr",
    "sklearn",
    "matplotlib",
    "papermill",
    "pytest",
    "sdoflare",
]


def fail(msg: str) -> None:
  print(f"[FAIL] {msg}", file=sys.stderr)
  sys.exit(1)


def main() -> None:
  if sys.version_info < (3, 10):
    fail(f"Python >= 3.10 required; found {sys.version}")
  print(f"[OK] Python {sys.version.split()[0]}")

  for mod in REQUIRED_IMPORTS:
    try:
      __import__(mod)
    except Exception as exc:  # pylint: disable=broad-except
      fail(f"Cannot import '{mod}': {exc}")
  print(f"[OK] All {len(REQUIRED_IMPORTS)} required packages import cleanly")

  for rel in REQUIRED_FILES:
    if not (ROOT / rel).exists():
      fail(f"Missing required path: {rel}")
  print(f"[OK] All {len(REQUIRED_FILES)} required repo, data & skill-template paths present")

  res_skill = subprocess.run(
      [
          sys.executable,
          str(ROOT / "env" / "validate_skill.py"),
          str(ROOT / "skills" / "sdo-ar-event-attribution"),
      ],
      capture_output=True,
      text=True,
      check=False,
  )
  if res_skill.returncode != 0:
    fail(f"Reference skill validation failed:\n{res_skill.stderr}")
  print("[OK] Reference skill (`skills/sdo-ar-event-attribution`) validated & registered")

  res = subprocess.run(
      [sys.executable, "-m", "pytest", "-q", str(ROOT / "tests")],
      capture_output=True,
      text=True,
      check=False,
  )
  if res.returncode != 0:
    fail(f"pytest failed:\n{res.stdout}\n{res.stderr}")
  print("[OK] Fast test suite (`pytest -q tests/`) passed")
  print("\n✅ SETUP GREEN — Ready for the workshop!")


if __name__ == "__main__":
  main()
```

---

## 10. R9 · Pre-workshop qualification protocol ("Definition of Done")

A workshop repository is **not ready** until the developer passes all three qualification gates below at least **5 days before the workshop**.

### Gate 1 — Automated Repo, Notebook, Skill & Offline Verification
Run the following commands with network access disabled (`SDOFLARE_OFFLINE=1`); all **MUST** exit `0`:
```bash
export SDOFLARE_OFFLINE=1
python env/check_setup.py
python env/validate_skill.py skills/sdo-ar-event-attribution
pytest -q tests/
for nb in notebooks/*.ipynb bughunt/*.ipynb; do
  papermill "$nb" "/tmp/$(basename "$nb")"
done
```

### Gate 2 — The 10-Prompt Agent Smoke Test (score $\ge 8/10$ required)
Open a fresh coding agent session in the repo root (with **no** prior conversation history) and run the **10 canonical smoke-test prompts** from Facilitator Guide §7.2 (which include invoking the pre-installed `sdo-ar-event-attribution` skill). Score each answer `1` only if:
- The agent cites the right `src/<toolkit>/<file>.py:<line>` and notebook cell without inventing function or keyword names, **and**
- Any code it writes uses the toolkit functions or registered skills rather than re-implementing them.

### Gate 3 — The 90-Minute Facilitator Dry Run (including `X3`–`X4` Skill Distillation)
Have one facilitator (who did **not** write the code) play a participant from **M0 to M6** using **Backup Case α** and only the **Participant Handouts**:
1. Time every step (`G1`–`G4`, `R1`–`R3`, `A1`, `S1`–`S6`, `P1`–`P5`, `X1`–`X5`).
2. Verify that the First-Light plot (`S2`) is reached within 45 minutes of starting `M4`.
3. Verify that running `X3` produces a valid skill (`skills/sdo-euv-channel-ablation/`) that passes `python env/validate_skill.py` and that a **fresh agent session (`X4`)** automatically discovers and executes the new skill on a perturbed query in under 2 minutes.

