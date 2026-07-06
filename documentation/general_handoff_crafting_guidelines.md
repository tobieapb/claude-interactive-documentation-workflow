# Handoff Crafting Guidelines: The Session-Bridge Standard

**Version:** 1.1.0
**Status:** In Review
**Last Updated:** 2026-06-15
**Owner:** MarNexii Platform Team
**Purpose:** This document is the single, self-contained, mandatory standard for writing a session handoff: the artifact that lets a fresh agent or session resume in-flight work losing nothing material. It is the peer of `general_plan_crafting_guidelines.md` (how to write a plan) and `general_documentation_crafting_guidelines.md` (how to write documentation). It governs handoffs only.

---

## Compliance Mandate

**A handoff that makes the next session re-derive a settled decision, re-read what was already read, or act on a claim that was never true has failed.** The measure is binary:

> Could a competent agent, holding this file plus the repository, resume the work and reach the correct next action without asking the prior session anything, and without repeating an assumption the prior session already corrected?

If the answer is no, the handoff is a draft, not a handoff. Every handoff must be evaluated against the self-review gate in Section 5 before it is delivered.

---

## 0. LLM/Agent Directive (Machine-Readable Instructions)

**This section is for the agent writing the handoff.**

### 0.1 The File Artifact Mandate

The deliverable is a FILE, named per Section 7, written to disk before the session is declared handed off. Internal reasoning, chat summaries, and scratchpad state are not a handoff. Whatever planning capability your runtime provides, the output of this process is the file.

### 0.2 Mandatory Pre-Reading Protocol

Before writing any load-bearing claim, read directly:

1. This document.
2. The work's own canonical artifacts: its plan in `plans/`, its governing documentation in `documentation/`, and the relevant source.
3. The primary source behind every load-bearing claim (Section 3.1 defines the calibration).

Reading via a subagent digest, or from your own earlier impression, does NOT satisfy a load-bearing claim. If you have only a subagent summary for a fact you intend to state as VERIFIED, read the source yourself first or label the claim DERIVED, PROPOSED, or OPEN per Section 3.2.

### 0.3 When to Write a Handoff

Write one when any of these holds:

| Trigger | Reason |
|---|---|
| The context window is under pressure and work is incomplete | The conversation will be summarized or reset; chat-only context is about to be lost |
| A working session ends with open work | A later session must resume from the exact point |
| A deliberate context reset is planned | The new context starts blind without the bridge |
| The user asks for a handoff | Explicit request |

### 0.4 Summary Checklist for the Agent

Before proceeding, confirm:

- [ ] I will write a file named per Section 7.
- [ ] I read the work's plan and governing docs directly, not via a digest.
- [ ] Every load-bearing claim traces to a primary source I read myself, or is labeled per the four registers.
- [ ] Every blanket assertion records its cross-product variant (Section 3.4).
- [ ] Every not-yet-made decision is an explicit OPEN item, not a guessed answer.
- [ ] I ran the Section 5 self-review gate.

---

## 1. Core Principles (Non-Negotiable)

### 1.1 A Handoff Is a Bridge, Not a Source of Truth

Canonical content lives in the plan (`plans/`), the documentation (`documentation/`), the schema (`er_model/`), and the code. A handoff carries the CHAT-ONLY residue: decisions reached in conversation, clarifications the user gave, and findings discovered, none of which is committed anywhere yet. The handoff carries that residue forward and points at the committed artifacts for everything else. It must never become a second, drifting source of truth. As each chat-only decision lands in its canonical home, the handoff cites that home.

### 1.2 Lose Nothing Material

Material means anything the next session would otherwise have to rediscover, re-decide, or get wrong. Every decision, clarification, correction, and finding from the session that is not yet in a committed artifact belongs in the handoff, with its provenance (Section 3.8).

### 1.3 Grounded, Read Directly, Calibrated

