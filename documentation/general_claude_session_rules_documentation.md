# Claude Session Rules and Machine Configuration

Standards for governing Claude Code behavior across sessions, machines, and projects. Covers the layered rules system, auto memory policy, bootstrapping, and machine setup.

---

## The Problem

Claude Code stores configuration at multiple levels, some git-tracked and some machine-local. Without explicit governance:

- Machine-local files accumulate rules invisible to git
- Rules written on one machine do not exist on another
- A fresh clone on a new machine loses behavioral context
- Claude may silently write to machine-local storage without the user's awareness

This document defines the rules system that prevents these failures.

---

## Rules Hierarchy

Claude Code rules live at four levels. When rules conflict, **the most specific level wins**.

| Level | Location | Git-Tracked | Scope | Survives Machine Change |
|-------|----------|-------------|-------|------------------------|
| 1. Subdirectory | `<subdir>/.claude/CLAUDE.md` | Yes | One module/feeder/subproject | Yes |
| 2. Project root | `.claude/CLAUDE.md` | Yes | One repository | Yes |
| 3. User-level | `~/.claude/CLAUDE.md` | No (machine-local) | All projects on this machine | No |
| 4. Auto memory | `~/.claude/projects/*/memory/MEMORY.md` | No (machine-local + path-specific) | One project on this machine at this path | No |

### What Goes Where

| Content Type | Correct Level | Rationale |
|-------------|---------------|-----------|
| Project-specific rules (banned tools, naming, conventions) | Level 2 (project root) | Travels with the repo, shared across team |
| Subproject-specific rules (feeder CLI usage, local conventions) | Level 1 (subdirectory) | Scoped tightly, avoids bloating root file |
| Universal personal preferences (git workflow, coding standards) | Level 3 (user-level) | Applies to all projects, but must be deployable from a canonical source |
| Nothing | Level 4 (auto memory) | See Auto Memory Policy below |

---

## Auto Memory Policy

### The Rule

**Never write to `~/.claude/projects/*/memory/` without ALL of the following:**

1. **Explicit disclosure:** Tell the user exactly what will be written, where, and that it is machine-local (does not persist in git)
2. **Double confirmation:** Ask the user to confirm. Then ask again: "This is machine-local only and will not survive across machines. Confirm you want this?"
3. **Never infer approval.** Explicit "yes" twice. Silence, ambiguity, or context do not count.
4. **Mandatory git-tracked pair:** Every auto memory entry must be paired with a corresponding rule in a git-tracked file. The git-tracked version is the canonical source. Auto memory is only a convenience cache.
5. **Default: do not write.** If in doubt, do not use auto memory.

### Rationale

Auto memory is:
- **Machine-specific:** Does not exist on other machines
- **Path-specific:** Tied to the absolute filesystem path of the project, not the repo itself
- **Invisible to git:** Cannot be audited, diffed, or reviewed through normal workflows
- **Silently written:** Claude Code's default behavior is to write here without explicit user approval

These properties make it unsuitable as a source of truth for anything.

### Auto Memory Audit (Once Per Project Per Session)

On the **first interaction** of a session:

1. **Read** the auto memory file for the current project. If it does not exist or contains only a "do not use" stub, skip — no action needed.
2. **Compare** against git-tracked rules (project and subdirectory `.claude/CLAUDE.md` files). Identify entries in auto memory not captured in git.
3. **If orphaned rules exist**, alert the user:
   - List each orphaned entry
   - Explain it exists only on this machine
   - Ask: "Should I migrate these to git-tracked `.claude/CLAUDE.md`? Or are they stale and should be removed?"
4. **Wait for explicit instructions.** Do not modify anything without direction.
5. **After resolution**, offer to stamp the memory file with an audit date (e.g., `Audited YYYY-MM-DD: no orphaned rules.`) so subsequent sessions skip the audit. Only write this note with user approval.

---

## Local Rules Convention (`.claude/` Subfolders)

### The Convention

Any subdirectory in a project may contain a `.claude/CLAUDE.md` with rules scoped to that area. These are git-tracked and follow the same format as the project root `.claude/CLAUDE.md`.

### The Trigger

**Before starting work in any subdirectory, check if it has a `.claude/CLAUDE.md` and read it first.** The trigger is the existence of the file, not a registry or listing elsewhere.

### When to Create One

Create a subdirectory `.claude/CLAUDE.md` when:
- The subdirectory has CLI tools or utilities with specific usage patterns
- The subdirectory has conventions that differ from or extend the project root
- Rules would bloat the project root file with context irrelevant to other parts of the repo

