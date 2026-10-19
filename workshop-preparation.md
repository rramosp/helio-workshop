# Workshop Preparation Guide for Project Developers

This document provides instructions and a one-week execution plan for project developers to organize, build, and verify the materials for our **1.5-day AI-Assisted Scientific Computing Workshop**.

---

## 1. Introduction: Overall Mechanics of the Workshop

### 1.1 The Core Paradigm: Prompt-Driven Jupyter Notebooks with AI Coding Agents

This workshop is built around **Jupyter Notebooks (`.ipynb`)** backed by our project's Python toolkit (`src/<toolkit>/`) and operated through **AI-assisted coding agents** (e.g., Gemini CLI, Claude Code, Codex, Cursor).

Unlike a traditional tutorial where participants passively press `Shift+Enter` on pre-written cells—or a traditional hackathon where participants spend hours writing and debugging raw boilerplate code by hand—this workshop uses an **agent-mediated scientific workflow**:

> **Participants do not manually write or edit large blocks of code inside the notebooks.** Instead, they act as **Principal Investigators and Scientific Directors**, issuing grounded natural-language prompts to a coding agent to:
> 1. **Import and ingest data** from pre-staged offline catalogs and arrays using the toolkit's official loaders,
> 2. **Modify, parameterize, and update Jupyter notebooks** (and companion `.py` helper modules) to test new scientific hypotheses, events, or parameter regimes,
> 3. **Run the notebooks and automated sanity checks**, and
> 4. **Analyze, validate, and critique the scientific results** against domain baselines, audit for subtle physical/statistical leakage, and package their validated workflow into a reusable **Agent Skill**.

```mermaid
flowchart LR
    Sci["Scientist<br/>(Principal Investigator)"] -->|"1. Grounded Prompt<br/>(@notebooks/..., @src/...)"| Agent["AI Coding Agent<br/>(Gemini CLI / Claude Code)"]
    Agent -->|"2. Proposes Plan &<br/>Cites file:line"| Sci
    Sci -->|"3. Approves Plan &<br/>Physical Assumptions"| Agent
    Agent -->|"4. Ingests Data, Updates<br/>.ipynb & work/<name>/src/*.py"| NB["Jupyter Notebooks &<br/>Toolkit Pipeline"]
    Agent -->|"5. Runs Notebooks<br/>& pytest"| Out["Figures, Metrics &<br/>Validation Tables"]
    Out -->|"6. Physical & Statistical<br/>Inspection"| Sci
```

### 1.2 Why This Mechanics Changes How Developers Must Prepare Notebooks

When human researchers read a messy notebook, they mentally fill in missing units, implicit coordinate frames, or undocumented globals. **When a coding agent reads a messy notebook, it hallucinates.** Left undirected or given monolithic 300-line notebook cells, a coding agent will:
- Bypass our toolkit completely and write raw `numpy` / `scipy` / `astropy` / `sunpy` boilerplate from scratch,
- Violate domain invariants silently (e.g., double-normalizing exposure/cadence, mixing coordinate frames, interpolating across instrument dropouts, or shuffling temporally autocorrelated time series into train and test splits),
- Corrupt `.ipynb` JSON when trying to edit massive cells inline.

To make the agent-mediated notebook mechanics work reliably across 15–25 participants, **our workshop repository must be engineered for both human scientists and coding agents**:
1. **Thin, modular notebooks (`notebooks/01_...` to `05_...`)**: Every cell is short ($\le 25$ lines), calls high-level functions from `src/<toolkit>/`, and defines configurable inputs clearly in the top cell.
2. **Helper logic extracted to `.py` files**: For custom participant analyses in Module 4, the agent writes reusable functions into `work/<name>/src/*.py` with unit tests in `work/<name>/tests/`, and updates the participant's notebook to import and plot those functions cleanly.
3. **Explicit domain context (`GEMINI.md` / `AGENTS.md` / `CLAUDE.md`)**: A concise ($\le 150$-line) root instruction file loaded automatically by the coding agent that enforces *Toolkit First*, *Plan Before Code*, *Cite `file:line`*, and our toolkit's non-negotiable physical and statistical conventions.
4. **Loud runtime guardrails**: Toolkit functions raise clear `ValueError` exceptions whenever an agent passes wrong units, uncalibrated data levels, naive timestamps, or leaky train/test splits.
5. **100% offline pre-staged data (`data/`)**: Zero live downloads during the workshop; data is organized into fast CPU embeddings/catalogs (Tier 1), medium downsampled cubes/arrays (Tier 2), and full-resolution benchmark event slices (Tier 3).

---

## 2. Structure of the Workshop: Modules `M0` to `M6` (The GRASP Ladder)

The 1.5-day curriculum follows the **GRASP** progression (**G**round $\to$ **R**eproduce $\to$ **A**rticulate $\to$ **S**olve $\to$ **P**rove & Skillify). Across modules `M0`–`M6`, the coding agent's autonomy expands only as the participant demonstrates mastery of the toolkit and its physical assumptions.