Every load-bearing claim is read from the primary source by the agent writing the handoff, at a depth calibrated to relevance (Section 3.1). The root cause of weak handoffs is writing on second-hand summaries.

### 1.4 Every Claim Is Tagged and Traceable

Each statement is exactly one of VERIFIED, DERIVED, PROPOSED, or OPEN (Section 3.2), and a VERIFIED or DERIVED claim carries its citation (Section 3.3). A claim that fits none of the four registers does not belong in the handoff.

### 1.5 Convert Unknowns to Explicit OPEN Items

A handoff cannot record a decision that has not been made. When a question is unresolved, state it as an OPEN item with the concrete options and what is required to settle it. Never paper over an undecided question with a confident wrong answer.

### 1.6 Calibration: a Bridge, Not an Encyclopedia

Favor content that makes the next session resume correctly with less guessing. Cut content that only adds length. Completeness is measured by "did the next session lose anything material," not by word count. Over-stuffing a handoff with re-stated committed facts buries the chat-only residue that is the handoff's actual job.

---

## 2. The Two Failure Modes (The Conceptual Core)

Every handoff gap is one of two kinds. The discipline that prevents each is different, and one of them cannot be fully eliminated.

| Failure mode | What it is | What cures it | Can it be eliminated? |
|---|---|---|---|
| Preventable-at-write-time | The fact already existed in a primary source the author did not read directly | Grounding (3.1), collision and prior-art sweep (3.6), drift check (3.7) | Yes, by direct reading and the sweeps |
| Not-yet-decided | The decision had not been made when the handoff was written | Force the question to surface as an explicit OPEN item with options (1.5, 3.2) | No, only converted from silent-wrong to explicit-open |

The practical consequence: discipline removes class one entirely, and class two is reduced to an honest open question instead of a confident wrong assertion. A handoff that asserts an answer to a question the session never actually settled is the most damaging defect, because the next session inherits the error as fact.

---

## 3. The Disciplines

### 3.1 Grounded Reading, Calibrated by Relevance

Read every load-bearing source directly. Calibrate depth so the reading is thorough where it matters and not wasteful where it does not:

| Source kind | Required depth |
|---|---|
| A document that directly governs the work (its plan, its primary spec) | Read in full |
| A large catalog or reference document (an endpoint reference, a scope catalog) | Read the conventions sections plus the sections directly relevant to the work, and disclose that triage in the handoff |
| The immutable schema (`er_model/sql_marnexii_station_er_model.sql`) | Targeted grep for the relevant tables, columns, constraints, and seed rows, then a range-read around each match |

A subagent digest or a recollection from earlier in the conversation never satisfies a load-bearing claim. "Read everything in full" is rejected as a mandate an author will quietly skip; relevance-calibrated reading with the triage disclosed is the standard.

### 3.2 The Four-Register Tagging Rule

Every statement is exactly one register. Never blur them.

| Register | Meaning | What it requires |
|---|---|---|
| VERIFIED | A fact cited to a primary source | A citation by file and line range or section (3.3) |
| DERIVED | A conclusion inferred from VERIFIED premises | A citation to the premises it follows from |
| PROPOSED | The author's recommendation, not yet approved | A note that it is unapproved |
| OPEN | A decision that belongs to the Project Owner | The concrete options and what settles it |

DERIVED exists because the most valuable handoff content is often a sound conclusion across cited facts (for example: "a session ceiling can only narrow and the new scope is in no role, therefore granting it needs a new seeded role"). That is not VERIFIED (no single line states it), not PROPOSED (it is not a preference), and not OPEN (it is not a decision). It is DERIVED, and it cites its premises.

### 3.3 The Citation Rule, Including Absence

A positive fact cites the exact source: a file and line range (`datamodel_locations_documentation.md:92-113`) or a section (`auth doc §2.6.5`).