### Format

Keep subdirectory rules files focused and concise:

```markdown
# Local Rules for <subdirectory name>

<Brief description of what this subdirectory is.>

## <Rule Section>

<Rules scoped to this area.>
```

### Optional Registry

The project root `.claude/CLAUDE.md` may include a table listing known subdirectories with local rules as a convenience for human readers. This table is informational — it is not the source of truth. The existence of a `.claude/` subfolder is the authoritative trigger.

---

## Bootstrapping: The New Machine Problem

### The Problem

User-level rules (`~/.claude/CLAUDE.md`) are machine-local. On a new machine:
1. Clone a project repo — project `.claude/CLAUDE.md` loads from git
2. User-level rules do not exist — protections are missing
3. Claude may write to auto memory, ignore conventions, or behave inconsistently

### The Solution: Inline Bootstrap + Canonical Reference

Every project `.claude/CLAUDE.md` must contain a **bootstrap block** — a small, self-contained set of critical rules that enforce correct behavior even when user-level rules are absent.

The bootstrap block contains:
1. The critical enforcement rules (auto memory prohibition, subdirectory check, audit trigger)
2. A URL pointing to this canonical repository for the full methodology
3. A pointer to the user-level rules template for machine setup

### Bootstrap Block Template

Place this at the top of any project's `.claude/CLAUDE.md`:

```markdown
## Bootstrap Rules (Critical — Enforced on Any Machine)

These rules apply regardless of whether `~/.claude/CLAUDE.md` is configured.

1. **NEVER** write to `~/.claude/projects/*/memory/` without explicit double-confirmed user approval.
2. **On first session**, read the auto memory file for this project. If it contains rules not in git-tracked `.claude/` files, alert the user and ask for instructions.
3. **Before working in a subdirectory**, check if it has a `.claude/CLAUDE.md` and read it.
4. **If `~/.claude/CLAUDE.md` does not exist**, inform the user: "User-level rules not found. Deploy from: https://github.com/<user>/claude-interactive-documentation-workflow"

Full methodology: https://github.com/<user>/claude-interactive-documentation-workflow
```

Replace `<user>` with the GitHub username.

### Deployment to a New Machine

```bash
# 1. Clone the canonical repo (or just fetch the template file)
git clone https://github.com/<user>/claude-interactive-documentation-workflow.git /tmp/claude-workflow

# 2. Copy the user-level rules template
cp /tmp/claude-workflow/documentation/general_user_level_rules_template.md ~/.claude/CLAUDE.md

# 3. Verify
cat ~/.claude/CLAUDE.md
```

After deployment, all projects on that machine inherit the full user-level rules.

---

## Interaction Between Levels

### Session Start Flow

```
1. Claude reads project .claude/CLAUDE.md (git-tracked, always available)
   ├── Bootstrap block fires — critical rules enforced immediately
   └── Project-specific rules loaded
2. Claude reads ~/.claude/CLAUDE.md (if it exists)
   ├── Full user-level rules supplement the bootstrap
   └── If missing → bootstrap suggests deployment
3. Claude checks auto memory file
   ├── Stub only → skip (no noise)
   └── Has content → audit against git-tracked rules, alert user if orphaned
4. User says "work on <subdirectory>"
   └── Claude checks <subdirectory>/.claude/CLAUDE.md → reads if exists
```

### Conflict Resolution

| Scenario | Resolution |
|----------|-----------|
| Subdirectory rule contradicts project root rule | Subdirectory wins (most specific) |
| Project rule contradicts user-level rule | Project wins (more specific) |
| Auto memory contradicts git-tracked rule | Git-tracked wins (auto memory is a cache, not source of truth) |
| Bootstrap rule contradicts detailed user-level rule | User-level wins (bootstrap is a floor, user-level is the ceiling) |

---

## Summary

| Principle | Implementation |
|-----------|---------------|
| Git is the source of truth | All rules in `.claude/CLAUDE.md` files within repos |
| Machine-local is a deployed artifact | `~/.claude/CLAUDE.md` deployed from canonical template |
| Auto memory is forbidden by default | Requires double confirmation + git-tracked pair |
| Bootstrapping prevents cold-start failures | Inline bootstrap block in every project |
| Subdirectory rules are trigger-based | Existence of `.claude/` folder, not a registry |
| Audit prevents silent drift | One-time per-session check of auto memory |

---

**Last Updated:** 2026-02-27
**Status:** Complete
