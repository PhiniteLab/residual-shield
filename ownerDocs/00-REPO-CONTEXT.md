# Repository context

**One page that orients an agent before it reads anything else in this repository.**
Read after `README.md` — the workspace area map — before touching code or manuscripts.

> Owner-owned and repository-specific. The template ships this file EMPTY; the scaffold
> creates it once and never overwrites it, and `--sync` never touches it. Fill it in your
> own words — the headings below are a prompt, not a schema.

---

## What this repository is

A simulation-only, companion-code artifact for arXiv:2606.01397 (`PAPER.yml`: `role:
companion-code`, `paper_of_record: false`). It ships a 12-state rigid-body fixed-wing UAV
simulator, a fixed/gain-scheduled autopilot, a bounded tabular Q-learning residual
supervisor, an HJB-inspired residual supervisor, a React/Three.js live visualization
frontend, and the benchmark/reproducibility outputs behind the paper's claims (`README.md`).
It is explicitly **not** the manuscript home — the arXiv LaTeX source lives in the companion
repository `PhiniteLab/sharq-twin` (`PAPER.yml`, `README.md`) — and it is **not** a
real-flight, HIL, or certified flight-safety system (`README.md` "Scope boundary").

## Scientific claim under development

The design claim stated by the code, not a single numeric hypothesis: bounded, HJB-inspired
residual command supervision (small residuals on commanded airspeed/altitude/heading) can
augment a fixed/gain-scheduled autopilot's disturbance-rejection tracking under
wind/gust/turbulence while never replacing the actuator-facing autopilot and staying
inspectable/bounded (`README.md`, "Method overview" and "Benchmark outputs"). **No numeric
success criterion is stated in this repository** — `reproducibility/VALIDATION_2026-05-19.md`
records pass/fail engineering checks (tests, lint, type, manifest hash), not a claim-level
numeric threshold, and no other file states one.

## Where the truth lives

- code that runs the experiments -> `backend/src/uavsim/` (simulator, autopilot, guidance,
  residual supervisors, WebSocket server, benchmark CLI) (`README.md`)
- the current method specification -> Not yet defined as a numbered spec: `ownerDocs/myMethods/`
  contains only the scaffold `README.md` and an empty `evidence/` directory — no method file has
  been registered there. The working method description lives in `README.md` ("Method
  overview") and in `PAPER.yml`.
- results an outside reader can trust -> `experiments/` (`experiments/claims/`,
  `experiments/results/{raw,data,tables,figures}`), cross-checked against
  `reproducibility/MANIFEST.sha256` and `reproducibility/VALIDATION_2026-05-19.md` (`README.md`
  "Canonical experiment paths")
- the corpus -> `ownerDocs/myLiterature/MANIFEST.yaml`

## Toolchain and how to run it

Backend checks (`README.md` "Quick start: backend checks"):

```bash
python -m venv .venv
. .venv/bin/activate
python -m pip install -e './backend[dev]'
cd backend
python -m py_compile $(find src tests -name '*.py' -print)
ruff check .
pyright src tests
python -m pytest -q
```

Expected backend test suite for this release: **41 passed** (`README.md`). Live demo needs a
second shell: `python -m uavsim.server` (backend) plus `npm install && npm run dev` (frontend),
then `http://localhost:3000`. **How long the smoke command takes is not stated in this
repository.**

## Constraints and non-goals

- License is Apache-2.0 and must not be replaced with another license used elsewhere in the
  collection (`PAPER.yml` notes).
- The former GitHub slug `PhiniteLab/pythalab-sharq-hjb-uav-command-supervision` is cited
  verbatim inside the published PDF of arXiv:2606.01397; nobody may ever register a new
  repository at that slug again (`PAPER.yml` notes, `README.md`).
- Non-goals, stated in `README.md` "Scope boundary": no real-flight validation, no
  hardware-in-the-loop validation, no certified flight safety, no real SAAB Gripen
  aerodynamics, no continuous high-dimensional HJB PDE solution, and no formal CBF/HJ safety
  certificate. The 3D aircraft mesh is a visual-only asset and does not affect backend physics
  or benchmark results.
- This repository is declared the **authoritative** copy of the `backend/src/uavsim/`
  package; the companion repository `sharq-twin` carries a working copy that has already
  drifted from it for `server.py` and `experiment_runner.py` (`sharq-twin`'s `PAPER.yml`
  notes) — that drift is `sharq-twin`'s open issue to resolve by syncing from here, not a
  constraint on this repository's own code.

## Current blockers

None recorded. `tasks/todo.md` is empty, and no open blocker is stated in `README.md`,
`CHANGELOG.md`, or `PAPER.yml`.
