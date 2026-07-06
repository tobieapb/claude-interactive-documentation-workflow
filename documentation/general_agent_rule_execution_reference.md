# General Agent Rule Execution Reference

Reference methodology for how an agent must discover, interpret, and obey repository instructions before taking repo-specific actions in any repository.

## Purpose

This document defines the generic execution discipline for agents working in this repository. Its purpose is to prevent a recurring failure mode: the agent chooses the shortest apparent path to an answer, command, edit, or test before first establishing which loaded instructions govern the task.

This is not a task-specific exception list. It is a control-flow reference that applies equally to code changes, documentation work, schema questions, deployment work, live-system testing, and any future task category introduced later.

## Core Failure Mode

The most dangerous agent failure is not ignorance of a single file. The dangerous failure is **premature shortcutting**:

1. The agent classifies the task too quickly
2. The agent infers a convenient method from general knowledge
3. The agent acts before resolving the authoritative rule set for the current task scope
4. The agent then escalates, edits, or answers based on the shortcut instead of the repository's stated procedure

Once this happens, later corrections are expensive because the first action was already taken under the wrong authority model.

## Governing Principle

For any repo-specific task, **repository instructions outrank convenience**.

The agent must not treat documentation, rule files, routed prompts, or subtree-local instructions as optional context. They are execution prerequisites when their trigger conditions match the task.

## Required Execution Sequence

Every repo-specific task must follow this sequence in order:

1. **Discover instructions**
2. **Load applicable rule sources**
3. **Extract obligations from those sources**
4. **Classify the task against the extracted obligations**
5. **Complete every triggered prerequisite reading step**
6. **Verify that the intended action is authorized by the loaded instructions**
7. **Only then execute commands, edits, tests, or repo-specific answers**

If any step is incomplete, the task is not ready for execution.

## Instruction Discovery

Instruction discovery is not limited to the repository root.

The agent must discover and load all applicable instruction sources for the current task scope, including:

- Root `AGENTS.md`
- Root `CLAUDE.md`
- All required files in root `.claude/rules/`
- Any follow-up instruction files explicitly referenced by those documents
- Any additional `.claude/rules/` directories in the active subtree

The agent must not assume that the root rules are the whole rule set if the task is taking place inside a more specific subtree.

## Obligation Extraction

After loading instruction sources, the agent must convert them into explicit operational obligations. At minimum, the agent must identify:

- Which tasks or conditions trigger a rule
- Which documents must be read before action
- Which actions require explicit user approval
- Which shortcuts or substitutions are forbidden
- Which files are authoritative for the current domain
- Which actions are prohibited entirely

This extraction step matters because instructions often express control flow indirectly, such as:

- "When a task involves testing, read X first"
- "When working in a subtree with its own rules, load those files too"
- "Before writing code involving Y, read Z"

These are not suggestions. They are blockers on execution.

## Task Classification

Before taking action, the agent must classify the task against the loaded obligations. The classification must be functional, not superficial.

Examples:

- A request to "test" something is not just a shell task; it may be a live-system testing task
- A request involving endpoints, tokens, health checks, or validation may be governed by testing procedures
- A request inside a subdirectory may trigger subtree-local instructions even if the command itself is simple
- A request touching deployment paths, users, roles, or units may trigger deployment-convention reading before any draft is written

The important question is not "What is the fastest way to do this?" The important question is "What category of governed work is this?"

## Prerequisite Reading

If a loaded rule routes the agent to another document before action, that reading is mandatory.

The agent must not replace routed prerequisite reading with an inferred shortcut, even if the shortcut appears technically equivalent. A shortcut is invalid if the governing instructions require a different pre-action workflow.

Examples of invalid behavior:

- Inspecting local state when the routed method requires validating through a documented external check
- Answering from general memory when the instructions require consulting a current source of truth
- Running a plausible command before reading the prompt or reference that governs the task class

## Action Authorization

Before any command, edit, test, or repo-specific answer, the agent must be able to identify which loaded instruction authorizes the action.

The agent should be able to answer both of these questions:

1. Which loaded instruction path authorizes this action?
2. Which loaded instruction path prohibits likely shortcuts here?

If the agent cannot answer those questions, the action is not ready to execute.

## Shortcut Prohibition

An inferred method is not acceptable merely because it seems simpler, faster, or locally equivalent.

A shortcut is prohibited when:

- A loaded rule prescribes a different method
- A loaded document defines a prerequisite read that has not been completed
- The shortcut bypasses an approval requirement
- The shortcut relies on unstated assumptions instead of repository authority

The agent must prefer the repository's explicit process over its own inferred convenience path.

## Subtree Re-Scan Requirement

If the task scope shifts into a different directory, or the target artifact is located under a subtree that may have its own `.claude/rules/` directory, the agent must re-scan that subtree before proceeding.

This prevents a common failure mode where the agent loads only root instructions, then later performs work under a more specific local ruleset without ever discovering it.

## Escalation Discipline

Requesting elevated execution does not repair a non-compliant action.

Before escalating a command, the agent must verify that:

1. The task has already passed instruction discovery and prerequisite reading
2. The exact command being escalated is the command authorized by the loaded instructions
3. No documented procedure has been replaced by an inferred shortcut

The correct sequence is:

1. Resolve the governing instructions
2. Determine the compliant action
3. Then decide whether that compliant action needs escalation

Escalating first and validating later is backwards.

## Practical Pre-Action Checklist

Before acting on any repo-specific task, the agent should be able to answer "yes" to all of the following:

- Have all applicable instruction sources for this task scope been discovered and loaded?
- Have all rule-referenced follow-up documents been read?
- Have any subtree-local `.claude/rules/` directories been checked for the active path?
- Has the task been classified against the loaded rules rather than by superficial convenience?
- Have all triggered prerequisite reads been completed?
- Is the intended action authorized by a loaded instruction source?
- Am I avoiding an inferred shortcut that bypasses the documented method?

If any answer is "no," the next step is more rule resolution, not execution.

## Why This Reference Exists

Repositories accumulate specific methodologies over time: testing prompts, deployment conventions, style constraints, schema authorities, and subtree-local rules. An agent that obeys only the rules it already expected will fail as the repository evolves.

This reference exists to enforce a more general discipline:

- New rules must be discoverable
- New rules must be load-bearing
- New rules must be allowed to redirect execution
- New subtree instructions must override generic convenience

That is the only scalable way for agent behavior to remain correct as the rule set expands.

## Relationship to `AGENTS.md`

`AGENTS.md` should remain short and enforce the hard gate at session start.

This document provides the fuller rationale and operating model behind that gate. It is the place to explain:

- why shortcutting is dangerous
- why routed prerequisite reading is mandatory
- why subtree-local rules matter
- why action authorization must come from loaded instructions rather than inference

---

**Last Updated:** 2026-04-06
**Status:** Active
**Owner:** MarNexii Platform Team
