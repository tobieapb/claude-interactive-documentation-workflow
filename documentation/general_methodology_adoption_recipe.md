# Methodology Adoption Recipe

**Canonical Source:** https://github.com/tobieapb/claude-interactive-documentation-workflow

**Purpose:** This document is the action-oriented reference for an agent that has been asked to seed a working repo with the Claude Interactive Documentation Workflow methodology, or with a specific subset of it.

**Audience:** agents. If you are a human reading this, you can follow it too, but the primary reader is an agent that has just cloned this canonical repository to a scratch location in response to a request like:

> "We don't have a documentation folder. Search GitHub for the claude-interactive-documentation-workflow repo, clone it locally, and seed this repo with the needed guidelines."

**Scope:** this document is not a scaffolder. It does not try to push files into new projects automatically. It is a lookup table plus a minimal behavioral contract: given a gap the user has identified, it tells the agent exactly what to copy from the canonical clone, where to put it in the working repo, and what to confirm before and after the action.

---

## 1. How This Recipe Is Used

The interaction pattern this recipe is designed to serve:

1. The user identifies a gap in the working repo (a missing guideline, a missing folder, a missing skill, or "we have nothing, set everything up").
2. The user points the agent at the canonical repository by URL or by a `gh repo search` phrase.
3. The agent clones the canonical repository to a scratch location outside the working repo (e.g. `/tmp/claude-workflow` or a sibling directory). The clone is a read-only reference; files are never executed from it.
4. The agent reads this recipe file inside the clone, matches the user's identified gap against the tier table in §4, and prepares an action list (which files to copy, which folders to create, which blocks to paste).
5. The agent states the action list to the user and waits for confirmation before touching the working repo.
6. On confirmation, the agent executes the actions and reports back: which files were copied, which folders were created, which blocks were pasted, and which canonical documents now live in the working repo.

The recipe supports **partial adoption** as a first-class case. A user saying "we don't have plan guidelines" triggers a single-file action; a user saying "we have nothing" triggers the full chain. Never execute more than the user asked for without explicit confirmation.

---

## 2. The Minimum Requirement

The methodology functions once a working repo contains these two files:

| File | Destination in working repo |
|---|---|
| `general_documentation_crafting_guidelines.md` | `documentation/general_documentation_crafting_guidelines.md` |
| `general_plan_crafting_guidelines.md` | `documentation/general_plan_crafting_guidelines.md` |

These two files are Tier 0. They give the agent in the working repo the rule set for producing compliant documentation and compliant plans — the two structured artifact types that carry the methodology's quality discipline. Everything else in this recipe is expansion on top of Tier 0.

The `documentation/` folder itself is created implicitly by the act of placing the two files.

---

## 3. Folder Structure the Methodology Assumes

The full folder shape is:

```
documentation/                # canonical docs + methodology reference files
investigations/               # loose-to-structured evidence gathering
plans/                        # implementation plans
archive/
  documentation/              # deprecated docs after ~3 months
  investigations/             # concluded and archived investigations
  plans/                      # completed or abandoned plans
.claude/
  CLAUDE.md                   # project rules (must contain the Bootstrap Block — see §6)
  skills/
    interview/
      SKILL.md                # optional, only if /interview is wanted
```

Subsystems within the working repo can mirror this shape locally (e.g. `backend/documentation/`, `backend/investigations/`, `backend/plans/`, `backend/archive/{documentation,investigations,plans}/`). The methodology is the same at every level.

Only create folders a tier action in §4 explicitly says to create. Do not preemptively create folders.

---

## 4. Gap → Action Lookup Table

Each row describes a gap the user may have identified, the tier it belongs to, and the exact action the agent must prepare and propose before touching the working repo.

### Tier 0: Minimum for the methodology to function

| Gap (user symptom) | Action |
|---|---|
| "We don't have documentation crafting guidelines" / "don't we have a documentation guideline?" | Copy `documentation/general_documentation_crafting_guidelines.md` → `{working-repo}/documentation/general_documentation_crafting_guidelines.md`. Create `documentation/` if it does not already exist. |
| "We don't have plan crafting guidelines" / "don't we have a plan guideline?" | Copy `documentation/general_plan_crafting_guidelines.md` → `{working-repo}/documentation/general_plan_crafting_guidelines.md`. Create `documentation/` if it does not already exist. |
| "Don't we have a documentation folder?" | If the gap is the folder only, create `{working-repo}/documentation/`. If the gap is the folder AND the guidelines, treat it as the combined case above. Propose both interpretations to the user. |

