# Plan Crafting Guidelines v2.0: The Unified Methodology

**Version:** 2.3.0
**Status:** Complete
**Last Updated:** 2026-02-01
**Purpose:** This document is the single, self-contained, mandatory standard for creating implementation plans. It supersedes all previous plan crafting guidelines and integrates all lessons from iterative plan development.

---

## Compliance Mandate

**A plan that does not meet these standards is not a plan—it is a sketch.** Every generated plan must be evaluated against the verification checklist in Section 11 before delivery. Any plan that fails verification must be regenerated.

---

## 0. LLM/Agent Directive (Machine-Readable Instructions)

**THIS SECTION IS SPECIFICALLY FOR LARGE LANGUAGE MODELS AND AI AGENTS.**

If you are an LLM or AI agent (Claude, Gemini, Grok, ChatGPT, Codex, or similar) that has been directed to this file with the intent of creating an implementation plan, the following instructions are **mandatory** and take precedence over any implicit assumptions about planning workflows.

### 0.1. The File Artifact Mandate

**THE PRIMARY GOAL IS TO CREATE A PLAN FILE.**

Regardless of what planning capabilities your runtime, harness, or agent framework provides (e.g., "plan mode," "reasoning mode," internal scratchpads), the output of this process is a **file artifact** located in the `plans/` directory.

- **Output:** A markdown file at `plans/<descriptive_name>_plan.md`
- **NOT Output:** Internal planning state, reasoning traces, or agent memory
- **Rationale:** The plan file is the deliverable that humans review, approve, and that powers subsequent implementation workflows

**The workflow is:**
1. Create the plan file artifact
2. User reviews and approves the plan
3. THEN the plan is used to guide implementation (potentially via agent planning features)

Do NOT skip step 1 by jumping directly into implementation. Do NOT treat your internal planning capabilities as a substitute for the file artifact.

### 0.2. Assumed Initial Instructions

When you encounter this guidelines document with the intent to create a plan, **assume the user has given you the following instruction** (derived from Section 14):

> "Create a complete, compliant implementation plan following the 4-Pass Methodology. Before you begin, identify the documentation file that describes the feature requiring a plan. Read that documentation file and ALL files it references before proceeding."

**Your first action must be:**
1. **If no documentation file was provided:** Ask the user for the path to the documentation file that describes the feature/system requiring a plan.
2. **If a documentation file was provided:** Read it completely.

### 0.3. Mandatory Pre-Reading Protocol

**Before writing ANY plan content, you MUST read:**

1. The primary documentation file (the feature specification)
2. Every file referenced in the "Related Documentation" or "See Also" sections
3. Every file path mentioned in code examples or configuration references
4. Any existing plan files in `plans/` that relate to the same system

**Rationale:** Plans that reference unread documentation inevitably contain inconsistencies, incorrect file paths, and contradictions. Reading first eliminates these errors.

### 0.4. Interaction Protocol

When creating a plan:

1. **Pass 1 (Skeleton):** Create structure, then STOP and present to user for review
2. **Pass 2 (Atomicity):** Refine actions, then STOP and present to user for review
3. **Pass 3 (Detail Enrichment):** Add context/code, then STOP and present to user for review
4. **Pass 4 (Verification):** Add verification commands, then STOP and present final plan

**Write the plan to the file after each pass.** The file is the source of truth, not your context window.

### 0.5. Summary Checklist for LLMs

Before proceeding, confirm:
- [ ] I will create a file at `plans/<name>_plan.md`
- [ ] I have identified or requested the source documentation file
- [ ] I will read ALL referenced files before writing plan content
- [ ] I will follow the 4-Pass Methodology with user checkpoints
- [ ] I understand the plan file is the deliverable, not my internal reasoning

---

## 1. Core Principles (Non-Negotiable)

### 1.1. Plans Are Language-Agnostic Algorithms

**The Primary Measure of Success:** "Could a competent developer use this plan to implement the feature in a different programming language (e.g., Go, Rust, TypeScript)?"

- If YES → It's a plan (describes algorithm, structure, contracts)
- If NO → It's an implementation disguised as a plan

| Aspect | A Plan Is... | A Plan Is NOT... |
|--------|--------------|------------------|
| **Nature** | A description of WHAT and WHY | A script describing HOW in one specific language |
| **Language** | Language-agnostic algorithms | Language-specific code |
| **Durability** | Survives technology changes | Brittle to library/framework changes |
| **Purpose** | Tells someone HOW TO THINK | Tells someone WHAT TO TYPE |
| **Transferability** | Works for Python, Rust, Go, TS | Locked to one language |

**Example - Algorithm Steps vs. Code:**

Plan (language-agnostic):
```markdown
**Algorithm:**
1. Record start time
2. Load image, extract dimensions
3. Run inference with gradients disabled
4. Post-process to extract bounding boxes
5. Convert absolute pixel coords to percentage coords
6. Return boxes and elapsed time
```

Implementation (language-specific - NOT a plan):
```python
start_time = time.perf_counter()
image = Image.open(image_path).convert("RGB")
width, height = image.size
with torch.no_grad():
    outputs = model(**inputs)
# ... 30 more lines
```

The algorithm steps work for ANY language. The Python code works ONLY for Python with specific libraries.

**Guiding Principle:**
> A plan should tell someone HOW TO THINK about implementing something, not provide the implementation itself.

#### Exception: Language-Locked Plans

The language-agnostic principle exists to prevent plans from becoming "pre-written code printers." However, some plans are **inherently language-locked** due to:

1. **Library dependencies** - Plans using Python-only libraries (e.g., `transformers`, `ultralytics`, `torch`) cannot be implemented in Go or Rust
2. **Framework requirements** - Plans for React components or Django views are inherently JavaScript/Python
3. **Toolchain constraints** - Prerequisites that verify specific language versions (e.g., "Check Python 3.10+ installed")

**For language-locked plans:**
- Language-specific code snippets in Algorithm sections are **acceptable**
- The spirit of the guideline is preserved as long as:
  - Complete runnable implementations are NOT provided (use stubs with `pass` or `// TODO`)
  - Code snippets are illustrative and scoped to individual checklist items
  - The plan still describes WHAT to do, not just WHAT TO TYPE verbatim

