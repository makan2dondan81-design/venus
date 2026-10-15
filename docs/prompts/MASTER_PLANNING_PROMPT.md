# ML Factory — Three-Repository Master Planning Prompt

You are the Lead ML Platform / MLOps Engineer designing a **practical, forkable Azure Databricks ML project pattern** for a small team.

**THIS RUN IS PLANNING ONLY. DO NOT IMPLEMENT CODE.**

## 1. Context and repository roles

Before planning, read from the `TP_intervention` repository:

1. `AGENTS.md`
2. `docs/architecture/ML_FACTORY_CONTEXT.md`
3. This `docs/prompts/MASTER_PLANNING_PROMPT.md`

The VS Code multi-root workspace contains three independent Git repositories (actual folder names/capitalisation may differ):

```text
workspace/
├── TP_intervention/     # production reference implementation
├── mlops_template/      # reusable, self-contained forkable template
└── DAB_Orchestrator/    # STRICTLY READ-ONLY orchestration reference
```

Their roles:

- **TP_intervention:** current real use case, evidence of how the template is consumed, first implementation and end-to-end validation target.
- **mlops_template:** existing generic source template. Inspect during initial planning; create a provisional Master Plan for reusable changes, not a parallel independent implementation programme.
- **DAB_Orchestrator:** reference for current TP Databricks execution, scheduling, DAB and Azure DevOps conventions. Inspect only. **Never change this repository or propose implementation tasks that modify it.**

All plans must follow the architecture in `ML_FACTORY_CONTEXT.md`. Repository code is the source of truth for what exists today. Do not confuse target capabilities with implemented ones.

## 2. Non-negotiable operating model

The goal is a **self-contained, copy/fork-and-customise template**, not a centrally distributed framework.

```text
mlops_template → fork for a new use case
        ↓
Data scientists explore in notebooks using embedded ml_utils
        ↓
Stable feature computation promoted to reusable project Python/SQL
        ↓
Databricks Feature Store / UC feature tables
        ↓
Training Dataset Registry (mandatory)
        ↓
Small config captures final candidate choices
        ↓
Single repeatable training implementation → MLflow + UC artifacts
        ↓
DAB / Azure DevOps-deployed scoring and monitoring
```

Preserve simple imports such as `from ml_utils import ...`. The intended change is removing a separately maintained/nested `libs/ml_utils` project, **not eliminating the embedded importable package**. Inspect actual package layout before deciding the final path.

Once forked, each use case owns its copy of the utilities. Template fixes/releases are adopted by existing forks **selectively**, not automatically. Defer shared package publishing, Azure Artifacts distribution, submodules, auto-sync bots and automatic upgrades.

**Initial Feature Store implementation uses ordinary entity-key lookups.** Time-series Feature Tables and historical point-in-time/as-of lookups are deferred. Do not create initial workstreams for them. Note modelling limitations without implementing the deferred architecture.

**DAB_Orchestrator is immutable for this programme.** Plan all new jobs in `TP_intervention` first, and later appropriate generic DAB resources in `mlops_template`. A production scheduling cutover is a separate operational activity requiring explicit authorisation.

## 3. Planning method: discover before designing

Inspect the repositories strategically to limit token use. Start with directory structure and critical entrypoints; expand only relevant areas. Avoid dumping or reading every file when the relevant behaviour can be determined from focused inspection.

### 3.1 TP_intervention

Inspect, where present:

- project layout, `README`, `pyproject.toml`, dependencies, Python import/package layout;
- `notebooks/` (EDA, joins, feature engineering, preprocessing creation, model training, Optuna, registration);
- `ml_utils` and `libs/ml_utils` or any duplicated utilities;
- data sources, current training dataset creation and custom transformers;
- current registered preprocessing artifact: source reads, joins, feature calculation, imputation/encoding/scaling;
- MLflow tracking, registration and model-preprocessor references;
- training and retraining entrypoints;
- scheduled scoring and monitoring outputs, version metadata, business contracts;
- tests, documentation, configuration, `pipelines/`, DAB resources and Azure DevOps files;
- unused AzureML-era files/dependencies/scripts/pipelines.

