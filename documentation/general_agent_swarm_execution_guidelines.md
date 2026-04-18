# Agent Swarm Execution Guidelines

**Canonical Source:** https://github.com/tobieapb/claude-interactive-documentation-workflow

**Purpose:** This document defines a general methodology for parallelizing work across multiple AI agents ("agent swarm"). The methodology is task-agnostic — it applies to plan passes, codebase-wide fixes, documentation updates, or any parallelizable work. The process has three phases: **Pre-Work** (formalize the goal, rules, and constraints into a dispatch brief), **Dispatch** (size the swarm, distribute work, launch agents), and **Post-Work** (validate the output against the dispatch brief and broader project constraints).

The dispatch brief is the connecting artifact: pre-work produces it, dispatch consumes it, post-work validates against it.

---

## 0. Activation Triggers

### 0.1 User-Initiated Triggers

When the user says any of the following (or close variations), the orchestrator agent must activate the swarm methodology defined in this document:

| Trigger Phrase | Intent |
|---------------|--------|
| "deploy a swarm" | Launch parallel agents for the current task |
| "use the swarm" | Same as above |
| "swarm this" | Same as above |
| "let's swarm it" | Same as above |
| "use the swarm methodology" | Same as above, with explicit methodology reference |
| "parallel agents" | Use multiple agents in parallel |
| "can we parallelize this" | Evaluate and offer the swarm option |
| "this is too much for one pass" | Evaluate and offer the swarm option |

When a trigger is detected, the orchestrator must:
1. Acknowledge the trigger
2. Begin Phase 1 (Pre-Work) immediately — gather the goal, rules, constraints
3. Present the dispatch brief to the user for approval before launching agents

### 0.2 Orchestrator-Initiated Evaluation

When working on a task that meets all four criteria in Section 1, the orchestrator should proactively offer the swarm option:

> "This work has [N] independent units that can be parallelized. I can deploy a swarm of [M] agents to complete this in parallel, following the swarm methodology. Each agent would handle [description of split]. Want me to proceed?"

The user can accept, decline, or adjust the proposed split. The orchestrator does NOT launch agents without user approval.

### 0.3 Plan Pass Integration

When an orchestrator agent is executing a plan pass (Pass 2, 3, or 4) and the plan has 100+ actions across 3+ non-N/A phases, the orchestrator should evaluate the swarm option and offer it before beginning the pass. See `documentation/general_plan_crafting_guidelines.md` for the pass methodology.

---

## 1. When to Use the Swarm

Use the swarm pattern when:

1. The work can be split into **non-overlapping units** where no agent needs another agent's output
2. There are **clear rules** that can be communicated unambiguously to each agent
3. The work is **large enough** that parallel execution provides meaningful time savings
4. The output can be **mechanically validated** against the rules

Do not use the swarm pattern when:

1. The work requires **sequential decisions** — one agent's output determines what the next agent should do
2. The rules are **ambiguous** or require judgment that can't be written down
3. The work is **small enough** for a single agent to complete in under 5 minutes
4. The units of work **overlap** in ways that would cause conflicting edits

---

## 2. The Three Phases

```
Phase 1: Pre-Work
  Input:  User's request + project context
  Output: Dispatch brief (formalized goal, rules, constraints, assignments)

Phase 2: Dispatch
  Input:  Dispatch brief
  Output: N agents working in parallel on non-overlapping units

Phase 3: Post-Work
  Input:  All agents' completed output + dispatch brief
  Output: Validated, consistent result ready for review
```

---

## 3. Phase 1: Pre-Work

The orchestrator gathers everything needed to instruct the agents clearly and validate their output afterward. The output is a **dispatch brief** — a structured artifact that answers every question an agent might have before it starts working.

### 3.1 What the Dispatch Brief Must Capture

The dispatch brief answers seven questions. Every answer must be explicit. If the orchestrator cannot answer a question, the swarm should not be launched — the ambiguity must be resolved first.

**A. What is the goal?**
What specific outcome does this swarm produce? What does "done" look like?