A negative or absence claim (something is undocumented, missing, deleted, moved, or unused) cites its SEARCH as evidence, not a line: the grep or history-pickaxe command and its result. If the search contradicts the claim, retract it and state the truth. Example of a compliant absence claim:

> The `team_assignment_permissions` JSON shape is not canonicalized in any document. Evidence: `grep -ril "team_assignment_permissions" documentation/` returns only incidental mentions; `git log --all -S "team_assignment_permissions"` shows no doc that defined and then removed it.

### 3.4 Exhaustiveness by Cross-Product

When the work has a model with interacting dimensions, enumerate the full cross-product and fill every cell. Do not leave a cell implied. Forbid a blanket assertion ("X only", "always", "never") for any cell unless the cross-cutting variant for that cell is recorded.

The canonical worked example is an authorization actor-by-verb matrix. For a user-owned resource, the actors are owner, same-customer peer (reached by publish), team member (reached by team-share), customer-admin, and public scope; the verbs are list, read, create, update, delete, reassign, share, and unshare. For each cell, record the base capability, any elevated cross-user variant, any delegated team-permission action, whether it is default-granted, and how it is revoked. This is exactly the discipline that turns a wrong blanket ("share is owner only") into a correct or explicitly-open cell ("self-share is the base scope `share:zone`, default-granted, revoked by narrowing the assignment ceiling; cross-user share is the elevated `share:customer_zone`").

The technique generalizes to other grids: state by transition, input by failure-mode, role by resource. Whenever the work has two or more interacting axes, build the grid.

### 3.5 Precondition and Grant Resolution

For every new capability, scope, flag, role, route, or dependency the work introduces, resolve HOW it is obtained or enabled by applying the system's actual resolution rules, not by assertion. Where the system has a resolver, apply it and state the consequence. Example: the central-api session resolver computes `role_union INTERSECT assignment_ceiling INTERSECT app_ceiling` and ceilings only narrow (`auth doc §2.6.5, §4.3`), so a new base scope must be granted by a seeded role; a ceiling cannot add it, only remove it. State that consequence explicitly rather than writing "the user gets the scope."

This applies equally to preconditions the next step CONSUMES, not only capabilities the work introduces. For every existing scope, endpoint, flag, or dependency the next step relies on, verify its CURRENT state by the system's resolution rules and a live or source check, citing it: is the scope actually granted to the operator now (resolve it from the session, not the role name), does the endpoint actually return the fields the next step needs now, is the flag actually on. "The tool consumes `read:zone`" is not resolution; "the current operator session resolves a zone scope, confirmed against `active_sessions.session_scope_resolved`" is.

### 3.6 Collision and Prior-Art Sweep

For every new name, identifier, scope string, route path, column, key, or constant the work introduces, grep the relevant catalogs and references for any existing or planned use of the same token or surface. Record every hit. Reconcile each overlap against the project's anti-patterns, distinguishing a legitimate shared surface from a different-semantics collision that must not collapse onto one name. Example: a proposed `list:zones` user scope collides with a planned station-side `read:zones` on the `/api/v1/zones` surface (`scopes doc §9`), and the two carry different semantics, so they cannot share one string.

The same sweep runs in the positive direction: for every capability the next step will BUILD, search for an existing mechanism, pattern, or asset of the same shape that it should REUSE rather than reinvent (an existing tool of the same kind, an existing gating hook, an existing control style, an existing store action), and record the reuse target with its citation. A handoff that sends the next session to reinvent a hook the codebase already has (for example a per-tool `requiredScope` filter that already gates the rail) has failed the prior-art half of this sweep.

### 3.7 Drift Check

Compare the committed artifacts (the plan, the canonical docs, the git state) against every decision reached only in conversation. Flag every divergence, including bookkeeping drift (status blocks, pass markers, version numbers, dates), each with the exact reconciliation step. Name the canonical home each chat-only decision must land in, and state that it is not there yet.

### 3.8 Provenance for Chat-Only Decisions

