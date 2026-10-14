# Heliophysics Toolkit Workshop — Participant Handouts (AI-Mature Toolkit Edition)

> [!IMPORTANT]
> **About the worked examples (`sdoflare`):** Throughout these handouts, all concrete prompts, MCP tool calls, declarative validation files, bug-hunt scenarios, and skill-creation examples use an **imaginary Python package called `sdoflare`** — a hypothetical, AI-mature toolkit for preprocessing full-disk **SDO/AIA** (EUV) and **SDO/HMI** (magnetogram) images and running **operational solar flare prediction models** against the **GOES XRS** flare catalog.
>
> Everything here is meant to be given to participants verbatim (print **H0**; put the rest in `prompts/`, `specs/`, and `templates/` in the workshop repo) after replacing `sdoflare` with your actual toolkit.

---

## Reference AI-Mature Toolkit (`sdoflare`) & Science Cases

`sdoflare` is organized into **three layers** that you will learn to combine during the workshop:

### Layer 0 · Bare Python Library & Notebooks (`src/sdoflare/`, `notebooks/`)
Use when inspecting exact equations (`file:line`) or writing **new scientific logic** not yet in the MCP servers:
- `src/sdoflare/io.py` — `load_aia_map()`, `load_hmi_los()`, `load_goes_xrs()`, `load_flare_catalog()`, `SDOTimeIndex`
- `src/sdoflare/prep.py` — `aia_l1_to_l15()`, `correct_aia_degradation()`, `correct_hmi_projection()`, `coalign_full_disk()`, `scale_channels()`
- `src/sdoflare/labels.py` — `label_flare_window()`, `carrington_split(buffer_days=27)`
- `src/sdoflare/models.py` — `FullDiskFlareNet`, `extract_disk_embeddings()`, `LinearProbeForecaster`, `PersistenceBaseline`, `ClimatologyBaseline`, `attribute_disk_regions()`
- `src/sdoflare/eval.py` — `find_optimal_tss_threshold()`, `compute_tss_hss()`, `brier_skill_score()`, `block_bootstrap_metric()`, `run_declarative_validation()`
- **Notebooks**: `01_sdo_ingest_and_l15_prep.ipynb` through `05_evaluation_attribution_and_events.ipynb`

### Layer 1 · Domain MCP Servers (`mcp/`)
Schema-enforced, provenance-logged tools that you will install into your coding agent in **Module 0**:
- **`sdoflare-data-mcp`** (Data, Catalog & Calibration Server):
  - **Tools**: `query_sdo_inventory`, `inspect_fits_wcs_and_quality`, `prepare_l15_cube`, `check_eclipse_and_gap_flags`, `get_ar_heliographic_trajectory`
  - **Resources**: `sdoflare://data/manifest`, `sdoflare://calibration/aia_degradation_v10`, `sdoflare://catalogs/goes_flares`
- **`sdoflare-forecast-mcp`** (Forecasting, Attribution & Declarative Validation Server):
  - **Tools**: `run_fulldisk_forecast`, `train_and_eval_probe`, `run_baselines`, `attribute_active_regions`, `audit_split_and_leakage`, `execute_validation_suite`
  - **Resources**: `sdoflare://models/registry`, `sdoflare://specs/catalog`

### Layer 2 · Declarative Validation Specs (`specs/`) & Agent Skills (`skills/`)
- **`specs/operational_24h_m10_validation.json` + `specs/validation_protocol.md`** — Machine-executable contract defining required Carrington split buffers, physical unit bounds, required baselines, validation-only threshold selection, positive benchmark events (`ar12673_201709`, `ar12192_201410`), quiet-Sun negative controls (`quiet_201905`), and block-bootstrap CIs.
- **`specs/preprocessing_contract.json`** — Declarative invariants for L1.5 registration, single `EXPTIME` division, AIA degradation correction, HMI $\mu$-masking, and inter-channel timestamp synchronization.
- **`skills/sdo-ar-event-attribution/`** — Gold-standard reference Agent Skill showing how MCP tools + a custom `.py` CLI + a declarative spec are packaged together.

### Three worked scientist personas
- **Case α (Elena — EUV channel ablation & degradation):** *"Does adding hot flaring channels (AIA 94 Å, 131 Å) to HMI $B_{\text{los}}$ + AIA 171 Å improve 24h $\ge\text{M1.0}$ flare skill across 2014–2017, or does uncorrected AIA sensitivity degradation create a spurious solar-cycle trend?"* → Skill **`sdo-euv-channel-ablation`**
- **Case β (Marco — Limb foreshortening & $\mu$-angle bias):** *"How badly does full-disk flare forecast skill drop when the flaring active region is near the solar limb ($|\text{lon}| > 60^\circ$, $\mu < 0.5$), and does off-limb AIA 193/211 Å coronal ring brightness ($0.95 < R/R_\odot < 1.15$) recover that skill?"* → Skill **`sdo-limb-foreshortening-audit`**
- **Case γ (Priya — Short-horizon peak X-ray flux around Sep 2017 AR 12673):** *"Can I use precomputed `FullDiskFlareNet` embeddings to predict continuous 6-hour peak $\log_{10}(\text{GOES 1–8 Å flux})$ on quiet-background pre-flare timestamps?"* → Skill **`sdo-preflare-flux-probe`**

---

## H0 · The Agent Operating Contract (keep next to your keyboard)

**You are the Principal Investigator. The agent is a fast, overconfident assistant equipped with three layers of tools.**

1. **Right layer for the job (MCP for plumbing, Declarative Specs for validation, Bare Python for novel science).**
   - Use **`sdoflare-data-mcp` and `sdoflare-forecast-mcp`** for inventory queries, Level 1.5 calibration, eclipse filtering, baseline runs, and validation suites. Never let the agent write 60 lines of `sunpy`/`astropy` glue code when an MCP tool does it with schema checks and provenance logging.
   - Drop down to **bare Python (`work/<name>/src/*.py` importing `sdoflare`)** *only* for the novel physical feature, mask, or model head that your science case introduces.