| Module | Time | Agent Role | Participant Role | How Participants Use the Agent with Notebooks | Key Deliverable / Gate |
|---|---|---|---|---|---|
| **M0 · Setup & First Contact** | Day 1<br/>30 min | **Environment Checker** | Operator | Prompt agent to summarize rules loaded from `GEMINI.md`, list the notebooks in `@notebooks/README.md`, and run `pytest -q tests/` + `env/check_setup.py`. | Green setup check; `work/<name>/` sandbox initialized. |
| **M1 · GROUND** | Day 1<br/>60 min | **Tutor**<br/>*(Read-Only)* | Learner & Fact-Checker | Prompt agent to map `@src/<toolkit>/` and `@notebooks/README.md` (`G1`), explain a tutorial notebook cell-by-cell (`G2`), quiz the participant on physical assumptions (`G3`), and assemble a personal cheat sheet of notebook snippets (`G4`). | `work/<name>/notes/toolkit_map.md` & `my_cheatsheet.md` with **3 manually verified `file:line` citations**. |
| **M2 · REPRODUCE** | Day 1<br/>60 min | **Navigator** | Experimentalist | Prompt agent to run a baseline tutorial notebook and summarize its metrics and warnings (`R1`), then plan, create, and run a modified notebook variant on a new event, channel set, or physical parameter (`R2`) and analyze how the output changed physically. | Executed variant notebook `work/<name>/<variant>.ipynb` + 3-sentence physical interpretation of the perturbation. |
| **M3 · ARTICULATE** | Day 1<br/>60 min | **Skeptical Collaborator**<br/>*(No Code Allowed)* | Principal Investigator | Prompt agent to challenge scope and draft a 5-hour Minimum Viable Science plan (`PLAN.md`) mapping every step to existing tutorial notebooks, toolkit functions, and pre-staged `data/` (`A1`–`A2`). | **`PLAN.md` signed off by a facilitator at the Plan Gate (`A3`)** before any custom code is written. |
| **M4 · SOLVE (Sprint 1)** | Day 1<br/>120 min | **Pair Programmer** | Scientific Director & Code Reviewer | Prompt agent in a strict *Plan $\to$ Approve $\to$ Update `.py` & `.ipynb` $\to$ Run $\to$ Sanity-Check* loop (`S1`–`S4`) to ingest data, build the analysis notebook, run physical unit/range assertions, and generate a First-Light plot. End Day 1 by generating `HANDOFF.md` (`S6`). | **90-Minute First-Light Plot (`figures/first_light.png`)** passing facilitator visual check + passing `test_sanity.py` + `HANDOFF.md`. |
| **M5 · SOLVE (Sprint 2) + PROVE** | Day 2<br/>85 min | **Hostile Journal Referee** | Defender & Validator | Finish MVP notebook (40 min), then switch agent to Hostile Referee mode (45 min): prompt agent to run domain baselines (`P1`), audit the notebook/code for data leakage & circularity (`P2`), run benchmark event & negative-control checks (`P3`), critique the findings as a hostile referee (`P4`), and add block-bootstrapped error bars (`P5`). | `results_table.md` (with baselines & CIs), `figures/main_result.png`, leakage audit report, and 1 executed referee test. |
| **M6 · PACKAGE, SKILLIFY & PRESENT** | Day 2<br/>100 min | **Skill Architect & Technical Editor** | Author & Presenter | Prompt agent to clean and re-run the final notebook end-to-end (`X1`), generate a 1-page `FINDINGS.md` (`X2`), distill the notebook pipeline + `ERROR_LOG.md` into a reusable **Agent Skill** (`skills/<name>/SKILL.md` + CLI script) (`X3`), test a pod-mate's skill in a fresh session (`X4`), and file toolkit GitHub issues (`X5`). | Reproducible `final_verified.ipynb`, `FINDINGS.md`, validated `skills/<name>/`, and 5-min lightning talk. |

---

## 3. One-Week Plan to Deliver the Workshop Materials

Below is the structured plan for the development team to design, implement, and verify all workshop materials during the preparation week. It is organized around the **three primary deliverables**—**(3.1) Scientific Goals & Cases**, **(3.2) Workshop Notebooks**, and **(3.3) The Participant Guide & Prompt Catalog**—followed by the **day-by-day execution schedule (3.4)**.

---

### 3.1 Deliverable 1: Scientific Goals & Benchmark Science Cases for Participants

Before writing or refactoring notebooks, the development team must define **what scientific outcomes participants will achieve** and stage the exact offline datasets needed to make those goals achievable within 5 hours of hands-on work.