Each chat-only decision records four things: what was decided, the rationale, who decided it and when, and whether it is now in a committed artifact or still chat-only. Provenance is what lets the next session trust the decision and know where it has to be written.

### 3.9 Forward Dependency Resolution (the launch-handoff discipline)

These guidelines otherwise assume the author built the work being handed off, so the surrounding system is already understood. A **launch handoff** is different: its purpose is to start a next step the session did NOT build, and the substrate that next step will extend may be unread. For a launch handoff, map the next step's **dependency surface** and verify each dependency's current state by direct reading or search, never by assumption:

- **Exists.** Does the code, wiring, store action, layer, or asset the next step builds on actually exist? Grep for the real consumer or implementation, not the document's description of it.
- **Granted.** Is every scope, flag, or permission the next step needs actually held by the operator now (resolve it per Section 3.5, from the session, not the role name)?
- **Returns what is needed.** Does every endpoint or projection the next step consumes actually return the fields it requires now?
- **Prior art to reuse.** Is there an existing mechanism of the same shape to reuse (Section 3.6, positive direction)?

The highest-risk class is a dependency the DOCS describe as if it were built while the CODE never wired it (documented-but-unbuilt). It is the most dangerous because the next session reads the doc, assumes the mechanism is there, and builds on a void. The cure is the same grep: confirm the consumer or implementation exists in code, and if it does not, say so as the load-bearing fact ("`mapMode` is set but no consumer reads it; the next step is the first to wire it"). The output of this discipline lands in Section 5 (verified substrate, including the negative facts) and in Section 1 (the next action names the dependency it must build first). A launch handoff that omits this is not a handoff; it is a wish.

---

## 4. Mandatory Handoff Structure

Every handoff contains these sections, in this order. Omitting a section invalidates the handoff unless the section is marked "none" with a reason.

```markdown
# Handoff: <work being handed off>

<one paragraph: repo, branch, date, and what this handoff is>

## 1. Resume point and the single next action
<where we are, and the one exact next action>

## 2. The settled model
<every claim tagged VERIFIED / DERIVED / PROPOSED / OPEN and cited>

## 3. Chat-only decisions (not yet committed)
<each: what, why, who and when, committed-or-chat-only, canonical home>

## 4. Open gating decisions
<each: the concrete proposal where one exists, and what settles it>

## 5. Verified facts about the substrate
<schema, contracts, current state, by targeted citation>

## 6. Documented-vs-gap
<what is genuinely missing, with search evidence, and the file that must hold it>

## 7. Required reading
<the precise sections that matter, and which were read directly this session>

## 8. Workflow conventions and project rules that govern this work

## 9. The owner's working style

## 10. Ready-to-paste first message for the fresh session
```

Each section carries the discipline of Section 3: Section 2 obeys the four registers and citation rule, Section 4 obeys 1.5, Section 6 obeys the absence-citation rule, and a model with interacting axes is rendered as a cross-product grid per 3.4.

---

## 5. Pre-Delivery Self-Review Gate

Run this before declaring the handoff done. It is binary: any unchecked box means the handoff is not finished.

- [ ] Every sentence is VERIFIED (with a resolvable citation), DERIVED (with cited premises), PROPOSED, or OPEN. No bare assertions.
- [ ] A sample of citations was spot-verified to resolve to the claimed content. Presence of a citation is not correctness of a citation.
- [ ] Every absence claim carries its search command and result.
- [ ] Every blanket assertion ("only", "always", "never") has its cross-product cell variant recorded.
- [ ] Every new name, scope, route, or column carries the result of its collision sweep.
- [ ] Every new capability states how it is granted or enabled, by applying the system's resolution rules.
- [ ] For a launch handoff (Section 3.9): every dependency the next step builds on is verified by a cited search to EXIST, be GRANTED to the operator, and RETURN what is needed, with any documented-but-unbuilt mechanism stated as the negative fact rather than assumed present.
- [ ] The drift list is reconciled, or each item carries its reconciliation step.
- [ ] No question the session did not actually settle is stated as an answer.
- [ ] No em dashes and no en dashes anywhere in the file (`.claude/rules/no_em_or_en_dashes.md`).
- [ ] The structure matches Section 4.

