# Investigation Guidelines

**Purpose:** Define the mandatory standard for writing forward-looking investigation files that gather the evidence needed before planning work for a feature, endpoint, subsystem, workflow, or behavior that does not yet exist or is not yet fully defined.

An investigation is the bridge between:

- a desired outcome
- the current documented and implemented state
- the eventual implementation plan

It exists to eliminate guesswork before planning. A weak investigation produces a speculative plan. A strong investigation produces a plan that can move straight into execution.

---

## 0. Activation Triggers

### 0.1 User-Initiated Triggers

When the user says any of the following, or close variations, the agent must activate investigation mode and begin creating an investigation file.

| Trigger Phrase | Intent |
|----------------|--------|
| "let's open an investigation on X" | Create a new investigation file scoped to X |
| "open an investigation on X" | Same |
| "start an investigation on X" | Same |
| "investigate X" | Same |
| "let's investigate X" | Same |
| "let's look into X" | Same, often loose-scratchpad framing |
| "let's dig into X" | Same |
| "gather info on X" / "gather info about X" | Same, evidence-gathering framing |
| "scratchpad this" / "let's scratchpad X" | Open a loose investigation |
| "what's going on with X" / "let's figure out what's going on with X" | Same, framed as active exploration |

When any of these triggers fire, the agent must, in order:

1. **Confirm the scope target.** Identify what X is and which subsystem it belongs to. If ambiguous, ask a single scoping question. If the repo has per-subsystem `investigations/` folders, pick the one that matches X. If the scope is cross-subsystem or system-wide, use the top-level `investigations/` folder. See §0.3.
2. **Choose a filename.** Format: `{subject}_investigation.md`, snake_case subject, describing the subject concretely. Avoid cute prefixes. Examples: `settings_reload_investigation.md`, `auth_drift_state_machine_investigation.md`, `ingress_rate_limiting_investigation.md`.
3. **Create a minimal starting scaffold.** The file at birth should contain at least a one-line problem statement and an empty findings area. It does NOT need to be well-organized at the first keystroke. The scaffold is a placeholder the investigation grows into.
4. **Begin gathering.** The first pass is allowed to be loose. Record what is observed, where it was observed, and what questions it raises. Use concrete references (file paths, function names, config keys, SQL names) whenever possible per §6. Do not prematurely classify findings into authority / current-implementation / gap / plan-decision / open-question (§3). Classification happens when the investigation matures toward the completion standard in §12.
5. **Persist early.** Write to disk before the notes feel "finished." The value of the `investigations/` folder is that rough work survives beyond a chat session.

### 0.2 What the Trigger Does NOT Require

The trigger does not demand that the investigation be complete, organized, or even coherent at start-time. It opens a file and begins capture. The completion standards in §12 describe what "ready to feed planning" looks like; that is a later state, not an entry requirement.

Common mistake on entry: treating a trigger as a demand to produce a finished investigation in one response. The correct initial action is to create the file with a thin scaffold, make the first few concrete entries, and continue from there.

### 0.3 Scope and Subsystem Placement

If the repo uses per-subsystem folders (`central-server/investigations/`, `stations/{service}/investigations/`, `tooling/{tool}/investigations/`, etc.), drop the file under the folder that matches the subject. If the scope is cross-subsystem or genuinely system-wide, use the top-level `investigations/` folder.

Never create a new `investigations/` folder without first checking that the target subsystem doesn't already own one. Folder proliferation is a sign the agent hasn't surveyed the repo.

### 0.4 Minimum Starting Scaffold

The first write to a freshly opened investigation should look roughly like:

```markdown
# {Subject} Investigation

**Status:** Open
**Opened:** YYYY-MM-DD

## Problem Statement

{One or two sentences: what are we investigating and why now.}

## Findings

{Empty. First findings go here as they come in — file paths, observations, questions.}

## Open Questions

{Empty. Questions accumulate here as they surface.}
```

This is not the final shape of the document; §3–§12 describe what the mature form looks like. The scaffold is the minimum the agent must produce to open the file successfully. Everything else accumulates through the normal work of the investigation.

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

### 4.9 Closing Summary

End the investigation with a section that makes the next step straightforward — whatever that next step turns out to be (§13 enumerates the options).

At minimum, summarize:

- what is already decided by authority
- what current implementation already provides
- what gap, if any, must be closed
- what decisions the next artifact (plan, documentation, or edit) must make
- what questions still need owner input

This section is what a reader — or a planner, or a documentation author, or a future self returning to an archived investigation — uses to orient without re-reading the whole file.

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
- the closing summary is clean and points at a concrete next step

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

## 13. Concluding an Investigation: Resolving the Next Step

An investigation concludes by **resolving what happens next** — not by handing off to a generic "next agent." At conclusion, one of the following paths is chosen and acted on.

### 13.1 Resolution Paths