2. **Ground every bare-code ask.** Point at `@src/sdoflare/...` and `@notebooks/...`, require `file:line` citations, and open at least one citation per answer.
3. **Lock in your declarative validation spec (`validation_spec.json`) before modeling.** Author your `work/<name>/validation_spec.json` in Module 3 before writing model code. You may *add* stricter checks later (in Module 5), never weaken them.
4. **Plan before code, one step at a time.** Approve which MCP tools and which `.py` functions will run before edits happen.
5. **Verify with both `execute_validation_suite` AND a Hostile Referee.** A green `validation_certificate.json` proves you respected standard rules; the Hostile Referee (`P4`) hunts for case-specific physics traps your initial spec missed.
6. **You own the physics.** Hypotheses, physical interpretations, and validation thresholds are human decisions.
7. **Log the misses — then bake them into your Skill and Spec.** Every error goes into `ERROR_LOG.md` and becomes a permanent rule in your Module 6 Skill and `validation_spec.json`.

**3-strikes rule:** after three failed fix attempts, stop prompting. Run prompt **S5**, inspect the source or MCP tool schema yourself, or ask a facilitator.

---

## H1 · Module 0 — Setup & Installing the Toolkit MCP Servers (45 min)

Goal: verify your Python environment, experience your coding agent *before* MCP is installed, then **install and verify `sdoflare-data-mcp` and `sdoflare-forecast-mcp`**.

### M0.1 · Base environment check (10 min)
```bash
cd sdoflare-workshop && git checkout -b <name>
python env/check_setup.py
mkdir -p work/<name>/{notes,src,tests,figures,runs}
cp templates/ERROR_LOG.md work/<name>/ERROR_LOG.md
```

### M0.2 · Observe the agent *before* MCP installation (5 min)
Start your coding agent in the repo root and ask:
```text
List all tools and MCP servers currently available to you. Do you have any domain-specific `sdoflare` tools registered yet, or only generic file/shell tools? Do not run any commands.
```
Notice that without MCP installed, the agent only has generic tools (`read_file`, `edit_file`, `run_command`) — to do *anything* with `sdoflare`, it would have to read source files and write custom Python scripts from scratch.

### M0.3 · Install & register the `sdoflare` MCP servers in your coding agent (15 min)

Run the installer for your coding agent (or inspect and run the CLI registration commands directly so you see how MCP servers are wired):

- **Option A — One-command installer (writes `.gemini/settings.json`, `.mcp.json`, and `.agents/mcp.json`):**
  ```bash
  python env/install_mcp.py --verify
  ```
- **Option B — Direct CLI registration (Gemini CLI / Claude Code):**
  ```bash
  # Register the Data & Calibration MCP Server
  gemini mcp add sdoflare-data-mcp -- python -m sdoflare.mcp.data_server --data-dir data --offline

  # Register the Forecast, Attribution & Validation MCP Server
  gemini mcp add sdoflare-forecast-mcp -- python -m sdoflare.mcp.forecast_server --data-dir data --specs-dir specs --offline
  ```

Open `.gemini/settings.json` (or `.mcp.json`) to see what was added:
```json
{
  "mcpServers": {
    "sdoflare-data-mcp": {
      "command": "python",
      "args": ["-m", "sdoflare.mcp.data_server", "--data-dir", "data", "--offline"],
      "env": {"SDOFLARE_OFFLINE": "1"}
    },
    "sdoflare-forecast-mcp": {
      "command": "python",
      "args": ["-m", "sdoflare.mcp.forecast_server", "--data-dir", "data", "--specs-dir", "specs", "--offline"],
      "env": {"SDOFLARE_OFFLINE": "1"}
    }
  }
}
```

