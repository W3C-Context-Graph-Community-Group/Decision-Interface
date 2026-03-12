# Decision Interface Specification

**W3C Context Graphs Community Group**

**Committee Chair:** Dr. Lorien Pratt

---

## Responsibilities

The Decision Interface specification defines the contract between a **coherence measurement** and any **decision model** that acts on it.

Upstream of this specification, the Coherence Protocol evaluates a system boundary and produces a structured uncertainty vector — a compact representation of what is known, what is misaligned, and what has not yet been determined about the data crossing that boundary. The Decision Interface specification defines what any decision model receives from that evaluation, what it must return, and what must be recorded so the decision is auditable and replayable.

The specification is **neutral to decision framework**. Threshold rules, Bayesian inference, utility models, reinforcement learning, human judgment — any of these can sit behind the interface. The committee standardizes the envelope, not the reasoning.

---

## The Problem in One Paragraph

When data crosses a boundary between two independently designed systems, three things can be misaligned: what the data **means**, how it is **structured**, and whether the **context** required to interpret it has been established. The Coherence Protocol measures these facets and expresses the result as a binary vector. But a measurement alone does not produce an action. Something must decide: given this uncertainty profile, do we **proceed**, **ask a clarifying question**, or **stop**? That decision — and the policy that governs it — is what this specification standardizes.

---

## Two Loops, One Interface

The Decision Interface applies at two scales. Both consume the same uncertainty vector. Both return the same action vocabulary. The specification must serve both.

### The Inner Loop: An Agent Making a Decision

An AI agent retrieves a financial filing. The filing contains 27 tagged references to "revenue," each with a different calculation methodology. The agent selects one and builds a dashboard. No error is raised. The number is real. It may not be the number the analyst needed.

