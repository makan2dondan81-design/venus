# ML Factory Architecture Context

## 1. Mission and non-goals

Create a **self-contained Databricks MLOps template** that a small ML team can fork and customise for each use case. `TP_intervention` is the first real reference implementation, used to prove the design before the reusable parts are brought back into `mlops_template`.

The goal is not a centrally hosted ML development platform, a mandatory shared package, or a replacement for notebooks. Keep the solution simple enough for data scientists and MLEs to own together.

The intended lifecycle is:

```text
Fork mlops_template
        ↓
Explore in notebooks using embedded ml_utils
        ↓
Promote proven joins/feature calculations to shared project code
        ↓
Materialise features into UC Feature Store tables
        ↓
Define/register a training dataset (TD001, TD002, ...)
        ↓
Capture candidate choices in a small config
        ↓
Run a standard training entrypoint in Databricks
        ↓
Evaluate → MLflow log → register fitted preprocessing/model
        ↓
DAB-deployed feature refresh / scoring / monitoring
```

## 2. Three repositories: one design, different roles

The working VS Code workspace includes:

| Repository | Purpose |
|---|---|
| `TP_intervention` | Existing fork, real data science workflow, first production implementation and end-to-end validation |
| `mlops_template` | Existing generic template; eventual release-ready source for future self-contained forks |
| `DAB_Orchestrator` | **Strictly read-only** evidence of today's Databricks scheduling, DAB configuration and Azure DevOps practices |

### One shared architecture, two implementation plans

Plan against both TP and the template from the beginning, but avoid two independent redesigns:

- **TP Master Plan (active):** real implementation, detailed workstreams and atomic tasks, migration of existing TP scoring/preprocessing, and full end-to-end validation.
- **Template Master Plan (initially provisional):** assess actual template code and gaps, map reusable candidates, choose non-TP-specific defaults, and record dependencies on results of TP validation. Do not prematurely create detailed coding tasks for unvalidated abstractions.
- **After TP validation:** revise the template plan in a separate review pass, derive executable template workstreams, selectively generalise proven changes, and test a fresh fork.

`docs/architecture/ML_FACTORY_CONTEXT.md` is the shared architectural source of truth for the planning phase. The two Master Plans must not invent incompatible interfaces. When template work begins, copy/adapt enough architecture and agent guidance into `mlops_template` for that repo to be independently maintainable.

### DAB_Orchestrator is never an implementation target

**Do not edit any `DAB_Orchestrator` file or plan changes to it.** In particular, inspect the current TP flow around `src/third_party_intervention_ml.py` where available, but put any new DAB job definitions under `TP_intervention` (and eventually generalise suitable scaffolding into `mlops_template`). Any switchover from legacy scheduled jobs is a separate, explicitly authorised operational cutover.

## 3. Fork and update policy (chosen option)

**Option A: self-contained fork with selective upstream updates.**

```text
mlops_template (versioned releases + changelog)
        ↓ fork / copy and customise
TP_intervention     another_use_case     future_use_case
    │                     │                     │
    └── each owns its embedded ml_utils and workflow code
```

Each fork remains independently deployable. A change in `mlops_template` **does not automatically change** an existing use case.

When TP establishes a reusable improvement:

1. Identify the reusable implementation and exclude TP-specific labels, claims business logic, paths and table names.
2. Port/adapt it into `mlops_template`, with simple defaults and relevant documentation.
3. Record the change in the template's changelog and create a Git tag/release using the team's normal release conventions. Optionally record the originating template version in each use case.
4. Existing forks adopt the change **selectively**, through an ordinary reviewed change/PR when useful.

**Deferred:** centrally published `ml_utils` package, Azure Artifacts feed, dependency-based auto-updates, submodules, bots and automated library synchronisation. Do not design those now.

## 4. Data science experience: explore → decide → configure

Data scientists should work freely in `notebooks/` to perform EDA, construct candidate datasets, investigate engineered features, use Optuna, compare models and inspect explanations. They should be able to import easy-to-use embedded helpers:

```python
from ml_utils import ...
```