### M0.4 · Reload the agent and inspect your new MCP tools & declarative specs (15 min)
Restart/reload your coding agent, run `/mcp` to confirm both servers show **`CONNECTED`**, then send this prompt:
```text
1. List all tools and resources exposed by `sdoflare-data-mcp` and `sdoflare-forecast-mcp`.
2. Inspect the input JSON Schema of `prepare_l15_cube`, `train_and_eval_probe`, and `execute_validation_suite`. Which parameters have strict constraints (enums, minimum buffer days, required threshold rules)?
3. Read resource `sdoflare://specs/catalog` (or `@specs/operational_24h_m10_validation.json`) and summarize in 5 bullets what a model run must satisfy to earn a PASS validation certificate.
```
Finally, run `python env/check_setup.py --with-mcp` and post ✅ in the workshop channel.

---

## H2 · Module 1 — GROUND & COMPARE: Bare Code vs. MCP Lab (75 min)

Goal: run a **controlled head-to-head experiment (`G1` vs `G2`)** on the exact same solar flare task so you see firsthand **what MCP servers + declarative specs buy you** over bare toolkit code — and **when you still need bare Python code**.

### The shared scientific task for `G1` and `G2`
> *"For the September 2017 AR 12673 event (`data/sdo_events/ar12673_201709/`): (1) check which timestamps between `2017-09-04` and `2017-09-10` have SDO eclipse or quality-flag dropouts, (2) calibrate the `2017-09-06T06:00:00Z` AIA 94/131/171 Å + HMI $B_{\text{los}}$ images to Level 1.5 with AIA degradation and HMI $\mu$-projection corrections, and (3) compute the 24h $\ge\text{M1.0}$ probability from `FullDiskFlareNet` and compare the 2014–2017 test TSS against `PersistenceBaseline`."*

### G1 · Run A: Bare Toolkit Code ONLY (MCP forbidden — 20 min)
```text
IMPORTANT CONSTRAINT FOR THIS RUN: Do NOT call any `sdoflare-data-mcp` or `sdoflare-forecast-mcp` tools or resources. Use ONLY file reads (`@src/sdoflare/`, `@notebooks/`) and write a standalone script `work/<name>/runs/g1_bare_code.py` to:
1. Check `data/sdo_events/ar12673_201709/` and `data/catalogs/` for eclipse/quality dropouts between 2017-09-04 and 2017-09-10.
2. Calibrate the `2017-09-06T06:00:00Z` AIA 94, 131, 171 Å and HMI B_los FITS files to a co-aligned 512x512 Level 1.5 cube.
3. Run `FullDiskFlareNet` on that cube and compute 2014–2017 test TSS against `PersistenceBaseline` using `data/embeddings/sdo_fulldisk_embeddings_6h_2011_2018.zarr`.
Run the script and report: (a) how many lines of Python you wrote, (b) how many files you had to read, (c) any errors/warnings you hit along the way, and (d) the resulting numbers.
```

### G2 · Run B: MCP Tools + Declarative Validation Spec (20 min)
Now execute the **exact same scientific task** using your installed MCP servers — plus a deliberate test of MCP schema safety:
```text
Now perform the exact same three tasks using ONLY the `sdoflare-data-mcp` and `sdoflare-forecast-mcp` MCP tools (write ZERO custom Python files):
1. Call `check_eclipse_and_gap_flags` and `prepare_l15_cube` for `ar12673_201709` at `2017-09-06T06:00:00Z` (save to `work/<name>/runs/g2_mcp/`).
2. Call `run_fulldisk_forecast` and `execute_validation_suite` using `@specs/operational_24h_m10_validation.json`.
3. DELIBERATE SCHEMA SAFETY TEST: Try calling `train_and_eval_probe` via MCP with two bad arguments: `split_buffer_days=0` (no solar-rotation buffer) and `threshold_source="test_set"`. Show me the exact MCP schema/validation error the server returns!
4. Fill in `work/<name>/notes/bare_vs_mcp_comparison.md` using the comparison table below.
```

#### Worksheet · `work/<name>/notes/bare_vs_mcp_comparison.md`

| Metric / Observation | `G1` · Bare Toolkit Code | `G2` · MCP + Declarative Spec | Why it matters for my science case |
|---|---|---|---|
| Lines of custom glue code written | *e.g., 68 lines* | *0 lines (3 MCP tool calls)* | Less glue code = fewer places to mis-scale units or mis-align timestamps |
| Wall-clock time & retries to first valid run | | | |
| Bugs or omissions in `G1` that `G2` prevented | *e.g., `G1` forgot to filter eclipse flags or picked TSS threshold on test split* | *Blocked by MCP JSON Schema & `operational_24h_m10_validation.json`* | |
| Provenance recorded automatically | *None unless coded by hand* | *`provenance.json` (`sha256`, `aia_degradation_v10`, `carrington_buffer_days: 27.0`)* | Full reproducibility for paper / referee |
| What `G1` CAN do that `G2` CANNOT do | *Define a brand-new physical feature or custom loss in Python* | *Restricted to parameters exposed by the MCP server* | Shows why M4 uses **MCP for plumbing + bare Python for novel science** |

### G3 · Peek under the MCP hood (verify `file:line` citations — 15 min)
MCP tools are **not** magic black boxes — every MCP tool wraps functions in `src/sdoflare/`. Verify how they connect:
```text
Read `@src/sdoflare/mcp/data_server.py`, `@src/sdoflare/mcp/forecast_server.py`, and `@src/sdoflare/prep.py`.
For `prepare_l15_cube` and `execute_validation_suite`:
1. Trace the exact `src/sdoflare/*.py` functions (`file:line`) each MCP tool calls under the hood.
2. Explain how `prepare_l15_cube` guarantees `EXPTIME` is divided once and how `execute_validation_suite` enforces `@specs/operational_24h_m10_validation.json`.
```
→ **Your job:** open **three** of the `file:line` citations in `src/sdoflare/` and mark them ✅/❌.

### G4 · My 3-Layer Architecture Map (15 min)
```text
My science case: <one sentence>.
Using what we learned in G1–G3, write `work/<name>/notes/my_architecture_map.md` with three sections:
1. Steps I will delegate to `sdoflare-data-mcp` and `sdoflare-forecast-mcp` (standard data prep, eclipse filtering, baselines, attribution, validation suite).
2. Rules I will lock into my declarative `work/<name>/validation_spec.json` (split buffer, physical bounds, benchmark events, subgroup checks).
3. The 1–2 genuinely novel scientific functions I MUST write in bare Python (`work/<name>/src/*.py` importing `sdoflare` primitives) because no MCP tool implements my specific hypothesis yet.
```

**Exit ticket (no agent, 2 min):** (1) Give one concrete reason to call an MCP tool instead of having the agent write Python against `src/sdoflare/`, and one case where you *must* write bare Python. (2) What happened when you tried passing `split_buffer_days=0` to the MCP server?

---

## H3 · Module 2 — REPRODUCE & DECLARATIVE BUG HUNT (60 min) · agent = navigator

Goal: see how declarative configuration files (`specs/*.json`) make perturbations reproducible and turn bug-hunting from guesswork into deterministic verification.

### R1 · Notebook vs. Declarative Spec execution (15 min)
```text
1. Execute `@notebooks/03_pretrained_fulldisk_flare_forecast.ipynb` headlessly via `papermill notebooks/03_pretrained_fulldisk_flare_forecast.ipynb work/<name>/runs/03_run.ipynb`.
2. Now run the equivalent declarative validation spec `@specs/operational_24h_m10_validation.json` on `FullDiskFlareNet` using the `execute_validation_suite` MCP tool (output to `work/<name>/runs/r1_spec_cert/`).
3. Compare the notebook printout with `work/<name>/runs/r1_spec_cert/validation_certificate.json` — which checks did the declarative suite run that the notebook didn't print?
```

### R2 · Perturb via a declarative config (20 min)
Instead of editing 8 scattered cells in a `.ipynb` file, create a declarative run configuration and execute it via MCP:

- **Example 1 (Switching benchmark event from Sep 2017 AR 12673 to Oct 2014 AR 12192):**
  ```text
  Use `query_sdo_inventory` (`sdoflare-data-mcp`) to inspect `ar12192_201410`. Then create a declarative config `work/<name>/runs/r2_ar12192_config.json` that runs `prepare_l15_cube`, `run_fulldisk_forecast`, and `attribute_active_regions` on `ar12192_201410` (`2014-10-18` to `2014-10-28`). Execute it via the MCP tools and explain physically why HMI `B_los` attribution vs EUV attribution differs between AR 12192 (confined X-flares) and AR 12673 (eruptive X-flares).
  ```
- **Example 2 (Switching forecast horizon and class threshold to 12h $\ge\text{C5.0}$):**
  ```text
  Copy `@specs/operational_24h_m10_validation.json` to `work/<name>/runs/r2_12h_c50_spec.json`, change `horizon_hours` to `12` and `min_flare_class` to `"C5.0"`, and run `train_and_eval_probe` + `execute_validation_suite` via `sdoflare-forecast-mcp`. Report how the positive base rate, optimal validation threshold, TSS, and BSS changed.
  ```

### R3 · Two-Pass Bug Hunt: Prose Guesswork vs. Declarative MCP Validator (25 min)

`bughunt/bug_A_high_skill.ipynb`, `bug_B_cycle_drift.ipynb`, and `bug_C_quiet_limb.ipynb` all run without throwing any Python exception, but their results are scientifically wrong.

**Pass 1 — Ask the agent to review the code in plain prose (10 min, NO MCP tools allowed):**
```text
Do NOT call any MCP tools yet. Read `@bughunt/bug_A_high_skill.ipynb`, `@bughunt/bug_B_cycle_drift.ipynb`, and `@bughunt/bug_C_quiet_limb.ipynb`. For each notebook, list what you think the bug is based on reading the code alone.
```
Record in `notes/` what the agent spotted and what it missed or excused (e.g., blaming Solar Cycle 24 for `bug_B`'s probability drop).

**Pass 2 — Run the Declarative MCP Validators on the three broken runs (15 min):**
```text
Now call `audit_split_and_leakage` and `execute_validation_suite` (using `@specs/operational_24h_m10_validation.json` and `@specs/preprocessing_contract.json`) on the outputs/configs of `bughunt/bug_A_high_skill.ipynb`, `bug_B_cycle_drift.ipynb`, and `bug_C_quiet_limb.ipynb`.
Produce a table comparing:
1. What your Pass 1 prose review guessed,
2. The exact declarative contract rules (`rule_id`, expected vs actual value) that FAILED in Pass 2, and
3. The physical consequence of each violation.
```

**Exit ticket:** Which bug did Pass 1 (prose review) miss or misdiagnose that Pass 2 (`specs/*.json` via MCP) caught deterministically?

---

## H4 · Module 3 — ARTICULATE & SPEC (60 min) · agent = skeptical collaborator · **no modeling code allowed**

Goal: produce **two** human-approved artifacts before writing any model code:
1. **`work/<name>/PLAN.md`** — your scientific & architectural plan (separating MCP steps from novel `.py` steps), and
2. **`work/<name>/validation_spec.json` (+ `validation_protocol.md`)** — your machine-executable validation contract adapted from `@specs/operational_24h_m10_validation.json`.

### A1 · Scope `PLAN.md` with a skeptic (20 min)

**Concrete example (Marco — Case β, limb foreshortening & off-limb EUV corona):**
```text
I study eruptive active regions near the solar limb. I want to quantify how badly `FullDiskFlareNet`'s 24h >=M1.0 skill degrades when the dominant active region is near the limb (`|lon| > 60 deg`, `mu < 0.5`), and test whether adding a custom off-limb EUV coronal brightness ring feature (`0.95 < R/R_sun < 1.15` in AIA 193 and 211 Å) recovers limb recall.
Step 1: Use `query_sdo_inventory` (`sdoflare-data-mcp`) to check what cubes, embeddings, and AR longitude catalogs exist in `data/` for 2014–2017. Do NOT write any Python code.
Step 2: Act as a skeptical senior collaborator. Ask me up to 3 clarifying questions, then draft `work/marco/PLAN.md` following `@templates/PLAN_TEMPLATE.md`. Clearly separate:
- Which steps use existing `sdoflare` MCP tools (`prepare_l15_cube`, `run_baselines`, `execute_validation_suite`), and
- Which single novel step we will write in bare Python (`work/marco/src/offlimb_ring.py`).
```

### A2 · Author your custom Declarative Validation Spec (`validation_spec.json` — 20 min)

Before writing a line of feature or model code, lock in the machine-readable rules that will judge your experiment tomorrow morning:

```text
Read `@specs/operational_24h_m10_validation.json`, `@specs/validation_spec.schema.json`, and `@work/<name>/PLAN.md`.
Create `work/<name>/validation_spec.json` and a 1-page human companion `work/<name>/validation_protocol.md` tailored to my science case:
1. Keep all mandatory solar-physics invariants (`carrington_split` with `min_buffer_days: 27.0`, `threshold_selection: "validation_split_only"`, `require_aia_degradation_corrected: true`, `forbid_double_exptime: true`, `ci_method: "carrington_block_bootstrap"`).
2. Set my task parameters (`channels`, `horizon_hours`, `min_flare_class`, `baselines`).
3. Add 2 case-specific declarative assertions under `custom_checks` (for example:
   - For Elena: `require_ablation_pair: ["degradation_corrected", "degradation_uncorrected"]` and `ongoing_flare_exclusion_threshold: "C5.0"`.
   - For Marco: `longitude_strata_deg: [[0, 30], [30, 60], [60, 90]]` and `min_limb_recall_gain_over_baseline: 0.05`.
   - For Priya: `task_type: "regression"`, `horizon_hours: 6`, `max_initial_goes_class: "C2.0"`).
4. Validate the file by running `python env/validate_spec.py work/<name>/validation_spec.json`.
```

#### Concrete example · `work/elena/validation_spec.json`
```json
{
  "$schema": "../../specs/validation_spec.schema.json",
  "spec_id": "elena_euv_hot_channel_ablation_24h_m10",
  "description": "24h >=M1.0 EUV 94/131 A channel ablation & AIA degradation sensitivity contract",
  "task": {
    "target_type": "binary_flare_class",
    "min_flare_class": "M1.0",
    "horizon_hours": 24,
    "base_channels": ["hmi_blos", "aia_171"],
    "candidate_channels": ["hmi_blos", "aia_171", "aia_94", "aia_131"]
  },
  "split_contract": {
    "method": "carrington_split",
    "min_carrington_buffer_days": 27.0,
    "forbid_shuffled_split": true,
    "forbid_eclipse_gap_as_negative": true
  },
  "preprocessing_contract": {
    "require_l15_registration": true,
    "require_aia_degradation_corrected": true,
    "forbid_double_exptime": true,
    "aia_ondisk_q99_range_dn_s": [1.0, 50000.0],
    "hmi_max_abs_median_gauss": 5.0
  },
  "evaluation_contract": {
    "primary_metrics": ["TSS", "HSS", "BSS"],
    "forbid_raw_accuracy": true,
    "threshold_selection": "validation_split_only",
    "required_baselines": ["PersistenceBaseline", "ClimatologyBaseline", "FullDiskFlareNet_base"],
    "ci_method": "carrington_block_bootstrap",
    "n_bootstrap": 1000,
    "ci_level": 0.95
  },
  "benchmark_controls": {
    "positive_events": [
      {"event_id": "ar12673_201709", "timestamp_utc": "2017-09-06T06:00:00Z", "min_prob": 0.50},
      {"event_id": "ar12192_201410", "timestamp_utc": "2014-10-24T15:00:00Z", "min_prob": 0.50}
    ],
    "negative_controls": [
      {"event_id": "quiet_201905", "max_prob_95th_pct": 0.05}
    ]
  },
  "custom_checks": {
    "require_degradation_off_ablation": true,
    "ongoing_flare_contamination_filter": {
      "exclude_when_t0_goes_flux_above": "C5.0",
      "max_allowed_tss_drop": 0.12
    }
  }
}
```

### A3 · Pod peer review & Facilitator Plan + Spec Gate (20 min)
Swap `PLAN.md` and `validation_spec.json` with a pod-mate. A facilitator signs off when `python env/validate_spec.py work/<name>/validation_spec.json` prints `VALID` and `PLAN.md` clearly separates MCP steps from your novel `.py` step.

---

## H5 · Module 4 — SOLVE, Sprint 1 (105 min) · Hybrid Mode (MCP + Bare Python)

Goal: use **MCP tools for all standard data staging and baselines (`S1a`)**, write **bare Python in `work/<name>/src/*.py` ONLY for your novel scientific step (`S1b`)**, and reach your **First-Light plot (`S2`)** by minute 75 (16:15).

### S1a · Delegate standard data staging & baselines to MCP (no custom boilerplate)
```text
We are executing the standard data-preparation and baseline steps in `@work/<name>/PLAN.md`:
1. Use `sdoflare-data-mcp` (`query_sdo_inventory`, `check_eclipse_and_gap_flags`, `prepare_l15_cube`) to stage the clean dataset/cubes we need and save the provenance receipt to `work/<name>/runs/prep_provenance.json`.
2. Use `sdoflare-forecast-mcp` (`run_baselines`) to compute and save the `PersistenceBaseline`, `ClimatologyBaseline`, and unmodified `FullDiskFlareNet` metrics on the exact Carrington split defined in `@work/<name>/validation_spec.json`.
Do NOT write custom Python code for these steps — invoke the MCP tools and report the saved artifact paths.
```

### S1b · Write bare Python (`work/<name>/src/*.py`) ONLY for your novel scientific step

**Concrete example (Marco — writing the custom off-limb EUV coronal ring extractor in bare Python):**
```text
Now we implement the novel step in `@work/marco/PLAN.md` that is NOT in the MCP server: extracting azimuthal off-limb EUV coronal brightness features (`0.95 < R/R_sun <= 1.15` in 18 heliographic latitude/East-West sectors from AIA 193 and 211 Å L1.5 maps) to pair with disk embeddings.
Constraints:
- Import `sdoflare.io` and `sdoflare.prep` primitives (see `@src/sdoflare/prep.py`). Do not rewrite FITS loading or L1.5 registration.
- Put the implementation in `work/marco/src/offlimb_ring.py` with docstrings stating units (`DN s^-1 pix^-1`), coordinate frame (`Helioprojective`), and radial bounds (`R/R_sun`).
- Add a unit test in `work/marco/tests/test_offlimb_ring.py` checking that on-disk pixels (`R/R_sun < 0.95`) are zeroed in the ring mask and that AR 12673 on the West limb (`2017-09-09T12:00:00Z`) peaks in the West equatorial sector.
First show me a 6-line plan with `src/sdoflare/` citations and wait for my approval.
```

### S2 · First-Light plot + Provenance check (must be done by 16:15)
```text
Create `work/<name>/figures/first_light.png` illustrating the physical signal of my novel feature/variant on a benchmark timestamp (e.g., AR 12192 or AR 12673), with physical units on all axes, solar limb circle, and UTC timestamp + `prep_provenance.json` hash in the subtitle.
```

### S3 · Physical sanity check against `validation_spec.json`
```text
Run the `preprocessing_contract` and `split_contract` sections of `@work/<name>/validation_spec.json` on our newly generated features/tensors using `sdoflare.eval.run_declarative_validation(..., stages=["preprocessing", "split"])` (or the MCP tool `audit_split_and_leakage`) and verify zero violations before we train or tune the model head.
```

### S4 · Explain back
```text
Explain `work/<name>/src/<module>.py` as if I am defending it at an AGU session tomorrow: (1) physical assumptions, (2) how it interfaces with the MCP-prepared inputs, and (3) one edge case where this calculation could fail silently.
```

### S5 · Stuck protocol (after 3 failed attempts)
```text
Stop changing code. Summarize: (1) what we are trying to accomplish, (2) the 3 attempts and exact errors (distinguishing MCP schema errors from Python tracebacks), and (3) 3 ranked hypotheses with a 1-minute test for each.
```

### S6 · End-of-day handoff (Day 1, 17:15)
```text
Write `work/<name>/HANDOFF.md`: status vs `PLAN.md`, which MCP artifacts are saved in `work/<name>/runs/`, what works in `work/<name>/src/`, what is broken, and the next 3 steps for Day 2 morning.
```

---

## H6 · Module 5 — SOLVE Sprint 2 + DECLARATIVE PROVE (85 min) · agent = declarative auditor & hostile referee

Finish your novel `.py` step in the first 35 minutes, then **stop building and run your declarative validation contract + hostile referee**.

### P1 · Execute your full Declarative Validation Suite via MCP (20 min)
```text
Call `execute_validation_suite` (`sdoflare-forecast-mcp`) with:
- `spec_path="work/<name>/validation_spec.json"`
- `predictions_path="work/<name>/runs/model_predictions.parquet"`
- `output_dir="work/<name>/runs/validation_cert"`
This runs all baselines (`PersistenceBaseline`, `ClimatologyBaseline`, `FullDiskFlareNet_base`), verifies the 27-day Carrington buffer and validation-only threshold selection, checks the positive (`ar12673_201709`, `ar12192_201410`) and quiet-Sun (`quiet_201905`) controls, computes Carrington-block bootstrap 95% CIs, and evaluates our `custom_checks`.
Show me the resulting `validation_report.md` and every rule status (`PASS` / `FAIL`) in `validation_certificate.json`.
```

### P2 · Diagnose and fix any `FAIL` in `validation_certificate.json` (10 min)
If any rule in `validation_certificate.json` returned `FAIL`:
- **Never weaken `validation_spec.json` to make a `FAIL` turn green!**
- Fix the bug in `work/<name>/src/` (or accept a genuine negative scientific finding if your hypothesis simply didn't beat the baseline — a cleanly validated negative result scores full marks!).

### P3 · Spatial attribution & physical plausibility check via MCP (10 min)
```text
Call `attribute_active_regions` (`sdoflare-forecast-mcp`) on `ar12673_201709` (`2017-09-06T06:00:00Z` disk and `2017-09-09T18:00:00Z` West limb) and plot `plot_event_timeline()`. Does our model's attribution align with the flaring active region, or is it latching onto limb artifacts / quiet-Sun network?
```

### P4 · Hostile Referee: find the hole your `validation_spec.json` missed (20 min)
Even a `PASS` certificate only checks the rules you wrote on Day 1. Now ask the agent to attack the spec itself:
```text
You are a hostile Space Weather / ApJ referee. Read `@work/<name>/PLAN.md`, `@work/<name>/validation_spec.json`, `@work/<name>/runs/validation_cert/validation_report.md`, and `@work/<name>/src/`.
Even though `validation_certificate.json` passed our initial rules, give the 4 strongest scientific or statistical objections to our claim.
Specifically identify at least ONE physical confounder or circularity trap that `@work/<name>/validation_spec.json` did NOT check (e.g., flare-in-progress EUV brightening at `t0`, solar-cycle phase imbalance between train and test, multi-AR ambiguity on full-disk images, or threshold sensitivity).
For the top objection, propose a cheap test (< 15 min) AND a new declarative rule we can add to `validation_spec.json`.
```

### P5 · Upgrade `validation_spec.json` and re-issue the certificate (15 min)
```text
Add the hostile referee's top test as a new declarative check in `@work/<name>/validation_spec.json`, implement the check, re-run `execute_validation_suite`, and save the final `validation_certificate.json` and main figure (`work/<name>/figures/main_result.png`) with Carrington-block 95% CIs.
```

---

## H7 · Module 6 — PACKAGE, SKILLIFY & PRESENT (45 min prep + 55 min talks)

Goal: leave behind (1) a clean notebook + `FINDINGS.md` backed by `validation_certificate.json`, and (2) an **MCP-backed Agent Skill (`skills/<skill-name>/`)** bundling `SKILL.md` + `scripts/<skill_cli>.py` + `specs/<skill_validation>.json` so any scientist can run your validated workflow in one prompt.

### X1 · Clean & reproduce (`papermill` + `README.md` — 8 min)
```text
Restructure my notebook into 5 thin sections (Setup & MCP Provenance → Data & Novel Feature → Results vs Baselines → Declarative Validation Certificate & Referee Test → Next Steps), verify `papermill work/<name>/analysis.ipynb work/<name>/runs/final_verified.ipynb` passes cleanly, and write a 3-command `work/<name>/README.md`.
```

### X2 · One-page findings (`FINDINGS.md` — 7 min)
```text
Write `work/<name>/FINDINGS.md` (max 1 page) summarizing the science question, hybrid architecture used (which MCP tools + which custom `.py` module), results table with Carrington-block 95% CIs, `validation_certificate.json` summary, outcome of the Hostile Referee test (`P4`/`P5`), and limitations.
```

### X3 · Distill your workflow into an MCP-backed Agent Skill (20 min)

In an AI-mature toolkit, a reusable Skill does **not** reinvent data loading or evaluation—it instructs the calling agent how to **orchestrate the `sdoflare` MCP tools**, run your **custom CLI script** for the novel scientific step, and enforce your **declarative validation spec (`specs/<skill_validation>.json`)**!

#### Step 1 — Brainstorm & design the MCP-backed Skill (no files written yet)
```text
We are going to package our validated workflow into a reusable Agent Skill in `skills/<skill-name>/`, modeled on `@skills/sdo-ar-event-attribution/SKILL.md` and `@templates/SKILL_TEMPLATE.md`.
Read `@work/<name>/PLAN.md`, `@work/<name>/validation_spec.json`, `@work/<name>/src/`, and `@work/<name>/ERROR_LOG.md`.
First propose the skill design (DO NOT write files yet):
1. `name` (hyphenated, `<= 64` chars) and YAML `description` ("Use when...", "Do not use for...").
2. How the skill orchestrates the 3 layers:
   - Step A (MCP Data Plumbing): which `sdoflare-data-mcp` tools the skill instructs the agent to call,
   - Step B (Novel Science CLI): the `argparse` subcommands in `skills/<skill-name>/scripts/<skill_cli>.py` wrapping `work/<name>/src/`,
   - Step C (Declarative Validation): the parameterized spec `skills/<skill-name>/specs/<skill_validation>.json` passed to `execute_validation_suite` (`sdoflare-forecast-mcp`).
3. Which pitfalls from `@work/<name>/ERROR_LOG.md` go into `## Common Mistakes`.
Wait for my approval.
```

#### Step 2 — Implement, bundle the declarative spec & validate
```text
Approved. Now create:
1. `skills/<skill-name>/SKILL.md` (following `@templates/SKILL_TEMPLATE.md`),
2. `skills/<skill-name>/scripts/<skill_cli>.py` (following `@templates/skill_cli_template.py`, writing outputs to `--output-dir`),
3. `skills/<skill-name>/specs/<skill_validation>.json` (adapted from `@work/<name>/validation_spec.json`),
and run `python env/validate_skill.py skills/<skill-name>` to validate all three components and register the skill symlinks in `.gemini/skills/`, `.claude/skills/`, and `.agents/skills/`.
```

#### Concrete example of an MCP-backed `SKILL.md` (`skills/sdo-euv-channel-ablation/SKILL.md`)
```markdown
---
name: sdo-euv-channel-ablation
description: >-
  Quantifies the impact of adding or removing SDO/AIA EUV channels (94, 131, 171,
  193, 211, 304, 335 A) alongside SDO/HMI B_los on operational solar flare
  prediction skill (TSS, HSS, BSS), audits sensitivity to AIA channel degradation,
  and issues a declarative validation certificate via `sdoflare-forecast-mcp`.
  Use when a scientist asks to compare SDO channel subsets, test whether hot
  flaring channels (94/131 A) improve C/M/X flare forecasts, or check if EUV
  degradation or ongoing flares bias skill. Do not use for single-frame spatial
  attribution alone (use `sdo-ar-event-attribution`).
---

# SDO EUV Channel Ablation & Degradation Sensitivity Skill

## Overview
Orchestrates `sdoflare-data-mcp`, a custom channel-ablation probe CLI (`scripts/euv_ablation_cli.py`), and `sdoflare-forecast-mcp` with a bundled declarative validation specification (`specs/euv_ablation_validation.json`) so any scientist can run a leakage-free, degradation-checked EUV channel ablation on any date range, horizon, and GOES flare class threshold.

## 3-Layer Execution Workflow
When this skill is triggered, execute these three steps in order:

### Step 1 · Inventory & Eclipse Check via `sdoflare-data-mcp`
Call MCP tools `query_sdo_inventory` and `check_eclipse_and_gap_flags` for the user's requested `--start-date`, `--end-date`, and channels (`--base-channels`, `--test-channels`) to verify offline data availability and record eclipse/quality mask counts.

### Step 2 · Run Channel & Degradation Ablation CLI (`scripts/euv_ablation_cli.py`)
```bash
python skills/sdo-euv-channel-ablation/scripts/euv_ablation_cli.py run-ablation \
  --data-path data/embeddings/sdo_fulldisk_embeddings_6h_2011_2018.zarr \
  --base-channels hmi_blos aia_171 \
  --test-channels aia_94 aia_131 \
  --start-date 2014-01-01 --end-date 2017-12-31 \
  --horizon-hours 24 --min-flare-class M1.0 \
  --exclude-ongoing-above C5.0 \
  --output-dir <output-dir>
```
Writes `predictions.parquet`, `ablation_metrics.json`, and `ablation_figure.png` to `<output-dir>`.

### Step 3 · Issue Declarative Validation Certificate via `sdoflare-forecast-mcp`
Call MCP tool `execute_validation_suite` with:
- `spec_path="skills/sdo-euv-channel-ablation/specs/euv_ablation_validation.json"`
- `predictions_path="<output-dir>/predictions.parquet"`
- `output_dir="<output-dir>/validation_cert"`
Never report the ablation result to the user without attaching the `PASS`/`FAIL` summary from `<output-dir>/validation_cert/validation_certificate.json`.

## Domain Guardrails (Enforced by CLI & Bundled Spec)
- **27-Day Carrington Buffer**: Enforced by `carrington_split(buffer_days=27.0)` and checked by `split_contract.min_carrington_buffer_days`.
- **Single `EXPTIME` & Degradation Check**: Verifies `sdoflare_degradation_corrected=True` on all EUV cubes.
- **Ongoing-Flare Contamination Check**: Recomputes skill after excluding timestamps with `GOES_flux(t0) >= C5.0`.

## Common Mistakes (from Workshop Error Logs)
1. **Dividing AIA by `EXPTIME` twice**: `correct_aia_degradation()` already normalizes to `DN s^-1 pix^-1`.
2. **Claiming hot channels (94/131 Å) predict flares when they only detect ongoing flares at `t0`**: Always inspect the `ongoing_flare_contamination_filter` result in `validation_certificate.json`.
```

### X4 · Fresh-session / pod-mate skill test (10 min)
Open a **brand-new agent session** (with MCP enabled) or swap branches with a pod-mate and test the skill with a perturbed natural-language prompt:
```text
Use the `sdo-euv-channel-ablation` skill to evaluate whether adding AIA 94 Å and 131 Å to HMI B_los + AIA 171 Å improves 12-hour >=C5.0 flare forecasting across 2015–2016, including the declarative validation certificate. Save all outputs to `work/elena/runs/skill_verify_12h_c50/`.
```
Verify that the agent calls the MCP inventory tool, runs `euv_ablation_cli.py`, calls `execute_validation_suite` on the bundled spec, and produces a `PASS` `validation_certificate.json`!

### X5 · Toolkit, MCP & Declarative Spec feedback harvest (5 min)
```text
From this session and `@work/<name>/ERROR_LOG.md`, list:
1. Any operation that should be promoted into a new tool or parameter on `sdoflare-data-mcp` or `sdoflare-forecast-mcp`,
2. Any check that should be added to the default `@specs/operational_24h_m10_validation.json`, and
3. Any ambiguous docstring or API gap in `src/sdoflare/`.
Format each as a concise GitHub issue.
```

### Template · 5-slide lightning talk (5 min)
1. **Question & Declarative Spec** — your physical question + the key custom rule you put in `validation_spec.json`.
2. **Hybrid Stack Used** — what you delegated to `sdoflare` MCP tools vs. the novel `.py` module you wrote.
3. **Result + Validation Certificate** — main figure with Carrington-block 95% CIs, baseline comparison, and `validation_certificate.json` badge.
4. **Referee Test & Agent/MCP Story** — the hole the Hostile Referee found in your initial spec, plus your favorite difference between Bare Code (`G1`) and MCP (`G2`).
5. **Reusable Skill Demo + Wishes** — your new skill (`skills/<skill-name>/`), the 1-sentence prompt any scientist can now run, and one wish for `sdoflare`'s MCP servers.

---

## H8 · Organizer templates (`GEMINI.md` — AI-Mature `sdoflare` Edition)

```markdown
# sdoflare — Agent Context, MCP Servers & Declarative Spec Rules

## 3-Layer Toolkit Architecture
1. Layer 1 — Domain MCP Servers (PREFER FOR ALL STANDARD OPERATIONS):
   - `sdoflare-data-mcp`: `query_sdo_inventory`, `inspect_fits_wcs_and_quality`, `prepare_l15_cube`, `check_eclipse_and_gap_flags`
   - `sdoflare-forecast-mcp`: `run_fulldisk_forecast`, `train_and_eval_probe`, `run_baselines`, `attribute_active_regions`, `audit_split_and_leakage`, `execute_validation_suite`
2. Layer 2 — Declarative Validation Specs (`specs/*.json`, `specs/*.md`) & Skills (`skills/`):
   - `specs/operational_24h_m10_validation.json` and `specs/preprocessing_contract.json` define machine-checkable rules.
   - `skills/` contains reusable workflows combining MCP tools, custom `.py` CLIs, and declarative specs.
3. Layer 0 — Bare Python Package (`src/sdoflare/`) & Notebooks (`notebooks/01..05`):
   - Use directly when inspecting equations (`file:line`) or writing genuinely new physical features/models in `work/<name>/src/*.py`.

## Mandatory Workflow Rules
- MCP FIRST FOR PLUMBING & VALIDATION: Never write custom Python scripts to inspect FITS headers, register L1->L1.5 cubes, filter eclipses, run baselines, or run standard validation suites when `sdoflare-data-mcp` or `sdoflare-forecast-mcp` provides a tool for it.
- BARE PYTHON FOR NOVEL SCIENCE: Put new scientific features, masks, or model heads in `work/<name>/src/*.py` importing `sdoflare` primitives, with a unit test in `work/<name>/tests/`. Keep notebooks thin.
- DECLARATIVE SPEC BEFORE MODELING: Every participant project must define `work/<name>/validation_spec.json` (validated via `python env/validate_spec.py`) before training/evaluating models, and must verify results via `execute_validation_suite`. Never weaken `validation_spec.json` to make a failing check pass.
- CITE SOURCES: When reading or extending `src/sdoflare/`, cite `src/sdoflare/<file>.py:<line>` and notebook cells.
- NO DOWNLOADS: `SDOFLARE_OFFLINE=1` is enforced. Use only pre-staged data in `data/` (query via `query_sdo_inventory` or `@data/README.md`).

## SDO & Solar Flare Domain Conventions (Enforced by MCP & `specs/*.json`)
1. Time: UTC observation times (`T_OBS`/`DATE-OBS`) only; never naive datetimes or file mtimes.
2. AIA (`sdoflare.prep`): `aia_l1_to_l15` registers pointing/roll/plate-scale only; `correct_aia_degradation(..., normalize_exptime=True)` corrects sensitivity degradation AND divides by `EXPTIME` once (`DN s^-1 pix^-1`).
3. HMI (`sdoflare.prep`): `B_los` in Gauss (`G`); `correct_hmi_projection()` divides by `mu = cos(theta)` down to `mu_min=0.2` and masks `R/R_sun > 0.95` to `NaN`.
4. Splits & Evaluation (`sdoflare.labels`, `sdoflare.eval`): `carrington_split(buffer_days=27.0)` mandatory; select probability thresholds on validation split only (`find_optimal_tss_threshold`); report TSS, HSS, and BSS against `PersistenceBaseline` and `ClimatologyBaseline` with Carrington-rotation block-bootstrap 95% CIs.
```

