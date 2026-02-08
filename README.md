# Claude Interactive Documentation Workflow

I used to whiteboard, then code. Now I markdown, then code.

There are many ways to vibe code, but this one will allow you to understand the code that is being built, come back to it 6 months later and actually understand it, and more importantly, craft user-facing documentation artifacts trivially. The canonical documentation and plan files are actually kind of awesome for giving LLMs immediate, complete, and effective context to keep them guardrailed, without too much wrangling.

**In short:** A structured methodology for creating high-quality technical documentation and implementation plans using Claude Code's (or any LLM for that matter) AI assistance.

## What Is This?

This is a **complete workflow** for the documentation lifecycle:

```
Interview → Documentation → Plan → Implementation
```

Each stage has explicit guidelines, checkpoints, and quality gates. The methodology is designed to:

- **Eliminate ambiguity** before it becomes technical debt
- **Build understanding progressively** (rough shape → detailed specs)
- **Produce documentation that doesn't require clarifying questions**
- **Create plans that any competent developer can follow**

## Origin Story

This workflow was yanked directly from a production project (a computer vision training pipeline). The documents contain references to that project's specific domain (maritime vessel tracking, Label Studio, YOLO models, etc.).

**These are features, not bugs.** Real examples are more useful than sanitized generic templates. Adapt the prefixes, terminology, and examples to your own project.

## Git Verified Results

This methodology was used in production across 3+ projects over 4 months before being extracted into this repository. The results were independently analyzed against git history, GitHub API data, and 32 conversation session logs. The full analysis is available in the case study.

| Metric | Value |
|--------|-------|
| In production use since | October 2025 |
| Projects using methodology | 3+ (maritime tracking, CV pipeline, database systems) |
| Best plan-to-execution ratio | 7:1 (18 hours planning, 2.5 hours executing) |
| Fastest phase execution | 43 minutes across Phases 0-V (nautical charts feature) |
| Free documentation artifacts per feature | 10 (quickstart, deployment, developer guide, troubleshooting, security audit, user guide, testing, smoke tests, sign-off, release notes) |
| Combined methodology output (one project) | 67,050 lines across 62 docs + 23 plans |

The methodology's own standards demand verifiable claims. This case study applies that principle to the methodology itself.

[**Full Case Study: Verified Results and Analysis**](documentation/general_methodology_case_study_documentation.md)

## Quick Start

### 1. Clone and Enter

```bash
git clone https://github.com/tobieapb/claude-interactive-documentation-workflow.git
cd claude-interactive-documentation-workflow
```

### 2. Start an Interview

The `/interview` skill is pre-installed in this project. In Claude Code:

```
/interview user authentication system
```

This launches a structured 4-pass interview:
1. **The Shape** - What is this? Why does it exist?
2. **The Flow** - How does it work end-to-end?
3. **The Detail** - What exactly happens at each step?
4. **The Completeness** - What could go wrong? What else is affected?

### 3. Review Generated Documentation

The interview produces a file in `documentation/` following the documentation crafting guidelines.

### 4. Create a Plan

Point Claude at the documentation and the plan crafting guidelines:

```
Create an implementation plan for documentation/your_feature_documentation.md
following documentation/general_plan_crafting_guidelines.md
```

Plans go in `plans/` and follow a similar 4-pass methodology.

## The Three Core Documents

| Document | Purpose |
|----------|---------|
| [`general_interview_methodology_skill.md`](documentation/general_interview_methodology_skill.md) | How to conduct structured interviews to extract requirements |
| [`general_documentation_crafting_guidelines.md`](documentation/general_documentation_crafting_guidelines.md) | Standards for documentation (the "what to build" artifact) |
| [`general_plan_crafting_guidelines.md`](documentation/general_plan_crafting_guidelines.md) | Standards for implementation plans (the "how to build" artifact) |

## Directory Structure

```
.
├── documentation/          # Canonical feature documentation lives here
│   ├── general_*           # The methodology documents
│   └── your_docs_here.md   # Your project's documentation
├── plans/                  # Implementation plans live here
├── archive/                # Completed/obsolete items
│   ├── plans/
│   └── documentation/
└── .claude/
    └── skills/
        └── interview/
            └── SKILL.md    # The installed /interview skill
```

## Key Principles

### The Clarifying Question Rule

> If a reader must ask a clarifying question, the documentation has failed.

Every sentence should pass this test: "Could someone unfamiliar with this system execute or understand this without asking me anything?"

### Plans Are Language-Agnostic Algorithms

> Could a competent developer use this plan to implement the feature in a different programming language?

Plans describe WHAT and WHY, not language-specific HOW. Implementation hints are clearly marked as language-specific.

### The 4-Pass Methodology

Both interviews and plans use iterative refinement (bones-to skin):

| Pass | Interview | Plan |
|------|-----------|------|
| 1 | The Shape (rough goal) | Skeleton (structure) |
| 2 | The Flow (end-to-end process) | Atomicity (single actions) |
| 3 | The Detail (per-stage specifics) | Detail Enrichment (context, code hints) |
| 4 | The Completeness (edge cases) | Verification (proof of completion) |

### No TBD Allowed

Ambiguity in documentation becomes bugs in code. The interview process forces decisions upfront. "It depends" is not an acceptable answer—pick a default and document when someone would change it.

## Customization

### Adapt the Prefixes

The documentation guidelines define prefixes for file naming:

| Original (Maritime Domain) | Your Domain |
|---------------------------|-------------|
| `station_` | `backend_`, `service_`, etc. |
| `webapp_` | `frontend_`, `web_`, etc. |
| `ingress_` | `etl_`, `pipeline_`, etc. |
| `cv_` | `ml_`, `ai_`, etc. |

Edit `documentation/general_documentation_crafting_guidelines.md` Section 12.1 to match your project.

### Adapt the Plan Phases

The plan guidelines define 9 mandatory phases. Not all apply to every project:

- No database? Mark Phase I as "Not Applicable"
- No frontend? Mark Phase IV as "Not Applicable"
- No external integrations? Mark Phase V as "Not Applicable"

The key is to explicitly acknowledge what's not applicable rather than silently omitting it.

## Using Without Claude Code

The methodology works with any LLM. The core documents are just markdown—reference them or paste them as context.

### Interview (Any LLM)

```
I want you to interview me about a feature following the methodology in this document:
[paste general_interview_methodology_skill.md]

The feature is: user authentication
```

### Documentation from Notes

If you've done research or have investigation notes, you can generate compliant documentation directly:

```
Produce a documentation file based on the investigation notes for feature X.
The final documentation file must be compliant with the documentation
guidelines file found in @documentation/general_documentation_crafting_guidelines.md
```

Or if your LLM doesn't support file references, paste the guidelines and your notes.

### Plans from Documentation

Same principle—once you have documentation, generate a plan:

```
Create an implementation plan for the feature documented in
@documentation/my_feature_documentation.md

The plan must be compliant with the plan crafting guidelines found in
@documentation/general_plan_crafting_guidelines.md
```

### The Pattern

The guidelines files are enforcement documents. Point any LLM at them with:
- Your input (notes, requirements, existing docs)
- The relevant guidelines file
- An instruction to produce compliant output

The `/interview` skill is a Claude Code convenience, not a requirement. The real value is in the guidelines themselves.

## License

MIT License - See [LICENSE](LICENSE)

This is a gift. Use it, adapt it, share it. Don't expect support. If you make lots of money, or save millions by using this, remember me!

## Contributing

Found an improvement? PRs welcome. The bar is high—these documents enforce their own standards.

---

**Last Updated:** 2026-02-01