**Test for compliance:** Does the plan provide complete copy-paste implementations, or does it provide guidance with illustrative snippets? The former violates the spirit; the latter is acceptable.

**Example - Acceptable for a Python-locked plan:**
```markdown
- [ ] **Add get_batch_images function**
    - **Algorithm:** Return `sorted(batch_dir.glob("*.jpg"))`
```

This is acceptable because:
- The plan's prerequisites verify Python installation
- The snippet is illustrative (one line), not a complete implementation
- A developer still needs to write the full function

### 1.2. The Documentation-First Mandate

A plan is a "how to build" document derived from a "what to build" document.

**Before any plan work begins:**
1. The feature/system must have complete documentation
2. The documentation must reference canonical files for shared concepts
3. The plan implementer must read all referenced documentation

**Documentation Readiness Assessment:**

Before starting a plan, the source documentation must answer YES to all:
- [ ] Are all data structures defined with field types?
- [ ] Are all configuration values specified (not "configure as needed")?
- [ ] Are all external dependencies listed with versions?
- [ ] Are all file paths specified?
- [ ] Are code patterns shown for complex operations?

If any answer is "no", **enhance the documentation first**. Plan creation often reveals documentation gaps. The correct response is to update documentation first, then continue planning—never encode undocumented decisions directly in the plan.

### 1.3. The Zero-Ambiguity Mandate

**If a developer must make a decision, guess, or infer anything, the plan has failed.**

The test: "If two competent developers would implement an action differently, it lacks sufficient detail."

Every step must be precise and unambiguous. No action should leave implementation choices to the executor.

### 1.4. The Verifiability Mandate

Every action and phase in a plan should be verifiable with a binary yes/no answer. The plan must include exact commands and expected outputs to prove completion.

### 1.5. Formal Structure

Consistent hierarchical format with no deviations. See Section 4 for the mandatory structure.

---

## The Documentation Lifecycle

Plans do not exist in isolation. They are one stage in a larger workflow:

```
Interview → Documentation (canonical truth) → Plan (references docs) → Execution
              ↑                                       ↑
              Creates/enriches                        Follows this document
```

### Documentation as Canonical Truth

The `documentation/` folder contains the **canonical source of truth** for what a feature IS:
- Requirements and constraints
- Decisions and their rationale
- Technical specifications
- Edge cases and failure modes

Plans reference this documentation. They describe HOW to build what the documentation defines.

### The Interview Process

Before documentation exists, it must be created through structured interviews. The interview process serves these purposes:

| Goal | Description |
|------|-------------|
| **Gather** | Requirements, constraints, preferences |
| **Decide** | Force resolution of ambiguities upfront - no "TBD" allowed |
| **Surface blind spots** | "Have you considered...?" |
| **Identify second-order effects** | "If X, then Y will also need..." |
| **Expose unintended consequences** | "This approach means Z won't work" |
| **Document** | Capture everything in canonical form |

The interview is complete when:
- All functional requirements are specified
- All technical constraints are identified
- All ambiguities are resolved with decisions
- All edge cases are documented
- Second-order effects are acknowledged
- Output is written to `documentation/<prefix>_<feature_name>_documentation.md`

### Why This Matters

A plan cannot achieve "zero ambiguity" (Section 1.3) if the underlying requirements are ambiguous. The interview process front-loads decision-making so that plans can be purely mechanical.

**Violation:** Creating a plan without canonical documentation forces the plan author to make undocumented decisions, which violates traceability and creates implicit knowledge.

---

## 2. The 4-Pass Methodology

Compliant plans **must** be created using this iterative, four-pass methodology. Each pass builds upon the last to progressively refine the plan from a high-level skeleton to a verifiable, zero-ambiguity blueprint.

| Pass | Name | Input | Output | Key Question |
|------|------|-------|--------|--------------|
| **1** | Skeleton | Source Documentation | Structure + Compound Actions | "What are all the things we need to do?" |
| **2** | Atomicity | Skeleton Plan | Atomic Actions | "Can this be executed in 5 seconds without thinking?" |
| **3** | Detail Enrichment | Atomic Plan | Enriched Actions with Context | "Is there any ambiguity left?" |
| **4** | Verification | Enriched Plan | Verifiable Actions | "How do we prove each action is done?" |

### 2.1. Pass 1: Skeleton

**Purpose:** Establish complete coverage without drowning in detail.

**Process:**
1. Read source documentation end-to-end
2. Identify all phases needed (map to 9-phase template in Section 4)
3. For each phase, list objectives
4. For each objective, list what needs to happen (compound actions are OK at this stage)
5. Add dependencies between objectives

**What compound actions look like at this stage:**
```markdown
- [ ] **Define StagedState**: Add TypedDict with fields for tracking staged images
- [ ] **Define validate_batch_dir**: Function that checks directory exists, contains state.json, contains .jpg files
```

These are acceptable in Pass 1 because they establish WHAT needs to happen without HOW details.

**Alignment Check:** Every objective should trace to a section in the documentation.

**Pass 1 Completion Criteria:**
- [ ] All documentation sections have corresponding plan objectives
- [ ] No orphan objectives (everything traces to documentation)
- [ ] Dependencies form a valid DAG (no cycles)
- [ ] No forbidden phrases from Section 9

**What NOT to do in Pass 1:**
- Don't try to make actions atomic yet
- Don't add code blocks yet
- Don't worry about line numbers yet

### 2.2. Pass 2: Atomicity

**Purpose:** Transform compound actions into single, executable steps.

**The 5-Second Rule:**
> If a developer reads an action and cannot begin executing within 5 seconds, the action is not atomic enough.

**Process:**
1. Read each action from Pass 1
2. Ask: "Does this require multiple operations?"
3. If yes, split into separate actions
4. Add file:line references where applicable
5. Ensure single verb per action

**Atomicity Requirements - Every action must:**
1. **Have a Single, Clear Intent:** The title contains exactly one goal
2. **Reference a Single Location:** The action applies to one file and one location
3. **Use a Single Verb:** Contains exactly one action verb (create, add, modify, delete, validate)
4. **Be Binary:** Can be marked "done" with a simple yes/no
5. **Contain No Conditionals:** No "if," "when," or "as needed" in the title

#### Atomicity Patterns

**Pattern 1: TypedDict/Struct Field Expansion**