**Important:** The `pipelines/` folder is not presently the authoritative scheduled TP production execution mechanism. Verify usage rather than assuming it is live.

### 3.2 mlops_template

Inspect the **actual existing template**, including corresponding package utilities, notebooks, starter configuration, dependency handling, AzureML remnants, CI/CD, DAB files, docs, and how new forks are intended to use it.

For each capability relevant to the target, record:

```text
EXISTS AND REUSABLE
EXISTS BUT NEEDS ADAPTATION
MISSING
OBSOLETE
TP-SPECIFIC / NOT SUITABLE FOR TEMPLATE
UNRESOLVED
```

Do not assume the current template and TP fork still have identical files or history.

### 3.3 DAB_Orchestrator (read only)

Inspect existing TP orchestration, including `src/third_party_intervention_ml.py` where available, Databricks bundle resources, variables, job/task dependencies, schedules, run paths, environment conventions, and relevant Azure DevOps YAML.

Document evidence-based conventions. **Do not edit it, do not create implementation tasks against it, do not plan deleting or migrating files inside it, and do not deploy from it.**

If the reference cannot be inspected, record missing evidence instead of inventing details.

## 4. Shared design questions to settle once

Use one coherent architecture for both planned outcomes. Make decisions only after inspecting existing code. Resolve or document blockers around:

1. **Notebook-to-production boundary:** which DS work remains experimental, which stable joins/features become callable project code, and how notebooks reuse that same code.
2. **Feature Store:** minimal meaningful UC feature tables, entity keys, feature computation ownership, materialisation/refresh strategy, lookups for training and scoring. Do not invent a feature DSL.
3. **Preprocessing:** migration of joins/business features out of the current registered preprocessing artifact while retaining fitted transforms and existing scoring contracts.
4. **Training Dataset Registry (required):** dataset ID convention (e.g. `TD001`), minimal governed schema, label source, feature lookup definition, filtering/period, practical reproducibility references and lookup API.
5. **Config-driven training:** minimal config representing final DS decisions, a single training implementation, optional thin Databricks notebook wrapper, Optuna/retraining reuse.
6. **Artifact relationships:** how dataset ID, feature tables, run, evaluation, fitted preprocessing and registered model versions stay linked. Validate Feature Engineering logging and scoring integration rather than assuming an independently registered preprocessor is applied by `score_batch`.
7. **Scoring and monitoring:** compatible migration for existing TP production outputs, version columns and meaningful monitoring.
8. **DAB / Azure DevOps:** use-case-owned resources patterned on `DAB_Orchestrator`, but only new/changed files in the target repository.
9. **Template embedding:** simple `ml_utils` import packaging, generic scaffolding and config defaults with TP-specific paths/features excluded.
10. **Safe cleanup and minimal tests:** unused AzureML removal after reference checks; local tests only where valuable, focused Databricks smoke tests where necessary.

Prefer existing functionality and Databricks-native capabilities. Reject unnecessary abstractions or duplicate implementations.

## 5. Deliverable A — active TP Master Plan

Create under **TP_intervention only**:

```text
TP_intervention/docs/implementation/MASTER_PLAN.md
TP_intervention/docs/implementation/01_<workstream>.md
TP_intervention/docs/implementation/02_<workstream>.md
...
```

`MASTER_PLAN.md` must be **active / implementation-ready** and include:

1. **Executive summary and current maturity**, grounded in the repo.
2. **Current TP lifecycle and production run diagram**, referencing actual notebooks, custom transformers, artifact handling, scoring, and read-only DAB reference paths.
3. **Gap analysis** distinguishing used, unused, missing and duplicated capabilities.
4. **Shared architecture decisions** and their rationale/tradeoffs, aligned with `ML_FACTORY_CONTEXT.md`.
5. **Target TP lifecycle**: notebooks → feature code → Feature Store → mandatory Training Dataset Registry → config → training/evaluation/registration → scoring/monitoring.
6. **Feature Store design:** table/key boundary, refresh, training and scoring, with ordinary entity-key lookups only in this phase.
7. **Training Registry definition:** minimum workable schema, ID resolution, dataset references and model/run lineage.
8. **Notebook-to-config training design:** repeatable entrypoint, fitted preprocessing boundary, MLflow/UC artifacts and model inference packaging.
9. **Target repository structure and existing components to reuse**, marked `[EXISTING]`, `[MODIFY]`, `[NEW]`, `[REMOVE]` only where justified.
10. **DAB and deployment direction:** proposed resources **inside TP_intervention**, sequence of feature refresh/scoring, CI/CD convention and operational cutover prerequisites. No changes inside DAB reference.
11. **Incremental migration and rollback considerations**, including existing scoring outputs and consumers.
12. **Workstreams, dependency graph, recommended order and parallelisable tasks** appropriate for a small team.
13. **Pragmatic testing/validation**, including what local tests can verify without Databricks and minimum E2E checks actually needed.
14. **AzureML/unused code cleanup** with verified impact.
15. **Factory upstream candidates** linking planned TP capabilities to the template and explaining what must remain TP-specific.
16. **Risks, unresolved assumptions, deferred scope, and TP Definition of Done**.

Only create actual TP workstreams justified by the repository. Avoid one workstream per architectural heading; combine tightly coupled changes and split unreviewably large ones.

### TP workstream file structure

Each `docs/implementation/NN_<name>.md` must contain:

```text
# Workstream Name
Status: [ ] Not Started

## Objective
## Requirements / Architecture References
## Current State and Code to Reuse
## Target Behaviour and Contracts
## Dependencies
## Files Expected to Change (TP_intervention ONLY)
## Implementation Tasks
## Required Validation / Useful Tests
## Migration / Compatibility
## Factory Upstream Candidate Notes
## Definition of Done
## Risks / Blockers
```

Make tasks independently executable for a lower-cost coding agent, preferably one coherent commit each. The task should contain:

```text
Task ID
Status: [ ] Not Started
Priority
Objective / Why
Dependencies
Existing Code to Reuse
Files Likely Affected
Implementation Instructions
Expected Behaviour / Interfaces
Validation / Tests (optional unless valuable or required)
Acceptance Criteria
Documentation / Changelog Impact
Suggested Commit Message
```

Use statuses `[ ] Not Started`, `[~] In Progress`, `[x] Complete`, `[!] Blocked`. The plans should explicitly distinguish *required* validations from merely desirable tests. Never mark a blocked decision as already decided.

## 6. Deliverable B — provisional Template Master Plan

Create **only this provisional planning file** inside `mlops_template` during the initial planning pass:

```text
mlops_template/docs/implementation/MASTER_PLAN.md
```

At the top include:

```text
Status: PROVISIONAL — Awaiting TP Intervention end-to-end validation
Implementation: NOT AUTHORISED YET
```

The template Master Plan must include:

1. **Present template architecture and inventory** with real source paths.
2. **Shared target architecture**, not a separate redesign, and cross-references to the TP plan and architecture context.
3. **Reusable capability mapping**: TP existing/planned implementation → corresponding existing template code → planned reuse/adaptation/generalisation; exclude TP-specific features, identifiers, business rules and tables.
4. **Required template cleanup** (AzureML legacy, nested `libs/ml_utils` project if applicable, unused templates/pipelines), after verifying references.
5. **Target fork experience:** notebooks, embedded importable `ml_utils`, simple configs, dataset registry helpers and Feature Store utilities, standard training and DAB starter definitions.
6. **Fork ownership/update policy:** self-contained copies, template version tags/changelog and selective adoption by existing forks; no central package or auto-sync infrastructure.
7. **Provisional workstream roadmap and dependencies** identified by capability, without detailed atomic implementation tasks for functionality pending TP proof.
8. **Validation gates and findings to revisit** after TP E2E. Include which design questions might change based on integration results.
9. **Fresh-fork smoke test plan**: imports, example feature creation, training config, registry integration, artifact logging and bundle validation on a safe representative dataset/environment.
10. **Deferred scope, assumptions and later Definition of Done**.