---

## 6. Forbidden Patterns (Auto-Invalid)

A handoff containing any of these is invalid until corrected:

| Pattern | Why it is forbidden |
|---|---|
| A load-bearing fact stated as VERIFIED but sourced from a subagent digest or memory | It is the root cause of class-one gaps (Section 2) |
| A blanket assertion ("owner only", "always", "never") with no cross-product variant recorded | It hides the cell that is actually different (Section 3.4) |
| An answer to a question the session never settled | It converts an OPEN item into an inherited error (Section 1.5) |
| An absence claim with no search evidence | It cannot be trusted or rechecked (Section 3.3) |
| A new name, scope, or route with no collision sweep | It plants a future contract conflict (Section 3.6) |
| In a launch handoff, a next-step dependency (a grant, an endpoint field, a wiring, a mechanism) stated as available with no search verifying it exists, is granted, or is built | It sends the next session to build on a void; documented-but-unbuilt is the worst case (Section 3.9) |
| A silent cap or truncation (top-N, sampled, skipped) presented as full coverage | It reads as complete when it is partial |
| Em dash or en dash characters | Project rule `.claude/rules/no_em_or_en_dashes.md` |

---

## 7. File Location and Naming

A handoff is ephemeral. Its content is superseded the moment its decisions land in the plan and the canonical docs, so it is NOT committed to the repository: committing it would create the second source of truth that Section 1.1 forbids.

- **Location and name:** `/tmp/<descriptive_name>_handoff.md`, mirroring the project rule that throwaway artifacts live in `/tmp/` and not in the repository tree. Example: `/tmp/zones_recipe_handoff.md`.
- **Descriptive name:** lowercase with underscores, naming the work, ending in `_handoff`.

If the Project Owner wants handoffs retained in a tracked location instead of `/tmp/` (for an audit trail across sessions), that is an owner decision that changes this section; record it here when made. Until then, `/tmp/` is the convention.

---

## 8. Worked Example (Compact)

A compliant fragment, showing the registers, citations, the cross-product, and an OPEN item, drawn from a real handoff:

```markdown
## 2. The settled model

- VERIFIED. A zone is private to its owner by default; visibility widens by
  customer publish, team share, customer-admin reach, and public scope.
  (datamodel_locations_documentation.md:92-113, datamodel_zones_and_zone_types_documentation.md:27-34)
- DERIVED. The base zone family must be granted by a seeded role, not a
  per-assignment ceiling, because ceilings only narrow and the role layer is the
  only grantor. (premises: auth doc §2.6.5, §4.3)

### Share, by actor (cross-product, per Section 3.4)
| Actor | Verb: share | Scope | Default-granted | Revoked by |
|---|---|---|---|---|
| Owner | own resource | base `share:zone` | yes | narrowing the assignment ceiling |
| Customer-admin | others' resources | elevated `share:customer_zone` | by admin role | removing the elevated scope |
| Team member | not permitted | none (share is not a team action) | no | N/A |

## 4. Open gating decisions
- OPEN. The `add` action in `team_assignment_permissions`: does it mean "create a
  resource already associated with a team" or "add an existing resource into a
  team"? Settles when the Project Owner rules. Proposed default: the former.
```

---

## 9. The Master Handoff Prompt Template

This is the copy-pasteable, domain-neutral prompt that operationalizes this guideline for one handoff. It is parameterized by the work; it bakes in no domain answer; it relies on the disciplines above to derive content. Fence it as text to distinguish it from code.