BEFORE (compound - violates atomicity):
```markdown
- [ ] **Add Sam3ProcessedState Fields**: Add fields `completed_images: list[str]`, `predictions_file: str`, `count: int`, `last_updated: str`
```

AFTER (atomic - compliant):
```markdown
- [ ] **Add Sam3ProcessedState Class Definition**: Add `class Sam3ProcessedState(TypedDict):` at line 14
- [ ] **Add Sam3ProcessedState Docstring**: Add docstring describing state after Stage 2
- [ ] **Add Sam3ProcessedState completed_images Field**: Add `completed_images: list[str]` field
- [ ] **Add Sam3ProcessedState predictions_file Field**: Add `predictions_file: str` field
- [ ] **Add Sam3ProcessedState count Field**: Add `count: int` field
- [ ] **Add Sam3ProcessedState last_updated Field**: Add `last_updated: str` field
```

**Why:** Each field is a separate operation. A developer can check off each action. Compound field lists require parsing and judgment.

**Pattern 2: Import Statement Specificity**

BEFORE (vague):
```markdown
- [ ] **Add Validation Imports**: Add validation function imports at line 18
```

AFTER (specific):
```markdown
- [ ] **Add Validation Imports**: Add `from pipeline_validation import validate_model_path, validate_device, ValidationError` at line 18
```

**Why:** "validation function imports" requires the developer to determine which functions. The specific version is copy-paste ready.

**Pattern 3: Function Definition Expansion**

BEFORE (compound):
```markdown
- [ ] **Define validate_batch_dir**: Function that checks directory exists, contains state.json, contains .jpg files
```

AFTER (atomic):
```markdown
- [ ] **Add validate_batch_dir Signature**: Add `def validate_batch_dir(batch_dir: Path) -> bool:` at line 18
- [ ] **Add validate_batch_dir Docstring**: Add docstring explaining validation purpose
- [ ] **Add validate_batch_dir Directory Check**: Add check that raises ValidationError if directory doesn't exist
- [ ] **Add validate_batch_dir State File Check**: Add check that raises ValidationError if state.json missing
- [ ] **Add validate_batch_dir Images Check**: Add check that raises ValidationError if no .jpg files
- [ ] **Add validate_batch_dir Return**: Add `return True`
```

**Why:** "Function that checks X, Y, Z" describes behavior but not implementation steps. Each check is a separate operation.

**Pattern 4: Decorator and Class Separation**

BEFORE:
```markdown
- [ ] **Add ImagePrediction Decorator and Class**: Add `@dataclass` followed by `class ImagePrediction:`
```

AFTER:
```markdown
- [ ] **Add ImagePrediction Decorator**: Add `@dataclass` decorator
- [ ] **Add ImagePrediction Class Definition**: Add `class ImagePrediction:` class declaration
```

**Why:** "and" in an action is a signal it should be split.

#### Atomicity Anti-patterns

| Anti-pattern | Problem | Fix |
|--------------|---------|-----|
| "Add fields X, Y, Z" | Multiple operations | One action per field |
| "Add imports at line N" | Unspecified imports | List exact imports |
| "Function that does A, B, C" | Multiple checks | One action per check |
| "X and Y" | Two operations | Split into two actions |
| "Add validation" | Unspecified | Name exact validation |
| "Handle errors" | Unspecified | One action per error type |
| "Update state" | Unspecified fields | Specify each field |

**Pass 2 Completion Criteria:**
- [ ] Every action has exactly one verb
- [ ] Every action references exactly one location
- [ ] No action contains "and" joining operations
- [ ] No embedded conditionals (if/when/as needed)
- [ ] File paths match documentation exactly

### 2.3. Pass 3: Detail Enrichment

**Purpose:** Eliminate all remaining ambiguity by adding context without writing full implementations.

**Critical Discovery:** The purpose is NOT to write complete code. It is to ensure zero ambiguity while maintaining language-agnostic algorithms.

**Process:**
1. For each objective or major section, add context fields
2. For every new file, add a boilerplate stub (NOT a complete implementation)
3. Ensure complex actions include rationale

#### The Enriched Atomic Action Pattern

This pattern resolves the conflict between language-agnostic plans and 5-second actionability. **Separate the intent (what to do) from the implementation hint (how to do it).**

**Structure:**
- **Action Title (The Intent):** A concise, language-agnostic description of the goal
- **Action Body (The Implementation Hint):** Context and a language-specific code snippet

**Example:**
```markdown
- [ ] **Check for batch directory existence**
    - **Rationale:** Fail fast if the source directory doesn't exist before any processing.
    - **Algorithm:**
        1. Check if the `batch_dir` path exists on the filesystem.
        2. If it does not exist, raise a `ValidationError` with the appropriate error code.
    - **Implementation (Python):**
      ```python
      if not batch_dir.exists():
          raise ValidationError(
              PipelineError.STATE_FILE_NOT_FOUND,
              f"Batch directory not found: {batch_dir}"
          )
      ```
    - **Prerequisite:** `ValidationError` class defined in Section 2.5
```

#### Enrichment Elements

| Element | When to Add | What It Answers |
|---------|-------------|-----------------|
| **Rationale** | Non-obvious approach chosen | "Why this way and not another?" |
| **Algorithm** | Complex multi-step logic | "What are the logical steps?" |
| **Implementation** | Any non-trivial action | "What does this look like in the target language?" |
| **Prerequisite** | Action depends on prior work | "What must exist before I can do this?" |
| **Assumption** | Design decision made | "What choice was made here?" |
| **Presumption** | External dependency assumed | "What must be true in the environment?" |

#### When to Add Each Element

**Add Rationale when:**
- Multiple valid approaches exist
- The chosen approach might seem odd without context
- Future maintainers might question "why not X?"

**Add Prerequisite when:**
- Action uses something defined elsewhere in plan
- Action requires a file/class/function from earlier section
- Order of execution matters

**Add Assumption when:**
- A design decision was made (not just implementing docs)
- Behavior in edge cases is defined
- Default values are chosen

**Add Presumption when:**
- External tools/libraries required
- Environment configuration assumed
- Version requirements exist

#### Boilerplate Stubs vs. Complete Code

Plans do NOT contain final, complete implementation code. For every new file, the plan provides a **Boilerplate Stub**.

