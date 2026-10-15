# AGENTS.md — ML Factory Engineering Rules

## Purpose

Build a **practical, self-contained, fork-per-use-case Databricks MLOps template** for a small ML team. `TP_intervention` is the first production reference implementation. Avoid a heavyweight central platform.

Normal lifecycle:

```text
fork mlops_template → explore in notebooks → promote stable feature code
→ materialise UC Feature Store features → configure a training candidate
→ resolve Training Dataset Registry ID → train/evaluate/register
→ score/monitor through Databricks Jobs deployed with DAB + Azure DevOps
```

This file defines durable engineering rules. The architecture lives in `docs/architecture/ML_FACTORY_CONTEXT.md`; repository-specific tasks live in implementation plans. **Never assume that an architecture requirement is already implemented.**

## 1. Repository boundaries

The multi-root VS Code workspace contains three separate Git repositories:

| Repository | Role | Allowed during initial planning |
|---|---|---|
| `TP_intervention` | Real use case, first implementation and end-to-end validation | Read code; write planning Markdown only |
| `mlops_template` | Self-contained, forkable target template | Read code; write **provisional** planning Markdown only |
| `DAB_Orchestrator` | Existing orchestration/deployment reference | **Read only, always** |

Repository folders might have different capitalisation. Resolve actual paths rather than guessing.

### Absolute prohibition: DAB_Orchestrator

**Do not edit `DAB_Orchestrator` during planning, implementation, validation, or template upstreaming.** Do not add/delete/refactor files, change jobs/schedules, change pipelines, deploy from this repo, or generate tasks requiring its modification. Do not list its files among files to change. Inspect it solely to understand the current TP scheduled execution (including `src/third_party_intervention_ml.py` where present), DAB resources, and Azure DevOps conventions.

Implement any replacement DAB jobs **inside `TP_intervention`**, and later generalise the pattern into `mlops_template`. Production cutover/removal of old orchestrator jobs is a **separately authorised operational activity**, not an automatic implementation step. A plan may document the needed coordination, but may not instruct an agent to edit the reference repo.

## 2. Planning and execution boundaries

**Initial planning stage:** inspect all three repositories, but create only:

- detailed TP `MASTER_PLAN.md` and executable TP workstream plans;
- **provisional** template `MASTER_PLAN.md`, describing target reusable changes and dependencies on TP validation.

No production code changes in either repository during planning. Do not produce detailed implementation tasks in the template for capabilities not yet validated in TP.

**TP implementation stage:** implement and validate tasks **only in `TP_intervention`**, one eligible task per coding-agent run. Do not begin template implementation.

**Template upstream stage:** after TP end-to-end validation and an explicit review of findings, update the provisional template plan, derive implementation-ready template workstreams, then implement **only in `mlops_template`**. Do not blindly merge or copy TP's entire repository.

If a task requires changing a settled architecture decision, crossing a repository boundary, or enlarging scope substantially, stop and report the discrepancy. Do not improvise a new design.

## 3. Self-contained fork ownership

`mlops_template` is **copied/forked and customised** for every new ML use case. The fork owns its embedded utilities and can run without referring back to the template repository.

- Embed `ml_utils` as an importable package within each fork. Its exact placement should be decided from existing packaging conventions. The intention is to eliminate a **separately maintained nested `libs/ml_utils` project**, not eliminate the `ml_utils` import or prohibit a standard Python `src/` layout.
- Notebook ergonomics should remain as simple as `from ml_utils import ...` (or a deliberately selected, documented equivalent).
- Keep common utilities in the template and use-case-specific feature code in the fork. Do not copy TP business rules into generic template helpers.
- Existing forks **do not automatically receive updates**. Upstream validated reusable improvements to `mlops_template`, record template releases/changelog, and selectively port fixes to existing forks **only after review and testing**.
- Do not introduce internal package publishing, private feed dependencies, submodules, automated sync bots, or automatic template updates in this phase.

## 4. Data scientist workflow

Notebooks are first-class tools for EDA, joins, feature experiments, Optuna, model comparisons, and analysis. They are not technical debt by default.

