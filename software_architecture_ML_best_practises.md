# Fundamentals of Software Architecture — Personal Rules for ML/AI Engineering

*Derived from Richards & Ford, "Fundamentals of Software Architecture" (2020),
applied for Data Science, ML Engineering, and AI Architecture practice.*

---

### Rule #1: Architecture decisions are the ones that are hard to undo.

Before starting a project, ask: which decisions will be expensive to reverse in six months? Those are architectural decisions. Everything else is design.

Architectural for ML: how training and serving are coupled, where the model registry lives, whether inference is synchronous or asynchronous, how data schemas are versioned.

Not architectural: which hyperparameters, how many lag features, whether to use ruff or pylint.

Document architectural decisions in `docs/decisions/` as ADRs at the moment they are made — not retrospectively.

### Rule #2: Document why, not just how.

The structure of a system can be read from the code. Why a particular decision was made disappears without explicit documentation.

In every ADR, the most important section is the Motivation — not the decision itself. "We use LightGBM" is recoverable from the code. "We use LightGBM because the dataset has high cardinality categoricals, training time is constrained to under 10 minutes, and sklearn compatibility is required for the serving pipeline" is not.

### Rule #3: Every ML system changes when data changes, not just when code changes.

Classical software breaks when code changes. ML systems break silently when data distributions shift — without a single line of code being touched. This is Hyrum's Law applied to data.

Design for this from the start: schema validation at ingestion, feature coverage monitoring, model performance tracking over time. These are not optional monitoring additions — they are architectural requirements.

### Rule #4: Treat Fitness Functions as first-class architecture artifacts.

A Fitness Function is any automated check that verifies an architectural characteristic is still being met. Examples for ML systems:

- CI test: model achieves minimum AUC on holdout set before deployment
- CI test: Docker container starts cleanly with all required env vars
- Alert: feature coverage drops below 80%
- Alert: prediction latency exceeds SLA threshold
- Test: serving pipeline produces identical output to training pipeline for the same input

If a characteristic matters, it needs a Fitness Function. If there is no automated check, the characteristic will eventually be violated silently.

### Rule #5: Guide technology choices, don't specify them.

An architect says "use a sklearn-compatible gradient boosting framework" not "use LightGBM." An architect says "use a Protocol-based client interface to allow provider swapping" not "use OpenAI."

This matters in practice: it keeps options open, forces you to think about what the actual requirement is (sklearn compatibility, provider agnosticism) rather than anchoring on a specific tool.

---

### Rule #6: Prefer breadth over depth when evaluating solutions.

It is more valuable to know that five feature store solutions exist and understand their trade-offs than to be an expert in one. Knowing that Feast, Tecton, Hopsworks, and a simple offline store each make different trade-offs on latency, infrastructure cost, and real-time capability is architectural knowledge. Knowing every configuration option of Feast is engineering knowledge.

When evaluating solutions, always ask: what are the alternatives and what do they trade off? Never present one option as the only option.

### Rule #7: Make trade-offs explicit and structured, not implicit and intuitive.

For every significant architectural decision, produce a trade-off table. Batch training vs. online learning:

| Dimension | Batch | Online |
|---|---|---|
| Reproducibility | High | Low |
| Infrastructure cost | Low | High |
| Model freshness | Hours/days | Minutes |
| Implementation complexity | Low | High |

"It depends" is not an answer. "It depends on X, and for this system X is Y, therefore we choose Z" is an answer.

### Rule #8: Stay hands-on through PoCs and tooling, not through owning critical path code.

An architect who owns the core model training code becomes a bottleneck. The team cannot progress without them; they cannot attend meetings without blocking the team.

Stay hands-on through: proof-of-concept implementations to validate architectural decisions, automation tooling that makes the team more productive (project templates, Makefiles, CI pipelines), and technical debt stories that free the team for feature work.

Your AI_Project_Template_V3 is the correct pattern — it is tooling that multiplies team productivity without putting you in the critical path.

### Rule #9: Avoid the Frozen Caveman Anti-Pattern in ML tooling choices.

An architect who had a bad experience with Kubernetes will route every project away from Kubernetes regardless of fit. An ML engineer who had a bad experience with feature stores will avoid them even when they are the right solution.

When evaluating a technology, separate the question "did this work badly in a previous context" from "does this fit the current context." Past failures are data, not verdicts.

---

### Rule #10: Feature Engineering and Model Serving belong in separate bounded contexts.

They share feature code — this looks like Cohesion. But they have fundamentally different lifecycle and scaling requirements — this is hidden Coupling.

Training runs infrequently, is compute-heavy, is batch-oriented, and tolerates latency. Serving runs continuously, is latency-sensitive, is request-oriented, and must scale horizontally. Putting them in the same module creates a system where you cannot scale serving without dragging training infrastructure along.

Shared feature transformation code lives in a shared library or domain layer. The training context and serving context each import from it — they do not import from each other.

### Rule #11: Never let feature column order be implicit.

Connascence of Position (CoP): a model trained on `[lag_1, lag_2, temp, humidity]` will produce wrong predictions if served features arrive in a different order — silently, without an exception.

Always enforce explicit column ordering in the serving pipeline. Use Pydantic schemas with named fields, not positional DataFrames. Assert column order matches training schema at serving time. This should be a test, not a manual check.