**A compliant boilerplate stub MUST contain:**
- The file's header, docstring, and all necessary imports
- Empty or stubbed-out function and class definitions with full type-hinted signatures
- `# Implementation stub` or `pass` where logic will be added

**A compliant boilerplate stub MUST NOT contain:**
- The final, complete implementation logic
- Any code beyond a minimal starting point

**Example - Compliant Boilerplate Stub:**
```python
# File: scripts/pipeline_state.py
"""Pipeline state file management utilities."""

from pathlib import Path
import json
from datetime import datetime, timezone
from .pipeline_types import PipelineState

PIPELINE_STATE_FILENAME = "pipeline_state.json"

def load_pipeline_state(batch_dir: Path) -> PipelineState:
    """Load pipeline state from batch directory, creating empty state if not exists."""
    # Implementation stub
    pass

def save_pipeline_state(batch_dir: Path, state: PipelineState) -> None:
    """Save pipeline state atomically using temp file and rename."""
    # Implementation stub
    pass
```

The atomic checklist items from Pass 2 guide what goes inside each stub.

**Pass 3 Completion Criteria:**
- [ ] Every new file has complete boilerplate stub
- [ ] Every function has typed signature
- [ ] Complex actions have rationale
- [ ] Dependent actions have prerequisites
- [ ] No action leaves implementation choice to executor
- [ ] Code blocks match documentation patterns

### 2.4. Pass 4: Verification

**Purpose:** Make every action provably complete.

**The Binary Test:**
> Can completion be verified with yes/no, requiring no judgment?

**Process:**
1. For each action, add a verification command or expected outcome
2. For each phase, add phase-level verification checklist
3. Ensure all verifications are executable (not just "check that...")

**Verification Patterns:**

| Action Type | Verification Pattern |
|-------------|---------------------|
| Create file | `test -f path && echo exists` |
| Add function | `grep "def function_name" path` |
| Add class | `grep "class ClassName" path` |
| Syntax check | `python3 -m py_compile path` |
| Import check | `python3 -c "from module import X"` |
| Type check | `mypy path` |
| API endpoint | `curl -s url \| jq .field` |

**Example - Verified Action:**
```markdown
- [ ] **Add Index**: Create index on `users.email` column
    - **Location**: `migrations/005_add_user_email_index.sql`
    - **Verification Command**:
      ```sql
      SELECT indexname, indexdef
      FROM pg_indexes
      WHERE tablename = 'users' AND indexname = 'idx_users_email';
      ```
    - **Expected Output**: 1 row with index definition containing `btree (email)`
```

**Pass 4 Completion Criteria:**
- [ ] 50%+ of actions have verification commands
- [ ] Each phase has verification checklist
- [ ] All verification commands are copy-paste executable

---

## 3. Cross-Pass Consistency Rules

### 3.1. Terminology Consistency

**Rule:** Once a term is used, it must be used identically throughout.

| Aspect | Example of Violation | Correct Approach |
|--------|---------------------|------------------|
| File names | `pipeline_state.py` vs `state.py` | Pick one, use everywhere |
| Function names | `load_state` vs `loadState` | Match documentation convention |
| Variable names | `batch_dir` vs `batchDir` vs `batch_directory` | Pick one |
| Class names | `StagedState` vs `StageState` | Match documentation exactly |

### 3.2. Path Consistency

**Rule:** File paths must match documentation exactly.

**Process:**
1. Documentation defines canonical paths
2. Plan references those paths verbatim
3. If plan needs different path, update documentation first

### 3.3. Reference Consistency

**Rule:** When referencing prior actions, use exact section/action identifiers.

- Good: "Requires `ValidationError` class from Section 2.5"
- Bad: "Requires the error class defined earlier"

### 3.4. Plan-Documentation Alignment Protocol

**Before Each Pass:**
1. Re-read relevant documentation sections
2. Note any terminology used
3. Note any file paths specified
4. Note any code patterns shown

**After Each Pass:**
1. Verify all terms match documentation
2. Verify all paths match documentation
3. Verify any code patterns follow documentation examples
4. If discrepancy found: update plan to match docs (or update docs first if docs are wrong)

**Handling Discrepancies:**

| Discrepancy Type | Resolution |
|------------------|------------|
| Plan uses different term than docs | Update plan to match docs |
| Plan needs path not in docs | Update docs first, then plan |
| Plan reveals docs gap | Document gap, update docs, continue |
| Plan contradicts docs | Stop - resolve before continuing |

---

## 4. Mandatory Plan Structure

### 4.1. Required Hierarchy

```markdown
# [Feature Name]: Implementation Plan
**Plan Version:** [semantic version]
**Estimated Atomic Actions:** [exact count]
**Last Updated:** [ISO 8601 timestamp]

## Required Reading
- Primary: [path to source documentation]
- Related: [paths to canonical files]

## Plan Development Status
[See Section 5]

## Compliance Checklist
- [ ] All file paths are absolute from project root
- [ ] All code snippets are syntactically valid
- [ ] All SQL is parameterized with exact types
- [ ] Every objective has 5+ atomic actions
- [ ] Zero instances of "implement", "build", "create" without specifics

## I. [Phase Name]
### 1.1: [Objective Title]
- **Objective:** [One sentence, max 20 words]
- **Success Criteria:** [Exact, testable condition]
- **Dependencies:** [List of blocking tasks by ID]
- **Estimated Actions:** [Count]
- **Rationale:** [Why this approach]
- **Assumption:** [Design decisions made]
- **Presumption:** [Environmental requirements]

<details>
<summary><strong>Specifications</strong></summary>
[Algorithm, data structures, contracts - language-agnostic]
</details>

- **Checklist:**
    - [ ] **[Action Category]**: [Language-agnostic intent]
        - **Implementation (Python):** [Code snippet]
        - **Verification:** [Command and expected output]
```

### 4.2. Format Violations That Invalidate a Plan

- Any heading without numbering
- Any objective without success criteria
- Any checklist item without a category prefix
- Any code block without language tag and file path comment
- Any "TODO" or "TBD" marker
- Any phrase "as needed", "if necessary", "appropriately"

### 4.3. The 9 Mandatory Phases

Every plan must include ALL of these phases, even if one phase is "No changes required" with rationale:

**Phase 0: Pre-Implementation**
- [ ] Verify Current State: List exact files and DB schema to be verified
- [ ] Backup Strategy: Exact commands for backing up affected data
- [ ] Feature Flag: Define flag name, default value, and check locations

**Phase I: Database Layer**
- [ ] Schema Verification: SQL queries to assert current schema
- [ ] Migration Script: Complete up/down migration with transaction
- [ ] Seed Data: If applicable, exact seed data script
- [ ] Index Creation: With EXPLAIN ANALYZE performance verification

**Phase II: Type Definitions & Contracts**
- [ ] Shared Types: Complete interfaces for all data structures
- [ ] API Contracts: OpenAPI/Swagger snippet or TypeScript types
- [ ] Error Codes: Enum or constant definitions
- [ ] Validation Schemas: Complete Zod/Joi schemas

**Phase III: Backend Implementation**
- [ ] Utility Functions: Pure functions with no side effects
- [ ] Data Access Layer: Database query functions
- [ ] Business Logic: Core feature logic
- [ ] API Routes: HTTP handlers with complete middleware chain
- [ ] Background Jobs: If applicable, job definitions with retry logic

**Phase IV: Frontend Implementation**
- [ ] Utility Hooks: Custom React hooks
- [ ] Shared Components: Reusable UI components
- [ ] Feature Components: Feature-specific components
- [ ] Page Integration: Modifications to existing pages
- [ ] State Management: Redux/Zustand/Context setup
- [ ] API Client: Fetch functions with error handling

**Phase V: Integration & Middleware**
- [ ] Middleware Updates: Modifications to auth, logging, etc.
- [ ] Background Job Registration: Cron or queue setup
- [ ] External Service Integration: Third-party API setup
- [ ] WebSocket/Realtime: If applicable, event handlers

**Phase VI: Testing (Mandatory Detail)**
- [ ] Unit Tests: One test case per function with exact assertions
- [ ] Integration Tests: Database round-trip tests
- [ ] API Tests: One test per endpoint per status code
- [ ] UI Tests: Component rendering and interaction tests
- [ ] E2E Tests: Complete user flows with exact selectors

**Phase VII: Observability**
- [ ] Logging: Exact log message formats with severity levels
- [ ] Metrics: Metric names, types (counter/gauge/histogram), and units
- [ ] Alerts: Threshold definitions and notification channels

**Phase VIII: Documentation**
- [ ] API Documentation: OpenAPI/Swagger updates
- [ ] README Updates: Changes to project README
- [ ] Runbook: Troubleshooting steps for common failures

**Phase IX: Deployment**
- [ ] Environment Variables: Exact names and example values
- [ ] Migration Commands: Exact sequence with rollback steps
- [ ] Deployment Checklist: Pre-flight and post-flight verification
- [ ] Rollback Plan: Exact steps to revert changes

---

## 5. Plan Development Tracking

Every plan should include a "Plan Development Status" section:

```markdown
## Plan Development Status

### Development Passes

| Pass | Focus | Status |
|------|-------|--------|
| Pass 1: Skeleton | Structure + compound actions | Complete |
| Pass 2: Atomicity | Single-verb, single-location actions | Complete |
| Pass 3: Detail Enrichment | Code blocks, rationales, prerequisites | In Progress |
| Pass 4: Verification | Verification commands | Pending |

### Current State
- **Current Pass:** Pass 3 (Detail Enrichment)
- **Next Pass:** Pass 4 (Verification)
- **Blocking Issues:** None

### Pass Completion Criteria

**Pass 1 (Skeleton):**
- [x] All phases present
- [x] Each objective has success criteria
- [x] Dependencies mapped

**Pass 2 (Atomicity):**
- [x] Every action single verb
- [x] Every action single location
- [x] No compound "and" actions

**Pass 3 (Detail Enrichment):**
- [ ] Every new file has boilerplate
- [ ] Complex actions have rationale
- [ ] Dependencies have prerequisites

**Pass 4 (Verification):**
- [ ] 50%+ actions have verification
- [ ] Phase checklists complete
```

**Why Track This:**
1. **Resumability:** Allows pausing and resuming plan development
2. **Visibility:** Makes progress observable
3. **Quality:** Ensures each pass is complete before moving on
4. **Audit:** Provides trail of plan development

---

## 6. Mandatory Specifications by Element Type

### 6.1. New File Specifications

Every plan that creates a new file must specify:

```markdown
- [ ] **Create File**: `src/exact/path/to/file.ts`
    - **Purpose**: [One sentence]
    - **Exports**: [List all exported items with types]
    - **Dependencies**: [Every import with version if external]
    - **Boilerplate:**
      ```typescript
      // Exact starting code with all imports
      import { Type } from 'library';

      export function functionName(param: Type): ReturnType {
        // Implementation stub
      }
      ```
```

### 6.2. Function Specifications

Every function must specify:

```markdown
- [ ] **Define Function**: `functionName` in `src/file.ts`
    - **Signature**: `(param1: Type1, param2: Type2) => ReturnType`
    - **Purpose**: [One sentence, max 15 words]
    - **Algorithm**:
        1. [Step 1 with exact operation]
        2. [Step 2 with exact operation]
    - **Edge Cases**:
        - `null` input: [exact behavior]
        - Empty array: [exact behavior]
    - **Example**:
      ```typescript
      functionName(input1, input2);
      // Expected output: [exact value or structure]
      ```
```

### 6.3. Database Operation Specifications

Every database operation must specify:

```markdown
- [ ] **Execute Query**: [Description] in `src/file.ts`
    - **Operation Type**: [SELECT | INSERT | UPDATE | DELETE | UPSERT]
    - **Tables**: [List with aliases]
    - **Parameters**:
      ```typescript
      {
        param1: string;  // Example: "user@example.com"
        param2: number;  // Example: 42
      }
      ```
    - **Query**:
      ```sql
      SELECT u.id, u.name, COUNT(o.id) as order_count
      FROM users u
      LEFT JOIN orders o ON o.user_id = u.id
      WHERE u.email = $1 AND u.status = $2
      GROUP BY u.id, u.name;
      ```
    - **Expected Result**:
      ```typescript
      Array<{ id: number; name: string; order_count: number; }>
      ```
    - **Index Requirements**: [List required indexes]
    - **Error Handling**:
        - Duplicate key: [exact response]
        - Foreign key violation: [exact response]
        - Connection timeout: [exact response]
```

