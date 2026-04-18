# Investigation Guidelines

**Purpose:** Define the mandatory standard for writing forward-looking investigation files that gather the evidence needed before planning work for a feature, endpoint, subsystem, workflow, or behavior that does not yet exist or is not yet fully defined.

An investigation is the bridge between:

- a desired outcome
- the current documented and implemented state
- the eventual implementation plan

It exists to eliminate guesswork before planning. A weak investigation produces a speculative plan. A strong investigation produces a plan that can move straight into execution.

---

## 1. What an Investigation Is

An investigation is a scoped technical discovery document that:

- defines the exact problem or desired end state
- establishes the authoritative sources for the domain
- maps the current state of the relevant code and documentation
- identifies what already exists and can be reused
- identifies what is missing
- distinguishes hard constraints from plan-level choices
- surfaces risks, dependencies, and open questions
- provides enough evidence that a plan can be written without invention

An investigation is NOT:

- a canonical specification
- a step-by-step implementation plan
- a vague brainstorm
- a design doc that silently declares new requirements without evidence

The investigation's job is to gather and organize the information needed to make good planning decisions.

---

## 2. Operating Principle

**An investigation must reduce planning uncertainty.**

Every section should answer one of these:

1. What do we know from authoritative sources?
2. What does the current system already do?
3. What gap exists between current state and desired state?
4. What constraints limit the solution space?
5. What choices remain open for planning to resolve?

If a section does not reduce uncertainty, it probably does not belong in the investigation.

---

## 3. Required Categories

Every substantive claim in an investigation must fit into one of these categories and should be labeled clearly by wording or section structure.

### 3.1 Authority / Canonical Facts

These are facts from authoritative documentation, approved plans, external standards, or owner-provided requirements.

Use these to establish:

- fixed behavioral rules
- fixed API or data contracts
- fixed threshold or policy rules
- fixed deployment or operational conventions
- naming, ownership, and architectural boundaries

### 3.2 Current Implementation Facts

These are facts about what the codebase or deployed system currently does.

Use these to establish:

- current entry points
- current handlers, services, and data paths
- existing database columns, tables, or stored data shapes
- existing tests, logs, metrics, and deployment behaviors
- partial implementations that may already solve part of the problem

### 3.3 Gap Findings

These are the real mismatches between:

- desired end state and current implementation
- canonical requirements and current implementation
- operational need and current observability/testing/deployment support

Gap findings are the reason the investigation exists.

### 3.4 Plan-Level Decisions To Be Made

These are not yet canonical and not yet implemented. They are the choices the plan will need to resolve.

Examples:

- internal JSON structure
- endpoint split or route naming
- whether to use array vs map storage
- retention cap choice
- handler/service ownership
- response contract shape

### 3.5 Open Questions

These are unresolved items that cannot be answered from the repo alone and need owner or stakeholder input.

Open questions are valid. Hidden questions are not.

---

## 4. What a Good Investigation Must Establish

### 4.1 Problem Statement

State clearly:

- what is being investigated
- why the investigation exists now
- what desired end state or capability is driving it
- what is explicitly out of scope

If the desired end state is known, state it plainly.

Bad:

- "Investigate drift JSON"

Good:

- "Investigate what must change so MOLID drift state can be discovered through a dual-path API: a fast live-state endpoint and a full forensic-detail endpoint backed by persisted JSONB history."

### 4.2 Authority Sources

List the files, documents, specs, or owner directives that define the non-negotiable parts of the domain.

For each source, say why it matters.

The investigation should make it obvious:

- what is immutable without approval
- what is merely current behavior
- what is still open to design

### 4.3 Current State Mapping

Map the current implementation with enough specificity that a planner does not need to rediscover the terrain.

Include:

- relevant files and functions
- key data structures
- persistence touchpoints
- endpoint handlers and route registration points
- related tests
- deployment/runtime touchpoints if relevant

This section should identify what already exists and what can be reused.

### 4.4 Gap Identification

Identify the real gap, not cosmetic differences.

A good gap statement has this form:

- canon / desired outcome says X
- current code does Y
- therefore the missing capability is Z

This is the core output of the investigation.

### 4.5 Constraints

List the things the eventual plan cannot casually change.

Examples:

