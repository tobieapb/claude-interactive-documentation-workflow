# Live System Testing Guidelines

**Canonical Source:** https://github.com/tobieapb/claude-interactive-documentation-workflow

**Purpose:** Define the methodology for testing deployed systems in their real operating environment. This is distinct from unit testing (which validates code correctness in isolation) and from CI integration testing (which validates code against a specification before deployment). Live-system testing validates that a system, with all its real dependencies, configurations, infrastructure, and data, behaves as expected in production or production-like conditions.

**Audience:** engineers and agents who need to validate, benchmark, or diagnose running systems. The methodology assumes a significant fraction of tests will be executed by LLM agents rather than humans, and is structured to make agent execution deterministic and auditable.

---

## 0. What This Is (and What It Isn't)

Live-system testing is its own discipline because it differs from unit and integration testing on six axes:

| Axis | Unit / CI Integration Tests | Live-System Tests |
|---|---|---|
| Subject of validation | Code against a specification | Deployed system against its operational requirements |
| Execution environment | Isolated, reproducible, CI-controlled | Real services, real dependencies, real configurations |
| Common failure causes | Code bugs | Code bugs, deployment drift, config mismatches, environment issues, data corruption, infrastructure faults |
| Authority boundary | Owned by code authors | Often run by ops, SRE, or agents against systems they don't own |
| Execution model | Automatic, on every commit, isolated | On-demand, against real systems, with immediate operational consequences |
| Result persistence | Ephemeral pass/fail | Archived evidence — benchmarks, validation records, regression baselines |
| Primary consumer | Compiler-style output for developers | Increasingly: LLM agents that read a procedure, execute it, and report |

A live-system test answers the question: *"Does the running system, as it is actually deployed right now, behave the way it is supposed to?"* No unit test can answer that question.

This methodology does **not** replace unit tests, CI integration tests, UI QA, or code-level test-driven development. Those live in the module alongside the code they validate (e.g., Go `_test.go` files, `test_*.py`, a `tests/` folder inside the module) or in the CI configuration. This methodology applies only to tests of the **running, deployed system**.

---

## 1. Folder Structure

```
tests/
├── README.md            # Folder intent + routing table + link to this methodology
├── documentation/       # Guides, methodologies, prompts, references
├── drivers/             # Executable test artifacts (optional, see §5)
└── investigations/      # Test result snapshots, benchmarks, validation records

tests/archive/
├── documentation/       # Obsolete guides and references
├── drivers/             # Retired drivers
└── investigations/      # Archived results superseded by current baselines
```

The top-level `tests/README.md` serves three purposes: state the folder's intent, provide the routing table (§6) that maps tasks to files, and link to this methodology document as the authority for everything else.

Projects with multiple subsystems can mirror this shape at each subsystem boundary (e.g. `backend/tests/{documentation,drivers,investigations}/`) so tests stay local to the service they validate. The methodology is the same at every level.

---

## 2. Artifact Types

Every file in `tests/documentation/` carries a type suffix that declares what kind of artifact it is. Four types exist.

### 2.1 Prompts (`_prompt.md`)

An agent-executable test procedure. **Must** follow the 7-section structure in §4.

Purpose: a prompt file is intended to be given to an LLM agent as its primary instruction for executing a specific test. The agent reads the file, gathers the referenced context, executes the steps against the live system, and produces the deliverable specified in the file — all without needing to ask clarifying questions.

Examples: `central_api_endpoint_testing_prompt.md`, `station_system_health_validation_prompt.md`.

### 2.2 References (`_reference.md`)

System description, test methodology, or validation guide. Does not itself execute a test; provides the knowledge needed to understand or execute related tests.

Purpose: a reference file documents the system under test so that prompts can stay focused on "what to do" while the reference carries the "what is this thing." A prompt that inlines the system's architecture, endpoint inventory, or data model bloats; referencing a dedicated `_reference.md` keeps the prompt tight.