### 6.4. Configuration Value Specifications

Every plan that introduces new values must specify whether they are configurable or constant:

```markdown
- [ ] **Configuration Requirement**: Identify whether value is configurable or constant
    - **Value**: [The value being introduced]
    - **Type**: [Environment Variable | Config File | Named Constant | Hardcoded]
    - **Justification**: [Why this classification]
    - **Location**: [Where defined - e.g., `.env`, `config.js`, inline constant]
```

**Classification Guidelines:**

| Classification | When to Use | Examples |
|----------------|-------------|----------|
| Environment Variable | Deployment-specific values | Ports, hostnames, credentials, API keys |
| Config File | Settings that may need runtime changes | Feature flags, thresholds, timeouts |
| Named Constant | Domain-specific fixed values | Protocol message types, schema names |
| Hardcoded | Truly immutable values | HTTP status codes, `Math.PI` |

### 6.5. Complete Type Definitions

All type definitions must be complete:

- **Database**: Complete `CREATE TABLE` for every new table
- **Database**: Complete `ALTER TABLE` for every modification
- **Types**: Complete TypeScript interface (no `any` types)
- **API Contracts**: Complete request/response JSON examples
- **Environment**: Complete `.env.example` entries
- **Validation**: Complete validation schema (Zod/Joi/etc)
- **Error Codes**: Complete error enum or constant definitions

---

## 7. Detail Multiplication Rules

When encountering these patterns, **multiply detail** by specified factor:

### 7.1. Creating an API Endpoint (Minimum 15 Actions)

Must decompose into:
- Route file creation (1)
- Type imports (1-3)
- Handler structure (1)
- Each HTTP method (1 each)
- Auth middleware (1)
- Request parsing (1)
- Each validation rule (1 per field)
- Database query (1)
- Success response (1)
- Each error case (1 per error type)
- Logging (1)

### 7.2. Creating a Component (Minimum 12 Actions)

Must decompose into:
- File creation (1)
- Import section (1)
- Props interface (1)
- Each state variable (1 each)
- Each effect (1 each)
- Each handler function (1 each)
- JSX/template structure (1)
- Styling (1)
- Export (1)

### 7.3. Adding a Database Table (Minimum 8 Actions)

Must decompose into:
- CREATE TABLE statement (1)
- Each column definition (1 per column)
- Each constraint (1 each)
- Each index (1 each)
- Migration file creation (1)
- Rollback migration (1)
- Type definition (1)

### 7.4. Writing a Test Suite (Minimum N + 4 Actions)

Where N = number of test cases:
- Test file creation (1)
- Test setup/teardown (1)
- Each individual test case (1 each)
- Test data fixtures (1)
- Assertion helpers if needed (1)

---

## 8. Plan Execution Protocol

**CRITICAL: When implementing a plan, EVERY instruction is limited to the current phase/section being worked on.**

### 8.1. Mandatory Section Transition Rules

1. **STOP at Section Boundaries**: When completing any phase or section (e.g., Phase II.6, Phase III), STOP immediately and report completion.

2. **REQUIRE Explicit Permission**: NEVER proceed to the next phase/section without explicit user confirmation.

3. **Interpret "Continue" Correctly**:
   - "Continue" means **continue ONLY until the end of the current section**
   - "Continue" does NOT mean continue through all remaining phases
   - When user says "continue from where you left off", complete ONLY the current section/phase

4. **Report Before Proceeding**:
   ```
   ✓ Phase [X] complete
   - Summary: [brief summary of what was done]
   - Files modified: [list]
   - Verification: [status]

   Ready to proceed. What would you like me to do next?
   ```

5. **Examples**:
   - ❌ **WRONG**: Agent finishes Phase II.6 and immediately starts Phase III
   - ✅ **CORRECT**: Agent finishes Phase II.6, reports completion, waits for explicit instruction

### 8.2. Scope Boundaries

- **Phase**: A major section (e.g., "Phase II: Main Component Restructure")
- **Section**: A subsection (e.g., "Phase II.6: Event Data Enhancement")
- **Rule**: Complete the current numbered item and STOP

### 8.3. Why This Matters

Plans can be large with many phases. Users need:
- Control over when each phase starts
- Ability to review work before proceeding
- Opportunity to provide feedback between phases
- Time to test/verify before moving forward

**VIOLATION OF THIS PROTOCOL IS A CRITICAL ERROR** that disrupts user workflow.

---

## 9. Forbidden Phrases (Auto-Fail)

Plans containing these phrases are **automatically invalid**:

### Tier 1 Violations (Vague Actions)
- "Implement [anything]"
- "Build [anything]"
- "Create [anything]" (without exact file path)
- "Add [anything]" (without exact location)
- "Handle [anything]"
- "Process [anything]"
- "Set up [anything]"
- "Configure [anything]"
- "Integrate [anything]"

### Tier 2 Violations (Ambiguous Qualifiers)
- "as needed"
- "if necessary"
- "appropriately"
- "properly"
- "correctly"
- "sufficiently"
- "where applicable"
- "as required"
- "similar to"
- "based on"

### Tier 3 Violations (Incomplete Specifications)
- "TODO"
- "TBD"
- "see documentation"
- "refer to"
- "check existing"
- "follow convention"
- "use standard"
- "implement best practices"

### Tier 4 Violations (Lazy Abstractions)
- "and other [anything]"
- "various [anything]"
- "relevant [anything]"
- "necessary [anything]"
- "appropriate [anything]"
- "etc."
- "..."

---

## 10. Quality Metrics

A compliant plan should achieve these numerical targets:

| Metric | Minimum | Target | Exceptional |
|--------|---------|--------|-------------|
| Atomic actions per objective | 5 | 10 | 15+ |
| Code block count | 10 | 25 | 50+ |
| SQL queries included | 5 | 10 | 20+ |
| TypeScript/Python interfaces | 5 | 10 | 20+ |
| Test cases specified | 10 | 25 | 50+ |
| Verification commands | 20% of actions | 50% of actions | 75%+ |
| Lines per atomic action | 1 | 2-3 | 5+ with verification |
| File paths specified | 100% | 100% | 100% |
| Total action count | 50 | 150 | 300+ |

---

## 11. Pre-Delivery Verification Checklist

