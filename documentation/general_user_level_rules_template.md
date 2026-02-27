# User-Level Rules Template

Deploy this file to `~/.claude/CLAUDE.md` on each machine. It provides universal rules that apply across all projects.

**Deployment:**
```bash
cp documentation/general_user_level_rules_template.md ~/.claude/CLAUDE.md
```

**Canonical source:** This template is maintained in the `claude-interactive-documentation-workflow` repository. When updating rules, update this file first, then redeploy to each machine.

---

**Everything below this line is the deployable content.** Copy from the next line to the end of the file.

---

# ABSOLUTE RULES (Override Everything)

## Auto Memory is FORBIDDEN

**NEVER use the `~/.claude/projects/*/memory/` auto memory system** without ALL of the following:

1. **Explicit disclosure:** Tell the user exactly what you intend to write, where it will be stored, and that it is machine-local (does not persist in git)
2. **Double confirmation:** Ask the user to confirm. Then ask AGAIN with: "This is machine-local only and will not survive across machines. Confirm you want this?"
3. **Never infer approval.** The user must explicitly say "yes" twice. Silence, ambiguity, or contextual hints do NOT count as approval.
4. **Mandatory git-tracked pair:** Every auto memory entry MUST be paired with a corresponding rule or documentation in a git-tracked project file (e.g., `.claude/CLAUDE.md`, `documentation/`, or `README.md`). The git-tracked version is the canonical source. The auto memory is only a convenience cache.
5. **If in doubt, do not write.** Default to NOT using auto memory.

**Rationale:** Auto memory is machine-specific and project-path-specific. It does not travel with the repo, does not survive machine changes, and creates invisible state that the user cannot audit through normal git workflows. The canonical source of truth must always be in git.

## Auto Memory Audit (Once Per Project Per Session)

On the **first interaction** of a session, before doing any work:

1. **Read the auto memory file** for the current project (`~/.claude/projects/*/memory/MEMORY.md`). If the file does not exist or contains only the "do not use" stub, skip the rest — no action needed.
2. **Compare against git-tracked rules** (project `.claude/CLAUDE.md` and any subdirectory `.claude/CLAUDE.md` files). Identify any entries in auto memory that contain project-relevant rules, patterns, or knowledge NOT already captured in git-tracked files.
3. **If orphaned rules are found**, alert the user:
   - List each orphaned entry with a summary of what it says
   - Explain that it exists only on this machine and is not in git
   - Ask: "Should I migrate these to the appropriate git-tracked `.claude/CLAUDE.md`? Or are they stale and should be removed?"
4. **Wait for explicit instructions.** Do not migrate, delete, or modify anything without the user's direction.
5. **After resolution**, offer to add a one-line note to the memory file recording that the audit was completed (e.g., `Audited YYYY-MM-DD: no orphaned rules.`) so subsequent sessions on this machine can skip the audit. Only write this note if the user approves.

This ensures rules written to auto memory on one machine are eventually captured in git or explicitly discarded — never silently lost.

---

# User-Level Rules

## Git Workflow

### Commit Strategy
- Commit frequently to local repository
- **Only push when explicitly authorized** ("push", "push this", "push to github")
- Stage ALL changes with `git add .` by default unless specific files are named
- **ALWAYS use `git add .`** even if only one file appears changed — the user may have added files out-of-band that Claude doesn't know about

### Commit Messages

**Claude-generated commits:**
- Use **PAST TENSE**: "Added feature X", "Fixed bug in Y", "Updated config"
- Be descriptive and specific
- **NO attribution footers or Co-Authored-By lines** unless explicitly requested

**User-provided commits:**
- Use **VERBATIM** - no modifications whatsoever
- Do not fix typos, grammar, or tense
- Do not add footers or attribution
- This overrides all other commit rules

## Coding Standards

- **Naming**: Use `snake_case` for variables, functions, and database columns
- **Configuration over hardcoding**: Extract configurable values (ports, hosts, timeouts, thresholds) to environment variables or config files
- **Extensibility**: Design with future expansion in mind; avoid hard-coded limits

## Documentation Organization

| Type | Location | Naming |
|------|----------|--------|
| High-level docs | `documentation/` | `lowercase_with_underscores.md` |
| Implementation plans | `plans/` | `lowercase_with_underscores.md` |
| Code-specific docs | With the code | `README.md`, `PascalCase.md` |

## Local Rules Convention (`.claude/` subfolders)

Subprojects, feeders, modules, and subdirectories may contain their own `.claude/CLAUDE.md` with rules scoped to that directory. These are **git-tracked** and follow the same conventions as the project root `.claude/CLAUDE.md`.

**When starting work in a subdirectory**, check if it has a `.claude/CLAUDE.md` and read it before proceeding. The trigger is the existence of the `.claude/` folder, not a registry or listing.

**Hierarchy (most specific wins):**
1. Subdirectory `.claude/CLAUDE.md` (most specific)
2. Project root `.claude/CLAUDE.md`
3. User-level `~/.claude/CLAUDE.md` (most general)

## Database Schema

- Schema files are **sacred** - notify user and justify before any proposed changes
- User makes schema changes out-of-band
- Use `grep` for schema lookups rather than reading entire files

---

## Reference

Canonical source: https://github.com/tobieapb/claude-interactive-documentation-workflow

Full methodology: See `documentation/general_claude_session_rules_documentation.md` in the canonical repo for rationale, bootstrapping details, and the interaction model between rule levels.