| Path | When to Choose | Action |
|------|----------------|--------|
| **Write new documentation** | The investigation established stable, authoritative understanding of a subsystem, feature, or behavior that does not yet exist in `documentation/`. | Create the doc per `general_documentation_crafting_guidelines.md`. The investigation feeds it. |
| **Craft a plan** | The investigation identified a concrete implementation gap and the decisions needed to close it are clear or well-scoped. | Create the plan per `general_plan_crafting_guidelines.md`. The investigation feeds it. |
| **Update existing documentation** | The investigation found that existing canonical documentation is incorrect, incomplete, or out of date — but the overall subsystem already has a canonical doc. | Edit the existing doc. Reference the investigation in the commit message. |
| **Archive with findings summary** | The investigation answered its question but the answer is not action-inducing (confirming correct behavior, closing off a hypothesis, capturing state for future reference, or ruling out a concern). | Prepend a Findings Summary (see §13.2) to the top of the investigation, then move the file to `archive/investigations/`. |
| **Split into multiple** | The investigation grew into multiple distinct concerns that deserve separate treatment. | Carve new investigation files out of the original, each with its own scope. The original becomes an index of pointers, or is archived with a summary. |

The paths are not mutually exclusive. A single investigation commonly produces both a documentation update *and* a plan. In that case, choose the order (usually documentation first, then plan against it), execute both, and record both in the findings summary.

### 13.2 The Findings Summary

Every concluded investigation — regardless of which resolution path is taken — must have a **Findings Summary** block at or near the top of the file. The summary states:

- What was investigated
- What was concluded
- Which resolution path was chosen
- Pointer to any artifact the investigation produced (plan file path, documentation file path, edit commit hash, or "archived — no action taken")

A reader returning to the file in six months should understand the investigation's conclusion from the summary alone, without re-reading the evidence below it. The summary's job is to make the file's evidence optional for the casual reader and retrievable for the investigator.

### 13.3 Criteria for Choosing a Path

The resolution path is not a free choice. Use these criteria, in order:

1. **Is the main need to formalize behavior, contracts, or rules that should become standing repo guidance before implementation?** → Write new documentation, or update existing documentation if the relevant canonical doc is already in place.
2. **Is the main need to resolve implementation or design decisions and drive execution?** → Craft a plan.
3. **Do 1 and 2 both apply?** → Documentation first, then plan against it. A plan that precedes its canonical doc tends to drift from canon and then silently redefine it.
4. **Neither 1 nor 2 applies?** → Archive with a findings summary. This is a valid and common outcome; many investigations conclude in "we now know this, but no change is needed."
5. **Is the investigation covering multiple distinct concerns?** → Split before applying 1–4 to each resulting file.

Never close an investigation without a resolution. "The investigation is complete" is not a conclusion on its own — it must be paired with one of the paths above.

### 13.4 Completion Bar for Each Path

The completion standard in §12 describes the strictest bar: ready to feed a plan. The paths in §13.1 have different bars:

- **Write/update documentation** and **craft a plan** paths → meet the full §12 standard before proceeding.
- **Archive with findings summary** path → the bar is lower. What is required is that the findings summary accurately states what was concluded and why no further action is needed. The file's evidence does not need the full §12 rigor; an archived investigation is a historical record, not a planning input.
- **Split into multiple** path → each resulting investigation is evaluated against §12 for its own chosen resolution path.

### 13.5 Who Decides the Path

The resolution path is a judgment call. The investigating agent must propose a path with **explicit reasoning**: state which §13.3 criterion drove the choice and, if not obvious, why the other paths were rejected. The user confirms or redirects before the path is executed. Do not default to a plan or documentation file silently — the "archive with findings summary" outcome is too common to skip over.

### 13.6 Executing the Chosen Path

Once the user confirms the path, the agent follows the requirements below before, during, and after producing the resulting artifact.

**Before writing:**

- Read the project's rules files (e.g. `CLAUDE.md` and any `.claude/rules/*.md`) and follow them.
- Read the crafting guidelines for the chosen path:
  - Writing new documentation or updating existing documentation → `general_documentation_crafting_guidelines.md`
  - Crafting a plan → `general_plan_crafting_guidelines.md`
- Read every canonical document the investigation references. Treat those as the source of truth for any fixed behavior, contract, threshold, deployment convention, or authority boundary identified in the investigation. **Never silently change canonical behavior.** If the investigation found that canon itself needs to change, surface that as an explicit decision requiring owner approval *before* the new artifact is written.
- If the path is "update existing documentation," read the existing doc end-to-end before editing. Do not patch a doc without first understanding what it currently says.

**While writing:**

Preserve the investigation's most important structural property in the resulting artifact: **keep the evidence layers cleanly separated.** The artifact must make it obvious which claims are:

- already canonically specified
- what the current system already implements
- what gap remains (if any)
- what decisions are still plan-level rather than canonical

If the artifact proposes structure, contracts, or details beyond what the canonical docs establish, label those proposals explicitly as plan-level decisions. Do not let them read as canon.

The artifact must be concrete enough for implementation (or, for documentation, operational use) to proceed without ambiguity, and it must match repo style and the relevant crafting guidelines.

**After writing:**

Report back to the user in the same response, stating:

- which file was created or updated (path)
- which §13.1 path was taken and why (referencing §13.3)
- the key decisions captured in the new artifact
- any open questions that still require owner input

---

**Last Updated:** 2026-04-18
**Status:** Complete
**Owner:** MarNexii Platform Team
