# My personal notes and lessons learned from the book:
# Software Engineering at Google

---

## Chapter 1 — What Is Software Engineering? 

**Tags:** Architecture, ML Engineering, Data Engineering

### Hyrum's Law for ML
Every observable behavior of an API will eventually be depended upon by someone. For ML: DataFrame column order, float precision, null handling, feature pipeline output schema — all become implicit dependencies. Countermeasures: Pydantic schemas, versioned MLflow artifacts, output contract tests.

### Sustainability for ML systems
Code is sustainable when you can react to any change for as long as it needs to live. ML models and pipelines must be designed for upgradeability: retraining mechanisms, schema evolution, dependency updates.

### Shifting Left
Finding problems earlier is cheaper. Applied to ML: validate data schemas at ingestion, test feature logic before training, catch serving contract violations before deployment. Bugs in production cost exponentially more.

### The Beyoncé Rule
"If you liked it, you shoulda put a test on it." If CI should protect a behavior — model output format, feature null handling — a test must exist for it.

---

## Chapter 2 — How to Work Well on Teams 

**Tags:** Architecture

### Bus Factor
No single person should be the only one who understands the ML pipeline. Document feature engineering decisions, model assumptions, and data source idiosyncrasies.

### Blameless Post-Mortems
When a model degrades in production: structured post-mortem with timeline, root cause, action items. Use findings to improve future model versions.

---

## Chapter 3 — Knowledge Sharing 

**Tags:** Architecture, ML Engineering

### Canonical Sources
One authoritative source for feature definitions, data dictionaries, model cards, evaluation metrics. No information islands where each team has their own feature engineering conventions.

### Documentation as Code
Model documentation (feature descriptions, training data lineage, evaluation metrics) should live in version control alongside the code. Static analysis as knowledge transfer: linters and type checkers automatically encode best practices.

### Tribal Knowledge is a Liability
If only one person knows why a feature was engineered a particular way, that's a bus factor problem. Code reviews and structured documentation are the countermeasure.

---

## Chapter 4 — Engineering for Equity 

**Tags:** Data Science, ML Engineering

### Training Data Bias
Google Photos classifying Black people as gorillas is a direct ML example. Training data that doesn't represent the population produces invalid results.

### Measure equity, don't assume it
"Don't assume equity; measure equity." For ML: monitor model performance across demographic subgroups. Confusion matrix per subgroup, not just globally.

### Prioritize data quality
"We should be willing to delay development in favor of trying to get more complete and accurate data." Directly applicable to ML: incomplete training data = invalid results.

---

## Chapter 7 — Measuring Engineering Productivity 

**Tags:** Data Science, ML Engineering, Architecture

### GSM Framework (Goals → Signals → Metrics)
Directly applicable to ML evaluation. Don't just measure what's easy (accuracy); measure what matters (business impact, latency, fairness). For every ML metric: which goal does it map to? Which signal is it a proxy for?

### QUANTS for ML
- **Q**uality — model quality
- **A**ttention — engineer flow during experimentation
- i**N**tellectual complexity — cognitive load of the pipeline
- **T**empo/Velocity — how fast can features be deployed?
- **S**atisfaction — satisfaction with MLOps tooling

Don't optimize only for model quality at the expense of other dimensions.

### Actionable metrics only
Only measure what you will actually use as a basis for decisions. If a metric won't trigger retraining, don't track it.

---

## Chapter 8 — Style Guides and Rules 

**Tags:** ML Engineering, Data Engineering

### Optimize for the reader
ML code (feature engineering, training scripts) is read far more often than it is written. Readability over cleverness. Especially important for pipeline code maintained by other teams.

### Automate enforcement
`black`, `mypy`, `pylint`, `ruff` — don't rely on human reviews for style. Automated enforcement scales; manual enforcement does not. DS-specific: a linter that flags `apply(lambda)` or blind `dropna()`.

### Consistency enables tooling
Consistent code enables automated refactoring. If all feature transformers follow the same interface, a schema change can be applied automatically.

---

## Chapter 9 — Code Review 

**Tags:** ML Engineering, Data Engineering, Architecture