#### A. Core Scientific & Methodological Goals for Every Participant
By the end of the workshop, every participant must achieve five concrete goals using our toolkit via their coding agent:
1. **Master the Toolkit's Physical Preprocessing & Data Model**: Trace raw inputs through calibration, coordinate registration, quality/gap masking, and feature/tensor assembly—verifying physical units and coordinate frames at every step rather than trusting agent defaults.
2. **Reproduce & Perturb End-to-End Notebook Workflows**: Execute reference notebooks through the coding agent and systematically evaluate how perturbing a physical parameter (target event, date range, spectral channel, cadence, or threshold) changes the scientific output.
3. **Enforce Domain-Valid Evaluation & Prevent Statistical Leakage**: Apply domain-appropriate splitting (accounting for physical autocorrelation timescales), avoid lookahead or threshold-tuning leakage, and audit agent-generated code for physical circularity.
4. **Complete a Falsifiable 5-Hour Science Investigation (MVP) with Rigorous Validation**: Formulate a focused science question, implement the analysis via agent-directed notebook + `.py` steps, benchmark against built-in domain baselines (e.g., Persistence and Climatology), validate on a **known positive benchmark event** and a **quiet/negative-control interval**, and compute domain-aware uncertainty intervals.
5. **Publish a Reusable Agent Skill for the Community**: Encapsulate their validated notebook workflow and hard-won domain guardrails into a standalone `skills/<skill-name>/SKILL.md` + parameterized CLI script so any researcher can run the analysis on new dates or events in a single prompt.

#### B. Define 3 Concrete Scientist Personas / Backup Science Cases (`Case α`, `Case β`, `Case γ`)
Not every participant arrives with a well-scoped 5-hour question. Furthermore, all worked prompt examples in the Participant Guide need realistic domain anchors. Developers **must define and pre-stage data for 3 canonical science cases**:

| Case | Archetype | What Developers Must Define for the Project | Target Reusable Skill in `M6` |
|---|---|---|---|
| **Case α** | **Sensor / Channel / Calibration Sensitivity** | A science question testing **input modality/channel ablation or instrument calibration sensitivity** (e.g., how adding/removing specific wavelengths or omitting time-dependent degradation correction impacts physical estimates or forecast skill). Pre-stage the multi-channel slice in Tier 1 & Tier 2 data. | `skills/<channel-or-calibration-skill>/` |
| **Case β** | **Geometric / Spatial / Regime Stratification** | A science question testing **performance across spatial coordinates, viewing geometry, latitude/longitude bins, or physical regimes** (e.g., center-to-limb foreshortening or high- vs. low-activity regimes), exposing where standard assumptions break down and testing a targeted correction. | `skills/<geometry-or-regime-audit-skill>/` |
| **Case γ** | **Target Horizon / Physical Mechanism Probing** | A science question using **Tier-1 precomputed features/embeddings** to test a custom physical target, lead-time horizon, or confounding-variable filter (e.g., separating true precursor signal from event-in-progress contamination) in seconds on CPU. | `skills/<horizon-or-mechanism-probe-skill>/` |

#### C. Stage 3 Benchmark Event Slices & 3 Compute Tiers in `data/`
To support these scientific goals with **zero network access** during the workshop, developers must populate `data/` with:
- **Tier 1 (`data/embeddings/`, `data/catalogs/`)**: Lightweight tabular catalogs and precomputed model embeddings/features ($< 250\,\text{MB}$) that load and run a complete training/evaluation loop in **$< 5$ seconds on CPU**.
- **Tier 2 (`data/cubes/`)**: Downsampled, co-aligned multi-channel arrays/cubes ($1\text{–}4\,\text{GB}$) for testing preprocessing, masking, and model inference in **$< 15$ seconds**.
- **Tier 3 (`data/events/`)**: Full-resolution slices for **3 specific benchmark intervals**:
  1. **Benchmark Event 1 (Primary Active Event)**: Used in Tutorial Notebooks `01` and `05` and sanity check `S3` as the canonical positive test case.
  2. **Benchmark Event 2 (Perturbation Active Event)**: Used in Module 2 (`R2`) when participants prompt the agent to retarget a notebook to a second major event with distinct physical characteristics.
  3. **Benchmark Event 3 (Quiet / Negative-Control Interval)**: Used in Module 5 (`P3`) to verify that participant models/analyses keep false-alarm rates low during quiet background conditions.

#### D. Developer Architecture Inventory Worksheet
As your first task on Day 1 of preparation, fill out this inventory table with the development team to lock in the public surface area exposed to participants and agents:

| Toolkit Layer | Target Module in `src/<toolkit>/` | Core Public Functions / Classes (15–20 Total) | Key Physical Units, Coordinate Frames & Guardrails to Enforce |
|---|---|---|---|
| **1. Data & Catalog Loaders** | `src/<toolkit>/io.py` | e.g., `load_observation()`, `load_catalog()`, `TimeIndex` | Enforce UTC-aware timestamps; block live network downloads in offline mode. |
| **2. Calibration & Registration** | `src/<toolkit>/prep.py` | e.g., `register_level15()`, `correct_degradation()`, `correct_projection()`, `coalign_channels()`, `scale_inputs()` | State exact input/output units; raise `ValueError` on double normalization or uncalibrated inputs. |
| **3. Target Builders & Domain Splitters** | `src/<toolkit>/labels.py` | e.g., `build_event_labels()`, `domain_temporal_split(buffer_days=...)` | Mask data/eclipse gaps as `NaN` (never `0`); enforce minimum buffer between train/val/test splits. |
| **4. Models, Probes & Baselines** | `src/<toolkit>/models.py` | e.g., `PretrainedBackbone`, `extract_embeddings()`, `LinearProbe`, `PersistenceBaseline`, `ClimatologyBaseline` | Shared `.fit()` / `.predict()` interface across models and baselines; check input tensor scaling. |
| **5. Evaluation, Attribution & Plots** | `src/<toolkit>/eval.py` | e.g., `compute_skill_metrics()`, `find_optimal_threshold()`, `block_bootstrap_metric()`, `plot_event_diagnostics()` | Require validation-tuned threshold as explicit kwarg; block bootstrap over physical rotation/event blocks. |