Before delivering any plan, verify **ALL** of these conditions:

### 11.1. Structural Compliance
- [ ] Every section follows exact format from Section 4
- [ ] All headings are numbered hierarchically
- [ ] All code blocks have language tags and file path comments
- [ ] All file paths are absolute from project root
- [ ] Plan includes all 9 mandatory phases
- [ ] Each phase has at least one objective
- [ ] Each objective has at least 5 atomic actions

### 11.2. Atomicity Compliance
- [ ] Every action passes the 5-second rule
- [ ] No action contains multiple verbs
- [ ] No action contains conditional language
- [ ] Action count is at least 3x the number of major features
- [ ] Every action has a category prefix (e.g., "**Create File**:", "**Add Function**:")

### 11.3. Content Completeness
- [ ] All database tables have complete CREATE TABLE statements
- [ ] All TypeScript interfaces are fully defined (no `any` types)
- [ ] All API endpoints have complete request/response examples
- [ ] All functions have signatures, purposes, and examples
- [ ] All error cases have specific status codes and messages
- [ ] All environment variables are documented with examples

### 11.4. Zero Ambiguity
- [ ] No forbidden phrases from Section 9 exist
- [ ] No "TODO" or "TBD" markers exist
- [ ] No external references without embedded content
- [ ] All decisions are pre-made (no options presented)
- [ ] All examples use realistic, specific data

### 11.5. Verification Capability
- [ ] At least 50% of actions have verification commands
- [ ] Each phase has a verification checklist
- [ ] All database changes have assertion queries
- [ ] All API endpoints have test case references

### 11.6. Language Agnosticism
- [ ] Action titles describe WHAT, not language-specific HOW
- [ ] Algorithms are language-agnostic
- [ ] Implementation hints are clearly marked as language-specific
- [ ] Plan could be used to implement in a different language

---

## 12. Plan File Naming Convention

### 12.1. Required Suffix

All plan files in the `/plans/` folder must end with `_plan.md`. This suffix:
- Clearly identifies the file as a planning document
- Distinguishes from documentation, code, or other artifacts
- Enables consistent file discovery via glob patterns

### 12.2. Pattern

```
descriptive_name_plan.md
```

### 12.3. Valid Examples

| Filename | Purpose |
|----------|---------|
| `mmolid_migration_plan.md` | Migration plan for MMOLID feature |
| `station_view_implementation_plan.md` | Implementation plan for station view |
| `typed_3d_marker_system_plan.md` | Plan for 3D marker system |

### 12.4. Invalid Examples

| Invalid | Why |
|---------|-----|
| `mmolid_migration.md` | Missing `_plan` suffix |
| `station_view_planning.md` | Wrong suffix |
| `notes.md` | Too vague, no suffix |

### 12.5. File Location

All plan files go in the `/plans/` folder. Completed plans should be moved to `/archive/plans/` (retaining the `_plan.md` suffix).

---

## 13. Example: Maximum Detail Demonstration

### 13.1. INSUFFICIENT DETAIL

```markdown
## II. Backend API
### 2.1: Create User Management API
- [ ] Implement user creation endpoint
- [ ] Add authentication
- [ ] Handle errors
- [ ] Write tests
```

### 13.2. COMPLIANT DETAIL

```markdown
## II. Backend API Development
### 2.1: User Creation Endpoint - POST /api/users
- **Objective:** Create a secure API endpoint that validates input, hashes passwords, inserts user records, and returns structured responses.
- **Success Criteria:** Endpoint returns 201 with user object for valid requests, 400 for invalid input, 409 for duplicate email, 500 for server errors.
- **Dependencies:** Phase I.1 (users table), Phase II.1 (User type)
- **Estimated Actions:** 18
- **Rationale:** User creation is foundational for all authenticated features.
- **Assumption:** bcrypt with cost factor 10 is sufficient for password hashing.
- **Presumption:** PostgreSQL is the database; `pg` driver is available.

<details>
<summary><strong>Endpoint Specification</strong></summary>

**Algorithm:**
1. Validate HTTP method is POST; return 405 for others
2. Parse JSON body; return 400 if invalid JSON
3. Validate email format (RFC 5322) and password length (8-72 chars)
4. Check email uniqueness in database
5. Hash password with bcrypt
6. Insert user record
7. Return sanitized user object (exclude password hash)

**Request Schema:**
```json
{
  "email": "user@example.com",
  "password": "securepassword123",
  "name": "John Doe"
}
```

**Response Schemas:**

201 Created:
```json
{
  "id": 123,
  "email": "user@example.com",
  "name": "John Doe",
  "createdAt": "2025-01-15T10:30:00Z"
}
```

400 Bad Request:
```json
{
  "error": "VALIDATION_ERROR",
  "message": "Field 'email' must be a valid email address"
}
```

409 Conflict:
```json
{
  "error": "USER_EXISTS",
  "message": "A user with this email already exists"
}
```

</details>

- **Checklist:**
    - [ ] **Create route file**: Create `src/pages/api/users/index.ts`
        - **Implementation (TypeScript):**
          ```typescript
          import type { NextApiRequest, NextApiResponse } from 'next';
          export default async function handler(req: NextApiRequest, res: NextApiResponse) {
            // Implementation stub
          }
          ```
        - **Verification:** `test -f src/pages/api/users/index.ts && echo "exists"`

    - [ ] **Add method routing**: Return 405 for non-POST requests
        - **Algorithm:** Check `req.method !== 'POST'`, return 405 with error JSON
        - **Implementation (TypeScript):**
          ```typescript
          if (req.method !== 'POST') {
            return res.status(405).json({ error: 'METHOD_NOT_ALLOWED' });
          }
          ```

    - [ ] **Import validation library**: Import Zod for schema validation
        - **Implementation (TypeScript):** `import { z } from 'zod';`
        - **Prerequisite:** Zod installed (`npm install zod`)

    - [ ] **Define validation schema**: Create Zod schema for user creation
        - **Implementation (TypeScript):**
          ```typescript
          const CreateUserSchema = z.object({
            email: z.string().email().max(255),
            password: z.string().min(8).max(72),
            name: z.string().min(1).max(255),
          });
          ```

    - [ ] **Validate request body**: Parse and validate incoming JSON
        - **Algorithm:** Call schema.parse(), catch ZodError, return 400 with details
        - **Implementation (TypeScript):**
          ```typescript
          try {
            const data = CreateUserSchema.parse(req.body);
          } catch (e) {
            if (e instanceof z.ZodError) {
              return res.status(400).json({ error: 'VALIDATION_ERROR', details: e.errors });
            }
          }
          ```

    [... continue with remaining 13 actions ...]

### 2.1 Phase Verification
- [ ] File exists: `test -f src/pages/api/users/index.ts && echo "exists"`
- [ ] TypeScript compiles: `tsc --noEmit src/pages/api/users/index.ts`
- [ ] Schema validates test data: Run unit test for schema
```

