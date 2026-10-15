# ML Factory — Targeted Architecture Amendment (Defer Feature Store)

## Objective

The initial ML Factory Master Plans and implementation workstreams have already been generated.

**Do NOT regenerate the plans from scratch.**

I am explicitly changing one architectural decision:

**Databricks Feature Store / Feature Engineering integration is deferred until after the core MLOps capabilities have been implemented and validated in both `TP_intervention` and `mlops_template`.**

The reason is to avoid making the core MLOps implementation dependent on investigating and refactoring the existing data scientists' feature engineering workflows.

Your task is to perform a **minimal, surgical update** to the existing Markdown documentation and implementation plans.

This is a documentation/planning amendment only.

## 1. Efficiency and token constraints

- Do not reanalyse the entire repository.
- Do not regenerate existing Master Plans or workstreams.
- Do not inspect notebooks, Python modules, or production code unless absolutely necessary to resolve an affected planning dependency.
- Use targeted searches such as `rg` to locate Feature Store references in the existing Markdown files.
- Read only the relevant sections and nearby dependencies.
- Preserve existing structure, task IDs, completed work, workstream numbering and unaffected content.
- Make the smallest coherent edits required.
- Avoid creating new documentation files unless absolutely necessary.
- Do not implement code or run Databricks workflows.

## 2. Updated Phase 1 architecture

The initial ML Factory implementation must support:

1. Self-contained embedded `ml_utils`.
2. Training Dataset Registry independent of Feature Store.
3. Config-driven, repeatable training.
4. MLflow experiment tracking and metadata.
5. Model and preprocessing artifact registration in Unity Catalog.
6. Model evaluation gates.
7. Model promotion, version management and rollback.
8. Automated batch scoring using existing data and preprocessing workflows.
9. Production monitoring, data quality checks and feature drift monitoring.
10. Databricks Asset Bundles and Azure DevOps CI/CD.
11. Appropriate lightweight testing and validation.

**Preserve the existing TP feature engineering and preprocessing implementations wherever practical.**

Do not require migration of joins, aggregations or feature calculations into Feature Store before implementing the above capabilities.

Ensure training and scoring continue to use compatible feature calculations and fitted preprocessing.

## 3. Training Dataset Registry adjustment

The Training Dataset Registry remains mandatory in Phase 1.

Remove its dependency on Feature Store tables, FeatureLookup objects and Feature Engineering APIs.

The registry should support existing Unity Catalog Delta training datasets or other verified existing training data sources.

Preserve the ability to record:

- Dataset ID, e.g. TD001
- Use case and training data source
- Target/label definition
- Relevant filters and population definition
- Source/table versions or reproducible snapshots where available
- Relevant code/config references
- MLflow run and registered model/preprocessor version relationships

Do not claim full reproducibility where the necessary source snapshots or versions are unavailable.

Design the contract so Feature Store references can be supported later without redesigning the entire registry.

## 4. Deferred Phase 2

Move the following out of initial implementation requirements:

- Databricks Feature Store / UC Feature Tables
- Feature materialisation and refresh jobs
- FeatureLookup-based training
- Feature Store-backed batch scoring
- Migration of existing business feature calculations into reusable feature-building modules solely for Feature Store adoption
- Point-in-time and time-series feature lookups

Document these as a future workstream requiring separate architectural review and collaboration with data scientists.

Feature Store must not appear as a dependency or acceptance criterion for any Phase 1 task or Definition of Done.

Preserve existing feature drift monitoring using available training/scoring feature datasets.

## 5. Files to amend

Apply the new decision consistently to the existing files where relevant:

**Architecture and agent guidance:**
- `TP_intervention/AGENTS.md`
- `TP_intervention/docs/architecture/ML_FACTORY_CONTEXT.md`
- `TP_intervention/docs/prompts/MASTER_PLANNING_PROMPT.md`

**Existing plans:**
- `TP_intervention/docs/implementation/MASTER_PLAN.md`
- Only affected `TP_intervention/docs/implementation/NN_*.md` workstream files
- `mlops_template/docs/implementation/MASTER_PLAN.md`

Resolve actual folder names and paths before editing. Do not create duplicate files.

Update any other existing planning Markdown only if it directly contradicts the revised architecture.

The template Master Plan must remain **PROVISIONAL**, with implementation dependent on successful validation in TP and a subsequent review.

## 6. Workstream amendments

For each affected workstream:

1. Remove Feature Store as an implementation prerequisite.
2. Replace Feature Store-dependent tasks with existing-dataset alternatives where needed.
3. Remove dedicated Phase 1 Feature Store implementation tasks, retaining their scope in the deferred roadmap.
4. Update dependencies, diagrams, acceptance criteria and Definitions of Done.
5. Preserve unaffected tasks, existing task IDs and task statuses.
6. Keep workstreams implementation-ready and logically sequenced.
7. Do not introduce unnecessary new abstractions or workstreams.

Do not renumber existing workstreams or rewrite entire documents.

If a task cannot be amended safely without evidence from existing code, flag the specific uncertainty rather than inventing a solution.

## 7. Repository boundaries

- `TP_intervention` remains the first implementation and validation target.
- `mlops_template` remains the self-contained forkable template and receives validated reusable capabilities afterwards.
- `DAB_Orchestrator` is strictly read-only. Do not modify any files in that repository.
- Do not perform implementation, commits, deployments or production cutovers.

## 8. Final consistency review

Before finishing, verify that:

- Both Master Plans reflect the same revised architecture.
- Phase 1 contains no mandatory Feature Store dependency.
- Training Dataset Registry, evaluation, promotion, scoring and monitoring remain required.
- Existing preprocessing and scoring contracts are preserved.
- The template remains provisional.
- Feature Store remains documented for future consideration.
- No unrelated implementation tasks or completed statuses were changed.
- No production code or DAB_Orchestrator files were modified.

## 9. Final response

Provide a concise summary containing:

1. Files modified.
2. Feature Store tasks removed or deferred.
3. Existing tasks amended to remove dependencies.
4. Any remaining blockers or contradictions.
5. Confirmation that Phase 1 can proceed without Feature Store.

**Do not regenerate the entire plan. Perform only the targeted amendments and stop.**
