# My personal notes and lessons learned from:

# 1. Rules of Machine Learning (Google, Martin Zinkevich) 
-> Focus on Business Value
# 2. Software Engineering at Google 
-> Focus on Engineering

---

# Part I: Rules of Machine Learning

*43 rules by Martin Zinkevich. Original guide: https://developers.google.com/machine-learning/guides/rules-of-ml*

The core philosophy in one sentence: **do machine learning like the great engineer you are, not like the great machine learning expert you aren't.** Most gains come from great features and solid infrastructure, not sophisticated algorithms.

---

## Before Machine Learning

### Rule #1: Don't be afraid to launch a product without machine learning.

ML requires data. If you think ML will give you a 100% boost, a heuristic will get you 50% of the way there. Use install rates, number of installs, spam history, or alphabetical ranking as heuristics. Don't use ML until you have data.

### Rule #2: First, design and implement metrics.

Before formalizing what your ML system will do, track as much as possible in your current system. Getting historical data now is cheaper than needing it later. Design your system with metric instrumentation in mind from day one. Also: an experiment framework that groups users into buckets and aggregates statistics by experiment is important.

### Rule #3: Choose machine learning over a complex heuristic.

A simple heuristic gets your product out the door. A complex heuristic is unmaintainable. Once you have data and a basic idea of what you're trying to accomplish, move to ML — it's easier to update and maintain than a complex heuristic.

---

## ML Phase I: Your First Pipeline

### Rule #4: Keep the first model simple and get the infrastructure right.

The first model provides the biggest boost, so it doesn't need to be fancy. Infrastructure issues will dominate. You need to determine: how to get examples to your algorithm, what "good" and "bad" mean for your system, and how to integrate the model into your application. Simple features make it easier to ensure features reach the algorithm correctly, the model learns reasonable weights, and features reach the model in serving correctly. Your simple model provides the baseline.

### Rule #5: Test the infrastructure independently from the machine learning.

