# Heliophysics Scientific Computing Workshop (`helio-workshop`)

This repository contains the design and facilitation guide for our hands-on scientific computing workshop, structured around a **"Missing Manual" exercise** across six progressive sessions (`M0` to `M5`).

Rather than following a passive, pre-scripted tutorial, participants (domain scientists) are asked to explore and understand the workshop repository, notebooks, datasets, and pretrained models/artifacts, attempt to formulate and implement their own science questions of choice, and combine their own independent thinking/coding with AI coding agents (such as **Claude Code** or **Antigravity**). Throughout the workshop, participants formulate a critical judgment of their experience so that the toolkit, its documentation, its agent-readiness, and its operational maturity can be improved.

---

## Repository Structure

```text
helio-workshop/
├── README.md   # Overview of the workshop structure and Missing Manual philosophy
└── guide.md    # Complete M0–M5 session guide & facilitator challenges
```

---

## Workshop Sessions (`M0` to `M5`)

See [`guide.md`](./guide.md) for the full session descriptions, facilitator framing, and **6 suggested challenges per session** (split evenly between **3 challenges for participants to think/implement by themselves** and **3 challenges for interacting with a coding agent**):

| Session | Name | Core Focus |
|---|---|---|
| **M0** | **Setup** | Bring up the environment, verify datasets and model artifacts, connect the coding agent, and audit the "Day-Zero" onboarding experience. |
| **M1** | **Discovery** | Explore the repository architecture, example notebooks, and pretrained artifacts; uncover hidden physical and operational assumptions. |
| **M2** | **Reproducibility** | Re-run reference notebooks from a clean state, perturb parameters or target events, and identify brittle assumptions or silent failure modes. |
| **M3** | **Scope** | Formulate a science question of choice, assess data/compute feasibility, and design an implementation strategy comparing human and agent plans. |
| **M4** | **Solve** | Implement the workflow across notebooks and code, ingest data, run models/artifacts, and produce diagnostic and result figures. |
| **M5** | **Prove** | Validate results against domain baselines, audit for methodological/leakage pitfalls, and synthesize the **Missing Manual** critical report. |