In the inner loop, the Decision Interface sits between the coherence measurement (the filing's revenue field has unresolved meaning — 27 candidates, no selection criteria from the user) and the agent's next action. The interface delivers the uncertainty vector to whatever decision logic the agent uses. The decision logic returns one of three protocol actions:

- **Act** — proceed with the selected interpretation.
- **Ask** — surface the ambiguity to the user or to an upstream system.
- **Halt** — the ambiguity is unresolvable within the current scope.

The inner loop is where most boundary crossings happen: API calls, database joins, retrieval-augmented generation, tool use, multi-agent message passing. Each is a boundary. Each needs a decision.

### The Outer Loop: A Decision About the Agent's Decision

A compliance officer reviews the agent's dashboard before it reaches the portfolio team. The coherence measurement at this boundary is different: it evaluates whether the agent's resolution trace — the record of what it checked, what it found, and what it decided at each upstream boundary — meets the organization's policy for that use case.

In the outer loop, the Decision Interface sits between a coherence measurement over a chain of prior decisions and a supervisory decision model. The uncertainty vector now includes not just the current boundary's state but the propagated state from upstream crossings. The decision logic may apply stricter thresholds, require human review for specific facet combinations, or enforce regulatory constraints.

The outer loop is where governance, oversight, and organizational policy enter. It is also where humans most naturally participate — not by inspecting raw data, but by setting the policies that determine when agents must ask versus act, and reviewing the resolution traces that record what happened.

### Why One Interface Serves Both

The inner and outer loops differ in scope, in the decision models behind them, and in who or what is making the decision. They do not differ in interface shape. Both receive an uncertainty vector indexed by facet. Both return a protocol action. Both produce a resolution trace. The specification standardizes this shared surface. What varies — thresholds, policies, reasoning frameworks — is on the other side of the interface, where it belongs.

---

## What the Specification will Define [OPEN]

### Input Surface

The decision model receives:

| Input | Description |
|---|---|
| **Uncertainty vector** | A facet-indexed binary vector representing the coherence state at this boundary. Each facet (meaning, structure, data) contributes 0 (aligned) or 1 (misaligned). A context-completion bit indicates whether the prerequisite resolutions have been recorded. |
| **Boundary metadata** | Identity of the systems on each side of the boundary, the field or claim under evaluation, and the timestamp of the evaluation. |
| **Policy parameters** | Risk tolerance, required confidence thresholds, domain-specific rules, regulatory constraints, or any other parameters the decision model needs. These are supplied by the deploying organization, not by the protocol. |
| **Resolution history** | For outer-loop evaluations: the resolution traces from upstream boundary crossings, expressing what was checked and decided at each prior hop. |

### Output Surface

The decision model returns:

| Output | Description |
|---|---|
| **Protocol action** | Exactly one of: **Act**, **Ask**, or **Halt**. |
| **Action rationale** | A structured record of why the action was selected — which conditions were evaluated, which thresholds applied, which facet states drove the decision. |
| **Trace entry** | A resolution trace record conforming to the Resolution Trace Format, suitable for append to the boundary's Context Graph. |

### What the Specification Does Not Prescribe

- **How the decision model reasons internally.** A lookup table, a neural network, a human reviewing a dashboard — all are valid implementations if they consume the input surface and produce the output surface.
- **What thresholds to use.** Risk tolerance is a policy choice, not a protocol choice. The specification defines where thresholds enter the interface. Organizations set their own values.
- **Which decision framework is correct.** The committee's job is to ensure that any framework — deterministic rules, stochastic models, utility maximization, satisficing heuristics, human judgment — can plug into the same interface without loss of information or auditability.

---

## Decision Model Classes

The specification must accommodate at least the following classes of decision logic. These are not prescriptive categories — they illustrate the range the interface must support.

**Deterministic.** If meaning mismatches, Halt. If context is incomplete, Ask. If all facets align, Act. Rule-based, fully auditable, no probabilistic reasoning. Appropriate for regulatory compliance, safety-critical systems, and environments where the cost of a wrong action far exceeds the cost of asking.

**Threshold-based.** Act if the proportion of aligned facets across a set of fields exceeds a configured threshold. Ask if below threshold but above a halt floor. Halt otherwise. Simple to implement, tunable per use case, interpretable by non-technical stakeholders.

**Probabilistic / Bayesian.** Maintain a posterior over possible interpretations given the uncertainty vector and prior observations. Act when the posterior concentrates sufficiently on a single interpretation. Ask when the expected information gain from a specific question exceeds a threshold. Halt when no available question can reduce uncertainty below the action threshold.

**Utility-based.** Assign costs to acting on a wrong interpretation, to asking (delay, resource use), and to halting (missed opportunity, escalation). Select the action that minimizes expected cost given the uncertainty vector. Appropriate when different kinds of errors carry different consequences.

**Human-in-the-loop.** Route the uncertainty vector and resolution history to a human decision-maker via a dashboard or notification. The human returns a protocol action. The specification treats human judgment as a valid decision model — the interface contract is the same.

---

## Auditability and Replay

Every decision that crosses the interface must produce a trace entry. The trace records the input surface (uncertainty vector, policy parameters, boundary metadata), the output surface (action, rationale), and the identity and version of the decision model that produced it. This enables:

- **Post-hoc audit.** Given a decision outcome, reconstruct exactly what the decision model saw and why it chose the action it chose.
- **Replay.** Given updated policy parameters or a different decision model, re-evaluate the same input surface and compare actions. This is how organizations test policy changes before deploying them.
- **Propagation analysis.** Trace how a decision at one boundary influenced decisions at downstream boundaries, using the resolution history that flows through the outer loop.

---

## Relationship to Other Committees

| Committee | Relationship to Decision Interface |
|---|---|
| **Coherence Protocol** | Produces the uncertainty vector that the Decision Interface consumes. The two specs share the Act / Ask / Halt vocabulary and must agree on the structure of the uncertainty vector. |
| **Syntax & Serialization** | Defines the concrete format (JSON, YAML, etc.) in which the input and output surfaces are expressed. The Decision Interface spec defines the logical contract; Syntax & Serialization defines how it is serialized on the wire. |
| **Semantic Alignment** | Provides the meaning-layer infrastructure (OWL, RDF, SKOS) that populates the meaning facet of the uncertainty vector. The Decision Interface consumes this as input — it does not evaluate semantic alignment directly. |
| **Applied Knowledge** | Bridges the specification to deployment. Validates that the interface contract is implementable in real systems and that the auditability requirements are practical. |
| **Business & Industry** | Sources the use cases and risk-tolerance profiles that drive policy parameter design. Validates that the decision model classes cover real-world needs. |
| **Agentic AI** | The primary consumer of the inner-loop interface. Validates that the spec supports the speed, scale, and autonomy requirements of agent-to-agent and agent-to-system boundary crossings. |

---

## Getting Started

If you are a decision intelligence practitioner, decision scientist, or builder of systems that make or govern automated decisions, here is how to engage:

1. **Read the charter's Decision Interface section** for the committee's scope and authority within the Community Group.
2. **Review the input and output surfaces above.** These are the core of the specification. If your decision framework can consume an uncertainty vector and return a protocol action with a trace, it fits.
3. **Bring a use case.** The most valuable contributions are real boundary-crossing scenarios where the decision — Act, Ask, or Halt — is nontrivial. What made it hard? What information was missing? What would have helped?
4. **Challenge the model classes.** If your decision framework does not fit the classes listed above, that is a signal the specification needs to expand. Tell us what is missing.

---

## Current Status

**Phase:** Pre-draft. The committee is assembling and scoping the first deliverable.

**First deliverable:** A minimal Decision Interface specification defining the input surface, output surface, trace requirements, and at least two reference decision models (one deterministic, one probabilistic) with worked examples.

**Chair:** Dr. Lorien Pratt — [contact via Community Group channels]

**Community Group Chair:** Ron Itelman

---

## Links

- [W3C Context Graphs Community Group](https://www.w3.org/community/context-graphs/)
- [Coherence Protocol Paper (Draft)](./docs/liquid-coherence-draft.pdf)
- [Community Group Charter](./CHARTER.md)
