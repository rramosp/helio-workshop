# Workshop Preparation Guide for Project Developers

This document provides instructions for project developers to organize and prepare the materials for our **1.5-day AI-Assisted Scientific Computing Workshop**.

## Executive Summary

- **Workshop in AI-Assisted Notebook Mode**: Participants (science users) interact with the project through Jupyter notebooks by prompting an AI coding agent—rather than writing code by hand—to download and ingest data, run the project's pretrained models and artifacts (without retraining), execute modified notebooks, and validate scientific results.
- **The Three Deliverables Each Team Must Prepare ([Section 3](#3-planning-workshop-materials))**:
  1. **Scientific Goals & Data/Compute Requirements ([§3.1](#31-deliverable-1-scientific-goals-benchmark-cases--hybrid-dataartifact-architecture))**: Define the operational and scientific learning goals for participants along with data volume and compute estimates for both project-provided and participant-brought datasets.
  2. **Workshop Notebooks List ([§3.2](#32-deliverable-2-list-of-notebooks-to-be-developed-and-organized))**: Provide a numbered sequence of modular, pre-executed Jupyter notebooks (`01_`, `02_`, ...) together with a brief explanation of what each one would do (not the actual notebook, just the list and description).
  3. **Draft Participant Guide & Prompt Examples ([§3.3](#33-deliverable-3-draft-participant-guide--how-to-interact-with-notebooks--prompt-the-coding-agent))**: Prepare an initial draft of `participant-guide.md` outlining practical workflow guidelines and preliminary prompt examples showing how science users will direct the coding agent across the workshop modules (to be refined into the final guide once the notebooks are built).

---

## 1. Introduction: Overall Mechanics of the Workshop

### 1.1 The Core Paradigm: Prompt-Driven Jupyter Notebooks with AI Coding Agents

This workshop is built around **Jupyter Notebooks (`.ipynb`)** backed by our project's Python toolkit (`src/<toolkit>/`) and operated through **AI-assisted coding agents** (e.g., Gemini CLI, Claude Code, Codex, Cursor).

Unlike a traditional tutorial where participants passively press `Shift+Enter` on pre-written cells—or a traditional hackathon where participants spend hours writing and debugging raw boilerplate code by hand—this workshop uses an **agent-mediated scientific workflow**:

> **Participants do not manually write or edit large blocks of code inside the notebooks.** Instead, they act as **science users**, issuing grounded natural-language prompts to a coding agent to:
> 1. **Discover, download, and ingest data**—whether fetching slices from our project's online data archives, downloading external datasets brought by the participant from the internet, or loading pre-staged local benchmark catalogs—using idempotent caching and official toolkit loaders,
> 2. **Modify, parameterize, and update Jupyter notebooks** (and companion `.py` helper modules) to run the project's pretrained models and artifacts on new events, datasets, or parameter regimes,
> 3. **Run the notebooks and automated sanity checks**, and
> 4. **Analyze, validate, and critique the scientific results** against domain baselines, audit for subtle physical/statistical pitfalls, and package their validated workflow into a reusable **Agent Skill**.


### 1.2 Operational Focus: Using Project Models & Artifacts (Not Retraining Models)

The primary goal of the workshop is **operational and scientific**: enabling science users to **use the pretrained models, calibrated pipelines, embeddings, and derived artifacts already built by the project** to generate predictions, analyze physical events, and answer domain science questions.

> [!IMPORTANT]
> **Inference & Analysis Only — No Model Retraining:**
> Science users **do not train or retrain machine learning models** during the workshop. Retraining models introduces heavy compute bottlenecks, hyperparameter tuning distractions, and training-pipeline complexity that divert time away from scientific exploration.
>
> Instead, all workshop notebooks and participant projects treat the project's models and artifacts as **operational scientific instruments**. Participants use the coding agent to:
> - **Run inference and generate predictions** with the project's pretrained model checkpoints on new time intervals, events, or input channel configurations,
> - **Query and analyze precomputed project artifacts** (such as model embeddings, prediction catalogs, reconstructions, or attribution maps) alongside observational datasets,
> - **Combine project predictions/artifacts with external datasets** downloaded from the internet or brought by the participant, and
> - **Evaluate operational performance and physical plausibility** across different physical regimes, viewing geometries, or event classes against domain baselines (e.g., Persistence, Climatology, or standard physical models).

### 1.3 Why This Mechanics Changes How Developers Must Prepare Notebooks & Data Access

To make the agent-mediated notebook mechanics work reliably across 15–25 participants, **our workshop repository must be engineered for both human science users and coding agents**:
1. **Thin, modular notebooks (`notebooks/01_...` to `05_...`)**: Every cell is short ($\le 25$ lines), calls high-level functions from `src/<toolkit>/` (for data ingestion, preprocessing, model inference, and artifact analysis), and defines configurable inputs clearly in the top cell.
2. **Helper logic (including custom data downloaders/adapters) extracted to `.py` files**: For custom participant analyses in Module 4, the agent writes reusable functions—including any download scripts or adapters for participant-brought external datasets—into `work/<name>/src/*.py` with unit tests in `work/<name>/tests/`, keeping the notebook cells clean.
3. **Explicit domain context (`GEMINI.md` / `AGENTS.md` / `CLAUDE.md`)**: A concise ($\le 150$-line) root instruction file loaded automatically by the coding agent that enforces *Toolkit First*, *Use Pretrained Models & Artifacts (Do Not Retrain)*, *Plan Before Code*, *Dry-Run & Cache Before Downloading*, *Cite `file:line`*, and our toolkit's non-negotiable physical and statistical conventions.
4. **Loud runtime guardrails**: Toolkit functions raise clear `ValueError` exceptions whenever an agent passes wrong units, uncalibrated data levels, naive timestamps, or invalid model inputs.


---

## 2. Suggested Structure of the Workshop: Modules `M0` to `M6` (The GRASP Ladder)

The 1.5-day curriculum follows the **GRASP** progression (**G**round $\to$ **R**eproduce $\to$ **A**rticulate $\to$ **S**olve $\to$ **P**rove & Skillify). Across modules `M0`–`M6`, the coding agent's autonomy expands only as the science user demonstrates mastery of the toolkit, its pretrained models/artifacts, and its physical assumptions.

| Module | Time | Agent Role | Participant Role | How Participants Use the Agent with Notebooks, Models & Data | Key Deliverable / Gate |
|---|---|---|---|---|---|
| **M0 · Setup & First Contact** | Day 1<br/>30 min | **Environment Checker** | Operator | Prompt agent to summarize rules loaded from `GEMINI.md`, list the notebooks in `@notebooks/README.md`, verify local data/model cache + remote archive connectivity via `env/check_setup.py`, and run `pytest -q tests/`. | Green setup, model checkpoint & data-access check; `work/<name>/` sandbox initialized. |
| **M1 · GROUND** | Day 1<br/>60 min | **Tutor**<br/>*(Read-Only)* | Learner & Fact-Checker | Prompt agent to map `@src/<toolkit>/` (loaders, preprocessing, pretrained model inference, artifacts) and `@notebooks/README.md` (`G1`), explain a tutorial notebook cell-by-cell (`G2`), quiz the participant on physical assumptions (`G3`), and assemble a personal cheat sheet (`G4`). | `work/<name>/notes/toolkit_map.md` & `my_cheatsheet.md` with **3 manually verified `file:line` citations**. |
| **M2 · REPRODUCE** | Day 1<br/>60 min | **Navigator** | Experimentalist | Prompt agent to run a baseline inference/prediction notebook and summarize its metrics and warnings (`R1`), then plan, create, and run a modified notebook variant that fetches/loads a new event window, channel configuration, or operational parameter (`R2`) and analyze how the predictions changed physically. | Executed variant notebook `work/<name>/<variant>.ipynb` + 3-sentence physical interpretation of the perturbation. |
| **M3 · ARTICULATE** | Day 1<br/>60 min | **Skeptical Collaborator**<br/>*(No Code Allowed)* | Science User (Scoping) | Prompt agent to challenge scope and draft a 5-hour Minimum Viable Science plan (`PLAN.md`) specifying how the project's pretrained models/artifacts will be applied, the exact data source (cached project data, bounded download from project archive, or participant-brought internet dataset), estimated download size, fallback slice, and toolkit functions per step (`A1`–`A2`). | **`PLAN.md` signed off by a facilitator at the Plan Gate (`A3`)** (verifying operational scope & data download size) before custom code is written. |
| **M4 · SOLVE (Sprint 1)** | Day 1<br/>120 min | **Pair Programmer** | Science User & Code Reviewer | Prompt agent in a strict *Plan $\to$ Approve $\to$ Download/Ingest $\to$ Update `.py` & `.ipynb` $\to$ Run $\to$ Sanity-Check* loop (`S1`–`S4`) to fetch and cache required data, adapt any external dataset headers/units, run pretrained model inference or artifact queries in the notebook, run physical unit/range assertions, and generate a First-Light plot. End Day 1 with `HANDOFF.md` (`S6`). | Cached & validated dataset + **90-Minute First-Light Plot (`figures/first_light.png`)** passing facilitator check + `test_sanity.py` + `HANDOFF.md`. |
| **M5 · SOLVE (Sprint 2) + PROVE** | Day 2<br/>85 min | **Hostile Journal Referee** | Defender & Validator | Finish MVP notebook (40 min), then switch agent to Hostile Referee mode (45 min): prompt agent to compare predictions against domain baselines (`P1`), audit the notebook/code for lookahead, alignment errors & circularity (`P2`), run benchmark event & negative-control checks (`P3`), critique the findings as a hostile referee (`P4`), and add block-bootstrapped error bars (`P5`). | `results_table.md` (with baselines & CIs), `figures/main_result.png`, evaluation audit report, and 1 executed referee test. |
| **M6 · PACKAGE, SKILLIFY & PRESENT** (optional) | Day 2<br/>100 min | **Skill Architect & Technical Editor** | Author & Presenter | Prompt agent to clean and re-run the final notebook end-to-end from cached data (`X1`), generate a 1-page `FINDINGS.md` (`X2`), distill the notebook pipeline + `ERROR_LOG.md` into a reusable **Agent Skill** (`skills/<name>/SKILL.md` + CLI script) (`X3`), test a pod-mate's skill in a fresh session (`X4`), and file toolkit GitHub issues (`X5`). | Reproducible `final_verified.ipynb`, `FINDINGS.md`, validated `skills/<name>/`, and 5-min lightning talk. |

---

## 3. Planning workshop materials

Below is the structured plan for the development team to design and prepare the workshop materials, organized around **three primary deliverables**: **(3.1) Scientific Goals & Data/Compute Requirements**, **(3.2) Workshop Notebooks List**, and **(3.3) A Draft Participant Guide & Suggested Prompts**.

---

### 3.1 Deliverable 1: Scientific Goals, Benchmark Cases & Hybrid Data/Artifact Architecture

Before writing or refactoring notebooks, the development team must define **what operational and scientific outcomes participants will achieve** using the project's models and artifacts, and engineer a **resilient data and artifact access layer** that supports both fast pre-staged slices and controlled live downloads.

#### A. Core Scientific & Operational Goals for Participant

Describe what are the learning goals to achieve by the participants, for instance:

- achieve an understanding of the architecture of the system
- achieve an understanding of the artifacts offered (models, catalogues, data, etc.)
- achieve an understanding on how use the artifacts (how to run inferene to make predictions, analyze data, etc.)
- combine the above to address a scientific question
 
#### B. Data and computing requirements

Describe and provide estimates for volumetry of

- the data that the participant might bring
- additional the data that we might need to provide
- the amount of computing required for the participant's workflow (for instance, the kind of machine required to run the notebooks)


---

### 3.2 Deliverable 2: List of Notebooks to Be Developed and Organized

Provide the list of sequential tutorial notebooks (`notebooks/01_...`, `02_...`, `03_...`, etc.) that participants and their coding agents will use as building blocks during the workshop, along with a short description of what each notebook demonstrates.

Use the `01_`, `02_`, `03_` naming convention to reflect the natural progression from data ingestion to operational model/artifact usage and scientific analysis. For example:

- **`notebooks/01_data_download_and_ingestion.ipynb`** — Querying, downloading/caching, and inspecting project data or external datasets, checking physical units, coordinate frames, and metadata.
- **`notebooks/02_preprocessing_and_model_inputs.ipynb`** — Calibrating observations, handling data gaps, co-aligning channels/catalogs, and assembling inputs for the project's models.
- **`notebooks/03_running_model_predictions.ipynb`** — Loading a pretrained project model and running inference/predictions on a target event or time interval, comparing against baseline methods.
- **`notebooks/04_analyzing_precomputed_artifacts.ipynb`** — Loading and exploring precomputed project artifacts (e.g., embeddings, prediction catalogs, derived physical quantities) combined with auxiliary or participant-brought data.
- **`notebooks/05_event_diagnostics_and_visualization.ipynb`** — End-to-end case study on a benchmark event with diagnostic plots, uncertainty intervals, and physical interpretation.

Include a brief index file (`notebooks/README.md`) mapping each notebook filename to the scientific/operational task it solves and the main `<toolkit>` functions it uses, so coding agents can quickly locate the right reference notebook.

#### Notebook Guidelines for Agent Compatibility
1. **Top Configuration Cell**: Define configurable inputs (data paths, date ranges, event IDs, model checkpoints, output directories) in the first code cell.
2. **Short, Modular Cells ($\le 25$ lines)**: Keep cells concise and move complex helper logic into `src/<toolkit>/`.
3. **Pre-Executed Outputs**: Commit notebooks with their outputs and figures already rendered so participants and agents can inspect them immediately.
4. **Local Caching for Downloads**: Ensure any data download step caches files locally so re-running a notebook does not re-download data.

---

### 3.3 Deliverable 3: Draft Participant Guide — How to Interact with Notebooks & Prompt the Coding Agent

Prepare an **initial draft** of `participant-guide.md` (not the final, definitive version) that outlines how science users will **interact with your planned notebooks, pretrained models/artifacts, and datasets through a coding agent**, along with **preliminary example prompts** they might use during the workshop. This draft serves as a working blueprint that will be refined and finalized later once the notebooks and codebase are ready.

Sections **A** and **B** below are offered as **suggestions and starting templates** for this draft—feel free to select, adapt, or sketch out only the guidelines and prompt ideas that best match your project's expected workflows.

#### A. Suggested Guidelines for Interacting with Notebooks & Online Data

You may find it helpful to include some of the following practical tips in your draft Participant Guide:

1. **You Direct; The Agent Downloads, Edits, and Runs**:
   - Participants can keep JupyterLab (or VS Code / Colab notebook viewer) open on half their screen to **view** markdown, code diffs, tables, and plots, while running their **AI Coding Agent CLI** (Gemini CLI, Claude Code, Codex) in a terminal on the other half.
   - Rather than typing code into notebook cells manually, participants can **issue prompts to the agent** referencing relevant files with `@` (e.g., `@notebooks/01_...ipynb`, `@src/<toolkit>/...`, `@data/README.md`).
2. **Operate Pretrained Models & Artifacts — Avoid Retraining**:
   - Remind participants that the goal is to use the project's pretrained models, embeddings, and artifacts as scientific instruments to make predictions and test hypotheses on cached or newly downloaded data.
   - If the coding agent proposes training a new neural network or fitting a model from scratch, participants can redirect it: *"Use the pretrained model / precomputed artifacts in `<toolkit>` instead of retraining."*
3. **A Two-Step Prompting Habit (*Plan First $\to$ Update & Run Second*)**:
   - Encourage participants not to ask the agent to *"download the data and build the whole analysis notebook"* in one shot.
   - **Step 1 (Plan, Estimate Size & Cite)**: Ask the agent to inspect the reference notebook and toolkit source, outline which cells or `.py` functions it plans to create or modify (with `file:line` citations), check file sizes if downloading remote data, and **wait for approval**.
   - **Step 2 (Apply, Execute & Inspect)**: Once approved, have the agent download/cache the data, update the helper module and notebook cells, execute the notebook, and summarize the output predictions, metrics, and figures.
4. **Safe Remote Data Downloading (*Dry-Run $\to$ Cache $\to$ Single-File Inspect*)**:
   - When pulling from the project's online archive or downloading a participant-brought dataset from an external URL/API, consider suggesting that participants:
     1. **Estimate first**: Have the agent check file counts and total size before downloading so workshop sessions aren't stalled by large transfers.
     2. **Cache locally**: Save downloaded data in a local cache folder (e.g., `data/cache/` or `work/<name>/data/`) so re-running a notebook does not re-download files.
     3. **Inspect 1 sample file before bulk processing**: For external internet data, have the agent download **one** sample file first to check its headers, units, timestamp format, and missing-value sentinels before processing the full date range.
5. **Keeping Reusable Code in `work/<name>/src/*.py` and Plots in `.ipynb`**:
   - Suggest that participants ask the agent to put larger data-ingestion or analysis functions in `work/<name>/src/<module>.py` (with a quick test in `work/<name>/tests/`) while keeping notebook cells short (importing the function and plotting the result). This avoids `.ipynb` JSON editing errors and makes it much easier to package the workflow into a reusable **Agent Skill** in Module 6.

---

#### B. Suggested Prompt Patterns & Illustrative Examples

Below are **six suggested prompt patterns** across the workshop modules that you can use as inspiration when sketching out draft prompts for your project. Pick the patterns that make sense for your workflow and adapt the placeholders (`<toolkit>`, planned notebook names, data sources) into rough draft examples—exact function names and file paths can be filled in later as the notebooks are implemented:

##### Pattern 1: Inspecting & Understanding an Existing Notebook (`M1 · Ground`)
Participants can prompt the agent to explain what a notebook does scientifically, how it fetches/loads data and runs pretrained models, and what physical assumptions it makes—without executing or modifying files.

> **Illustrative Prompt — Cell-by-Cell Scientific Walkthrough (`G2`):**
> ```text
> Walk me through @notebooks/01_data_download_and_ingestion.ipynb cell by cell, for a domain scientist who is an intermediate Python programmer.
> For each cell, state:
> (a) What it does scientifically (including how it queries, caches, or loads the data),
> (b) Which `<toolkit>` function it calls and what that function assumes (physical units, data calibration level, coordinate frame, cadence, masking), citing `src/<toolkit>/<file>.py:<line>`, and
> (c) One thing that would silently go wrong scientifically if I pointed this notebook at a different downloaded dataset.
> Keep it under 600 words. Do not execute or modify any files.
> ```
>
> **Illustrative Follow-Up Prompt — Checking Pretrained Model Input Conventions:**
> ```text
> Look closely at Cell 5 of @notebooks/03_running_model_predictions.ipynb and @src/<toolkit>/models.py.
> What exact channel order, physical units, and intensity scaling does `load_pretrained_model()` expect during inference, and where in `@src/<toolkit>/prep.py` is normalization applied? Cite exact `file:line` numbers so I don't feed uncalibrated inputs into the pretrained model.
> ```

##### Pattern 2: Reproducing an Inference Notebook & Perturbing Parameters or Target Events (`M2 · Reproduce`)
Participants can prompt the agent to run a reference inference notebook, inspect its predictions and warnings, and then create and run a modified notebook variant that generates predictions on a new event or operational configuration.

> **Illustrative Prompt — Baseline Inference Notebook Execution (`R1`):**
> ```text
> Run @notebooks/03_running_model_predictions.ipynb from top to bottom without modifying its code.
> Report:
> 1. Total execution time,
> 2. Every warning emitted during cell execution (including header, coordinate, or NaN warnings), and
> 3. The exact operational prediction metrics and baseline comparison scores printed in the final cells.
> Do not edit any files—just execute and report.
> ```
>
> **Illustrative Prompt — Modifying a Notebook to Fetch & Predict on a New Event (`R2`):**
> ```text
> I want to create a modified version of @notebooks/05_event_diagnostics_and_visualization.ipynb that runs the pretrained model and diagnostics on a different active event (`<START_DATE>` to `<END_DATE>`).
>
> Step 1 (Plan & Data Check only — do NOT edit files or download bulk data yet):
> - Check `@data/README.md` to see if this date range is already available locally. If not, check how many files and megabytes it would take to download into `data/cache/`.
> - List every cell and parameter in `@notebooks/05_event_diagnostics_and_visualization.ipynb` that should change (file paths/query dates, event identifier, plotting bounds).
> - Flag any physical or calibration assumptions in `<toolkit>` that could behave differently for this event.
> Wait for my approval.
>
> Step 2 (After I approve):
> - Fetch/cache the slice if needed, create `work/<name>/05_event_variant.ipynb`, apply the changes, execute all cells, and summarize how the predicted timeline and diagnostics changed between the two events.
> ```

##### Pattern 3: Downloading Remote Data, Ingesting Participant-Brought Datasets & Running Pretrained Predictions (`M4 · Solve`)
In Module 4, participants might either (a) download a targeted slice from the project's online archive and run the pretrained model on it, or (b) download and adapt an external dataset they brought from the internet to compare against the project's predictions/artifacts. Here are suggested prompt templates for both scenarios:

> **Illustrative Prompt 3A — Downloading a Slice from the Project Archive & Running Pretrained Inference (`S1 — Project Archive`):**
> ```text
> We are implementing Step 1 of @work/<name>/PLAN.md:
> "Fetch and cache observations for `<START_DATE>` to `<END_DATE>` across channels `<CHANNELS>`, construct calibrated input variants (with vs. without `correct_degradation()`), and run `<toolkit>.models.run_inference()` using the pretrained checkpoint."
>
> Constraints:
> 1. First run a dry-run query using `<toolkit>.io` (`cite src/<toolkit>/io.py:<line>`) to verify the total download size is under 500 MB and check what is already cached in `data/cache/`.
> 2. Do NOT retrain any model—use the pretrained model checkpoint and inference helpers in `<toolkit>.models`.
> 3. Write a reusable data-fetching, preprocessing, and inference module in `work/<name>/src/inference_pipeline.py` that caches downloaded files idempotently in `data/cache/`.
> 4. Write a fast unit test in `work/<name>/tests/test_inference_pipeline.py` on a 2-timestamp sample verifying local cache hits, finite physical units, and valid prediction outputs.
> 5. Update `work/<name>/analysis.ipynb` so Cell 2 imports `inference_pipeline`, runs the predictions on the cached dataset, and displays a summary table of time ranges, channel medians, and predicted outputs.
>
> Show me the dry-run size estimate and a 6-line implementation plan first, and wait for my approval.
> ```
>
> **Illustrative Prompt 3B — Downloading & Adapting a Participant-Brought Internet Dataset (`S1 — External Data`):**
> ```text
> For my science project in @work/<name>/PLAN.md, I want to download an external dataset from `<URL / API / Zenodo / CDAWeb / VSO>` and analyze/evaluate it alongside `<toolkit>`'s pretrained model predictions and artifacts in `data/artifacts/`.
>
> Step 1 (Single-Sample Probe & Compatibility Plan — do NOT bulk download yet):
> 1. Download ONLY a single sample file (or the first 100 rows / 1 day of data) into `work/<name>/data/sample/` and inspect its structure.
> 2. Compare its format against `@src/<toolkit>/io.py` and `@src/<toolkit>/prep.py`, and report:
>    - Exact column names, array shapes, and FITS/NetCDF/Parquet metadata keys,
>    - Time standard and timezone (and how to convert it to UTC observation time matching `<toolkit>`),
>    - Physical units and coordinate frame (and what unit conversion or rescaling is required),
>    - Missing-data or fill-value sentinels (e.g., `-9999`, `NaN`) and cadence gaps,
>    - Estimated total download size for my full target date range (`<START_DATE>` to `<END_DATE>`).
> Wait for my approval.
>
> Step 2 (After I approve):
> 1. Write `work/<name>/src/external_adapter.py` with a cached downloader (`work/<name>/data/`) and an adapter function that aligns the external data with `<toolkit>`'s predictions/artifacts and physical units.
> 2. Write `work/<name>/tests/test_external_adapter.py` asserting UTC timestamps, unit consistency, and `NaN` masking of fill values.
> 3. Update `work/<name>/analysis.ipynb` to import `external_adapter`, ingest the data, and print a verification summary.
> ```
>
> **Illustrative Prompt 3C — Adding a First-Light Diagnostic Plot Cell (`S2`):**
> ```text
> Add a "First-Light Diagnostic" section to `work/<name>/analysis.ipynb` (and save the figure to `work/<name>/figures/first_light.png`) that plots:
> 1. Top row: a multi-panel view at `<benchmark_timestamp_utc>` showing each ingested physical channel in its calibrated physical units alongside the pretrained model's spatial attribution/output using `<toolkit>`'s plotting helpers, with coordinate grid overlays and labeled axes.
> 2. Bottom row: time series of the pretrained model's predictions over the target interval compared against our calibrated input summary and target catalog on the right axis.
> Show me the cell plan first. After I approve, update the notebook, execute the cells, and confirm that `work/<name>/figures/first_light.png` was created cleanly.
> ```

##### Pattern 4: Auditing the Participant's Notebook for Evaluation Circularity & Data Alignment Bugs (`M5 · Prove`)
Participants can prompt the agent to cross-check their notebook and helper modules (including any downloaded/external data joins) against the toolkit's domain rules to help catch subtle methodological flaws.

> **Illustrative Prompt — Auditing the Notebook for Lookahead, Alignment & Physical Circularity (`P2`):**
> ```text
> Audit my analysis notebook `@work/<name>/analysis.ipynb` and helper modules in `@work/<name>/src/` for evaluation circularity, timestamp misalignment, and physical bugs. Check every item below and cite exact cell numbers and `file:line`:
> 1. Forecast horizon lookahead: for issue time `t0`, do any input channels, rolling statistics, or external catalog joins use observations recorded after `t0`?
> 2. Multi-source / downloaded data alignment: when we joined downloaded or external data with project predictions/artifacts, were they matched strictly on UTC observation timestamps (`max_dt` tolerance) without forward-filling future values or misinterpreting timezones/cadence windows?
> 3. Event-in-progress contamination: how do the operational metrics change if we exclude timestamps where an active event is already underway at `t0`?
> 4. Threshold & calibration discipline: was the operational decision threshold fixed from the reference calibration set rather than tuned retrospectively on our target evaluation events?
> 5. Data gap & fill-value handling: are instrument dropout intervals and external fill values masked out as `NaN` rather than treated as real zero/quiet measurements?
> Output a severity-ranked markdown table. Do not modify any files yet.
> ```

##### Pattern 5: Running Baselines, Uncertainty Quantification & Hostile Referee Tests in the Notebook (`M5 · Prove`)
Participants can prompt the agent to augment their notebook with fair baselines, block-bootstrapped error bars, and a quick empirical check addressing a potential referee question.

> **Illustrative Prompt — Adding Baselines & Block-Bootstrapped Error Bars (`P1` + `P5`):**
> ```text
> Update `work/<name>/analysis.ipynb` to add a "Baseline & Uncertainty Evaluation" section:
> 1. Over the exact same evaluation timestamps and pre-calibrated threshold policy, compare our pretrained model predictions/variants against `<toolkit>.models.PersistenceBaseline` and `<toolkit>.models.ClimatologyBaseline`.
> 2. Compute 95% confidence intervals on all evaluation metrics for every method using `<toolkit>.eval.block_bootstrap_metric` grouped by independent physical blocks/rotations (never i.i.d. frame resampling).
> 3. Export the comparison table to `work/<name>/results_table.md` and update `work/<name>/figures/main_result.png` with the error bars.
> Show me the plan first, wait for approval, then update and run the notebook.
> ```

##### Pattern 6: Cleaning the Notebook & Distilling It into a Reusable Agent Skill (`M6 · Skillify`)
Once the science results are ready, participants can prompt the agent to clean up their notebook for end-to-end reproducibility (running cleanly from cached data and pretrained artifacts) and package the workflow into `skills/<skill-name>/`.

> **Illustrative Prompt — Final Notebook Cleanup & End-to-End Verification (`X1`):**
> ```text
> Restructure `@work/<name>/analysis.ipynb` into 6 clean, well-documented sections:
> 1. Setup, Configurable Parameters & Cached Data/Artifact Paths
> 2. Data Fetching/Caching, Ingestion & Physical Preprocessing
> 3. Pretrained Model Inference / Artifact Analysis (imported from `work/<name>/src/`)
> 4. Results vs. Domain Baselines (with block-bootstrapped 95% CIs)
> 5. Physical Plausibility & Hostile-Referee Validation Checks
> 6. Conclusions & Next Steps
> Remove any dead scratch cells, verify that all reusable logic lives in `work/<name>/src/` and that all data loads from local cache without redundant downloads, and execute the entire notebook from top to bottom in a clean kernel (`work/<name>/runs/final_verified.ipynb`).
> Confirm that `results_table.md` and `figures/main_result.png` are regenerated identically with zero errors.
> ```
>
> **Illustrative Prompt — Distilling the Validated Notebook Workflow into an Agent Skill (`X3`):**
> ```text
> Read the gold-standard reference skill `@skills/<reference-skill>/SKILL.md`, `@templates/SKILL_TEMPLATE.md`, my validated code in `@work/<name>/src/`, my notebook `@work/<name>/analysis.ipynb`, and `@work/<name>/ERROR_LOG.md`.
> We are going to package my validated operational workflow into a reusable Agent Skill under `skills/<skill-name>/` so any science user can run this prediction/analysis pipeline on a new date range, event, or external dataset in a single prompt.
>
> Phase 1 (Design only — do NOT write files yet):
> 1. Propose the hyphenated skill name and YAML `description` (including explicit "Use when..." and "Do not use for..." triggers).
> 2. List which notebook parameters become required CLI flags (`--data-path` or `--fetch-dates`, `--start-date`, `--end-date`, `--output-dir`, etc.) vs. which domain guardrails (caching, download size caps, unit checks, pretrained checkpoint selection) remain strictly locked inside the code.
> 3. Propose the CLI wrapper script `skills/<skill-name>/scripts/<skill_cli>.py` and the top 3 domain pitfalls from `ERROR_LOG.md` to include under `## Common Mistakes`.
> Wait for my approval before creating the files and running `python env/validate_skill.py skills/<skill-name>`.
> ```

---