---

## 14. Master Prompt Template for Compliant Plan Generation

Use this prompt to steer an agent to create a compliant implementation plan.

---

**Your task is to create a complete, compliant, and executable implementation plan.**

You must operate as a senior software architect creating a detailed, language-agnostic blueprint for a development team. Your primary output is a plan that describes **how to think** about implementing the feature, not a pre-written implementation.

**CRITICAL GUIDING PRINCIPLE:** Your final plan must be a **reusable algorithm**, not a language-specific script. The measure of success is: "Could another developer use this plan to implement the feature in a different programming language (e.g., Go, Rust, TypeScript)?" If the answer is no, the plan has failed.

**Mandatory Reference:** You must adhere strictly to the rules within `documentation/general_plan_crafting_guidelines.md`.

You will generate the plan by following the prescribed **4-Pass Methodology**, deriving all requirements from the single source-of-truth documentation file provided. You must generate each pass sequentially and await confirmation before proceeding to the next.

**The Task:** Create an implementation plan based on the feature specification documented in the following file: **[INSERT PATH TO DOCUMENTATION FILE]**

---

### Pass 1: The Skeleton

**Your Goal:** Establish the complete structure of the plan, covering all required phases and objectives based on the provided documentation.

**Your Instructions:**
1. Read the source documentation file provided in "The Task" above.
2. Create the plan structure with all 9 mandatory phases (0-IX), marking any as "Not Applicable" with a clear rationale.
3. For each applicable phase, define the high-level objectives that correspond to sections in the source documentation.
4. Under each objective, create a checklist of **compound actions**. At this stage, actions may be high-level (e.g., "Define the state management class").
5. Add a `Required Reading` section at the top of the plan.
6. Add the `Plan Development Status` section, marking Pass 1 as "In Progress."

**STOP and await confirmation before proceeding to Pass 2.**

---

### Pass 2: Atomicity

**Your Goal:** Decompose all compound actions into single, unambiguous, 5-second actions.

**Your Instructions:**
1. Review every checklist item from Pass 1.
2. Decompose each compound action into a series of atomic actions, adhering to the "5-second rule."
3. Ensure every action's title describes a language-agnostic intent.
4. Ensure every action has a single verb and references a single file location.
5. Expand definitions (TypedDicts, classes) into one action per field.
6. Expand function definitions into separate actions for signature, docstring, and each logical step.
7. Do not include implementation code in the titles—that goes in Pass 3.
8. Update the `Plan Development Status` section.

**STOP and await confirmation before proceeding to Pass 3.**

---

### Pass 3: Detail Enrichment

**Your Goal:** Eliminate all ambiguity by adding context, algorithms, and implementation hints.

**Your Instructions:**
1. For each objective or major section, add these context fields:
   - **Rationale:** Explain *why* the approach is being taken.
   - **Assumption:** Document design decisions made.
   - **Presumption:** State environmental requirements.
   - **Prerequisite:** Reference earlier sections that must be completed first.

2. For every **new file**, add a `<details>` block containing a **boilerplate stub** (imports + empty function signatures, NOT complete implementation).

3. For each atomic action, add an expanded body containing:
   - **Algorithm:** (If complex) Language-agnostic step-by-step logic.
   - **Implementation ([Language]):** A language-specific code snippet showing how to perform the action.

4. The action TITLE remains language-agnostic (the WHAT). The action BODY contains the language-specific hint (the HOW).

5. Update the `Plan Development Status` section.

**STOP and await confirmation before proceeding to Pass 4.**

---

### Pass 4: Verification

**Your Goal:** Make the plan self-verifying.

**Your Instructions:**
1. For at least 50% of the actions, add a `Verification` field containing a copy-paste executable command and the expected output.
2. At the end of each Phase, add a `Phase Verification Checklist` with commands to verify the state of the phase as a whole.
3. Update the `Plan Development Status` section, marking all passes complete.

After completing all four passes, present the final, fully-compliant plan.

---

## 15. Version Control

**Specification Version:** 2.3.0
**Last Updated:** 2026-02-01

**Changes in v2.3.0:**
- Added "The Documentation Lifecycle" section establishing the Interview → Documentation → Plan → Execution workflow
- Added Interview Process goals table (Gather, Decide, Surface blind spots, Identify second-order effects, Expose unintended consequences, Document)
- Added interview completion criteria

**Changes in v2.2.0:**
- Added Section 0: LLM/Agent Directive with machine-readable instructions
- Added File Artifact Mandate (plan file is the deliverable, not internal reasoning)
- Added Assumed Initial Instructions for first-encounter scenarios
- Added Mandatory Pre-Reading Protocol
- Added Interaction Protocol with per-pass checkpoints
- Added Summary Checklist for LLMs

**Changes in v2.1.0:**
- Added "Exception: Language-Locked Plans" subsection to Section 1.1
- Clarified that language-specific code is acceptable in Algorithm sections for plans tied to single-language ecosystems (e.g., Python-only ML libraries)

**Changes from v1.x to v2.0.0:**
- Added "Plans Are Language-Agnostic Algorithms" as core principle
- Added "Documentation-First Mandate"
- Added 4-Pass Methodology (Skeleton, Atomicity, Detail Enrichment, Verification)
- Added Enriched Atomic Action Pattern (language-agnostic title + language-specific body)
- Added Cross-Pass Consistency Rules
- Added Plan-Documentation Alignment Protocol
- Added Plan Execution Protocol (stop at phase boundaries)
- Added Plan Development Tracking section
- Enhanced atomicity patterns with concrete examples
- Added quality metrics table
- Consolidated all verification requirements

---

**Last Updated:** 2026-02-01
**Status:** Complete