Test getting data into the algorithm — check that feature columns that should be populated are populated. Test getting models out of the training algorithm — make sure the model in your training environment gives the same score as in your serving environment (see Rule #37).

### Rule #6: Be careful about dropped data when copying pipelines.

When copying an existing pipeline, the old pipeline may drop data that you need. Common pattern: logging only data seen by the user, making negative examples invisible. Always audit what a copied pipeline drops before using it.

### Rule #7: Turn heuristics into features, or handle them externally.

Existing heuristics contain intuition you don't want to throw away. Four options: (1) preprocess using the heuristic, (2) create a feature directly from the heuristic score, (3) mine the raw inputs of the heuristic as separate features, (4) modify the label. Use existing heuristics to create a smooth transition to ML.

---

## Monitoring

### Rule #8: Know the freshness requirements of your system.

How much does performance degrade if the model is a day old? A week old? A quarter old? This determines monitoring priorities. Ad-serving models may need daily updates; other models can be updated infrequently. Freshness requirements change when feature columns are added or removed.

### Rule #9: Detect problems before exporting models.

Do sanity checks right before exporting. Make sure model performance is reasonable on held-out data. Issues on unexported models require an email alert; issues on user-facing models may require a page. Better to wait and be sure before impacting users.

### Rule #10: Watch for silent failures.

ML systems adjust to stale data gradually — behavior stays reasonable but slowly decays. A table that hasn't been updated for months can be refreshed for a larger gain than any launch that quarter. Track statistics of the data and manually inspect data on occasion.

### Rule #11: Give feature columns owners and documentation.

Know who created or is maintaining each feature column. If the person who understands a feature column is leaving, transfer that knowledge. Document what the feature is, where it came from, and how it is expected to help.

---

## Your First Objective

### Rule #12: Don't overthink which objective you choose to directly optimize.

Early in ML development, most metrics go up together even if you only optimize one. Keep it simple. Don't balance multiple metrics when you can still easily increase all of them. But don't confuse the objective with the ultimate health of the system — see Rule #39.

### Rule #13: Choose a simple, observable and attributable metric for your first objective.

The ML objective should be easy to measure and a proxy for the true objective. Easiest to model: directly observed user behavior attributable to a system action (click, download, forward, rating, spam report). Avoid modeling indirect effects first (daily active users, time on site). Use indirect effects as metrics in A/B testing and launch decisions, not as training objectives.

### Rule #14: Starting with an interpretable model makes debugging easier.

Linear regression, logistic regression, and Poisson regression produce predictions interpretable as probabilities or expected values. This makes them easier to debug than models optimizing classification accuracy directly. With these models, deviations between training predictions and production predictions reveal problems. Use probabilistic predictions for decisions, but remember the decision quality matters more than the likelihood of data given the model.

### Rule #15: Separate Spam Filtering and Quality Ranking in a Policy Layer.

Quality signals become obvious to bad actors who will game them. Keep quality ranking focused on good-faith content. Spam filtering is a different system entirely — features need constant updates, and the reputation of the content creator plays a major role. Integrate the two systems' outputs carefully, and remove spam from quality classifier training data.

---

## ML Phase II: Feature Engineering

### Rule #16: Plan to launch and iterate.

The model you're working on now is not the last one you'll launch. Consider whether the complexity you're adding will slow down future launches. Teams launch a model per quarter or more for years. Think about how easy it is to add, remove, or recombine features. Think about how easy it is to create a fresh copy of the pipeline and verify its correctness.

### Rule #17: Start with directly observed and reported features as opposed to learned features.

Learned features (from clustering, factored models, deep learning) can be useful but have many pitfalls. External system objectives may be weakly correlated with your objective. Snapshots go stale. Non-convex models don't guarantee optimal solutions, making it hard to distinguish meaningful changes from random variation. Get an excellent baseline from direct features first, then try more esoteric approaches.

### Rule #18: Explore with features of content that generalize across contexts.

Features observed in one context (searches, watches, ratings) can be valuable in another (recommendations, rankings). These features allow bringing new content into a context where you have no data yet. Note: this is about figuring out if someone likes content in a context first, then who likes it more or less.

### Rule #19: Use very specific features when you can.

With lots of data, learning millions of simple features is easier than learning a few complex features. Don't be afraid of features that apply to a small fraction of your data if overall coverage is above 90%. Use regularization to eliminate features that apply to too few examples.

### Rule #20: Combine and modify existing features to create new features in human-understandable ways.

Two standard approaches: discretization (turning a continuous feature into discrete bins — basic quantiles give most of the impact) and crosses (combining two or more feature columns to create interaction features). Crosses that produce very large feature columns may overfit. For text: dot products or intersections.

### Rule #21: The number of feature weights you can learn in a linear model is roughly proportional to the amount of data you have.

Scale learning to the size of your data. 1,000 examples: use dot products and a dozen human-engineered features. 1 million examples: use feature crosses with regularization, potentially millions of features. 1 billion examples: cross feature columns with document and query tokens, use feature selection. Statistical learning theory gives guidance for a starting point, not tight bounds.

### Rule #22: Clean up features you are no longer using.

Unused features create technical debt. If you're not using a feature and combining it with others isn't working, drop it from your infrastructure. Keep coverage in mind: a feature covering only 8% of users is unlikely to be effective. But a feature covering 1% of data where 90% of those examples are positive is worth keeping.

---

## Human Analysis of the System

### Rule #23: You are not a typical end user.

You are too close to the code, subject to confirmation bias, and your time is too valuable. Anything that looks reasonably near production should be tested further — by paying laypeople on crowdsourcing platforms or through live experiments. Use user experience methodologies: create user personas, do usability testing.

### Rule #24: Measure the delta between models.

Before users see your new model, calculate how different the new results are from production. Use symmetric difference of results weighted by ranking position. Small difference: little change expected without running an experiment. Large difference: make sure it's good. Also verify that a model compared with itself has zero symmetric difference — ensure system stability.

### Rule #25: When choosing models, utilitarian performance trumps predictive power.

The key question is what you do with the prediction, not how accurate the prediction itself is. If using predictions to rank, the quality of the final ranking matters more than the prediction. If predicting spam probability, the precision of what is allowed through matters more. If a change improves log loss but degrades system performance, look for another feature.

### Rule #26: Look for patterns in the measured errors, and create new features.

Find training examples the model got wrong. Those are examples the system knows it got wrong and wants to fix. Create features that allow it to fix those errors. Features created for examples the system doesn't see as mistakes will be ignored. Once you find patterns in errors, add a dozen features in that direction and let the model figure out what to do with them.

### Rule #27: Try to quantify observed undesirable behavior.

Turn gripes about the system into solid numbers. If you think too many gag apps are appearing, have human raters identify them. If issues are measurable, you can start using them as features, objectives, or metrics. General rule: **measure first, optimize second**.

### Rule #28: Be aware that identical short-term behavior does not imply identical long-term behavior.

A system that only uses historical data for each query has no way to learn that a new document should be shown. Identical A/B test behavior doesn't mean identical long-term behavior. The only way to understand long-term behavior is to train on data acquired when the model was live.

---

## Training-Serving Skew

### Rule #29: Save features used at serving time and pipe them to training.

The best way to ensure you train like you serve is to log the set of features used at serving time, and use those features for training. Even for a small fraction of examples, this lets you verify consistency between serving and training. Teams that have made this measurement at Google were sometimes surprised by the results.

### Rule #30: Importance-weight sampled data, don't arbitrarily drop it.

When you have too much data, don't just ignore files 13-99. Data never shown to users can be dropped, but for the rest use importance weighting: if you sample example X with 30% probability, give it weight 10/3. With importance weighting, calibration properties still hold.

### Rule #31: Beware that if you join data from a table at training and serving time, the data in the table may change.

Features joined from a table at serving time may have different values at training time. Your model's prediction for the same document can then differ between training and serving. Easiest fix: log features at serving time. If the table changes slowly, snapshot it hourly or daily.

### Rule #32: Re-use code between your training pipeline and your serving pipeline whenever possible.

Create an object where query/join results are stored in a human-readable way, then run a common method to bridge to the ML system's expected format — for both training and serving. This eliminates a source of training-serving skew. Avoid using two different programming languages between training and serving.

### Rule #33: If you produce a model based on data until January 5th, test the model on data from January 6th and after.

Test on data gathered after training data — this better reflects production behavior. Performance will be slightly worse on new data, but shouldn't be radically worse. AUC should be reasonably close even if average click rates differ due to daily effects.

### Rule #34: In binary classification for filtering, make small short-term sacrifices in performance for very clean data.

In filtering tasks, negatives blocked by the filter can't be used for training without sampling bias. Solution: label 1% of all traffic as "held out" and send those to the user. Use the held-out examples as training data for a clean, unbiased signal.

### Rule #35: Beware of the inherent skew in ranking problems.

When you change a ranking algorithm radically, you change the data your algorithm sees in the future. Design your model around this: higher regularization on features covering many queries, only allowing features to have positive weights, avoiding document-only features.

### Rule #36: Avoid feedback loops with positional features.

Content position dramatically affects interaction likelihood. Add positional features so the model learns to weight them. At serving time, don't provide positional features or give all candidates the same default value. Keep positional features separate from the rest of the model — don't cross them with document features.

### Rule #37: Measure Training/Serving Skew.

Three parts to measure: (1) performance on training data vs. holdout data — always exists, not always bad; (2) performance on holdout vs. next-day data — tune regularization to maximize next-day performance, large drops indicate time-sensitive features; (3) performance on next-day data vs. live data — any discrepancy here is likely an engineering error.

---

## ML Phase III: Slowed Growth, Optimization Refinement, and Complex Models

### Rule #38: Don't waste time on new features if unaligned objectives have become the issue.

When measurements plateau, the issue is often that product goals are not covered by the existing ML objective. Change either your objective or your product goals before investing in new features.

### Rule #39: Launch decisions are a proxy for long-term product goals.

A model that improves install rate but drops daily active users by 5% might not get launched. Launch decisions depend on multiple criteria, only some of which can be directly optimized with ML. Measurable A/B metrics are proxies for longer-term goals: user satisfaction, growth, partner satisfaction, profit. The only easy launch decisions are when all metrics improve.

### Rule #40: Keep ensembles simple.

Each model should either be an ensemble taking only outputs of other models, or a base model taking many features — not both. Use a simple model for ensembling that takes only the output of your base models as inputs. Enforce that an increase in a base model's predicted probability does not decrease the ensemble's predicted probability.

### Rule #41: When performance plateaus, look for qualitatively new sources of information.

When you've exhausted features, tuned regularization, and seen less than 1% improvement for several quarters: build infrastructure for radically different features. User history from the last day/week/year. Data from a different property. Knowledge graph entities. Deep learning. Weigh the benefit of new features against the cost of increased complexity.

### Rule #42: Don't expect diversity, personalization, or relevance to be as correlated with popularity as you think they are.

If your system measures clicks, watches, shares — you're measuring popularity. Personalization and diversity features often get less weight (or different sign) than expected. This doesn't mean they're not valuable: use post-processing to increase diversity or relevance, or modify the objective directly if longer-term metrics improve.

### Rule #43: Your friends tend to be the same across different products. Your interests tend not to be.

Social graph connections transfer well across products. Interest-based personalization typically does not. What sometimes works: using raw behavioral data from one property to predict behavior on another, or using the presence of activity on two products as a signal in itself.

---

---

# Part II: Software Engineering at Google — ML/AI Insights

*Chapter-by-chapter extraction from: Software Engineering at Google (Winters, Manshreck, Wright, 2020)*

Relevance:  = very high,  = high,  = medium,  = low

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

* **Q**uality — model quality
* **A**ttention — engineer flow during experimentation
* i**N**tellectual complexity — cognitive load of the pipeline
* **T**empo/Velocity — how fast can features be deployed?
* **S**atisfaction — satisfaction with MLOps tooling

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

* **Reference docs:** docstrings for all public functions
* **Design docs:** why LightGBM over XGBoost, why a 24h horizon
* **Tutorials:** how to start training locally, how to add a feature
* **Conceptual docs:** how the forecasting pipeline works end-to-end

### WHO/WHAT/WHEN/WHERE/WHY

Every design doc should answer these in the first two paragraphs. Especially important: WHY (motivation, alternatives considered) and WHEN (freshness date). Stale docs are worse than no docs.

### Docs next to code

Documentation in the same repo as the code. `docs/decisions/` for ADRs. `model_card.md` next to the training script. Freshness dates on design docs.

---

## Chapter 11 — Testing Overview 

**Tags:** ML Engineering, Data Engineering, CI/CD

### Test sizes for ML

* **Small:** pure Python domain logic (feature calculations, validation rules) — no I/O, no sklearn, runs in milliseconds
* **Medium:** sklearn pipeline integration with synthetic DataFrames, FastAPI test client
* **Large:** end-to-end with real ENTSO-E data through the full training pipeline

### 80/15/5 Pyramid

* 80% unit (domain)
* 15% integration (pipeline)
* 5% E2E (real data)

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

* Configuration issues (does the Docker container start with all env vars?)
* Emergent behaviors with real data
* Load issues
* Hyrum's Law violations

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

* Does the project have tests?
* Who maintains it?
* What compatibility guarantees does it make?
* How long will we depend on it?

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

# My personal learnings:

### Return Objects for Pipeline Orchestration

Multi-step orchestrators (training, ingestion, prediction) should return structured result objects, not None. A dataclass with execution metadata (rows_written, duration_seconds, rows_dropped) gives the caller inspectable output without log parsing. This enables downstream monitoring, conditional logic in schedulers (Airflow, Step Functions), and consistent contracts at every orchestration boundary. Exceptions remain the error signal; the return object confirms success with context.

