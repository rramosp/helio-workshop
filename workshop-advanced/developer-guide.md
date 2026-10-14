# Heliophysics Toolkit Workshop — Toolkit Developer Guide (AI-Mature Toolkit Edition)

> [!IMPORTANT]
> **About the worked examples (`sdoflare`):** Throughout this Developer Guide (as in the companion **Facilitator Guide** and **Participant Handouts**), all concrete code snippets, MCP server implementations, declarative JSON validation schemas, docstrings, file trees, and tests use an **imaginary Python package called `sdoflare`** — a hypothetical, AI-mature toolkit for preprocessing full-disk **SDO/AIA** (EUV) and **SDO/HMI** (magnetogram) images and running **operational solar flare prediction models** evaluated against the **GOES XRS** flare catalog.
>
> Use `sdoflare` as a concrete reference implementation: replace its module names, MCP tool names, physical units, coordinate frames, and declarative validation rules with those of your actual heliophysics toolkit.

---

## 1. Purpose of this guide

In this workshop, your toolkit is delivered not just as a bare Python package and notebooks, but as a **three-layer AI-mature stack**:
1. **Layer 0 — Bare Python Package & Notebooks (`src/sdoflare/`, `notebooks/`)**: Inspectable, unit-tested physical routines and extensible primitives for novel science.
2. **Layer 1 — Domain MCP Servers (`src/sdoflare/mcp/`)**: Schema-enforced, provenance-logging Model Context Protocol servers (`sdoflare-data-mcp`, `sdoflare-forecast-mcp`) that participants install into their coding agents in **Module 0** and compare head-to-head against bare code in **Module 1**.
3. **Layer 2 — Declarative Validation Specs (`specs/*.json`, `specs/*.md`) & Agent Skills (`skills/`)**: Machine-checkable contracts defining split buffers, physical bounds, baselines, and benchmark controls that participants customize in **Module 3**, execute in **Module 5** to earn a `validation_certificate.json`, and bundle into reusable **Agent Skills** in **Module 6**.

This guide specifies the **mandatory (`MUST`)** and **recommended (`SHOULD`)** engineering requirements a toolkit developer must follow so every module in the Facilitator Guide and Participant Handouts works out of the box.

### Traceability: how workshop activities depend on repo deliverables

| Workshop module & prompt cards | What the participant / agent does | What the developer **MUST** provide in the repo |
|---|---|---|
| **M0 · Setup & MCP Install** (`M0.1`–`M0.4`) | Runs `check_setup.py`, observes agent without MCP, installs `sdoflare` MCP servers, inspects tool schemas | `env/check_setup.py`, `env/install_mcp.py`, stdio MCP servers (`sdoflare.mcp.data_server`, `sdoflare.mcp.forecast_server`), `GEMINI.md` (+ `AGENTS.md`/`CLAUDE.md` symlinks) |
| **M1 · GROUND & COMPARE** (`G1`–`G4`) | Runs A/B lab: Bare Code (`G1`) vs MCP + Spec (`G2`), tests MCP schema rejection, verifies `file:line` citations (`G3`), builds 3-layer map (`G4`) | Pydantic/JSON-Schema constraints on MCP tools (e.g., rejecting `split_buffer_days < 13.5` and `threshold_source="test_set"`), provenance receipts (`provenance.json`), structured docstrings in `src/sdoflare/` |
| **M2 · REPRODUCE & AUDIT** (`R1`–`R3`) | Runs notebooks via `papermill` + specs via MCP; perturbs JSON config; runs 2-pass Bug Hunt (prose review vs declarative MCP validator) | `papermill`-ready notebooks; `specs/operational_24h_m10_validation.json` & `specs/preprocessing_contract.json`; 3 broken runs in `bughunt/` whose saved artifacts deterministically trip the declarative validator |
| **M3 · ARTICULATE & SPEC** (`A1`–`A3`) | Uses `query_sdo_inventory` MCP tool; writes `PLAN.md` + custom `work/<name>/validation_spec.json` before modeling | `specs/validation_spec.schema.json`, `env/validate_spec.py`, 3-tier offline `data/` + `data/README.md`, 3 `backup_cases/` (each with `PLAN.md` + `validation_spec.json`) |
| **M4 · SOLVE (Hybrid)** (`S1`–`S6`) | Uses MCP tools for data staging/baselines (`S1a`) and bare Python in `work/<name>/src/*.py` for novel science (`S1b`); First-Light plot (`S2`) | Clean Python primitives in `src/sdoflare/` that accept MCP-staged cubes/embeddings; loud runtime guardrails (Rule 2.2) |
| **M5 · DECLARATIVE PROVE** (`P1`–`P5`) | Runs `execute_validation_suite` via MCP to get `validation_certificate.json`; runs Hostile Referee (`P4`) and adds a new check to `validation_spec.json` (`P5`) | `execute_validation_suite` MCP tool & `sdoflare.eval.run_declarative_validation()` supporting extensible `custom_checks` |
| **M6 · PACKAGE & SKILLIFY** (`X1`–`X5`) | Distills workflow into `skills/<name>/` (`SKILL.md` + `scripts/` + `specs/`) (`X3`) and verifies in a fresh MCP-enabled session (`X4`) | Reference skill `skills/sdo-ar-event-attribution/` (with bundled spec); `templates/SKILL_TEMPLATE.md`, `templates/skill_cli_template.py`, and `env/validate_skill.py` |