### Tier 1: Common expansion

| Gap (user symptom) | Action |
|---|---|
| "We don't have an investigations folder" | Create `{working-repo}/investigations/` and `{working-repo}/archive/investigations/`. If the investigation guidelines are also absent, copy them in the same action (next row). |
| "We don't have investigation guidelines" / "don't we have an investigation guideline?" | Copy `documentation/general_investigation_review_guidelines.md` → `{working-repo}/documentation/general_investigation_review_guidelines.md`. |
| "We don't have a plans folder" | Create `{working-repo}/plans/` and `{working-repo}/archive/plans/`. |
| "We don't have an archive folder" | Create `{working-repo}/archive/` and any of `archive/documentation/`, `archive/investigations/`, `archive/plans/` whose corresponding top-level folder already exists in the working repo. |

### Tier 2: Adopter's choice

| Gap (user symptom) | Action |
|---|---|
| "We want to use `/interview`" / "we don't have the interview skill" | Copy `documentation/general_interview_methodology_skill.md` → `{working-repo}/documentation/`. Copy `.claude/skills/interview/SKILL.md` → `{working-repo}/.claude/skills/interview/SKILL.md`. Create both parent directories if absent. |
| "We don't have swarm guidelines" / "how do we parallelize across agents?" | Copy `documentation/general_agent_swarm_execution_guidelines.md` → `{working-repo}/documentation/`. |
| "We don't have session rules" / "we want the governance doc" | Copy `documentation/general_claude_session_rules_documentation.md` → `{working-repo}/documentation/`. Note: this is the full 213-line governance document; the Bootstrap Block it describes is reproduced inline in §6 of this recipe so the agent does not need to chase a second file to apply the minimum rules. |
| "We want the user-level rules template for our machine" | Copy `documentation/general_user_level_rules_template.md` → `{working-repo}/documentation/`, or directly `cp` to `~/.claude/CLAUDE.md` if the user confirms this is the target. This is a per-machine action, not a per-project action; confirm which intent is active before copying. |
| "We want this recipe file itself in the working repo" | Copy `documentation/general_methodology_adoption_recipe.md` (this file) → `{working-repo}/documentation/`. This is useful when the working repo should remember how to re-adopt from the canonical source later. |

### Tier Full: "We have nothing, set everything up"

Execute Tier 0 + Tier 1 in order, then ask the user which Tier 2 items they want. Do not apply all Tier 2 items by default — Tier 2 is explicitly adopter's-choice.

---

## 5. Project-Level Rules (`.claude/CLAUDE.md`)

The methodology assumes the working repo has a `.claude/CLAUDE.md` with at minimum the Bootstrap Block reproduced in §6. If the working repo has no `.claude/CLAUDE.md`, the agent's action is to create one containing only the Bootstrap Block and state this clearly to the user before doing so. If a `.claude/CLAUDE.md` exists but lacks the block, the agent proposes inserting the block near the top of the file and waits for confirmation. Do not silently modify an existing `.claude/CLAUDE.md`.

---

## 6. The Bootstrap Block (inline)

This is the minimum set of rules every working repo adopting this methodology should carry in its `.claude/CLAUDE.md`. Paste this block verbatim, replacing `<user>` with the GitHub user or organization that owns the canonical fork you are adopting from (use `tobieapb` if you are adopting from the canonical upstream).