Examples: `central_api_endpoint_testing_reference.md`, `station_health_testing_reference.md`.

### 2.3 Methodologies (`_methodology.md`)

Cross-cutting testing strategy that applies across multiple tests or subsystems.

Purpose: when several prompts or references share a common approach (how to run smoke tests, how to interpret health endpoints, how to name benchmark runs, how to load bearer tokens), the common part is factored into a methodology file and cited from the specific tests.

Examples: `general_testing_methodology_reference.md`, `general_benchmark_interpretation_methodology.md`.

### 2.4 Investigations (`_investigation.md`)

Test result snapshots, benchmark records, validation artifacts. These are the durable evidence produced by running tests. They live in `tests/investigations/`, not `tests/documentation/`.

Purpose: investigations in `tests/investigations/` serve the same role as top-level investigations (see `general_investigation_review_guidelines.md`) — evidence captured for future reference, organized into findings, archived when superseded.

Examples: `central_drift_noise_validation_investigation.md`, `station_throughput_baseline_investigation.md`.

---

## 3. File Naming Convention

`${area}_${description}_${type}.md`

Where:
- `${area}` identifies the subsystem or scope. One per major subsystem of your project, plus `general_` for cross-cutting content.
- `${description}` is a snake_case description of what the test or reference is about.
- `${type}` is one of: `prompt`, `reference`, `methodology`, `investigation`.

Small projects with a single subsystem can collapse area prefixes and use the description directly (`api_endpoint_testing_prompt.md`). The prefix pattern starts paying off once three or more subsystems share the `tests/` folder and naming collisions become possible.

Examples across project sizes:

| Project Size | Example |
|---|---|
| Single subsystem | `api_health_prompt.md` |
| Multiple subsystems | `central_api_health_prompt.md`, `station_health_prompt.md` |
| Cross-cutting reference | `general_testing_methodology_reference.md` |
| Result snapshot | `central_api_latency_benchmark_investigation.md` |

Pick your project's area prefixes explicitly and list them in `tests/README.md`. Do not accrete prefixes one at a time as new tests arrive — that leads to overlapping or redundant prefixes (`central_` vs `central-server_` vs `server_`). Decide on the set and hold it.

---

## 4. The 7-Section Prompt Structure (Mandatory for `_prompt` Files)

Every `_prompt` file **must** have these seven sections, in this order, with these exact headings. The structure exists so that any competent LLM agent can execute the test without rediscovery, without asking clarifying questions, and without making up missing context.

### 4.1 Objective

One or two sentences stating what the test validates. Must be specific enough that success vs. failure is unambiguous.

Weak: "Verify the API is working."

Strong: "Verify that the central-server API `GET /events/search` endpoint returns a 200 response with a properly paginated JSON body containing events within the requested date range and respects the `limit` parameter."

### 4.2 Required Reading

A list of documents the agent must read before executing. Typical entries:
- The relevant `_reference.md` for the system under test
- The system architecture or deployment doc if the test touches infrastructure
- The `general_testing_methodology_reference.md` if the project has one
- Any prior `_investigation.md` files that establish the baseline this test extends

The agent does not begin Step 1 until Required Reading is complete. This section exists because agents that skip context-gathering produce wrong conclusions confidently.

### 4.3 Prerequisites

What must be true of the live system before the test can execute. Typically:
- Which services must be running
- Which credentials or tokens must be loaded
- Which data must already exist in the system
- Which command-line tools must be installed

Each prerequisite should include a verification command when applicable. Example: `run 'systemctl status <service>' and verify 'active (running)' before proceeding`.

A prerequisite that cannot be verified by command is a liability — the agent will skip checking it.

### 4.4 Steps

The test procedure as an ordered list of executable actions. Each step must be:

- **Concrete.** Exact command, exact payload, exact endpoint. Not "call the API."
- **Verifiable.** Each step should include the assertion that confirms the step succeeded, not only the action. A step with only an action is incomplete.
- **Atomic.** One action per step. If a step reads "run X, then verify Y," it's two steps.