---

### 3.2 Deliverable 2: List of Notebooks to Be Developed and Organized

Developers must prepare and test **8 notebooks total** (5 Core Tutorial Notebooks + 3 Backup Case Starter Notebooks) plus a master `notebooks/README.md` index table. Every notebook must be structured so a coding agent can read, explain, modify, and execute it reliably.

#### Group 1: The 5 Core Tutorial Notebooks (`notebooks/01_...` to `05_...`)
These five notebooks form the backbone of the workshop. Participants use them in **M1 (Ground)** to learn the toolkit, in **M2 (Reproduce)** to run baseline and perturbed workflows, and in **M3–M5 (Articulate, Solve, Prove)** as building blocks for their own investigation.

| Notebook Filename (Adapt Name to Domain) | Pedagogical Role in Workshop | What the Notebook Must Demonstrate | Target Data Tier & Max Runtime |
|---|---|---|---|
| **`notebooks/01_ingest_and_calibration.ipynb`** | **Physical Data Ingestion & Calibration** | Loading raw/uncalibrated files from `data/events/`, inspecting headers/metadata/units, applying pointing/coordinate registration, instrument degradation/exposure normalization, boundary/quality masking, and plotting calibrated vs. uncalibrated physical quantities. | Tier 3 (`data/events/`)<br/>$\le 20\,\text{s}$ |
| **`notebooks/02_coalignment_and_labels.ipynb`** | **Multi-Source Alignment, Scaling & Domain Splits** | Co-aligning multiple instruments/channels onto a synchronized spatial/temporal grid, handling observation gaps safely, scaling physical channels into model-ready tensors, constructing target labels, and performing a leak-free domain split. | Tier 2 (`data/cubes/`) + `catalogs/`<br/>$\le 30\,\text{s}$ |
| **`notebooks/03_pretrained_model_and_baselines.ipynb`** | **End-to-End Inference & Baseline Benchmarking** | Running the core pretrained model or primary physical algorithm on Tier-2 data, calibrating decision thresholds strictly on the validation split, and comparing domain metrics against `PersistenceBaseline` and `ClimatologyBaseline`. **Used in `R1` reproduction.** | Tier 2 (`data/cubes/`)<br/>$\le 40\,\text{s}$ |
| **`notebooks/04_embedding_probing_and_variants.ipynb`** | **Fast Iteration, Transfer Learning & Parameter Sweeps** | Loading precomputed Tier-1 embeddings/features (`data/embeddings/`) to train a lightweight probe or run a parameter/channel ablation in seconds on CPU, including block-bootstrapped confidence intervals. **Primary template for `R2` and `M4` MVPs.** | Tier 1 (`data/embeddings/`)<br/>$\le 10\,\text{s}$ |
| **`notebooks/05_event_diagnostics_and_attribution.ipynb`** | **Event Case Studies, Attribution & Visual Diagnostics** | Running deep-dive diagnostics on a benchmark event (`data/events/`): event timelines, reliability/calibration diagrams, and spatial/physical attribution maps. **Wrapped by the gold-standard reference skill in `skills/`.** | Tier 3 (`data/events/`)<br/>$\le 30\,\text{s}$ |

> **Mandatory Companion Index (`notebooks/README.md`)**:
> Create `notebooks/README.md` containing a single markdown lookup table:
> `| Notebook | Science / Engineering Question Answered | Key Toolkit Functions Demonstrated | Data Tier Used | Expected Runtime |`
> Coding agents read `@notebooks/README.md` first whenever a participant asks *"How do I do X with this toolkit?"*

#### Group 2: The 3 Backup Science Case Starter Notebooks (`backup_cases/`)
For participants who need a ready-made project at the Module 3 Plan Gate (`A3`), provide three folders (`backup_cases/case_alpha/`, `case_beta/`, `case_gamma/`), each containing:
- A pre-approved `PLAN.md` scoped to $\le 5$ hours with its target `M6` skill name,
- A starter notebook `starter_analysis.ipynb` (and 15-line `starter_check.py`) that loads the exact `data/` slice and sets up the top configuration cell so the participant can immediately begin prompting their coding agent in Module 4.