Example: "Transform all compound actions in the plan into atomic, single-verb actions per the plan crafting guidelines Pass 2 rules."

Example: "Replace all references to /opt/marnexii/ with the correct deployment convention paths across 4 documentation files."

**B. What are the rules?**
What constraints govern how each agent does its work? These must be specific enough that two agents following the same rules produce consistent output.

Rules come from:
- Task-specific standards (e.g., plan crafting guidelines §2.2 for atomicity)
- Project conventions (e.g., deployment conventions doc for paths)
- Project instructions (e.g., snake_case naming)
- User-specified constraints for this particular task

**C. What is the source of truth?**
Which documents, files, or artifacts does each agent read to get correct field names, types, paths, contracts, and values? List every file path explicitly.

**D. What is prohibited?**
What must agents NOT do? This includes:
- Forbidden phrases or patterns
- Files or sections they must not modify
- Decisions they must not make (reserved for the orchestrator or user)
- Conventions that have known incorrect alternatives (e.g., "NEVER use /opt/marnexii/")

**E. What are the deliverables?**
What does each agent write? To which file(s)? In what format?

**F. What are the boundaries?**
How is the work divided? What does each agent own? What must they not touch?

**G. How will success be measured?**
What specific checks will the orchestrator run in post-work? These must be concrete — ideally grep commands, counts, or pattern matches. Vague criteria like "looks good" are not acceptable.

### 3.2 Sizing the Swarm

Based on the dispatch brief, the orchestrator decides:

| Decision | Guidance |
|----------|---------|
| **How many agents?** | 3-5 is typical. Fewer than 3 doesn't justify overhead. More than 5 increases coordination risk. |
| **How to split the work?** | By natural boundaries in the target artifact (phases in a plan, files in a codebase fix, sections in a doc). Never split within a boundary unit. |
| **How to balance load?** | Estimate output size per unit. Assign heavy units to dedicated agents. Group light units together. |

### 3.3 Dispatch Brief Template

```markdown
## Dispatch Brief: [Task Description]

### Goal
[One paragraph: what does "done" look like?]

### Rules
[Numbered list: every constraint that governs agent behavior]

### Source of Truth
[File paths: every document agents must read for correct values]

### Prohibited
[Explicit list: what agents must NOT do, use, or modify]

### Deliverables
[What each agent writes, to which file(s), in what format]

### Agent Assignments

| Agent | Work Unit | Boundary | Estimated Load |
|-------|-----------|----------|---------------|
| A | [description] | [what they own] | [relative size] |
| B | [description] | [what they own] | [relative size] |
| ... | ... | ... | ... |

### Success Criteria (Post-Work Checks)
- [ ] [Concrete check 1 — ideally a grep or count]
- [ ] [Concrete check 2]
- [ ] [Concrete check 3]
- [ ] ...
```

---

## 4. Phase 2: Dispatch

### 4.1 Building Agent Prompts

Each agent receives a prompt constructed from the dispatch brief. The prompt includes:

1. **The goal** — what this agent is producing
2. **The rules** — identical for all agents (from the dispatch brief)
3. **The source of truth** — which files to read
4. **The prohibitions** — what not to do
5. **The boundary** — which unit of work this specific agent owns
6. **The deliverable format** — how to write the output

The orchestrator should use a consistent template so all agents receive the same structure. Only the boundary assignment differs between prompts.

### 4.2 Prompt Template

```
You are performing [GOAL] on [TARGET ARTIFACT].

**RULES:**
[Numbered list from dispatch brief — identical for all agents]

**SOURCE OF TRUTH:**
Read these files for correct values: [file list from dispatch brief]

**PROHIBITED:**
[List from dispatch brief — identical for all agents]

**YOUR ASSIGNMENT:** [This agent's specific work unit and boundary]

**BOUNDARY RULES:**
- Only modify [your assigned unit]
- Do NOT modify [other units, headers, shared sections]

**TASK:** [Specific instructions for what to do within the boundary]
```

### 4.3 Launch Protocol