```markdown
## Bootstrap Rules (Critical — Enforced on Any Machine)

These rules apply regardless of whether `~/.claude/CLAUDE.md` is configured.

1. **NEVER** write to `~/.claude/projects/*/memory/` without explicit double-confirmed user approval.
2. **On first session**, read the auto memory file for this project. If it contains rules not in git-tracked `.claude/` files, alert the user and ask for instructions.
3. **Before working in a subdirectory**, check if it has a `.claude/CLAUDE.md` and read it.
4. **If `~/.claude/CLAUDE.md` does not exist**, inform the user: "User-level rules not found. Deploy from: https://github.com/<user>/claude-interactive-documentation-workflow"

Full methodology: https://github.com/<user>/claude-interactive-documentation-workflow
```

The Bootstrap Block is required content if the user asks the agent to "set up project rules." It is optional content if the user only asked for guideline files; in that case the agent should surface that the block exists and offer to add it, but not insert it without permission.

---

## 7. Reverse Discovery

Every `general_*.md` file in this canonical repository carries a `**Canonical Source:**` header pointing back to the upstream URL. This means a working repo holding even one copied guideline file can reach the rest of the methodology by following the URL. Agents arriving in a foreign repo that contains, for example, only `general_documentation_crafting_guidelines.md` can:

1. Read the `**Canonical Source:**` header in that file.
2. Clone the canonical repository to a scratch path.
3. Read this recipe.
4. Propose what else the working repo needs, per the user's direction.

This is the reverse of the forward seeding flow described in §1. Both flows use the same lookup table in §4.

---

## 8. Agent Conduct Rules

When executing a recipe action, the agent must:

1. **Clone to scratch, not inside the working repo.** The canonical clone is a read-only reference, never a nested subdirectory of the host repo. Use `/tmp/` or a sibling directory.
2. **Propose before touching.** Present the action list and wait for user confirmation before any copy, mkdir, or file edit inside the working repo.
3. **Never invent files.** If a gap is not covered by a row in §4, say so and ask the user rather than guessing what the right action is.
4. **Never auto-create Tier 2.** Tier 2 items are explicitly adopter's-choice; requesting Tier Full does not silently include them.
5. **Preserve existing content.** If a file the agent would copy already exists in the working repo, propose a diff or an overwrite prompt; do not silently overwrite.
6. **Never modify `.claude/CLAUDE.md` silently.** Show the proposed Bootstrap Block insertion and wait for confirmation.
7. **Report back.** After the actions complete, report which files were copied, which folders were created, which blocks were pasted into which rules file, and which Tier 2 items were offered but not applied.
8. **Clean up the clone.** Once the actions are complete and reported, the scratch clone can be deleted. State this to the user as part of the final report.

---

## 9. What This Recipe Does NOT Do

- It does not automate anything. No scripts, no `./seed.sh`, no runnable commands beyond the standard `git clone` that brings the canonical repo to a scratch location. The agent is the executor.
- It does not push updates into a working repo that has already adopted an older copy of the methodology. Update strategy is a separate concern (diff the two copies, propose changes, wait for confirmation) and is not covered by this recipe.
- It does not enforce consistency across multiple working repos that share an owner. Each working repo is adopted independently.
- It does not prescribe what a "successful" adoption looks like beyond "the requested gap is filled." Whether the working repo should have Tier 1 or Tier 2 items is a user decision, not a recipe decision.

---

## 10. Relationship to the Other Methodology Documents

This recipe sits alongside the other `general_*.md` documents rather than above them:

- [`general_documentation_crafting_guidelines.md`](general_documentation_crafting_guidelines.md) — the rule set for producing a compliant documentation file.
- [`general_plan_crafting_guidelines.md`](general_plan_crafting_guidelines.md) — the rule set for producing a compliant plan file.
- [`general_investigation_review_guidelines.md`](general_investigation_review_guidelines.md) — the rule set for writing, concluding, and resolving an investigation.
- [`general_interview_methodology_skill.md`](general_interview_methodology_skill.md) — the structured interview process that produces a documentation file from scratch.
- [`general_agent_swarm_execution_guidelines.md`](general_agent_swarm_execution_guidelines.md) — the rule set for parallelizing methodology work across multiple agents.
- [`general_claude_session_rules_documentation.md`](general_claude_session_rules_documentation.md) — the full governance model for Claude Code rules across sessions, machines, and projects.
- [`general_user_level_rules_template.md`](general_user_level_rules_template.md) — the deployable `~/.claude/CLAUDE.md` template for per-machine setup.
- [`general_methodology_case_study_documentation.md`](general_methodology_case_study_documentation.md) — verified results from 4 months of production use.

This recipe's job is to decide which of the above a given working repo needs and to place them correctly. The documents themselves are the source of truth for how the methodology actually operates; the recipe is the adoption-time index.

---

**Status:** Complete
**Last Updated:** 2026-04-18