#### Mandatory Engineering Standards for All Workshop Notebooks
Every notebook in `notebooks/` and `backup_cases/` **MUST** satisfy four rules:
1. **Top Configuration Cell**: Cell 1 of every notebook must define all configurable variables (`DATA_DIR`, `EVENT_ID`, `START_DATE`, `END_DATE`, `CHANNELS`, `HORIZON_HOURS`, `OUTPUT_DIR`) with sensible defaults so the coding agent can inspect and modify experimental parameters in one predictable location.
2. **Short, Self-Contained Cells ($\le 25$ lines)**: Break long cells into logical steps with descriptive markdown headers (`## Step 1: Load & Register...`, `## Step 2: Physical Sanity Check...`). Move complex loops or plotting boilerplate into `src/<toolkit>/`.
3. **Committed Outputs**: Execute all tutorial notebooks before committing so participants and agents can inspect existing cell outputs and figures immediately upon cloning.
4. **Offline Execution Guarantee**: Every notebook must execute top-to-bottom offline in under 60 seconds with zero network calls.

---

### 3.3 Deliverable 3: Participant Guide — How to Interact with Notebooks & Prompt the Coding Agent

Developers must prepare `participant-guide.md` (and `prompts/*.md`) so participants have a clear operational manual explaining **how to work with notebooks through a coding agent** and **concrete copy-pasteable prompt examples** for every module of the workshop.

Below is the exact operational framework and prompt catalog that developers must customize with the project's function names, data paths, and science cases.

#### A. How Participants Interact with Notebooks (The Golden Rules for the Participant Guide)

Include this workflow explanation at the front of the Participant Guide:

1. **You Direct; The Agent Edits and Runs**:
   - Keep JupyterLab (or VS Code / Colab notebook viewer) open on half your screen to **view** markdown, code diffs, tables, and plots.
   - Keep your **AI Coding Agent CLI** (Gemini CLI, Claude Code, Codex) open in your terminal in the repository root on the other half of your screen.
   - Instead of typing code into notebook cells manually, **issue prompts to the agent** referencing exact files with `@` (e.g., `@notebooks/01_ingest_and_calibration.ipynb`, `@src/<toolkit>/prep.py`, `@data/README.md`).
2. **The Two-Step Prompting Ritual (*Plan First $\to$ Update & Run Second*)**:
   - **Never** ask the agent to *"build my whole analysis notebook"* in one shot.
   - **Step 1 (Plan & Cite)**: Ask the agent to inspect the reference notebook and toolkit source, list which cells or `.py` functions need to be created/modified with exact `file:line` citations, state the physical units/assumptions, and **wait for your approval**.
   - **Step 2 (Apply, Execute & Inspect)**: Once you approve the plan, instruct the agent to write reusable processing functions into `work/<name>/src/<module>.py`, update the notebook cells to call those functions, execute the notebook, and report the output metrics and figures.
3. **Why Reusable Code Goes in `work/<name>/src/*.py` and Plots Stay in `.ipynb`**:
   - Having the agent edit 200-line algorithms directly inside `.ipynb` JSON is slow, error-prone, and impossible to unit-test with `pytest`.
   - Instead, instruct the agent: *"Put the data-processing/modeling function in `work/<name>/src/<module>.py`, write a quick unit test in `work/<name>/tests/`, and keep the notebook cell to 5–10 lines that import the function and plot the result."*
   - Bonus: In Module 6 (`X3`), wrapping `work/<name>/src/*.py` into a reusable **Agent Skill** takes 5 minutes because your logic is already modular!

---

#### B. Catalog of Prompt Patterns & Concrete Examples (To Adapt in `participant-guide.md`)

Developers must provide concrete examples for each of the **six core notebook interaction patterns** below, substituting our toolkit's actual module/function names and staged datasets:

##### Pattern 1: Inspecting & Understanding an Existing Notebook (`M1 · Ground`)
Participants prompt the agent to explain what a notebook does scientifically, verify toolkit assumptions, and extract working snippets without executing or modifying files.

> **Example Prompt — Cell-by-Cell Scientific Walkthrough (`G2`):**
> ```text
> Walk me through @notebooks/01_ingest_and_calibration.ipynb cell by cell, for a domain scientist who is an intermediate Python programmer.
> For each cell, state:
> (a) What it does scientifically,
> (b) Which `<toolkit>` function it calls and what that function assumes (physical units, data calibration level, coordinate frame, cadence, masking), citing `src/<toolkit>/<file>.py:<line>`, and
> (c) One thing that would silently go wrong scientifically if I changed the input data.
> Keep it under 600 words. Do not execute or modify any files.
> ```
>
> **Example Follow-Up Prompt — Checking a Specific Physical Convention in a Cell:**
> ```text
> Look closely at Cell 5 of @notebooks/01_ingest_and_calibration.ipynb and @src/<toolkit>/prep.py.
> When the notebook calls `register_level15()` followed by `correct_degradation()`, which function normalizes pixel values by exposure time? Cite the exact lines in `src/<toolkit>/prep.py` so I don't accidentally normalize by exposure time twice when I adapt this notebook.
> ```