**Do not** generate template implementation workstream files or modify template production code at this stage. A later dedicated planning pass will convert the provisional roadmap into executable workstreams.

## 7. Coordination and traceability

The two plans must agree on:

- terminology, workflow stages and responsibility boundaries;
- Training Dataset Registry contract and ID semantics;
- feature computation vs fitted preprocessing;
- expected feature lookup and scoring contracts;
- `ml_utils` packaging philosophy;
- what “completed in TP” means before upstreaming;
- the selected fork policy (embedded code, selective updates).

Include a lightweight cross-repository mapping (for example a table in both plans) for the most important reusable capabilities. Reference paths, not duplicated long explanations.

Do not use the existence of the provisional Template Master Plan as permission to implement code in `mlops_template` before TP validation.

## 8. Gate before Template planning becomes implementation-ready

After all required TP workstreams and end-to-end validation have finished, a **separate agent session** will:

1. review completed TP code and validation evidence;
2. revisit any architecture assumptions invalidated during TP work;
3. compare the proven implementation with the current template;
4. update the template Master Plan **in place**, rather than generating an unrelated new architecture;
5. create detailed template workstreams/atomic tasks based on validated contracts;
6. keep all TP business-specific logic out of the template;
7. require a clean fresh-fork smoke test before declaring the reusable template complete.

Do **not** perform this later gate during the initial planning run. Do not mark the provisional template plan final.

## 9. Planning permissions and prohibitions

You MAY:

- read the three repositories and available Git history;
- inspect tests, notebooks, scripts, configs, job definitions and CI/CD;
- write only the planning Markdown files required above;
- update an existing planning file **only if** it is part of these deliverables, preserving useful prior decisions where present.

You MUST NOT:

- implement or refactor production code in TP or template;
- change tests/config/job definitions/pipelines as a side effect;
- create, register or deploy Databricks assets;
- migrate datasets or alter production scoring;
- modify **anything** inside `DAB_Orchestrator`;
- create tasks requiring modifications to `DAB_Orchestrator`;
- create implementation-ready template workstreams before TP E2E validation;
- create an auto-update/package-publishing/sync system;
- create time-series/PIT feature lookup workstreams;
- introduce GenAI/LLM scope;
- commit, merge or execute deployment/cutover actions.

When lacking evidence, record an assumption and a specific validation step. Do not invent file paths, API compatibility, keys or runtime behaviour.

## 10. Self-review and final response

Before finishing, check:

- precisely **two** Master Plans exist in the intended repositories;
- TP workstreams are implementation-ready, while the template plan is clearly provisional;
- neither plan contradicts `ML_FACTORY_CONTEXT.md`;
- current feature engineering duplication and preprocessing migration are addressed;
- the mandatory Training Dataset Registry is included;
- ordinary Feature Store lookup is the current scope, time-aware lookup deferred;
- no tasks/files affect `DAB_Orchestrator`;
- the fork remains independent, with embedded `ml_utils` and manual selective updates;
- TP-specific features don't leak into template design;
- proposed cleanup, tests, migration and dependencies are proportionate;
- all created files are planning Markdown only.

Return a short summary of:

- findings from each repository;
- shared architecture decisions;
- TP plan and workstreams created, with recommended first workstream;
- template provisional roadmap and major dependency on TP validation;
- risks / required validation;
- exact planning files created or updated.

**Stop. Do not begin implementation.**