1. Construct one prompt per agent from the template
2. Launch **all agents in parallel**
3. Do not begin post-work until ALL agents have completed
4. Do not duplicate any agent's work while waiting

---

## 5. Phase 3: Post-Work

The orchestrator validates the merged output against the dispatch brief's success criteria and the broader project constraints.

### 5.1 Validate Against Dispatch Brief

Walk the success criteria checklist from the dispatch brief item by item. For each item, run the corresponding check and record pass/fail. Every item must pass before the work is considered complete.

### 5.2 Mechanical Checks

These are automated checks the orchestrator runs. They should be scripted, not ad-hoc. Common patterns:

| Check Type | Method |
|-----------|--------|
| Prohibited patterns absent | `grep -rn "[pattern]" [files]` returns 0 matches |
| Required conventions present | `grep -c "[correct pattern]" [file]` returns expected count |
| Count consistency | Compare header/summary counts with actual `grep -c` counts |
| Cross-reference integrity | Verify that references between sections still resolve |
| No out-of-boundary edits | `git diff` shows changes only in assigned sections |

### 5.3 Judgment Checks

The orchestrator manually reviews:

1. **Consistency across agents** — do all agents use the same terminology, naming, and style?
2. **Boundary compliance** — did any agent modify sections outside its assignment?
3. **Semantic correctness** — are the changes actually correct, or just mechanically formatted?

### 5.4 Conflict Resolution

| Issue Type | Resolution |
|-----------|------------|
| Agents wrote conflicting values to a shared field (e.g., header count) | Orchestrator sets the correct value from source (e.g., `grep -c`) |
| Naming inconsistency between agents | Orchestrator picks the correct name from the source of truth and fixes all occurrences |
| Agent modified outside its boundary | Orchestrator reverts the out-of-boundary change |
| Prohibited pattern introduced | Orchestrator fixes directly |
| Success criterion fails | Orchestrator fixes, or re-dispatches a single agent for that unit |

### 5.5 Completion

After all checks pass:
1. Update any summary counts or status fields in the target artifact
2. Commit with a descriptive message that references agent count, action count, and validation results
3. Present for user review

---

## 6. Application: Plan Pass Execution

The most common use of the swarm is executing plan passes. Here is how the generic methodology maps to plan-specific work:

### 6.1 Goal Mapping

| Pass | Goal |
|------|------|
| Pass 2 (Atomicity) | Transform compound actions into single-verb, single-location atomic actions |
| Pass 3 (Detail Enrichment) | Add boilerplate stubs, typed signatures, rationale, prerequisites to actions |
| Pass 4 (Verification) | Add verification commands to high-risk actions and phase-level checklists |

### 6.2 Rules Mapping

| Pass | Rules Source |
|------|-------------|
| Pass 2 | Plan crafting guidelines §2.2 (atomicity patterns, 5-second rule, single verb) |
| Pass 3 | Plan crafting guidelines §2.3 (enrichment elements, boilerplate stubs, code snippets) |
| Pass 4 | Plan crafting guidelines §2.4 (verification patterns, coverage guidance, phase checklists) |

### 6.3 Work Unit Mapping

| Pass | Split By | Rationale |
|------|---------|-----------|
| Pass 2 | Phase boundaries | Actions within a phase are self-contained |
| Pass 3 | Complexity (new files vs existing) | Boilerplate stubs require more work than annotations |
| Pass 4 | Risk level (DB/auth/transactions vs output/docs) | High-risk actions need more verification effort |

### 6.4 Success Criteria Mapping

Every plan pass post-check should include:

```bash
PLAN="[path to plan file]"

# Forbidden phrases
grep -in "forbidden pattern" "$PLAN" | grep -v "exceptions" || echo "CLEAN"

# Deployment conventions
grep -n "/opt/marnexii\|User=marnexii\|chmod 750" "$PLAN" || echo "CLEAN"

# Path conventions
grep -n '`central-server/' "$PLAN" | grep -v '`\./central-server/' || echo "CLEAN"

# Action count consistency
HEADER=$(grep "Estimated Atomic Actions" "$PLAN" | grep -oP '\d+')
ACTUAL=$(grep -c "    - \[ \]" "$PLAN")
echo "Header: $HEADER, Actual: $ACTUAL"
[ "$HEADER" -eq "$ACTUAL" ] && echo "MATCH" || echo "MISMATCH"

# Section sum consistency
SUM=$(grep "Estimated Actions:" "$PLAN" | grep -oP '\d+' | paste -sd+ | bc)
echo "Sum: $SUM, Header: $HEADER"
[ "$SUM" -eq "$HEADER" ] && echo "MATCH" || echo "MISMATCH"
```

---

## 7. Anti-Patterns

| Anti-Pattern | Why It Fails | Correct Approach |
|-------------|-------------|-----------------|
| Launching without a dispatch brief | Agents receive inconsistent instructions | Always produce the brief first |
| Splitting within a natural boundary | Conflicting edits to the same section | Split on boundary lines only |
| Letting agents update shared state (headers, counts) | Last writer wins | Orchestrator owns shared state |
| Copying rules ad-hoc into each prompt | Rules drift between prompts | Use the brief as single source |
| Skipping post-work | Inconsistencies ship to the user | Always validate before presenting |
| Equal-sized assignments | Unbalanced load, long tail | Balance by estimated output size |
| Sequential agent launch | Loses parallelism benefit | Boundary rules and post-work ARE the safety mechanism |
| Trusting agent output without checking | Agents may introduce violations | The post-check is mandatory |

---

## 8. Orchestrator Checklist

### Phase 1: Pre-Work
- [ ] Understand the user's request and desired outcome
- [ ] Identify all relevant rules, conventions, and constraints
- [ ] Identify all source-of-truth documents
- [ ] Identify all prohibitions
- [ ] Estimate work per unit
- [ ] Size the swarm and assign boundaries
- [ ] Produce the dispatch brief (Section 3.3 template) with all fields filled
- [ ] Verify success criteria are concrete and mechanically checkable
- [ ] Verify no boundary overlaps between agents

### Phase 2: Dispatch
- [ ] Construct one prompt per agent from the brief
- [ ] Launch all agents in parallel
- [ ] Confirm all agents are working on non-overlapping units

### Phase 3: Post-Work
- [ ] Wait for ALL agents to complete
- [ ] Walk the dispatch brief's success criteria item by item
- [ ] Run mechanical checks
- [ ] Run judgment checks
- [ ] Resolve any conflicts
- [ ] Update shared state (counts, status) from source
- [ ] Commit with descriptive message
- [ ] Present for user review

---

## 9. Lessons from Deployments

### 9.1 First Deployment: Plan Pass 2 Atomicity (2026-03-17)

4 agents, splitting by plan phase. Task: transform 299 compound actions into 503 atomic actions.

| Agent | Work Unit | Before | After | Duration |
|-------|-----------|--------|-------|----------|
| A | Phase II + VII | 45 | 178 | ~16 min |
| B | Phase III | 55 | 110 | ~10 min |
| C | Phase V + VI | 43 | 55 | ~6 min |
| D | Phase VIII + IX | 48 | 56 | ~4 min |

**What worked:** Consistent output across all agents. Same naming, same paths, same format. Post-check was mechanical — 9 automated checks, all passed.

**What to improve:** Agent A took 4x longer than Agent D. Better load balancing would assign Phase II alone (largest struct expansion) to one agent.

### 9.2 Second Deployment: Deployment Path Correction (2026-03-17)

3 agents, splitting by file. Task: replace all `/opt/marnexii/` references with correct deployment convention paths across 4 files.

**What worked:** Simple, well-bounded task. Each agent owned specific files with no overlap.

**What to improve:** One agent's working directory was wrong, causing false "file not found" reports. Dispatch should specify the working directory explicitly.

---

**Last Updated:** 2026-03-17
**Status:** Complete
**Owner:** MarNexii Platform Team