When useful features are accepted, **promote the actual feature calculation** from exploratory notebooks into reusable PySpark/SQL/Python code and import the **same implementation** from notebooks and feature-refresh jobs. Do not require an MLE to translate those calculations into a second set of custom sklearn transformers.

Configs describe **decisions** (dataset ID, model, final parameters, preprocessing, evaluation, registration), not how to program joins or write feature formulas. Use a single repeatable training implementation; a thin Databricks notebook may call it.

## 5. Feature Store and preprocessing

Feature computation code performs joins, aggregations, and deterministic business feature calculations. Databricks Feature Engineering / Unity Catalog Feature Tables store the **computed feature values** for training and scoring.

Keep fitted, model-specific transforms (e.g., imputation/encoding/scaling learned from training data) in the preprocessing stage. Existing registered preprocessing artifacts may need migration because they currently include joins and feature logic. Do not remove fitted preprocessing where needed, or inadvertently change output schemas.

**Initial Feature Store scope:** use ordinary entity-key lookup tables. Time-series feature tables, point-in-time/as-of lookup implementation, and their supporting workstreams are **deferred**. Note any known limitation without implementing historical lookup systems.

Validate runtime/API compatibility for Feature Engineering model logging and batch scoring before relying on exact Databricks API calls. Ensure the model's inference packaging correctly applies required fitted preprocessing; don't assume `score_batch` automatically loads an independently registered preprocessor.

## 6. Required Training Dataset Registry

A Training Dataset Registry **must** be included in the initial Factory. Prefer a small governed Unity Catalog Delta table unless an adequate solution already exists.

Assign stable dataset IDs (for example `TD001`) and record the essential reproducible definition: use case, label source/definition, feature tables and selected columns, lookup key, applicable filters/time window, and relevant code/config or table version references.

- **Training Dataset Registry:** what training dataset definition was used?
- **MLflow:** how did this training run execute and evaluate?
- **UC model/artifact registry:** what fitted model/preprocessing artifacts were registered?

Keep references consistent; don't build another enterprise metadata platform.

## 7. DAB, CI/CD and legacy cleanup

The eventual use-case fork should own its DAB deployment definitions for applicable feature refresh, training, scoring, and monitoring jobs. Use existing `DAB_Orchestrator` conventions as a reference **without modifying it**. Azure DevOps owns CI/CD; Databricks Jobs own runtime orchestration.

The previous template contains AzureML-era material. Identify obsolete dependencies, scripts, pipelines, and docs; **remove only after confirming no active callers**. Do not retain unused AzureML structure just for historical consistency.

Treat `TP_intervention/pipelines/` as potentially unused; inspect actual callers and scheduled jobs before calling it active.

## 8. Tests and validation

Testing is **risk-based, not coverage-driven**. Prefer small local tests when they validate meaningful feature logic, config, registry interfaces, preprocessing or metadata without starting Databricks or MLflow services. Tests are optional for low-risk wiring/docs. Use focused Databricks integration/smoke validation when actual Feature Store, UC Registry, or DAB behaviour must be verified.

Never mark a task complete if a required acceptance check was not done. Explicitly report environments or validations that are unavailable.

## 9. Coding-agent task discipline

For a selected workstream:

1. Read the repo's `AGENTS.md`, its `MASTER_PLAN.md`, and the current workstream file.
2. Choose one eligible `[ ]` task; resume `[~]` first. Check dependencies.
3. Inspect existing code and tests; reuse appropriate existing components.
4. Set task `[~]`, implement only that task, and do the required validation.
5. Set task `[x]` only when acceptance criteria are met; use `[!]` for genuine blockers.
6. Update workstream status, relevant docs/changelog, and a suggested focused commit message.
7. **Stop**. Do not start the next task or automatically commit/merge without explicit authorisation.

Task/workstream statuses: `[ ] Not Started`, `[~] In Progress`, `[x] Complete`, `[!] Blocked`.

## 10. Scope guardrails

Prefer small modules and Databricks-native facilities. Avoid custom feature DSLs, plugin/dependency-injection frameworks, new workflow engines, unnecessary service layers, excessive YAML, large test harnesses, and speculative abstractions. GenAI/LLM Factory work is out of scope. The template remains independently usable after a fork.