(Or an equivalent chosen once from the current repository.) Eliminate the need for a *separately maintained* `libs/ml_utils` project, while retaining normal Python packaging where convenient. `ml_utils` is shipped **inside the fork**.

The handoff is not “data scientist writes features, MLE rewrites sklearn transformers”. Instead:

```text
Explore a feature in a notebook
        ↓
Promote stable feature computation into project Python/SQL
        ↓
Reuse the SAME code in notebook and scheduled feature job
        ↓
Select dataset/model/preprocessing choices in config
        ↓
Run standard training implementation
```

Use config for **choices**, not as a custom language for join logic or feature formulas. Keep the number of config files small.

## 5. Feature computation vs fitted preprocessing

The current TP preprocessing artifact may perform raw source reads, joins, aggregations, business feature engineering and fitted transforms together. Its migration should reduce this coupling while retaining training/scoring consistency.

| Concern | Preferred home |
|---|---|
| Source-table joins, aggregation, deterministic date/claim features | Feature calculation Python/SQL in use-case repo |
| Computed reusable feature values | Unity Catalog Feature Store tables |
| Learned imputation/encoding/scaling and other fitted operations | Model preprocessing artifact or model inference pipeline |
| Predictions | Model inference using feature lookups and required fitted preprocessing |

The model-serving/inference packaging must ensure fitted preprocessing executes correctly. Whether it is logged as part of the model inference pipeline, separately registered and explicitly resolved, or both should be decided against real existing code and the supported Feature Engineering APIs. Do not assume feature lookup alone applies the fitted preprocessing artifact.

## 6. Feature Store: initial scope

Use Databricks Feature Engineering with governed Unity Catalog feature tables. Data scientists define features; production code materialises values; models use feature lookups rather than rejoining raw tables inside custom sklearn transformers.

```text
Claims / policies / contacts (UC Delta)
             ↓
Project feature-building functions or SQL
             ↓
UC feature tables with entity keys
        ┌────┴────┐
        ↓         ↓
     Training   Scoring
```

For initial TP implementation:

- Use ordinary entity-key feature lookups (for example `appclaimid`, if verified).
- Derive appropriate table boundaries/refresh cadence from the real use case; don't create one table for every intermediate DataFrame.
- Keep feature computation in source-controlled project code; Feature Store stores feature *values*, not the code that creates them.
- Use the supported Databricks Feature Engineering APIs for training-set creation, model logging and batch scoring where appropriate; verify current runtime/client compatibility before coding.
- **Defer time-series Feature Tables, point-in-time/as-of lookups and historical time-aware feature joins.** These must not become initial implementation workstreams. Acknowledge that current-value lookup is not an automatic historical snapshot and document any relevant modelling limitation.

Feature tables require a refresh job. Order refresh and scoring based on required freshness and existing operations.

## 7. Mandatory Training Dataset Registry

The first implementation **must have a Training Dataset Registry**. This is not a deferred stretch goal. Prefer a small governed Unity Catalog Delta table unless TP already has an adequate reusable implementation.

A training dataset identity such as `TD001` should map to the essential reproducible *definition* of the training population, not merely an unqualified table name. Capture only information needed to resolve and explain the training set:

- unique dataset ID and use case;
- label source, target definition and relevant filters/window;
- referenced feature table(s), feature names and entity lookup key(s);
- stable source/table snapshot references or other reproducibility information **where available and practical**;
- relevant code/configuration reference and creation metadata.

The registry must be able to resolve a dataset ID for a repeatable training entrypoint. If precise historical recreation cannot be guaranteed by the initial table strategy, document that limitation rather than claiming full reproducibility.

**Separation of concerns:**

| System | Question answered |
|---|---|
| Training Dataset Registry | Which governed dataset definition did we train against? |
| MLflow | Which parameters, metrics, artifacts and code/config were used in the run? |
| Unity Catalog Model Registry / MLflow artifacts | Which registered model and fitted preprocessing artifacts were produced? |

Do not build a general-purpose registry for everything. Persist the relationship between dataset ID, MLflow run, model version and any separately registered preprocessing version.