### Code is a Liability
Every line of ML pipeline code is maintenance burden. Delete dead feature engineering code rather than commenting it out. Good ML code: less, clearer, testable.

### Write small changes
ML PRs should be small and focused. Don't combine "new feature + refactoring + dependency update" in one PR. Easier to debug and revert on regressions.

### Write good change descriptions
"Refactor feature pipeline" is bad. "Extract lag feature calculation into FeatureService to enable unit testing" is good. The description is a historical artifact — crucial when debugging two years from now.

---

## Chapter 10 — Documentation 

**Tags:** ML Engineering, Architecture

### Documentation types for ML
- **Reference docs:** docstrings for all public functions
- **Design docs:** why LightGBM over XGBoost, why a 24h horizon
- **Tutorials:** how to start training locally, how to add a feature
- **Conceptual docs:** how the forecasting pipeline works end-to-end

### WHO/WHAT/WHEN/WHERE/WHY
Every design doc should answer these in the first two paragraphs. Especially important: WHY (motivation, alternatives considered) and WHEN (freshness date). Stale docs are worse than no docs.

### Docs next to code
Documentation in the same repo as the code. `docs/decisions/` for ADRs. `model_card.md` next to the training script. Freshness dates on design docs.

---

## Chapter 11 — Testing Overview 

**Tags:** ML Engineering, Data Engineering, CI/CD

### Test sizes for ML
- **Small:** pure Python domain logic (feature calculations, validation rules) — no I/O, no sklearn, runs in milliseconds
- **Medium:** sklearn pipeline integration with synthetic DataFrames, FastAPI test client
- **Large:** end-to-end with real ENTSO-E data through the full training pipeline

### 80/15/5 Pyramid
- 80% unit (domain)
- 15% integration (pipeline)
- 5% E2E (real data)

ML anti-pattern: "ice cream cone" — only E2E tests, no unit tests. Result: slow, flaky, hard to debug.

### The Beyoncé Rule
If CI should catch a regression in model output format, a test must exist for it. Also applies to null handling, column order, data types.

### Automated testing enables change
Without tests you cannot safely refactor the feature pipeline. Tests are what makes refactoring possible in the first place — especially important when upstream data schemas change.

---

## Chapter 12 — Unit Testing 

**Tags:** ML Engineering, Data Engineering

### Test behaviors, not methods
Not `test_calculate_lag_features()`, but `test_lag_features_handle_nulls_at_series_start()`. Each test describes a behavior, not a method.

### Test via public APIs
Test the `FeatureService.transform()` interface, not internal helper functions. Tests that only touch the public API don't break during internal refactorings.

### Test state, not interactions
Assert on the output DataFrame, not on which internal methods were called. `assert_frame_equal()` over `verify(mock_transformer).transform()`. Interaction tests are brittle.

### DAMP over DRY in tests
In tests, some duplication is fine if it makes the test more understandable. Every test should be comprehensible without context. DRY tests often become unreadable.

---

## Chapter 13 — Test Doubles 

**Tags:** ML Engineering, Data Engineering

### Prefer real implementations
Real sklearn transformer > mock. Only mock external services: MLflow, databases, APIs. Too much mocking produces tests that test nothing.

### Fakes over mocks
An in-memory feature store fake is better than a mock. Fakes are realistic implementations; mocks are just call recorders.

### Stub the ENTSO-E API
In integration tests: stub ENTSO-E API responses. Don't hit the real API in CI. But always test your own feature pipeline with real sklearn transformers.

---

## Chapter 14 — Larger Testing 

**Tags:** ML Engineering, Data Engineering, CI/CD

### Fidelity gaps in unit tests
Unit tests with mocked data miss:
- Configuration issues (does the Docker container start with all env vars?)
- Emergent behaviors with real data
- Load issues
- Hyrum's Law violations

### A/B diff testing
Run old and new model with the same inputs and compare outputs. Critical on model updates: explicitly classify every deviation in prediction output as expected or unexpected.

### Hermetic SUTs
For integration tests: in-memory databases and a local MLflow tracking server instead of production backends. No dependency on external services in presubmit tests.

### Configuration testing
Test that the Docker container starts cleanly with all required env vars — smoke test. Many production outages are configuration errors, not code bugs. For ML: MLflow URI, feature store connection, model registry credentials.