An agent reading a step should need to make zero guesses about what to execute. If the agent needs to pick between "curl vs. httpie" or "which auth header format," the step is underspecified.

### 4.5 Expected Outcomes

What success looks like, in concrete terms. Specific values, not vague phrasings.

Weak: "The response should be successful."

Strong: "HTTP 200 with `Content-Type: application/json` and a body matching the shape `{events: array, next_offset: number, total: number}` where `events.length > 0` if the time window contains at least one event and `total >= events.length` always."

Include at least one specific verifiable value or pattern per expected outcome. An outcome without a matchable check cannot be verified by an agent.

### 4.6 Failure Indicators

What to check if the actual outcome doesn't match the expected. Typically organized as:

- Specific error messages and what they mean
- Log lines to grep for (with the exact grep commands and log file paths)
- Database queries to inspect state
- Related services whose status might affect this one

The purpose is to let an agent diagnose a failure in one pass without having to improvise. An improvising agent is a noise factory.

### 4.7 Delivery Requisites

What the agent must produce when the test is complete, regardless of pass or fail. Typically:

- A structured results report (specify the exact format: markdown table with given columns, JSON with given fields, etc.)
- A findings summary pointing to the underlying evidence
- For a benchmark: the raw metrics and the interpretation
- For a failure: the full failure context, not just "failed"

The agent is not done until the deliverable matches this section's specification. This section prevents the "the test passed" declaration with no evidence behind it.

---

## 5. Driver Options

A "driver" is the executable piece that actually performs the test against the live system. Drivers come in four shapes; choose the one that matches the test.

### 5.1 Inline (no drivers folder needed)

If the test consists of a handful of shell commands, `curl` calls, or `psql` queries that fit comfortably in the Steps section of the prompt, there is no separate driver. The prompt is the procedure.

When to use: simple health checks, API smoke tests, configuration verification.

### 5.2 Scripts (`drivers/*.sh` or equivalent)

Shell scripts, Python files, Node scripts that can be run directly.

When to use: multi-step procedures with reusable logic, or when the test needs file I/O, temporary files, or conditional logic that would be awkward in a prompt.

Layout: `tests/drivers/<driver-name>.<ext>` — flat folder, no `src/bin` split needed for scripts.

### 5.3 Compiled binaries (`drivers/src/` + `drivers/bin/`)

Go, Rust, or other compiled driver programs.

When to use: the test has non-trivial logic, is run repeatedly, benefits from typed parsing of API responses, or needs performance beyond what a shell script can deliver.

Layout: `tests/drivers/src/<driver-name>/` for source, `tests/drivers/bin/<driver-name>` for the built binary. Add a `.gitignore` in `drivers/` that ignores the contents of `drivers/bin/` so rebuilds stay reproducible and binaries don't pollute diffs. Document the build command either in the driver's local `README.md` or in the prompt's Prerequisites section.

### 5.4 LLM-orchestrated (no driver at all)

The prompt specifies the test objective and constraints; the agent executes using whatever tools the agent has available (bash, curl, filesystem access, existing project binaries).

When to use: exploratory validation, one-off diagnostic tasks, or any test where the value is in the agent's judgment on the results rather than in mechanical pass/fail.

The distinction between §5.1 and §5.4 is subtle but real: §5.1 is "the prompt contains the exact commands"; §5.4 is "the prompt describes the intent and lets the agent pick the commands." Reserve §5.4 for cases where the commands genuinely cannot be fixed in advance.

---

## 6. The Routing Table Pattern (`tests/README.md` Template)

Every `tests/README.md` should include a routing table that maps a task to the file the agent should read first. This is how agents navigate without rediscovering the folder's structure on each visit.

Minimal template:

```markdown
# Tests

Live-system testing for [project name] deployed infrastructure. This folder validates running production systems — not development code.

**Distinction:** code-level tests (unit tests, CI integration tests) live in their respective modules and validate code correctness. This folder validates that deployed systems behave correctly in production.

## Structure

tests/
├── documentation/   — guides, methodologies, prompts, references
├── drivers/         — executable test artifacts (if any)
└── investigations/  — test result snapshots, benchmark and validation records

## File Naming

`${area}_${description}_${type}.md`

**Area prefixes:** [list your project's prefixes, e.g. `api_`, `frontend_`, `db_`, `general_`]

**Type suffixes:** `_prompt`, `_reference`, `_methodology`, `_investigation`

## Routing Table

| When you need | Read this first |
|---|---|
| [task description] | `documentation/[file].md` |
| [task description] | `documentation/[file].md` |

Benchmark and validation results are stored in `investigations/`. Append new results; don't overwrite.

## Full methodology

See `documentation/general_live_system_testing_guidelines.md` for the methodology this folder implements.
```

The routing table is a living document. Every time a new `_prompt` or `_reference` is added, its row is added to the table. A `tests/` folder whose `README.md` hasn't been updated in six months is accruing undiscoverable tests.

---

## 7. Result Persistence

Test runs produce evidence. That evidence belongs in `tests/investigations/`, not in ephemeral output, chat logs, or developer memory.

Pattern for a `tests/investigations/` file:

- One file per durable finding or per benchmark baseline.
- Timestamped header: when the test ran, against what version or deployment, who (human or which agent) ran it.
- Evidence body: raw output, tables, anomalies observed, references to log files or database snapshots.
- Findings Summary at or near the top, in the same pattern as top-level investigations (see `general_investigation_review_guidelines.md` §13.2).

**Append new results; don't overwrite.** When a benchmark is re-run, either append the new results as a new section with its own timestamp, or open a new investigation file that cross-references the old one. Overwriting erases history; history is how regressions are detected.

When a benchmark or validation is obsolete (the system has changed enough that the old baseline no longer applies), move the file to `tests/archive/investigations/`. Do not delete — archived results are how regressions are detected later, and how auditors reconstruct past system states.

---

## 8. LLM-Agent Execution Considerations

Live-system tests are increasingly executed by LLM agents rather than by humans. The methodology is shaped for that consumer:

- **Required Reading exists because agents skip context.** An agent that hasn't read the relevant reference will make wrong assumptions with confidence. §4.2 forces the agent to gather context before executing.
- **Expected Outcomes must be matchable.** An agent cannot verify "the response should look right" — it needs exact values or matchable patterns. §4.5 enforces this.
- **Failure Indicators must be searchable.** An agent troubleshooting a failure needs structured search paths (exact grep commands, exact log file paths) rather than prose hints. §4.6 enforces this.
- **Delivery Requisites prevent "the test passed" with no evidence.** Agents declare victory readily; §4.7 forces a structured artifact that can be audited after the fact.
- **Prerequisites prevent running against the wrong environment.** An agent that runs a production test against a staging URL produces wrong results confidently. §4.3 with verification commands catches this before Step 1.

These five discipline patterns are why the 7-section structure is non-negotiable for `_prompt` files. Skipping any one of them invites the agent to improvise, and improvised test execution is test-shaped noise, not evidence.

---

## 9. When to Use This Methodology

Use this methodology for any of:

- Health checks of deployed services
- API endpoint validation against running services
- Benchmarks against real data or real traffic
- Regression tests against production or production-like environments
- Diagnostic procedures for operational issues
- Validation of deployment changes
- Any test that requires a live system to be running

Do **not** use this methodology for:

- Unit tests — those belong in the module alongside the code (e.g., `_test.go`, `test_*.py`).
- Integration tests that run in CI — those belong in the CI configuration and the module's test directory.
- UI/UX QA — use a dedicated QA methodology (see the `gstack` skill or equivalent).
- One-off ad-hoc commands that will never be re-run — just run them.