##### Pattern 2: Reproducing a Notebook Run & Perturbing Parameters/Events (`M2 · Reproduce`)
Participants prompt the agent to run a reference notebook, inspect its outputs and warnings, and then create and run a modified notebook variant targeting a new physical event, channel subset, or forecast horizon.

> **Example Prompt — Baseline Notebook Execution (`R1`):**
> ```text
> Run @notebooks/03_pretrained_model_and_baselines.ipynb from top to bottom without modifying its code.
> Report:
> 1. Total execution time,
> 2. Every warning emitted during cell execution (including header, coordinate, or NaN warnings), and
> 3. The exact test-set metrics and baseline comparison scores printed in the final cells.
> Do not edit any files—just execute and report.
> ```
>
> **Example Prompt — Modifying a Notebook for a New Benchmark Event or Parameter (`R2`):**
> ```text
> I want to create a modified version of @notebooks/05_event_diagnostics_and_attribution.ipynb that evaluates model predictions and spatial attribution on Benchmark Event 2 (`data/events/<event_2>/`) instead of Benchmark Event 1 (`data/events/<event_1>/`).
>
> Step 1 (Plan only — do NOT edit files yet):
> - List every cell and parameter in `@notebooks/05_event_diagnostics_and_attribution.ipynb` that must change (file paths, UTC date strings, event identifier, plotting bounds).
> - Flag any physical or calibration assumptions in `<toolkit>` that could behave differently between these two events (e.g., instrument degradation epoch, viewing geometry, cadence).
> Wait for my approval.
>
> Step 2 (After I approve):
> - Create `work/<name>/05_event2_variant.ipynb`, apply the changes, execute all cells, and summarize how the predicted timeline and attribution map changed between the two events.
> ```

##### Pattern 3: Importing/Ingesting New Data Slices & Adding Processing Steps to a Notebook (`M4 · Solve`)
Participants prompt the agent to ingest data from `data/`, write a clean helper function in `work/<name>/src/`, test it with `pytest`, and update their analysis notebook to run it and generate the First-Light diagnostic plot.

> **Example Prompt — Data Ingestion & Feature Construction (`S1`):**
> ```text
> We are implementing Step 1 of @work/<name>/PLAN.md:
> "Ingest the co-aligned multi-channel cubes from `data/cubes/` (see @data/README.md) and construct three calibrated input variants: (A) baseline channels only, (B) baseline + additional channels with `correct_degradation()`, and (C) baseline + additional channels WITHOUT `correct_degradation()`."
>
> Constraints:
> 1. Use existing `<toolkit>.io` and `<toolkit>.prep` functions (see @work/<name>/notes/my_cheatsheet.md). Search `src/<toolkit>/` before writing any custom code.
> 2. Write the reusable data-loading and variant-builder functions in `work/<name>/src/channel_variants.py` with docstrings stating physical units, coordinate frames, and array shapes.
> 3. Write a fast unit test in `work/<name>/tests/test_channel_variants.py` on a 3-timestamp slice verifying finite values and expected physical units.
> 4. Update `work/<name>/analysis.ipynb` so Cell 2 imports `channel_variants`, loads the data, and displays a summary table of shapes, time ranges, and channel medians.
>
> First show me a concise 6-line plan citing the exact `<toolkit>` functions (`file:line`) you will call, and wait for my approval before creating or editing any files.
> ```
>
> **Example Prompt — Adding a First-Light Diagnostic Plot Cell (`S2`):**
> ```text
> Add a "First-Light Diagnostic" section to `work/<name>/analysis.ipynb` (and save the figure to `work/<name>/figures/first_light.png`) that plots:
> 1. Top row: a multi-panel view at `<benchmark_timestamp_utc>` showing each physical channel in its calibrated physical units using `<toolkit>`'s plotting helpers, with coordinate grid overlays and labeled axes.
> 2. Bottom row: time series of disk-integrated median channel intensity BEFORE vs. AFTER `correct_degradation()`, with the reference catalog activity level overplotted on the right axis.
> Show me the cell plan first. After I approve, update the notebook, execute the cells, and confirm that `work/<name>/figures/first_light.png` was created cleanly.
> ```

##### Pattern 4: Auditing the Participant's Notebook for Domain Leakage & Circularity (`M5 · Prove`)
Participants prompt the agent to cross-check their notebook and helper modules against the toolkit's domain rules to catch subtle methodological flaws that do not throw Python exceptions.