- canonical algorithm must remain unchanged
- no schema changes allowed
- existing endpoint must remain fast
- deployment must follow repo conventions
- output must remain bounded

Constraints narrow the valid solution space and prevent bad plans.

### 4.6 Reuse Opportunities

Call out:

- existing structs that can be extended
- handlers that can be mirrored
- tests that establish a pattern
- existing JSON fields or DB columns that should be reused
- existing logging/auth/deployment patterns that should be followed

An investigation should not treat the system as greenfield if relevant pieces already exist.

### 4.7 Risks and Failure Modes

List the risks a plan must account for.

Examples:

- hidden consumer dependence on current JSON shape
- pathologically large forensic payloads
- expensive DB reads on hot paths
- auth drift between similar endpoints
- documentation and Swagger divergence
- null handling ambiguity

This section is especially important for anything with persistence, contracts, or operational impact.

### 4.8 Open Planning Decisions

Separate true facts from plan-level choices the investigation is teeing up.

Examples:

- exact JSON layout
- exact response contract
- retention bound
- where canonical comparison values should come from
- which subsystem owns the orchestration logic

These should be phrased as decisions to resolve, not disguised as settled fact.

### 4.9 Plan Handoff

End the investigation with a section that makes planning straightforward.

At minimum, summarize:

- what is already decided by authority
- what current implementation already provides
- what gap must be closed
- what the next plan must decide
- what questions still need owner input

The investigation should hand planning a clean starting point.

---

## 5. Questions Every Investigation Must Answer

Before an investigation can be considered complete, it must answer these:

1. What exactly are we trying to enable, change, or learn?
2. What authoritative sources define the fixed rules?
3. What does the current code already do in this area?
4. Which parts of the current system are relevant to the future change?
5. What is the actual implementation gap?
6. What constraints limit the solution space?
7. What can be reused instead of rebuilt?
8. What risks or second-order effects should planning account for?
9. Which decisions are still open?
10. What specifically should the implementation plan pick up next?

If any of these remain vague, the investigation is not ready to feed planning.

---

## 6. Evidence Standards

An investigation must be evidence-based.

### 6.1 Required Evidence Types

Use concrete references wherever possible:

- documentation sections
- file paths
- function names
- route registrations
- SQL column/table names
- request/response structures
- test file names
- deployment/runtime locations when relevant

### 6.2 What Counts as Weak Evidence

Avoid unsupported statements like:

- "the system probably does..."
- "this seems to be..."
- "it likely would be best if..."

If you infer, say it is an inference and explain the basis.

### 6.3 What Counts as Strong Evidence

Strong evidence looks like:

- "`GetDriftState` returns only live cache fields in `./central-server/api/handler/drift.go`"
- "`WriteMilestoneSnapshot` overwrites the JSON root in `./central-server/api/service/driftcache.go`"
- "§12.3 names the forensic JSON keys but does not define the nested object shapes"

The investigation should let a reader verify its claims quickly.

---

## 7. Scope Discipline

### 7.1 Good Investigation Scope

A good investigation:

- explores enough adjacent context to avoid bad planning
- stays centered on the target capability
- distinguishes required context from optional future work

### 7.2 Bad Investigation Scope

Bad scope includes:

- rewriting canonical behavior
- solving implementation details prematurely
- turning into a step-by-step plan
- mixing unrelated subsystem redesign into the same document
- hiding major decisions in passing prose

---

## 8. What Belongs in Planning, Not Investigation

An investigation should not fully expand into planning.

The following usually belong in the plan, not the investigation:

- atomic action checklists
- exact sequencing of edits
- per-file edit order
- execution verification commands
- deployment runbooks
- task decomposition

The investigation may point toward these, but should not try to become the plan.

---

## 9. When an Investigation Is Strong

A strong investigation has these properties:

- a reader can understand the target outcome on first read
- authoritative constraints are obvious
- the current implementation map is concrete and trustworthy
- the real gap is clearly identified
- open decisions are explicit
- the plan handoff is clean

After reading it, a planner should not need to rediscover:

- where the work lives
- what must not change
- what already exists
- what the hard problems are

---

## 10. Common Failure Modes

### 10.1 False certainty

The investigation presents a preference as if it were already decided.