---

## Chapter 15 — Deprecation 

**Tags:** Architecture, ML Engineering

### Code is a liability
Delete old feature engineering code that is no longer needed. Don't comment it out, don't move it to a `deprecated/` directory. Dead code paths increase cognitive load.

### Deprecation warnings with Python
`warnings.warn(..., DeprecationWarning)` for deprecated feature functions. Before migration: provide automated tooling-based updates rather than manual changes.

### Hyrum's Law and deprecation
Users will have grown dependent on deprecated behavior. Provide migration tools — don't just announce "stop using the old thing."

---

## Chapter 16 — Version Control and Branch Management 

**Tags:** Data Engineering, ML Engineering, CI/CD

### Trunk-based development
Commit to `main` frequently. Feature branches should be short-lived (hours to days, not weeks). For ML: no long-lived `feature/new-model` branches. Use feature flags to hide unfinished code instead.

### One-Version Rule
Pin all dependencies explicitly in `pyproject.toml`. No floating versions (`latest`). For ML: one version of pandas, sklearn, lightgbm — not multiple in parallel. Avoid the diamond dependency problem.

### Version control for configuration
All model hyperparameters, feature configs, and deployment configs in version control. Not just code — configuration too. Configuration changes should be reviewed the same way as code changes.

### No long-lived dev branches
DORA research: trunk-based development is a significant positive predictor of high-performing engineering organizations. Long-lived dev branches are an anti-pattern for ML teams too.

---

## Chapter 18 — Build Systems and Build Philosophy 

**Tags:** Data Engineering, ML Engineering, CI/CD

### Reproducible builds for ML
Same inputs → same outputs. For ML: same data + same code → same model artifact. `pyproject.toml` with pinned dependencies enables reproducible training runs.

### Fine-grained modules (1:1:1 rule)
One module per concern:
```
src/energy_forecast/domain/
src/energy_forecast/application/
src/energy_forecast/infrastructure/
```
Fine-grained modules scale better and enable parallelization in the build.

### External dependency management
`requirements.txt` with pinned versions. Mirror critical dependencies internally. Cryptographic hashes for external artifacts — supply chain attacks on ML pipelines are real.

### Build caching for ML
Docker layer caching, pip caching in CI. Don't rebuild what hasn't changed. Especially important for large training images.

---

## Chapter 20 — Static Analysis 

**Tags:** ML Engineering, Data Engineering

### Integrate into CI
Run `mypy`, `pylint`, `ruff`, `bandit` on every PR. Block merges on failures. Keep the false-positive rate low — engineers will ignore tools that produce too much noise.

### Detect DS-specific anti-patterns
Static analysis can catch pandas anti-patterns: `apply(lambda)`, blind `dropna()`, slice assignments without `.copy()`. Encode these in custom linter rules.

### Empower users to contribute
Domain experts write their own analyzer rules. An ML engineer who finds a recurring bug can write a check for it. Tooling as knowledge transfer.

---

## Chapter 21 — Dependency Management 

**Tags:** Dependencies, Data Engineering

### Diamond dependency problem
LightGBM and MLflow both depend on numpy — but potentially different versions. `pip-compile` resolves this deterministically. Never specify `latest` as a version.

### SemVer is lossy
Version numbers are estimates, not guarantees. Hyrum's Law: even "patch releases" can break ML pipelines if they depend on observable-but-undocumented behavior.

### Minimum Version Selection
When updating: take the smallest step forward possible. Don't jump straight to the latest version. Fewer unexpected changes, easier to debug.

### Questions to ask before importing
- Does the project have tests?
- Who maintains it?
- What compatibility guarantees does it make?
- How long will we depend on it?

For a production ML pipeline with a 3+ year lifespan, these are critical questions.

---

## Chapter 22 — Large-Scale Changes 

**Tags:** Architecture, ML Engineering

### Automated refactoring for ML
When a feature interface change affects many files: use automated tools (sed, AST transformers) rather than manual edits. Small, independent PRs that can each be tested individually.

### Test coverage enables LSCs
Without tests, large refactorings are too risky. Tests are what makes safe refactoring possible at all — including in ML pipelines.

---