> **Example Prompt — Auditing the Notebook for Leakage & Physical Circularity (`P2`):**
> ```text
> Audit my analysis notebook `@work/<name>/analysis.ipynb` and helper module `@work/<name>/src/` for data leakage and domain circularity. Check every item below and cite exact cell numbers and `file:line`:
> 1. Temporal / physical persistence leakage: is there an adequate buffer (`domain_temporal_split(buffer_days=...)`) between train, validation, and test splits so the same evolving physical region or structure never straddles two splits?
> 2. Forecast horizon lookahead: for issue time `t0`, do any input features, rolling statistics, or scalers use observations recorded after `t0`?
> 3. Event-in-progress contamination: how does performance change if we exclude timestamps where an active event is already underway at `t0`?
> 4. Threshold & scaler discipline: were feature scalers and the classification threshold (`find_optimal_threshold`) fitted strictly on train/validation and frozen before evaluating on the test split?
> 5. Data gap handling: are instrument dropout or eclipse intervals masked out rather than labeled as quiet (`0`)?
> Output a severity-ranked markdown table. Do not modify any files yet.
> ```

##### Pattern 5: Running Baselines, Uncertainty Quantification & Hostile Referee Tests in the Notebook (`M5 · Prove`)
Participants prompt the agent to augment their notebook with fair baselines, block-bootstrapped error bars, and a fast empirical check addressing a journal referee's objection.

> **Example Prompt — Adding Baselines & Block-Bootstrapped Error Bars (`P1` + `P5`):**
> ```text
> Update `work/<name>/analysis.ipynb` to add a "Baseline & Uncertainty Evaluation" section:
> 1. Using the exact same test split and validation-tuned threshold policy, evaluate `<toolkit>.models.PersistenceBaseline`, `<toolkit>.models.ClimatologyBaseline`, and the unmodified reference model from `@notebooks/03_pretrained_model_and_baselines.ipynb`.
> 2. Compute 95% confidence intervals on all evaluation metrics for every model using `<toolkit>.eval.block_bootstrap_metric` grouped by independent physical blocks/rotations (never i.i.d. frame resampling).
> 3. Export the comparison table to `work/<name>/results_table.md` and update `work/<name>/figures/main_result.png` with the error bars.
> Show me the plan first, wait for approval, then update and run the notebook.
> ```

##### Pattern 6: Cleaning the Notebook & Distilling It into a Reusable Agent Skill (`M6 · Skillify`)
Once the science is proven, participants prompt the agent to clean up their notebook for end-to-end reproducibility and package the workflow into `skills/<skill-name>/`.