```text
You are writing a session handoff so a fresh session resumes <THE WORK> losing nothing material. The deliverable is a file at /tmp/<descriptive_name>_handoff.md. Follow documentation/general_handoff_crafting_guidelines.md exactly.

GROUNDING: before any load-bearing claim, read the primary source DIRECTLY and at the depth its relevance warrants (in full for the documents governing the work; conventions plus relevant sections for large catalog or reference docs, disclosing that triage; the immutable schema by targeted grep then range-read). A subagent digest or a memory does NOT satisfy a load-bearing claim.

TAGGING: every statement is exactly one of VERIFIED (cite file:line or section), DERIVED (cite the premises), PROPOSED (your unapproved recommendation), or OPEN (a Project-Owner decision, with the concrete options and what settles it). A positive fact cites a line or section; an absence claim cites its search command and result, and is retracted if the search contradicts it.

EXHAUSTIVENESS: where the work has interacting dimensions, build the full cross-product and fill every cell. Never write a blanket ("only", "always", "never") for a cell without recording its cross-cutting variant. For an authorization model, build the actor-by-verb matrix and record per cell the base capability, the elevated cross-user variant, the delegated team action, whether default-granted, and how revoked.

RESOLUTION AND COLLISION: for every new capability, scope, flag, role, route, or name, resolve HOW it is granted or enabled by applying the system's real resolution rules and state the consequence; and grep the catalogs and references for any existing or planned use of the same token or surface, recording and reconciling every hit.

DRIFT: compare the committed plan and docs against every chat-only decision; flag every divergence including bookkeeping, name each decision's canonical home, and give the reconciliation step.

CONTENT: produce the Section 4 structure: resume point and single next action; the settled model (tagged and cited); chat-only decisions with provenance; open gating decisions with concrete proposals; verified substrate facts; documented-vs-gap with search evidence; required reading with which sections were read directly; workflow conventions and project rules; the owner's working style; a ready-to-paste first message.

SELF-REVIEW: run the Section 5 gate. Spot-verify a sample of citations actually resolve. Confirm no undecided question is stated as an answer, no blanket lacks its cross-product cell, no absence claim lacks its search, and no em or en dashes appear. If any check fails, you are not finished.
```

---

## 10. Version Control

**Version:** 1.1.0
**Changes in 1.1.0:**
- Added the launch-handoff discipline (Section 3.9: Forward Dependency Resolution) for a handoff whose purpose is to start a next step the session did not build. It requires mapping the next step's dependency surface and verifying each dependency by direct search (exists, granted, returns what is needed, prior art to reuse), and names documented-but-unbuilt as the highest-risk class. Broadened Section 3.5 to consumed preconditions (verify the current state of existing scopes/endpoints the next step relies on, not only capabilities the work introduces) and Section 3.6 to the positive prior-art direction (sweep for existing mechanisms to reuse, not only naming collisions to avoid). Added the matching self-review gate item (Section 5) and forbidden pattern (Section 6). Motivated by a real launch handoff that asserted a consumed scope grant and a `mapMode` map interaction without verifying either: the scope grant was unconfirmed and the `mapMode` consumer did not exist in code.
- **Changes in 1.0.0:** Initial standard. Establishes the bridge-not-source-of-truth principle, the two-failure-mode taxonomy, the four-register tagging rule (VERIFIED / DERIVED / PROPOSED / OPEN), the citation rule including absence claims, exhaustiveness by cross-product, precondition and grant resolution, the collision and prior-art sweep, the drift check, provenance, the mandatory structure, the self-review gate, the file-location convention, a worked example, and the embedded master handoff prompt template.

---

**Last Updated:** 2026-06-15
**Status:** In Review
**Owner:** MarNexii Platform Team

**Note:** This document follows the documentation crafting guidelines for its own form (status, metadata footer, tagged code fences, no forbidden phrases, no em or en dashes) and the plan crafting guidelines for its house structure (compliance mandate, LLM directive, core principles, mandatory structure, self-review gate, embedded master prompt, version control).