The bar for "belongs here" is: if this test might be re-run, by a different person or agent, at a different time, against a different version of the system, then it belongs in `tests/` with the 7-section prompt structure. If it's truly single-use, it doesn't.

---

## 10. Relationship to Other Methodology Artifacts

Live-system tests sit in a specific position in the methodology flow:

- **Downstream of plans.** A plan describes what to implement and ship. Once implementation is deployed, live-system tests prove it works in its actual environment. A plan's verification phase (see `general_plan_crafting_guidelines.md`) typically includes references to relevant live-system tests or specifies new tests to add.
- **Upstream of investigations.** A failing live-system test is a common trigger for opening a new top-level investigation (see `general_investigation_review_guidelines.md` §0 Activation Triggers). The investigation digs into why; the test detected that something was wrong.
- **Produces its own investigations.** Benchmark runs and validation campaigns produce investigation files in `tests/investigations/`. Those are their own durable artifacts, scoped to the test folder and independent of the top-level `investigations/` folder.

A project that has plans, documentation, and investigations but no `tests/` folder has no mechanism for proving that what was planned is what actually runs in production. That gap manifests as "we shipped it but we don't know whether it works against real data."

---

## 11. Completion Standard

### 11.1 A `_prompt` file is ready to execute when:

1. Objective is specific and verifiable.
2. Required Reading names concrete files (not "the relevant docs").
3. Prerequisites include verification commands.
4. Steps are atomic, concrete, and verifiable.
5. Expected Outcomes include specific matchable values.
6. Failure Indicators include searchable patterns (grep commands, log paths, SQL queries).
7. Delivery Requisites specify the exact format of the deliverable.

### 11.2 A `_reference` file is ready when:

- It contains enough context that a prompt can reference it without restating the same content.
- It does not prescribe a test procedure (that is what `_prompt` files are for).
- Its scope is clearly bounded (what it covers, what it does not).

### 11.3 A `_methodology` file is ready when:

- It describes an approach that applies to at least two different tests.
- It defines terminology used by the prompts and references that cite it.
- Its scope is clearly cross-cutting (not subsystem-specific).

### 11.4 An `_investigation` file (in `tests/investigations/`) is ready when:

- It has a timestamped header identifying when the test ran, against what version or deployment, and who ran it.
- It has a Findings Summary at or near the top.
- Its evidence is concrete (raw values, file paths, log excerpts) rather than prose.
- If it extends or supersedes a prior investigation, it cross-references the prior file.

---

## 12. Common Failure Modes

### 12.1 Prose Steps
Steps written as prose paragraphs instead of atomic ordered actions. Agents cannot reliably extract executable instructions from prose.

### 12.2 Vague Expected Outcomes
"The response should be successful" or "the service should be healthy." The agent has no matchable value to compare against and will declare success on any response that doesn't throw an exception.

### 12.3 Missing Failure Indicators
The prompt describes the happy path only. When the test fails, the agent improvises diagnostic commands and often picks wrong ones, producing noise instead of diagnosis.

### 12.4 No Delivery Requisites
The agent reports "the test passed" with no evidence. No record is produced. The work is not auditable after the fact.

### 12.5 Improvised Naming
Prompts named without the area prefix or type suffix. They become un-discoverable in the routing table. Eventually an agent creates a duplicate test because the original couldn't be found.

### 12.6 Results Overwrite
Benchmark results overwrite the previous run's file. No history is preserved. Regressions become invisible.

### 12.7 Drivers Without Source
Compiled binaries checked in without their `drivers/src/` counterpart. The test can be run but cannot be rebuilt, modified, or audited.

### 12.8 Uncommitted Tests
Tests exist in local scratch directories instead of `tests/`. No one else can run them. The discipline this methodology provides applies only to tests that are committed to the repo.

### 12.9 Stale Routing Table
The `tests/README.md` routing table was last updated months ago. New tests have been added but not registered. Agents doing task-to-file lookup miss the newer tests entirely.

---

**Last Updated:** 2026-04-19
**Status:** Complete