## Chapter 23 — Continuous Integration 

**Tags:** CI/CD, ML Engineering, Data Engineering

### CI for ML = testing the entire ecosystem
Not just code, but also data schemas, model artifacts, configuration, upstream API compatibility. An ML pipeline often breaks due to data schema changes, not code bugs.

### Fast feedback loops

| Stage | Contents | Latency |
|---|---|---|
| Presubmit | Unit tests (domain) | Seconds |
| Post-submit | Integration tests (pipeline) | Minutes |
| Nightly | E2E with real data | Hours |

The earlier a problem is found, the cheaper it is to fix.

### Hermetic testing for ML
Synthetic data and a local MLflow server for presubmit tests. Don't hit the real ENTSO-E API in CI. Hermetic = deterministic, isolated, reproducible.

### CI is alerting
A failing CI test is like a production alert. Fix it immediately or explicitly disable it with a tracked bug — never ignore it. Actively eliminate flaky tests; they erode confidence.

### Configuration in CI
The Docker container starts cleanly with all env vars — smoke test. Configuration errors cause most production outages. For ML: MLflow URI, feature store connection, model registry credentials.

---

## Chapter 24 — Continuous Delivery 

**Tags:** CI/CD, Serving, ML Engineering

### Faster is safer
Small, frequent model updates are safer than large, infrequent ones. Each update is easier to debug. Applies to ML: weekly retraining beats quarterly.

### Flag-guard all changes
Flag-guard new model versions. Deploy to 10% of traffic, monitor, then roll out. Not 100% immediately. Feature flags decouple code deployment from feature activation.

### Release train for ML
Set a regular retraining cadence (e.g. weekly). If a feature misses the train, it waits for the next one — no stopping the entire release for one late feature.

### No binary is perfect
Ship when metrics exceed a threshold, not when it's perfect. Define acceptance criteria upfront. Don't hold model updates waiting for perfection.

### Phased rollout for models
Staging → Canary (10%) → Full Production. Automatic rollback when metrics degrade. A/B testing between old and new model.

---

## Chapter 25 — Compute as a Service 

**Tags:** Serving, Architecture, ML Engineering

### Cattle, not pets
ML training jobs and serving containers should be stateless and replaceable. Don't SSH into a training worker to repair it — kill it and restart it.

### Architecting for failure
Training jobs should periodically checkpoint state to external storage. If a worker dies, restart from the last checkpoint. For batch inference: split work into small chunks and assign dynamically.

### Batch vs. Serving

| Type | Optimization goal | Infrastructure |
|---|---|---|
| Training (batch) | Throughput | GPU cluster, spot instances |
| Inference (serving) | Latency | Dedicated, stable pods |

Don't optimize both with the same setup.

### Managing state
All model artifacts, feature statistics, and training data in external storage (MLflow, S3/GCS), not on local disk. In-process state is transient; real persistence requires external storage.

### Containerization for reproducibility
Docker containers ensure a reproducible environment. Same container in dev, CI, and production. Same Python version, same library versions — eliminates "works on my machine."

### Serverless for low-traffic ML
For infrequent batch inference jobs: serverless (AWS Lambda, Cloud Functions) can be more cost-effective than persistent containers. Trade-off: less control, no local state.

---

## Summary: Top Priorities for Your Energy Forecasting Project

### Immediately actionable (before March 31)
1. **Pydantic schemas** for feature pipeline outputs (Hyrum's Law protection)
2. **Write small tests** for domain logic — lag features, null handling, no I/O
3. **Set up GitHub Actions CI:** `black` + `mypy` + `pytest` on every PR
4. **Pin all dependencies** in `pyproject.toml` with exact versions
5. **`docs/decisions/`** with ADRs for key architectural decisions
6. **Docker smoke test:** container starts cleanly with all env vars

### Architecture principles confirmed
- **DDD layering:** Domain (pure Python) → Application (use cases) → Infrastructure (MLflow, FastAPI, DB)
- **Trunk-based development:** short-lived branches, frequent commits to `main`
- **Test pyramid:** 80% unit (domain), 15% integration (pipeline), 5% E2E (real data)
- **One-Version Rule:** pin all dependencies, no floating versions