## 8. Config-driven candidate training

A candidate's final settings should be recorded in a small configuration file after notebook experiments. An illustrative shape (not a mandated schema):

```yaml
training:
  dataset_id: TD003
  model: lightgbm
  parameters:
    learning_rate: 0.03
    num_leaves: 31
  preprocessing: standard
  evaluate: [roc_auc, precision]
  register_as: tp_intervention_model
```

Use the config to invoke **one standard training implementation**, not separate notebook and production implementations.

```text
Training config
   ↓
Resolve Training Dataset Registry ID
   ↓
FeatureLookup(s) + training set creation
   ↓
Fit necessary preprocessing → train/tune → evaluate
   ↓
Log MLflow run + metadata/config
   ↓
Register appropriate model/preprocessing artifacts
```

The core implementation should be Python modules. A Databricks notebook may be a thin interactive entrypoint accepting a config path; DAB may invoke the same logic. Existing useful Optuna and custom transformer helpers should be reused.

Keep evaluation and registration standards practical, with use-case-specific thresholds/configuration where appropriate.

## 9. Scoring, monitoring and orchestration

The target batch scoring flow:

```text
Scheduled feature refresh → UC feature tables
                                  ↓
Scoring population with entity IDs
                                  ↓
Model-associated feature lookup
                                  ↓
Required fitted preprocessing + registered model
                                  ↓
Persist scored data with model/preprocessor versions
                                  ↓
Current useful monitoring
```

Do not blindly replace the existing TP scoring path. Preserve outputs and downstream contracts until the new path is validated, then plan a controlled adoption. Continue useful existing volume/completeness/prediction and feature-drift monitoring where it is already working; don't build a large new observability platform.

The eventual fork owns applicable DAB resources and Azure DevOps deployment configuration. The existing `DAB_Orchestrator` supplies **conventions only**, never code-edit targets. Exact job decomposition (feature refresh, scoring, training on demand, monitoring) should reflect actual operational needs rather than mandatory one-job-per-stage rules.

## 10. AzureML legacy and repository structure

This template originally included AzureML-specific content. The current target is **Azure Databricks**. Remove unreferenced AzureML dependencies, pipeline configuration, deployment scripts and docs *only after verifying usage*. Likewise, inspect `pipelines/` rather than treating its presence as proof of active production use.

Prefer the smallest useful repository change, roughly supporting:

```text
notebooks/              # EDA and experiments
ml_utils/ or src/ml_utils/  # embedded reusable Python package
project_features/ or equivalent  # use-case-specific feature calculations
configs/                # small training/scoring configs
pipelines/ or jobs/      # thin repeatable entrypoints
resources/ + databricks.yml  # use-case-owned DAB resources
```

The final layout should be derived from existing code/packaging; this is illustrative, not a mandatory restructure.

## 11. Testing and Definition of Done

Use **lightweight, risk-based tests**, especially for useful deterministic transforms, config parsing and registry helpers that can be checked without external services. Prefer targeted Databricks integration/smoke validation for actual Feature Store, MLflow/UC and DAB functionality. There is no unit-test coverage quota.

**TP implementation Done** means the required flow is demonstrated with a real or safe representative TP data path: feature computation/materialisation → registry-backed config-driven training → evaluation/artifact registration → scoring with Feature Store lookups → persistent outputs/monitoring, preserving necessary production contracts. Any production cutover is separately authorised.

**Template implementation Done** means the proven generic pattern is usable after a *fresh fork*, with working imports, appropriately parameterised configs, no TP-specific assumptions, and a small representative feature/training/scoring smoke check.

## 12. Deferred scope

- Databricks time-series features and point-in-time/as-of lookup implementation;
- GenAI/LLM Factory;
- shared `ml_utils` package distribution / Azure Artifacts publishing;
- automatic template-to-fork syncing or upgrades;
- complex custom metadata platforms beyond the required Training Dataset Registry;
- custom feature DSL or workflow engine;
- extensive testing infrastructures not justified by risk;
- edits to `DAB_Orchestrator` (prohibited, not merely deferred).