> **Example Prompt — Final Notebook Cleanup & End-to-End Verification (`X1`):**
> ```text
> Restructure `@work/<name>/analysis.ipynb` into 6 clean, well-documented sections:
> 1. Setup & Configurable Parameters
> 2. Data Ingestion & Physical Preprocessing
> 3. Method Implementation (imported from `work/<name>/src/`)
> 4. Results vs. Domain Baselines (with block-bootstrapped 95% CIs)
> 5. Physical Plausibility & Hostile-Referee Validation Checks
> 6. Conclusions & Next Steps
> Remove any dead scratch cells, verify that all reusable logic lives in `work/<name>/src/`, and execute the entire notebook from top to bottom in a clean kernel (`work/<name>/runs/final_verified.ipynb`).
> Confirm that `results_table.md` and `figures/main_result.png` are regenerated identically with zero errors.
> ```
>
> **Example Prompt — Distilling the Validated Notebook Workflow into an Agent Skill (`X3`):**
> ```text
> Read the gold-standard reference skill `@skills/<reference-skill>/SKILL.md`, `@templates/SKILL_TEMPLATE.md`, my validated code in `@work/<name>/src/`, my notebook `@work/<name>/analysis.ipynb`, and `@work/<name>/ERROR_LOG.md`.
> We are going to package my validated notebook workflow into a reusable Agent Skill under `skills/<skill-name>/` so any scientist can run this analysis on a new date range, event, or threshold in a single prompt.
>
> Phase 1 (Design only — do NOT write files yet):
> 1. Propose the hyphenated skill name and YAML `description` (including explicit "Use when..." and "Do not use for..." triggers).
> 2. List which notebook parameters become required CLI flags (`--data-path`, `--start-date`, `--end-date`, `--output-dir`, etc.) vs. which domain guardrails remain strictly locked inside the code.
> 3. Propose the CLI wrapper script `skills/<skill-name>/scripts/<skill_cli>.py` and the top 3 domain pitfalls from `ERROR_LOG.md` to include under `## Common Mistakes`.
> Wait for my approval before creating the files and running `python env/validate_skill.py skills/<skill-name>`.
> ```

---

### 3.4 Day-by-Day Developer Schedule During Preparation Week

Use this 5-day schedule to coordinate the development team and hit all qualification gates before the workshop begins.

| Day | Focus Area | Concrete Developer Tasks | End-of-Day Deliverable |
|---|---|---|---|
| **Day 1 (Mon)** | **Architecture Inventory, Scientific Goals & Offline Data Staging** | 1. Complete the Toolkit Architecture Inventory table (§3.1.D).<br/>2. Define the 3 Worked Scientist Personas / Backup Cases (`Case α, β, γ`) and workshop scientific goals (§3.1.A–B).<br/>3. Extract and verify the 3-tier offline dataset in `data/` (Tier 1 fast embeddings/catalogs $< 250\,\text{MB}$, Tier 2 cubes, Tier 3 active + quiet benchmark events).<br/>4. Write `data/README.md` with exact file paths, schemas, physical units, coordinate frames, and known data gaps. | `data/` populated & verified offline; `data/README.md` complete; 3 science cases defined. |
| **Day 2 (Tue)** | **Agent-Ready Toolkit API, Guardrails & Core Tutorial Notebooks (`01`–`05`)** | 1. Audit the 15–20 core public functions in `src/<toolkit>/`: add structured docstrings (units, frames, what each function does *not* do, 3-line examples) and loud runtime `ValueError` guardrails.<br/>2. Ensure built-in domain baselines (`PersistenceBaseline`, `ClimatologyBaseline`), leak-free splitter, and block bootstrap exist in `src/<toolkit>/`.<br/>3. Build/refactor Tutorial Notebooks `01` through `05` (§3.2 Group 1) with top configuration cells and $\le 25$-line modular cells.<br/>4. Write `notebooks/README.md` and `docs/CHEATSHEET.md`. | `notebooks/01_...` to `05_...` execute top-to-bottom in $< 60\,\text{s}$ offline; `docs/CHEATSHEET.md` ready. |
| **Day 3 (Wed)** | **Backup Case Starters, Reference Skill & Agent Context (`GEMINI.md`)** | 1. Create `backup_cases/case_{alpha,beta,gamma}/` with pre-approved `PLAN.md` and starter notebooks (`starter_analysis.ipynb`).<br/>2. Build 1 gold-standard reference skill in `skills/<reference-skill>/` wrapping Notebook `05`.<br/>3. Write the root `GEMINI.md` ($\le 150$ lines) with our toolkit's domain conventions and symlink `AGENTS.md -> GEMINI.md` and `CLAUDE.md -> GEMINI.md`. | `backup_cases/`, reference skill, and `GEMINI.md` committed and verified. |
| **Day 4 (Thu)** | **Participant Guide Customization & Verification Scripts** | 1. Customize `participant-guide.md` and `prompts/*.md` with our toolkit's function names, notebook filenames, and the 6 notebook prompt patterns (§3.3).<br/>2. Configure and test `env/check_setup.py`, `env/validate_skill.py`, and `templates/` (`PLAN_TEMPLATE.md`, `ERROR_LOG.md`, `HANDOFF.md`, `SKILL_TEMPLATE.md`, `skill_cli_template.py`). | Complete draft of `participant-guide.md`; reference skill passes `python env/validate_skill.py`. |
| **Day 5 (Fri)** | **Pre-Workshop Qualification Gates & Facilitator Dry Run** | Execute all 3 mandatory Qualification Gates (below) with network access disabled. Fix any docstrings, notebook cells, or prompt cards where the coding agent stumbles. Tag the release branch for participants. | **All 3 Qualification Gates GREEN**; repository frozen and ready for participants. |

---

## 4. Final Pre-Workshop Qualification Checklist ("Definition of Done")

Do **not** consider the workshop materials complete until a developer who did not author the notebooks runs and passes all three checks on Friday:

- [ ] **Gate 1 — Automated Offline Repo, Notebook & Skill Sweep**:
  With network access disabled, run:
  ```bash
  python env/check_setup.py
  python env/validate_skill.py skills/<reference-skill-name>
  pytest -q tests/
  jupyter nbconvert --to notebook --execute notebooks/*.ipynb --output-dir /tmp/nb_verify
  ```
  Every command **must** exit `0`, and each notebook must finish in $< 60\,\text{s}$.

- [ ] **Gate 2 — The 10-Prompt Agent Smoke Test ($\ge 8/10$ Required)**:
  Start a fresh coding agent session in the repository root and run 10 canonical prompts covering:
  1. Tracing calibration/registration functions and unit conversions (`file:line`),
  2. Coordinate projection and boundary masking rules,
  3. Model input channel ordering and scaling requirements,
  4. Domain-safe train/validation/test splitting without autocorrelation leakage,
  5. Invoking the pre-installed reference skill in `skills/` on Benchmark Event 1,
  6. Handling observation gaps/dropouts during target labeling,
  7. Running Notebook `03` and reporting test skill scores against `PersistenceBaseline` and `ClimatologyBaseline`,
  8. Perturbing Notebook `04` or `05` for a new threshold or benchmark event,
  9. Explaining validation-only threshold selection discipline, and
  10. Computing block-bootstrapped 95% confidence intervals.
  Verify that the agent cites accurate `src/<toolkit>/<file>.py:<line>` and notebook cells without inventing APIs.

- [ ] **Gate 3 — End-to-End 90-Minute Facilitator Dry Run**:
  Have a team member step through `M0` $\to$ `M6` as a participant using **Backup Case α** and only `participant-guide.md`—issuing prompts to the coding agent to ingest data, modify and run the notebook, generate `figures/first_light.png` within 45 minutes, run the `P1`–`P5` validation suite, and distill a working Agent Skill (`X3`–`X4`) that passes `env/validate_skill.py` and executes in a fresh agent session.