### Rule #12: Never let feature encoding be implicit.

Connascence of Meaning (CoM): if `0` means "missing" and `1` means "present" and that convention exists only in someone's memory, it will eventually be violated.

Use explicit Python Enums or named constants for all categorical encodings. Document encoding conventions in the feature schema, not in comments. Validate at ingestion, not at training time.

### Rule #13: Share transformation code between training and serving pipelines — no exceptions.

Connascence of Algorithm (CoA): if training applies `log1p` normalization and serving applies `log` normalization, you have training-serving skew that will not produce an exception and will not appear in unit tests.

The only safe approach is a single implementation used by both pipelines. sklearn Pipelines serialized with the model artifact are the standard solution. If your serving pipeline reimplements any transformation that also exists in training, that is an architectural defect.

### Rule #14: Diagnose existing systems using the Zone of Pain / Zone of Uselessness heuristic.

Zone of Pain: too much implementation, not enough abstraction. The notebook that became a script that became a 2,000-line monolith. Symptoms: cannot add a feature without touching five files, cannot test without running the full pipeline, cannot swap the model without rewriting the serving layer.

Zone of Uselessness: too much abstraction, not enough implementation. The system with 12 layers of indirection where no one can figure out where to add a new feature. Symptoms: engineers spend more time navigating the architecture than writing business logic.

When joining a project or doing a consulting engagement, diagnose which zone the system is in before proposing solutions. The fix for Pain is different from the fix for Uselessness.

---

### Rule #15: Define Architecture Characteristics before writing code, not after.

The first artifact in any ML project should be a list of the critical Architecture Characteristics with their measurement criteria. Before the first line of code, answer:

- What is the acceptable prediction latency? (Performance)
- How stale can the model be? (Freshness — an ML-specific operational characteristic)
- What is the minimum acceptable model quality? (Quality threshold — trigger for rollback)
- How quickly must the system recover from failure? (Reliability)
- Who needs to audit which decisions, and how? (Auditability)
- What are the data retention and deletion requirements? (Privacy/Legal)

If you cannot answer these questions, you cannot evaluate whether your architecture succeeds.

### Rule #16: Replace "nonfunctional requirements" with "Architecture Characteristics" in every conversation.

"Nonfunctional" implies secondary. "Architecture Characteristics" implies structural and critical.

In stakeholder conversations: "We need to define the Architecture Characteristics that determine whether this system succeeds — specifically freshness, latency, auditability, and reproducibility." This frames the conversation correctly and signals architectural thinking to the client.

### Rule #17: Identify implicit Architecture Characteristics — the ones no one will write down.

No requirements document will ever contain these for an ML system. You must add them:

- Training-serving consistency: the serving pipeline must apply identical transformations to those used during training
- Model reproducibility: the same code and data must produce the same trained artifact
- Data drift detection: the system must detect when input distributions diverge from training distributions
- Feature coverage monitoring: the system must alert when feature availability drops below acceptable levels
- Graceful model degradation: when model quality drops below threshold, the system must fall back to a heuristic or previous model version

These are implicit Architecture Characteristics. Making them explicit — and building Fitness Functions for them — is the specific value an AI Architect adds that a junior engineer will not add.

### Rule #18: Choose the fewest Architecture Characteristics, not the most.

Every characteristic you add increases architectural complexity. A daily batch forecasting tool does not need sub-100ms latency, zero-downtime deployments, and horizontal auto-scaling. A real-time recommendation engine does.

Forcing every project to meet enterprise-grade characteristics for every dimension produces over-engineered systems that are expensive to build and maintain. Match characteristics to actual usage, not to an idealized version of the system.

Anti-pattern: copying the architecture of a high-traffic production system for a proof-of-concept. The PoC does not need the same Reliability and Scalability characteristics. It needs Extensibility and Testability so it can grow into the production system without being thrown away.

### Rule #19: Add ML-specific Architecture Characteristics to every AI system assessment.

The standard "-ilities" from the book cover classical software. ML systems require additional characteristics that have no equivalent in classical software:

- **Model Freshness**: how old can a model be before prediction quality becomes unacceptable? Measured in hours or days, defined by how fast the underlying phenomenon changes.
- **Reproducibility**: given the same code and data, does training produce the same model? Required for debugging, auditing, and regulatory compliance.
- **Observability**: can you see what the model is predicting, on what inputs, with what confidence, in real time? Distinct from application-level logging.
- **Fairness**: does model performance hold across relevant subgroups? Required explicitly in regulated industries, implicitly everywhere.
- **Explainability**: can a prediction be explained to a stakeholder or regulator? Architectural decision — it affects model choice, not just visualization.

For each engagement, decide which of these are critical, which are desirable, and which are out of scope. Document the decision.

### Rule #20: Treat GDPR and data deletion as architectural problems, not legal ones.

"The right to be forgotten" is an architectural requirement for any ML system trained on personal data. If a user requests deletion, can you remove their influence from a trained model?

Today, Machine Unlearning is an unsolved problem. The architectural response is to design for it before it becomes urgent: maintain training data lineage so you know which records influenced which model version, version all model artifacts with their training data manifests, and have a documented policy for what "deletion" means in the context of a trained model.

This is not a legal department problem. It is an architectural decision that must be made before training begins.