### 10.2 Fake completeness

The investigation sounds polished but does not actually identify the real gap or missing constraints.

### 10.3 Rediscovery burden

The investigation names a problem but leaves the planner to find all relevant files, contracts, and integration points from scratch.

### 10.4 Scope bleed

The investigation drifts into implementation planning or canonical redesign.

### 10.5 Hidden unknowns

Important owner decisions are buried or omitted instead of being surfaced as explicit open questions.

---

## 11. Relationship to Other Artifacts

| Artifact Type | Primary Purpose | Review Standard |
|---------------|-----------------|----------------|
| Investigation | Gather facts, map current state, identify gap, surface decisions | This document |
| Documentation | Define stable system understanding or operating instructions | `general_documentation_crafting_guidelines.md` |
| Plan | Define execution-ready implementation steps | `general_plan_crafting_guidelines.md` |

The normal flow is:

1. investigate
2. document or clarify canon if needed
3. plan
4. implement

Not every change needs a separate documentation artifact before planning, but every non-trivial plan should be grounded in investigation-quality understanding.

---

## 12. Completion Standard

An investigation is ready to feed planning when:

- the target outcome is explicit
- authority sources are identified
- current state is mapped concretely
- the real implementation gap is stated clearly
- constraints are listed
- reuse opportunities are identified
- risks are surfaced
- open decisions are explicit
- the next plan can be written without major rediscovery

If a planner would still need to ask "what exactly are we building?", "what already exists?", or "what is actually fixed vs open?", the investigation is not complete.

---

## 13. Post-Signoff Handoff

When an investigation is found complete enough to feed planning, the reviewing agent must do all of the following in the same response:

1. State explicitly that the investigation is ready to feed planning.
2. State any non-blocking observations that remain.
3. Provide the standard initiation prompt for the next agent without waiting to be asked.

The handoff prompt must instruct the next agent to:

- read the investigation as the starting point
- decide whether the correct next artifact is a plan or a documentation file
- follow `documentation/general_plan_crafting_guidelines.md` if producing a plan
- follow `documentation/general_documentation_crafting_guidelines.md` if producing documentation
- create the file in the correct repo location
- summarize which file was created, why that artifact type was chosen, and any open questions still requiring owner decision

A signoff without the handoff prompt is incomplete.

### Standard Initiation Prompt

```text
Read the investigation at `<path-to-investigation>` and use it as the starting point for the next authoritative implementation artifact.

Your task is to decide whether the correct next artifact is:
1. a plan file, following `documentation/general_plan_crafting_guidelines.md`, or
2. a documentation file, following `documentation/general_documentation_crafting_guidelines.md`.

Decision rule:
- Choose a plan if the main need is to resolve implementation/design decisions and drive execution.
- Choose a documentation file if the main need is to formalize behavior/specification that should become standing repo guidance before implementation.
- State explicitly why you chose one or the other.

Requirements:
- Before writing anything, read and follow:
  - `CLAUDE.md`
  - `.claude/rules/project_instructions.md`
- If producing a plan, read and follow:
  - `documentation/general_plan_crafting_guidelines.md`
- If producing documentation, read and follow:
  - `documentation/general_documentation_crafting_guidelines.md`
- Read all canonically relevant docs referenced by the investigation.
- Treat the canonical documentation as the source of truth for any fixed behavior, contract, threshold, deployment convention, or authority boundary identified by the investigation.
- Do not silently change canonical behavior.
- Make the resulting artifact clearly separate:
  - what is already canonically specified
  - what is currently implemented
  - what gaps remain
  - what decisions are still plan-level rather than canonical
- If you propose structure or contract details beyond the canonical docs, label them explicitly as implementation/plan-level decisions, not canon.
- The result must be concrete enough for implementation to proceed without ambiguity and must match repo style and crafting guidance.

Deliverable:
- Create the appropriate file in the correct repo location.
- In your final response, summarize:
  - which file you created
  - why that artifact type was chosen
  - the key implementation decisions captured
  - any open questions that still require owner decision
```

When using the standard prompt, replace `<path-to-investigation>` with the actual repository-relative path of the signed-off investigation file.

---

**Last Updated:** 2026-03-21
**Status:** Complete
**Owner:** MarNexii Platform Team