---

## 2. R1 · Mandatory repository layout

```text
sdoflare-workshop/
├── GEMINI.md                        # Primary agent context & 3-layer rules (<= 150 lines)
├── AGENTS.md -> GEMINI.md           # Symlink for Codex / Cursor / Antigravity
├── CLAUDE.md -> GEMINI.md           # Symlink for Claude Code
├── README.md                        # Quickstart, schedule, and architecture overview
├── pyproject.toml                   # Editable install (`pip install -e .`) + pinned dependencies
├── env/
│   ├── environment.yml              # Conda/mamba lockfile (or uv.lock / requirements.txt)
│   ├── check_setup.py               # Pre-flight verification (`--with-mcp` flag; §10)
│   ├── install_mcp.py               # Registers MCP servers in .gemini/, .mcp.json, .agents/ (§4)
│   ├── validate_spec.py             # Validates declarative `validation_spec.json` files (§5)
│   └── validate_skill.py            # Validates `skills/<name>/` (SKILL.md + CLI + spec) & registers symlinks (§9)
├── src/
│   └── sdoflare/
│       ├── __init__.py
│       ├── io.py                    # Data & catalog loaders (enforces SDOFLARE_OFFLINE=1)
│       ├── prep.py                  # L1->L1.5 registration, AIA degradation, HMI mu-projection, co-alignment
│       ├── labels.py                # Operational flare window labeling & `carrington_split`
│       ├── models.py                # `FullDiskFlareNet`, `LinearProbeForecaster`, `PersistenceBaseline`, `ClimatologyBaseline`
│       ├── eval.py                  # `compute_tss_hss`, `block_bootstrap_metric`, `run_declarative_validation`
│       └── mcp/                     # Layer 1 · Domain MCP Servers (§4)
│           ├── __init__.py
│           ├── data_server.py       # `sdoflare-data-mcp` (inventory, FITS/WCS/quality, L1.5 cube prep)
│           └── forecast_server.py   # `sdoflare-forecast-mcp` (forecast, probe, baselines, attribution, validation suite)
├── specs/                           # Layer 2 · Declarative Validation Specifications (§5)
│   ├── README.md                    # How declarative validation specs work
│   ├── validation_spec.schema.json  # JSON Schema enforced by `env/validate_spec.py` and MCP
│   ├── operational_24h_m10_validation.json
│   ├── validation_protocol.md       # Human-readable companion to `operational_24h_m10_validation.json`
│   └── preprocessing_contract.json  # L1.5, EXPTIME, degradation, mu-mask, and co-alignment invariants
├── skills/                          # Layer 2 · Reusable Agent Skills Catalog (§9)
│   ├── README.md
│   └── sdo-ar-event-attribution/    # Gold-standard reference skill (MCP + CLI + Spec)
│       ├── SKILL.md
│       ├── scripts/
│       │   └── ar_attribution_cli.py
│       └── specs/
│           └── ar_attribution_validation.json
├── .gemini/
│   ├── settings.json                # Populated by `env/install_mcp.py` during Module 0
│   ├── commands/                    # Optional prompt cards as slash commands
│   └── skills/                      # Symlinks to `../../skills/*`
├── .claude/
│   └── skills/                      # Symlinks to `../../skills/*`
├── .agents/
│   └── skills/                      # Symlinks to `../../skills/*`
├── notebooks/                       # Layer 0 · Parameterized tutorial notebooks (`01`..`05` + `README.md`)
├── docs/
│   └── CHEATSHEET.md                # Public Python API + MCP tool quick reference
├── data/                            # 3-tier offline dataset (`catalogs/`, `embeddings/`, `cubes_512/`, `sdo_events/`)
├── bughunt/                         # 3 broken notebooks + pre-saved run artifacts for `R3` Pass 1 & Pass 2
├── backup_cases/                    # 3 fallback cases, each with `PLAN.md` + `validation_spec.json`
├── prompts/                         # Standalone prompt files (`M0`, `G1`–`G4`, `R1`–`R3`, `A1`–`A2`, `S1`–`S6`, `P1`–`P5`, `X1`–`X5`)
├── templates/                       # `PLAN_TEMPLATE.md`, `ERROR_LOG.md`, `HANDOFF.md`, `SKILL_TEMPLATE.md`, `skill_cli_template.py`
├── tests/                           # Fast unit + MCP + spec tests (`pytest -q tests/` < 30s)
└── work/                            # Participant workspaces (`work/<name>/`)
```

---

## 3. R2 · Layer 0 standards: Bare Python API, docstrings & runtime guardrails

Even with MCP servers available, scientists drop down to `src/sdoflare/` in **Module 1 (`G3`)** to verify physical equations and in **Module 4 (`S1b`)** to write their novel scientific feature or model head.

### Rule 2.1 — Structured domain docstrings with explicit MCP cross-references
Every public function in `src/sdoflare/` **MUST** state:
1. What it does and what it **does not** do (e.g., `aia_l1_to_l15()` registers pointing/roll/scale but does **not** correct degradation).
2. Physical units, coordinate frames, and tensor shapes.
3. Metadata keys read and written (`sdoflare_degradation_corrected`, `sdoflare_exptime_normalized`).
4. Which MCP tool wraps it for standard pipelines (e.g., *"Wrapped by MCP tool `prepare_l15_cube` in `sdoflare-data-mcp`"*).

### Rule 2.2 — Loud runtime guardrails in bare Python
Your bare Python functions **MUST** fail fast with descriptive `ValueError`s when called improperly in `G1` or `S1b`:
- **Double `EXPTIME` or double degradation**: check and set `meta["sdoflare_exptime_normalized"] = True` and `meta["sdoflare_degradation_corrected"] = True`.
- **Naive or non-UTC datetimes**: reject `dt.tzinfo is None`.
- **Unbuffered splits**: `carrington_split(index, buffer_days=27.0)` **MUST** raise `ValueError` if `buffer_days < 13.5` unless `allow_leakage_for_testing=True`.
- **Test-set threshold tuning**: `compute_tss_hss(y_true, y_prob, *, threshold: float)` **MUST** require an explicit `threshold: float` computed via `find_optimal_tss_threshold(y_val, y_prob_val)`.

---

## 4. R3 · Layer 1 standards: Building the Domain MCP Servers (`src/sdoflare/mcp/`)

### Rule 3.1 — Architectural division between MCP and Bare Python
Design your MCP servers around this boundary so **Module 1 (`G1` vs `G2`)** and **Module 4 (`S1a` vs `S1b`)** teach the right lesson:
- **Expose as MCP Tools**: Standard, repeatable operations where parameter schemas and provenance matter — inventory discovery, header/quality/eclipse inspection, standard L1.5 cube calibration, baseline model runs, active-region attribution, leakage audits, and declarative validation suite execution.
- **Keep in Bare Python (`src/sdoflare/` primitives)**: Low-level array/map transformations that scientists compose in `work/<name>/src/*.py` when inventing a custom feature, custom coronal mask, or custom model head.

### Rule 3.2 — Schema-enforced domain contracts & provenance receipts
Every MCP tool **MUST**:
1. Use strict `pydantic.Field` / JSON Schema constraints so invalid domain parameters (such as `split_buffer_days=0` or `threshold_source="test_set"` in `G2`'s deliberate schema-safety test) are **rejected before execution** with an educational error message.
2. Write large outputs (cubes, predictions, plots) to files in `output_dir` and return a compact JSON dictionary containing a **cryptographic provenance receipt (`provenance_path`, `sha256`, `domain_guarantees_verified`)** so the agent's context window stays clean.

#### Reference implementation (`src/sdoflare/mcp/forecast_server.py`)
```python
"""sdoflare-forecast-mcp: Forecasting, Attribution & Declarative Validation MCP Server."""

from datetime import datetime, timezone
import hashlib
import json
from pathlib import Path
from typing import Literal
from mcp.server.fastmcp import FastMCP
from pydantic import BaseModel, Field

from sdoflare import eval as sdo_eval
from sdoflare import labels as sdo_labels
from sdoflare import models as sdo_models

mcp = FastMCP("sdoflare-forecast-mcp")


class ProbeEvalRequest(BaseModel):
  embeddings_path: str = Field(
      description="Path to precomputed Zarr embeddings or prepared cube in data/."
  )
  channels: list[
      Literal[
          "hmi_blos",
          "aia_94",
          "aia_131",
          "aia_171",
          "aia_193",
          "aia_211",
          "aia_304",
          "aia_335",
      ]
  ]
  start_date: str = Field(description="ISO UTC start date (YYYY-MM-DD).")
  end_date: str = Field(description="ISO UTC end date (YYYY-MM-DD).")
  horizon_hours: int = Field(ge=1, le=72, default=24)
  min_flare_class: Literal["C1.0", "C5.0", "M1.0", "M5.0", "X1.0"] = "M1.0"
  split_buffer_days: float = Field(
      ge=13.5,
      default=27.0,
      description=(
          "Mandatory Carrington rotation buffer (days) between train/val/test "
          "splits. Must be >= 13.5 days (half solar rotation) to prevent the "
          "same active region from appearing in both splits."
      ),
  )
  threshold_source: Literal["validation_split_only"] = Field(
      default="validation_split_only",
      description=(
          "TSS decision threshold MUST be selected on the validation split "
          "only; test_set optimization is forbidden."
      ),
  )
  output_dir: str = Field(description="Directory where predictions and provenance are written.")


@mcp.tool()
def train_and_eval_probe(req: ProbeEvalRequest) -> dict:
  """Trains a LinearProbeForecaster on SDO embeddings/cubes with strict Carrington buffering."""
  out_dir = Path(req.output_dir)
  out_dir.mkdir(parents=True, exist_ok=True)

  # Calls Layer 0 (`src/sdoflare/labels.py`, `models.py`, `eval.py`) under the hood
  result = sdo_models.run_probe_experiment(
      embeddings_path=req.embeddings_path,
      channels=req.channels,
      start_date=req.start_date,
      end_date=req.end_date,
      horizon_hours=req.horizon_hours,
      min_flare_class=req.min_flare_class,
      split_buffer_days=req.split_buffer_days,
      output_dir=out_dir,
  )

  provenance = {
      "tool": "sdoflare-forecast-mcp.train_and_eval_probe",
      "timestamp_utc": datetime.now(timezone.utc).isoformat(),
      "request": req.model_dump(),
      "domain_guarantees_verified": {
          "carrington_buffer_days": req.split_buffer_days,
          "threshold_selected_on": "validation_split_only",
          "eclipse_gaps_excluded": True,
      },
      "predictions_sha256": hashlib.sha256(
          (out_dir / "predictions.parquet").read_bytes()
      ).hexdigest(),
  }
  prov_path = out_dir / "provenance.json"
  prov_path.write_text(json.dumps(provenance, indent=2) + "\n")

  return {
      "status": "OK",
      "metrics_summary": result["metrics_summary"],
      "predictions_path": str(out_dir / "predictions.parquet"),
      "provenance_path": str(prov_path),
      "underlying_code_citations": [
          "src/sdoflare/labels.py:carrington_split",
          "src/sdoflare/models.py:LinearProbeForecaster",
          "src/sdoflare/eval.py:find_optimal_tss_threshold",
      ],
  }


@mcp.tool()
def execute_validation_suite(
    spec_path: str,
    predictions_path: str,
    output_dir: str,
) -> dict:
  """Executes a declarative validation specification (specs/*.json) and emits a certificate."""
  return sdo_eval.run_declarative_validation(
      spec_path=Path(spec_path),
      predictions_path=Path(predictions_path),
      output_dir=Path(output_dir),
  )


if __name__ == "__main__":
  mcp.run(transport="stdio")
```

### Rule 3.3 — Ship the multi-agent MCP installer (`env/install_mcp.py`)
In **Module 0 (`M0.3`)**, participants register `sdoflare-data-mcp` and `sdoflare-forecast-mcp`. Ship `env/install_mcp.py` so registration works across Gemini CLI (`.gemini/settings.json`), Claude Code (`.mcp.json`), and Codex/Antigravity (`.agents/mcp.json`) and verifies the stdio handshake:

```python
#!/usr/bin/env python3
"""Registers sdoflare MCP servers in local agent configs and verifies stdio startup."""

import argparse
import json
from pathlib import Path
import sys

ROOT = Path(__file__).resolve().parent.parent

MCP_CONFIG = {
    "mcpServers": {
        "sdoflare-data-mcp": {
            "command": sys.executable,
            "args": [
                "-m",
                "sdoflare.mcp.data_server",
                "--data-dir",
                str(ROOT / "data"),
                "--offline",
            ],
            "env": {"SDOFLARE_OFFLINE": "1"},
        },
        "sdoflare-forecast-mcp": {
            "command": sys.executable,
            "args": [
                "-m",
                "sdoflare.mcp.forecast_server",
                "--data-dir",
                str(ROOT / "data"),
                "--specs-dir",
                str(ROOT / "specs"),
                "--offline",
            ],
            "env": {"SDOFLARE_OFFLINE": "1"},
        },
    }
}

TARGET_CONFIG_FILES = [
    ROOT / ".gemini" / "settings.json",
    ROOT / ".mcp.json",
    ROOT / ".agents" / "mcp.json",
]


def install_configs() -> None:
  for cfg_path in TARGET_CONFIG_FILES:
    cfg_path.parent.mkdir(parents=True, exist_ok=True)
    existing = {}
    if cfg_path.exists():
      existing = json.loads(cfg_path.read_text())
    existing.setdefault("mcpServers", {}).update(MCP_CONFIG["mcpServers"])
    cfg_path.write_text(json.dumps(existing, indent=2) + "\n")
    print(f"[OK] Registered sdoflare MCP servers in {cfg_path.relative_to(ROOT)}")


def main() -> int:
  parser = argparse.ArgumentParser()
  parser.add_argument("--verify", action="store_true", help="Import & smoke-test MCP tools.")
  args = parser.parse_args()
  install_configs()
  if args.verify:
    from sdoflare.mcp import data_server, forecast_server  # pylint: disable=import-outside-toplevel
    assert data_server.mcp.name == "sdoflare-data-mcp"
    assert forecast_server.mcp.name == "sdoflare-forecast-mcp"
    print("[OK] Verified sdoflare-data-mcp and sdoflare-forecast-mcp tool registries")
  return 0


if __name__ == "__main__":
  sys.exit(main())
```

---

## 5. R4 · Layer 2 standards: Declarative Validation Specifications (`specs/*.json`, `specs/*.md`, `env/validate_spec.py`)

Declarative validation specs are what allow participants to:
1. Catch all three planted bugs in **Module 2 (`R3` Pass 2)** deterministically,
2. Lock in their scientific acceptance criteria at the **Module 3 Plan & Spec Gate (`A2`–`A3`)** before writing model code, and
3. Generate a tamper-evident `validation_certificate.json` in **Module 5 (`P1`–`P5`)** and **Module 6 (`X3`–`X4`)**.

### Rule 4.1 — Ship `specs/validation_spec.schema.json` and `env/validate_spec.py`
The developer **MUST** provide a JSON Schema (`specs/validation_spec.schema.json`) and a CLI validator (`env/validate_spec.py`) that rejects any participant `validation_spec.json` that attempts to weaken core domain invariants:
- `split_contract.min_carrington_buffer_days` **MUST** be $\ge 13.5$ (default $27.0$),
- `split_contract.forbid_shuffled_split` **MUST** be `true`,
- `preprocessing_contract.require_aia_degradation_corrected` **MUST** be `true`,
- `preprocessing_contract.forbid_double_exptime` **MUST** be `true`,
- `evaluation_contract.threshold_selection` **MUST** be `"validation_split_only"`,
- `evaluation_contract.required_baselines` **MUST** include `"PersistenceBaseline"` and `"ClimatologyBaseline"`,
- `evaluation_contract.ci_method` **MUST** be `"carrington_block_bootstrap"`.

#### Reference implementation (`env/validate_spec.py`)
```python
#!/usr/bin/env python3
"""Validates a declarative validation_spec.json against domain invariants."""

import json
from pathlib import Path
import sys


def fail(msg: str) -> None:
  print(f"[SPEC INVALID] {msg}", file=sys.stderr)
  sys.exit(1)


def validate_spec(path: Path) -> None:
  if not path.exists():
    fail(f"File not found: {path}")
  spec = json.loads(path.read_text())

  for section in [
      "spec_id",
      "task",
      "split_contract",
      "preprocessing_contract",
      "evaluation_contract",
      "benchmark_controls",
  ]:
    if section not in spec:
      fail(f"Missing required top-level section: '{section}'")

  sc = spec["split_contract"]
  if sc.get("min_carrington_buffer_days", 0) < 13.5:
    fail("split_contract.min_carrington_buffer_days must be >= 13.5 days")
  if sc.get("forbid_shuffled_split") is not True:
    fail("split_contract.forbid_shuffled_split must be true")

  pc = spec["preprocessing_contract"]
  if pc.get("require_aia_degradation_corrected") is not True:
    fail("preprocessing_contract.require_aia_degradation_corrected must be true")
  if pc.get("forbid_double_exptime") is not True:
    fail("preprocessing_contract.forbid_double_exptime must be true")

  ec = spec["evaluation_contract"]
  if ec.get("threshold_selection") != "validation_split_only":
    fail("evaluation_contract.threshold_selection must be 'validation_split_only'")
  baselines = set(ec.get("required_baselines", []))
  if not {"PersistenceBaseline", "ClimatologyBaseline"}.issubset(baselines):
    fail("evaluation_contract.required_baselines must include PersistenceBaseline and ClimatologyBaseline")
  if ec.get("ci_method") != "carrington_block_bootstrap":
    fail("evaluation_contract.ci_method must be 'carrington_block_bootstrap'")

  bc = spec["benchmark_controls"]
  if not bc.get("positive_events") or not bc.get("negative_controls"):
    fail("benchmark_controls must specify at least 1 positive_event and 1 negative_control")

  print(f"[OK] VALID SPEC: {path} (spec_id='{spec['spec_id']}')")


if __name__ == "__main__":
  if len(sys.argv) != 2:
    fail("Usage: python env/validate_spec.py <path/to/validation_spec.json>")
  validate_spec(Path(sys.argv[1]))
```

### Rule 4.2 — What `sdoflare.eval.run_declarative_validation()` MUST output
When called directly or via the `execute_validation_suite` MCP tool, `run_declarative_validation(spec_path, predictions_path, output_dir)` **MUST** write two files into `output_dir`:
1. **`validation_certificate.json`**:
   ```json
   {
     "spec_id": "elena_euv_hot_channel_ablation_24h_m10",
     "overall_status": "PASS",
     "timestamp_utc": "2026-10-14T10:12:00Z",
     "predictions_sha256": "8f3c...",
     "checks": [
       {"rule_id": "split_contract.min_carrington_buffer_days", "expected": ">= 27.0", "actual": 27.0, "status": "PASS"},
       {"rule_id": "preprocessing_contract.forbid_double_exptime", "expected": true, "actual": true, "status": "PASS"},
       {"rule_id": "evaluation_contract.threshold_selection", "expected": "validation_split_only", "actual": "validation_split_only", "status": "PASS"},
       {"rule_id": "benchmark_controls.positive_events.ar12673_201709", "expected": "prob >= 0.50", "actual": 0.84, "status": "PASS"},
       {"rule_id": "benchmark_controls.negative_controls.quiet_201905", "expected": "p95 <= 0.05", "actual": 0.018, "status": "PASS"},
       {"rule_id": "custom_checks.ongoing_flare_contamination_filter", "expected": "tss_drop <= 0.12", "actual": 0.07, "status": "PASS"}
     ],
     "metrics_with_95ci": {
       "CandidateModel": {"TSS": [0.68, 0.62, 0.74], "HSS": [0.51, 0.45, 0.57], "BSS": [0.34, 0.28, 0.40]},
       "PersistenceBaseline": {"TSS": [0.44, 0.38, 0.50], "HSS": [0.39, 0.33, 0.45], "BSS": [0.12, 0.06, 0.18]},
       "ClimatologyBaseline": {"TSS": [0.00, 0.00, 0.00], "HSS": [0.00, 0.00, 0.00], "BSS": [0.00, 0.00, 0.00]}
     }
   }
   ```
2. **`validation_report.md`**: A concise markdown table rendering the checks and the `metrics_with_95ci` comparison table ready to paste into `FINDINGS.md`.

---

## 6. R5 · Tutorial notebooks (`notebooks/`) & 3-tier offline data (`data/`)

1. **`notebooks/README.md` Lookup Table**: Maps each science question to its tutorial notebook (`01`–`05`), the bare `sdoflare` functions it demonstrates, **and the equivalent MCP tool + declarative spec**.
2. **Headless `papermill` Execution**: Every tutorial notebook has a `"parameters"` cell and executes in $< 60\,\text{s}$ offline.
3. **Three-Tier Offline Data (`data/`)**:
   - **Tier 1 (`data/embeddings/`, `data/catalogs/`, $< 250\,\text{MB}$)**: Precomputed `FullDiskFlareNet` 6-hour embeddings + GOES/AR catalogs + eclipse bitmasks (runs in $< 5\,\text{s}$ on CPU).
   - **Tier 2 (`data/cubes_512/`, $1\text{–}4\,\text{GB}$)**: Co-aligned $512\times 512$ L1.5 AIA+HMI cubes (`2014–2017`).
   - **Tier 3 (`data/sdo_events/`, $< 2\,\text{GB}$)**: Raw L1 & registered L1.5 FITS slices for `ar12192_201410`, `ar12673_201709`, and `quiet_201905`.

---

## 7. R6 · Engineering the 3 Bug-Hunt notebooks (`bughunt/`) for the Two-Pass Audit (`R3`)

In **Module 2 (`R3`)**, participants first inspect `bughunt/bug_A_high_skill.ipynb`, `bug_B_cycle_drift.ipynb`, and `bug_C_quiet_limb.ipynb` using plain prose prompts (Pass 1) and then run `execute_validation_suite` via MCP on the saved run artifacts (Pass 2).

Therefore, for each notebook in `bughunt/`, the developer **MUST** commit:
1. The pre-executed `.ipynb` file (running cleanly in $< 30\,\text{s}$ with zero Python exceptions), **and**
2. Its saved output directory `bughunt/runs/bug_{A,B,C}/` containing `predictions.parquet` and `run_metadata.json` so that calling `execute_validation_suite(spec_path="specs/operational_24h_m10_validation.json", predictions_path="bughunt/runs/bug_A/predictions.parquet", ...)` immediately flags the exact violated declarative rules!

---

## 8. R7 · MCP-backed Skill scaffolding & validator (`skills/`, `templates/`, `env/validate_skill.py`)

In **Module 6 (`X3`–`X4`)**, participants package their validated workflow into `skills/<skill-name>/` containing:
- `SKILL.md` (orchestrating Step 1: `sdoflare-data-mcp` inventory/prep $\to$ Step 2: `scripts/<skill_cli>.py` for the novel science step $\to$ Step 3: `sdoflare-forecast-mcp` `execute_validation_suite`),
- `scripts/<skill_cli>.py` (parameterized `argparse` CLI writing to `--output-dir`), and
- `specs/<skill_validation>.json` (the declarative validation spec for that skill).

### Updated `env/validate_skill.py` (validates `SKILL.md` + CLI script + bundled `specs/*.json`)
```python
#!/usr/bin/env python3
"""Validates an MCP-backed skill folder in skills/<name> and registers symlinks."""

import os
from pathlib import Path
import re
import subprocess
import sys

ROOT = Path(__file__).resolve().parent.parent
REQUIRED_SECTIONS = [
    "## Overview",
    "## 3-Layer Execution Workflow",
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
    fail("Frontmatter 'description:' must include 'Use when...' trigger guidance")

  for sec in REQUIRED_SECTIONS:
    if sec not in text:
      fail(f"SKILL.md is missing required section: '{sec}'")

  if "execute_validation_suite" not in text:
    fail("SKILL.md must invoke 'execute_validation_suite' to verify outputs against its declarative spec")

  scripts_dir = skill_dir / "scripts"
  scripts = list(scripts_dir.glob("*.py")) if scripts_dir.exists() else []
  if not scripts:
    fail(f"No CLI script found in {scripts_dir}/")

  for script in scripts:
    src = script.read_text()
    if re.search(r"work/[a-zA-Z0-9_-]+/", src):
      fail(f"{script.name} contains a hardcoded 'work/<user>/' path; use --output-dir")
    res = subprocess.run(
        [sys.executable, str(script), "--help"],
        capture_output=True,
        text=True,
        check=False,
    )
    if res.returncode != 0 or ("--output-dir" not in res.stdout and "--output" not in res.stdout):
      fail(f"{script.name} --help must succeed and expose --output-dir")

  specs_dir = skill_dir / "specs"
  spec_files = list(specs_dir.glob("*.json")) if specs_dir.exists() else []
  if not spec_files:
    fail(f"Missing bundled declarative validation spec in {specs_dir}/*.json")
  for sp in spec_files:
    res_sp = subprocess.run(
        [sys.executable, str(ROOT / "env" / "validate_spec.py"), str(sp)],
        capture_output=True,
        text=True,
        check=False,
    )
    if res_sp.returncode != 0:
      fail(f"Bundled spec {sp.name} failed validation:\n{res_sp.stderr}")

  for agent_dir in AGENT_SKILL_DIRS:
    agent_dir.mkdir(parents=True, exist_ok=True)
    link = agent_dir / skill_name
    if link.is_symlink() or link.exists():
      link.unlink()
    link.symlink_to(os.path.relpath(skill_dir, agent_dir))

  print(f"[OK] Validated MCP-backed skill '{skill_name}' (CLI + declarative spec)")
  print(f"[OK] Registered symlinks in .gemini/skills/, .claude/skills/, and .agents/skills/")


if __name__ == "__main__":
  if len(sys.argv) != 2:
    fail("Usage: python env/validate_skill.py skills/<skill-name>")
  validate_and_register(Path(sys.argv[1]))
```

---

## 9. R8 · Pre-workshop qualification protocol ("Definition of Done")

A workshop repository is **not ready** until the developer passes all three qualification gates at least **5 days before the workshop**:

### Gate 1 — Automated Repo, MCP, Spec, Skill & Notebook Verification
```bash
export SDOFLARE_OFFLINE=1
python env/install_mcp.py --verify
python env/check_setup.py --with-mcp
python env/validate_spec.py specs/operational_24h_m10_validation.json
python env/validate_skill.py skills/sdo-ar-event-attribution
pytest -q tests/
for nb in notebooks/*.ipynb bughunt/*.ipynb; do
  papermill "$nb" "/tmp/$(basename "$nb")"
done
```

### Gate 2 — The 10-Prompt Agent + MCP Smoke Test (score $\ge 9/10$ required)
Run the 10 canonical smoke-test prompts in Facilitator Guide §7.1 in a fresh agent session with `sdoflare-data-mcp` and `sdoflare-forecast-mcp` enabled. Verify that:
- The agent uses MCP tools for inventory, L1.5 prep, baselines, and validation suites rather than writing boilerplate Python,
- The MCP server rejects `split_buffer_days=2` via schema enforcement,
- `execute_validation_suite` deterministically flags the planted violations in `bughunt/bug_A`, `bug_B`, and `bug_C`.

### Gate 3 — The 90-Minute Facilitator Dry Run (M0 through M6)
Have a facilitator run through **M0 → M6** using **Backup Case α**:
1. Install the MCP servers (`M0.3`),
2. Run the Bare Code (`G1`) vs MCP (`G2`) comparison lab and confirm `G2` is at least $5\times$ faster and blocks bad schema parameters,
3. Customize `validation_spec.json` (`A2`), run the Hybrid Solve loop (`S1a` MCP + `S1b` `.py`), generate `validation_certificate.json` (`P1`), and package + test the resulting MCP-backed skill (`X3`–`X4`).

