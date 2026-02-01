# Documentation Crafting Guidelines: Enforcement Edition

**Purpose:** Establish mandatory documentation standards for the MarNexii project. These are not suggestions—they are **hard requirements** that must be met for documentation to be considered complete.

---

## Compliance Mandate

**Documentation that requires clarifying questions is a draft, not documentation.** Every document must be self-contained, precise, and actionable. If a developer or operator must ask "what does this mean?" or "how exactly do I do this?", the documentation has failed.

Documents that do not meet these standards must be marked `**Status:** Draft` until corrected.

---

## Core Principles (Non-Negotiable)

1. **Self-Sufficiency:** A reader should never need to consult another source to understand or execute the documentation.
2. **Zero Ambiguity:** Every instruction, description, or specification must be precise and unambiguous.
3. **Verifiability:** Every claim, configuration, or procedure should include a way to verify correctness.
4. **Currency:** Documentation must reflect the current state of the system—stale docs are worse than no docs.
5. **Discoverability:** File names and structure must enable quick identification of relevant content.
6. **Completeness:** Partial documentation is not documentation. Every element must be fully specified.

---

## 1. The Clarifying Question Rule

**If a reader must ask a clarifying question, the documentation has failed.**

Every sentence should pass this test: "Could someone unfamiliar with this system execute or understand this without asking me anything?"

### Applying the Rule

| ❌ Fails the Rule | ✅ Passes the Rule |
|-------------------|-------------------|
| "Configure the database connection" | "Set `DB_HOST=localhost` and `DB_PORT=5432` in `stations/webapp-service/api/.env`" |
| "Use the appropriate event type" | "Use event type UUID `a1b2c3d4-...` for 'vessel first seen' events" |
| "The API returns vessel data" | "The API returns `{mmsi, name, lat, lon, speed, course, heading, shiptype}`" |
| "Run the migration" | "Run `psql -h localhost -U postgres -d marnexiidatabase -f migrations/001_schema.sql`" |

### Questions That Indicate Failure

If a reader asks any of these, the documentation needs revision:

- "What value should I put here?"
- "Which file is this in?"
- "What does this return?"
- "What happens if this fails?"
- "Is this required or optional?"
- "What's the format for this?"
- "Where do I find this?"

---

## 2. Content Depth Rules

### 2.1: Single Concept Per Section

Each section addresses exactly one topic. If you find yourself writing "also" or "additionally" for unrelated items, split into new sections.

**❌ Violation:**
```markdown
## Database Configuration
Set up the connection pool. Also, make sure to configure the station code.
The logging level should be set appropriately. Additionally, the WebSocket
port needs to be specified.
```

**✅ Compliant:**
```markdown
## Database Configuration
[Database content only]

## Station Identity Configuration
[Station code content only]

## Logging Configuration
[Logging content only]

## WebSocket Configuration
[WebSocket content only]
```

### 2.2: No Assumed Knowledge

If you reference a concept, either:
- Link to where it's explained, OR
- Explain it inline (1-2 sentences minimum)

Never assume the reader knows project-specific terms.

| Term | Minimum Inline Explanation |
|------|---------------------------|
| MMSI | Maritime Mobile Service Identity - a 9-digit vessel identifier |
| MOLID | MarNexii Object Lifecycle ID - stable identity across MMSI changes |
| objects_lk | The "last known" table storing current vessel state |
| Ingress pipeline | The service that receives AIS data and writes to PostgreSQL |
| Station code | Unique identifier for this installation (e.g., `PTSC001`) |

### 2.3: Executable State

Every code example must be in a runnable state. No shortcuts.

**❌ Non-executable:**
```javascript
// ... imports ...
const result = await db.query(/* query here */);
// ... handle result ...
```

**✅ Executable:**
```javascript
// File: stations/webapp-service/api/routes/vessels.js
const db = require('../utils/db');

async function getVessel(mmsi) {
  const result = await db.query(
    'SELECT * FROM mrnxstation.objects_lk WHERE obj_mmsi = $1',
    [mmsi]
  );
  return result.rows[0] || null;
}
```

### 2.4: Complete Context

Every example must include enough context to understand where it fits.

**Required context for code examples:**
- File path (in comment or preceding text)
- Relevant imports
- Function/class it belongs to (if applicable)
- What triggers this code (if applicable)

### 2.5: Demonstrate Configuration Patterns

Code examples must model proper configuration practices. Values that would be configurable in production code should be shown accessed through configuration mechanisms, not hardcoded inline.

| ❌ Hardcoded in Logic | ✅ Configurable Pattern |
|-----------------------|------------------------|
| `db.connect('localhost', 5432)` | `db.connect(process.env.DB_HOST, process.env.DB_PORT)` |
| `if (count > 100)` | `if (count > config.MAX_VESSEL_COUNT)` |
| `fetch('http://localhost:3001/api')` | `fetch(\`${API_BASE_URL}/api\`)` |
| `const schema = 'mrnxstation'` | `const schema = process.env.DB_SCHEMA \|\| 'mrnxstation'` |

**Compatibility with Section 6 (Realistic Examples):** Examples still use realistic values per Section 6, but show *how* those values are accessed through configuration rather than hardcoded inline. The example values demonstrate what a properly configured system looks like.

**Example - Showing Both Principles:**
```javascript
// File: stations/webapp-service/api/utils/db.js
const { Pool } = require('pg');

// ✅ Configuration pattern with realistic default values
const pool = new Pool({
  host: process.env.DB_HOST || 'localhost',
  port: parseInt(process.env.DB_PORT) || 5432,
  database: process.env.DB_DATABASE || 'marnexiidatabase',
  user: process.env.DB_USER || 'postgres',
});
```

---

## 3. Mandatory Specifications by Element Type

### 3.1: Code Examples

Every code example must have:

| Requirement | Example |
|-------------|---------|
| Language tag | ` ```sql `, ` ```typescript `, ` ```bash ` |
| File path | `// File: stations/ingress-service/vesselState.js` |
| Imports shown | `const db = require('../utils/db');` |
| Complete snippet | No `// ...` or `/* TODO */` |
| Realistic values | Actual table names, actual field names |

### 3.2: Tables

Every table must have:

| Requirement | Why |
|-------------|-----|
| Header row | Identifies columns |
| Minimum 2 data rows | Single-row tables suggest incompleteness |
| No empty cells | Use `-` or `N/A` explicitly |
| Consistent alignment | Readability |

**❌ Invalid table:**
```markdown
| Name | Type |
|------|------|
| id | uuid |
```

**✅ Valid table:**
```markdown
| Name | Type | Nullable | Description |
|------|------|----------|-------------|
| id | uuid | NO | Primary key |
| name | varchar(255) | YES | Vessel name from AIS |
| created_at | timestamptz | NO | Record creation time |
```

### 3.3: Commands

Every command must include:

| Element | Example |
|---------|---------|
| Exact command | `psql -h localhost -U postgres -d marnexiidatabase -c "\dt mrnxstation.*"` |
| Working directory | `cd stations/webapp-service/api` (if required) |
| Expected output | What success looks like |
| Failure indication | What failure looks like (if non-obvious) |

**Example:**
```markdown
**Command:**
```bash
curl -s http://localhost:3001/api/v1/station/config | jq '.stationCode'
```

**Expected Output:**
```
"PTSC001"
```

**If this fails:**
- Empty response: API server not running
- Connection refused: Wrong port or server down
- `null`: Station not configured in database
```

### 3.4: Configuration Values

Every configuration value must specify:

| Element | Required |
|---------|----------|
| Variable name | Yes |
| Type | Yes (string, integer, boolean, etc.) |
| Valid range/values | Yes (if constrained) |
| Default value | Yes (or "none - required") |
| Example | Yes (realistic value) |
| What breaks if wrong | Yes |

**Example:**
```markdown
### DB_PORT

- **Type:** Integer
- **Valid range:** 1-65535
- **Default:** 5432
- **Example:** `5432`
- **What breaks if wrong:** Database connection fails with "Connection refused" or "Invalid port"
```

---

## 4. Detail Multiplication by Document Type

When documenting certain patterns, minimum content requirements apply. Partial documentation is invalid.

### 4.1: Database Table Documentation

| Must Include | Why | Invalid Without It |
|--------------|-----|-------------------|
| All columns with types | Schema must be complete | Missing columns cause runtime errors |
| All constraints (PK, FK, UNIQUE, CHECK) | Integrity rules are part of schema | Incomplete constraint knowledge causes insert failures |
| All indexes with purpose | Performance context | Missing index info leads to slow queries |
| Column descriptions | Semantics matter | Ambiguous column meanings |
| At least one INSERT example | Shows required fields | Developers guess at required fields |
| At least one SELECT example | Shows typical usage | Developers write inefficient queries |
| Schema verification query | Proves accuracy | No way to validate doc matches reality |

### 4.2: API Endpoint Documentation

| Must Include | Why | Invalid Without It |
|--------------|-----|-------------------|
| HTTP method and path | Basic identity | Can't call the endpoint |
| Authentication requirements | Security context | Unauthorized errors |
| All parameters (path, query, body) | Complete contract | Missing required params |
| All response status codes | Client error handling | Unhandled error cases |
| Response body shape per status | Data contract | Client parsing failures |
| All error codes and messages | Debugging support | Opaque error handling |
| Rate limits / quotas | Operational limits | Unexpected 429 errors |
| curl example | Quick testing | Slow onboarding |

### 4.3: Configuration Documentation

| Must Include | Why | Invalid Without It |
|--------------|-----|-------------------|
| Every environment variable | Complete config picture | Missing config = broken deploy |
| Type and validation rules | Prevents misconfiguration | Silent misconfig bugs |
| Default value | Required for optional configs | Unknown behavior |
| Example with realistic value | Not `your_value_here` | Copy-paste errors |
| What breaks if wrong | Debugging aid | Hours wasted on config issues |
| Related variables | Dependency awareness | Partial config changes |

### 4.4: Event Type Documentation

| Must Include | Why |
|--------------|-----|
| Event type UUID | Required for queries |
| Event type name | Human readability |
| When it fires | Trigger conditions |
| Payload structure | What data is captured |
| Example payload | Concrete illustration |
| Related events | Event flow understanding |

### 4.5: Troubleshooting Documentation

| Must Include | Why |
|--------------|-----|
| Exact symptom description | Problem identification |
| Exact error messages | Pattern matching |
| Diagnostic commands | Investigation path |
| All possible root causes | Comprehensive debugging |
| Resolution per cause | Actionable fixes |
| Verification of fix | Confirm resolution |
| Prevention steps | Avoid recurrence |

---

## 5. The "No External Dependencies" Rule

Documentation must not require consulting other sources to be understood.

### 5.1: What This Means

| ❌ Violation | ✅ Compliant |
|-------------|-------------|
| "See the code for validation rules" | Embed the validation rules in the doc |
| "Check the schema for column types" | Include the column types table |
| "Refer to the ingestion paradigm doc" | Summarize relevant parts inline, then link for deep dive |
| "Use the standard format" | Specify exactly what the format is |
| "Follow the usual process" | Document the process steps |
| "As documented elsewhere" | State where, and summarize key points |

### 5.2: Linking Policy

Links to other documentation are allowed IF:

1. **You provide a 1-sentence summary** of what's at the link
2. **The link is for "deep dive" only** - the current doc is usable without clicking
3. **The link is to internal documentation** - external links need access date

**❌ Invalid linking:**
```markdown
For configuration details, see [this document](./other_doc.md).
```

**✅ Valid linking:**
```markdown
The station uses SASL/SCRAM-SHA-512 authentication with per-station credentials.
For the full authentication flow including token refresh, see
[`authentication_architecture.md`](./authentication_architecture.md).
```

### 5.3: Embedding vs. Linking Decision

| Situation | Action |
|-----------|--------|
| < 10 lines of context needed | Embed inline |
| 10-50 lines of context needed | Embed with collapsible section or summary + link |
| > 50 lines of context needed | Summary paragraph + link |
| Critical for understanding | Always embed, regardless of length |
| Nice-to-have deep dive | Link with summary |

---

## 6. Realistic Examples Mandate

All examples must use actual project values, not generic placeholders.

### 6.1: Forbidden Generic Values

| ❌ Generic | ✅ Project-Specific |
|-----------|---------------------|
| `schema.table_name` | `mrnxstation.positions` |
| `example.com` | `localhost:3001` |
| `user@example.com` | `operator@marnexii.local` |
| `123` for IDs | `550e8400-e29b-41d4-a716-446655440000` |
| `foo`, `bar`, `baz` | `obj_mmsi`, `position_lat`, `event_type_id` |
| `some_value` | `PTSC001` |
| `your_password_here` | `(do not commit - see .env.example)` |
| `<placeholder>` | Actual value or explicit instruction |
| `...` in code | Complete code or explicit "truncated for brevity" note |

### 6.2: Project-Specific Reference Values

Use these actual values in examples:

| Domain | Example Values |
|--------|---------------|
| Station codes | `PTSC001`, `PTSC002` |
| Database | `marnexiidatabase`, schema `mrnxstation` |
| Tables | `positions`, `events`, `objects_lk`, `rawais` |
| Ports | API: `3001`, PostgreSQL: `5432`, Client: `5173` |
| MMSIs | `311234567`, `319876543` (valid format, 9 digits) |
| UUIDs | `550e8400-e29b-41d4-a716-446655440000` (valid v4 format) |
| Timestamps | `2025-01-10T14:30:00Z` (ISO 8601) |
| Coordinates | `-61.5186, 10.6544` (Port of Spain) |

### 6.3: When Placeholders Are Acceptable

Placeholders are allowed ONLY when:

1. **Passwords/secrets:** Use `(do not commit)` note
2. **User-specific values:** Provide format specification
3. **Generated values:** Explain how to generate

**Example:**
```markdown
```bash
DB_PASSWORD=<your-password>  # Do not commit. Must be at least 12 characters.
API_KEY=$(openssl rand -hex 32)  # Generate with this command
STATION_CODE=XXXX001  # Replace XXXX with your organization code
```
```

---

## 7. Document Invalidation Criteria

A document is **automatically invalid** (must be marked `Draft`) if any of these apply:

### 7.1: Structural Failures

- [ ] Missing any required section from template (Section 10)
- [ ] No `Last Updated` date at bottom
- [ ] No `Status` indicator at bottom
- [ ] Headings not hierarchical (skipping levels)
- [ ] No document title (H1)

### 7.2: Content Failures

- [ ] Contains any Tier 1-5 forbidden phrase (Section 9)
- [ ] Any code block without language tag
- [ ] Any command that fails when executed
- [ ] Any internal link that 404s
- [ ] Any external link that 404s (without archive note)
- [ ] References to files that don't exist
- [ ] References to tables/columns that don't exist
- [ ] Code examples with `// ...` or `/* TODO */`

### 7.3: Completeness Failures

- [ ] Partial table schemas (missing columns)
- [ ] Partial API responses (missing fields or status codes)
- [ ] Partial config lists (missing required variables)
- [ ] Examples with placeholder values like `<your-value>`
- [ ] Commands without expected output
- [ ] Config values without types or defaults
- [ ] Database tables without all constraints/indexes

### 7.4: Self-Containment Failures

- [ ] "See the code" without embedding relevant code
- [ ] "Check the schema" without including schema
- [ ] Links without summary of linked content
- [ ] Assumed knowledge of project-specific terms without definition

---

## 8. Self-Verification Protocol

Before marking a document `**Status:** Complete`, apply ALL of these tests:

### 8.1: The New Developer Test

Give the document to someone unfamiliar with the system (or imagine such a person).

- Can they follow every instruction without asking questions?
- Do they understand every term used?
- Can they complete the described task successfully?

**If any answer is "no", the document needs revision.**

### 8.2: The Execution Test

For every command and code example in the document:

1. Copy the command exactly as written
2. Execute it in the documented context
3. Verify the output matches documented expectations

**If any command fails or produces unexpected output, fix the document.**

### 8.3: The Link Test

For every link in the document:

1. Click the link
2. Verify it resolves to the intended content
3. Verify the summary accurately describes the linked content

**If any link is broken or misleading, fix it.**

### 8.4: The Currency Test

For every technical reference in the document:

1. File paths: Verify the file exists at that path
2. Table names: Verify the table exists in the database
3. Column names: Verify columns exist with documented types
4. Config variables: Verify they're used in the code
5. API endpoints: Verify they exist and behave as documented

**If any reference is stale, update it.**

### 8.5: The Completeness Test

For the document type, verify ALL required sections are present per Section 10 templates.

| Document Type | Required Sections Checklist |
|--------------|----------------------------|
| ADR | Context, Decision Drivers, Options, Decision, Consequences |
| API Reference | Overview, Auth, Endpoints (with all status codes), Examples |
| Database Schema | Overview, Schema table, Constraints, Indexes, Relationships, Examples, Verification |
| Deployment Guide | Prerequisites, Env Vars, Steps (with verification each), Rollback |
| Troubleshooting | Quick Diagnostics, Issues (with symptoms, diagnosis, resolution, prevention) |

---

## 9. Forbidden Phrases (Auto-Fail)

Documentation containing these phrases is **automatically considered Draft status**:

### Tier 1: Incomplete Markers
- "TBD"
- "TODO"
- "FIXME"
- "will be documented"
- "to be added"
- "coming soon"
- "WIP"

### Tier 2: Vague References
- "see above" (without section link)
- "as mentioned" (without specific reference)
- "similar to previous" (without explicit reference)
- "refer to documentation" (without specific file/section)
- "check the code" (without file:line reference)

### Tier 3: Vague Quantifiers
- "various"
- "several"
- "some"
- "many"
- "few"
- "etc."
- "and so on"
- "and others"
- "..." (when used to imply continuation of a list)

**Exception for ellipsis (`...`):** Ellipsis is permitted when truncating long identifiers, keys, UUIDs, or breadcrumbs where showing the full value is impractical. The ellipsis must clearly indicate truncation of a specific value, not continuation of a list.

| ❌ Forbidden | ✅ Permitted |
|-------------|-------------|
| "supports cargo, tanker, tug, ..." | `gpkey_abc123def456...` (truncated key) |
| "and other vessel types..." | `uuid: 550e8400-e29b-41d4-a716-...` (truncated UUID) |
| "configure settings, ports, ..." | `path/to/deeply/nested/.../file.txt` (truncated path) |

### Tier 4: False Simplicity
- "straightforward"
- "simply"
- "just"
- "obviously"
- "clearly"
- "trivially"
- "easy"
- "basic"

### Tier 5: Ambiguous Instructions
- "appropriately"
- "properly"
- "correctly"
- "as needed"
- "if necessary"
- "when applicable"
- "where relevant"

---

## 10. Mandatory Document Templates

Every document type has required sections. Omitting required sections invalidates the document.

### 10.1: Architecture Decision Records (ADR)

```markdown
# [Decision Title]

**Status:** [Proposed | Accepted | Deprecated | Superseded]
**Date:** YYYY-MM-DD
**Deciders:** [List of people involved]

## Context
[What is the issue that we're seeing that motivates this decision?]
[2-5 sentences minimum]

## Decision Drivers
- [Driver 1]
- [Driver 2]
- [Driver 3]

## Options Considered

### Option 1: [Name]
- **Description:** [What is this option?]
- **Pros:** [List advantages]
- **Cons:** [List disadvantages]
- **Effort:** [Low | Medium | High]

### Option 2: [Name]
[Same structure]

## Decision
[Which option was chosen and why, in 2-3 sentences]

## Consequences

### Positive
- [Positive consequence 1]
- [Positive consequence 2]

### Negative
- [Negative consequence 1]
- [Trade-off accepted]

### Risks
- [Risk and mitigation strategy]

---

**Last Updated:** YYYY-MM-DD
```

### 10.2: API Reference Documentation

```markdown
# [API Name/Endpoint Group]

## Overview
[One paragraph describing the API's purpose]

## Authentication
[Exact auth requirements - header name, token format, example]

## Endpoints

### [METHOD] /path/to/endpoint

**Description:** [One sentence]

**Authentication:** [Required | Optional | None]

**Rate Limit:** [X requests per Y period, or "None"]

**Parameters:**

| Name | Location | Type | Required | Description | Example |
|------|----------|------|----------|-------------|---------|
| id | path | integer | Yes | Resource ID | 123 |
| limit | query | integer | No | Max results (default: 50) | 100 |

**Request Body:** (if applicable)
```json
{
  "field": "value",
  "nested": {
    "field": "value"
  }
}
```

**Responses:**

**200 OK:**
```json
{
  "id": 123,
  "name": "Example",
  "createdAt": "2025-01-15T10:30:00Z"
}
```

**400 Bad Request:**
```json
{
  "error": "VALIDATION_ERROR",
  "message": "Field 'email' is required",
  "field": "email"
}
```

**401 Unauthorized:**
```json
{
  "error": "UNAUTHORIZED",
  "message": "Invalid or missing authentication token"
}
```

**500 Internal Server Error:**
```json
{
  "error": "INTERNAL_ERROR",
  "message": "An unexpected error occurred"
}
```

**Example Request:**
```bash
curl -X POST "http://localhost:3001/api/v1/endpoint" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"field": "value"}'
```

---

**Last Updated:** YYYY-MM-DD
```

### 10.3: Database Schema Documentation

```markdown
# [Table/Entity Name]

## Overview
[Purpose of this table in 2-3 sentences. What data it holds and why.]

## Schema

**Table:** `mrnxstation.table_name`

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | uuid | NO | gen_random_uuid() | Primary key |
| name | varchar(255) | NO | - | Display name |
| status | varchar(20) | NO | 'active' | Record status |
| created_at | timestamptz | NO | now() | Creation timestamp |
| updated_at | timestamptz | YES | - | Last modification time |

## Constraints

| Name | Type | Columns | Details |
|------|------|---------|---------|
| pk_table_name | PRIMARY KEY | id | - |
| uq_table_name_email | UNIQUE | email | - |
| fk_table_name_user | FOREIGN KEY | user_id | References users(id) ON DELETE CASCADE |
| chk_table_name_status | CHECK | status | status IN ('active', 'inactive', 'pending') |

## Indexes

| Name | Columns | Type | Purpose |
|------|---------|------|---------|
| idx_table_name_email | email | btree | Email lookups |
| idx_table_name_created | created_at DESC | btree | Recent records queries |
| idx_table_name_status_created | (status, created_at) | btree | Filtered time queries |

## Relationships

```
[parent_table] 1──────N [this_table]
     │
     └── parent_id → parent_table.id

[this_table] 1──────N [child_table]
     │
     └── id ← child_table.parent_id
```

## Example Queries

**Insert:**
```sql
INSERT INTO mrnxstation.table_name (name, email, user_id)
VALUES ('John Doe', 'john@example.com', '550e8400-e29b-41d4-a716-446655440000')
RETURNING id, created_at;
```

**Common Select:**
```sql
SELECT t.id, t.name, t.status, u.email as user_email
FROM mrnxstation.table_name t
JOIN mrnxstation.users u ON t.user_id = u.id
WHERE t.status = 'active'
  AND t.created_at > NOW() - INTERVAL '7 days'
ORDER BY t.created_at DESC
LIMIT 100;
```

**Update:**
```sql
UPDATE mrnxstation.table_name
SET status = 'inactive', updated_at = NOW()
WHERE id = '550e8400-e29b-41d4-a716-446655440000'
RETURNING id, status, updated_at;
```

## Schema Verification

```sql
-- Verify table exists with correct columns
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_schema = 'mrnxstation' AND table_name = 'table_name'
ORDER BY ordinal_position;

-- Verify constraints
SELECT conname, contype, pg_get_constraintdef(oid)
FROM pg_constraint
WHERE conrelid = 'mrnxstation.table_name'::regclass;

-- Verify indexes
SELECT indexname, indexdef
FROM pg_indexes
WHERE schemaname = 'mrnxstation' AND tablename = 'table_name';
```

---

**Last Updated:** YYYY-MM-DD
```

### 10.4: Deployment Guide

```markdown
# [Deployment Name] Guide

## Overview
[What is being deployed and why, in 2-3 sentences]

## Prerequisites

| Requirement | Version | Verification Command | Expected Output |
|-------------|---------|---------------------|-----------------|
| Node.js | 18+ | `node --version` | `v18.x.x` or higher |
| PostgreSQL | 14+ | `psql --version` | `psql (PostgreSQL) 14.x` or higher |
| npm | 9+ | `npm --version` | `9.x.x` or higher |

## Environment Variables

| Variable | Required | Type | Default | Example | Description |
|----------|----------|------|---------|---------|-------------|
| DB_HOST | Yes | string | - | localhost | Database server hostname |
| DB_PORT | No | integer | 5432 | 5432 | Database server port |
| DB_PASSWORD | Yes | string | - | (do not commit) | Database password |
| API_PORT | No | integer | 3001 | 3001 | API server listen port |

## Deployment Steps

### Step 1: [Action Name]

**Working Directory:** `stations/webapp-service/api`

**Command:**
```bash
npm ci --production
```

**Expected Output:**
```
added 127 packages in 8.2s
```

**Verification:**
```bash
test -d node_modules && echo "SUCCESS" || echo "FAILED"
```

**If This Fails:**
- `npm ERR! network`: Check internet connection
- `npm ERR! 404`: Check npm registry access
- Permission errors: Check directory ownership

### Step 2: [Action Name]
[Same structure]

## Post-Deployment Verification

| Check | Command | Expected Result | If Fails |
|-------|---------|-----------------|----------|
| API responds | `curl http://localhost:3001/health` | `{"status":"ok"}` | Server not running |
| DB connected | `curl http://localhost:3001/api/v1/station/config` | JSON with stationCode | DB connection issue |

## Rollback Plan

### When to Rollback
- API health check fails after deployment
- Error rate exceeds baseline by 10x
- Critical functionality broken

### Immediate Rollback Steps
```bash
# 1. Stop current deployment
pm2 stop all

# 2. Restore previous version
git checkout HEAD~1

# 3. Reinstall dependencies
npm ci --production

# 4. Restart services
pm2 start ecosystem.config.js

# 5. Verify rollback
curl http://localhost:3001/health
```

### Database Rollback (if migrations were run)
```bash
# Only if migrations were part of this deployment
npm run migrate:down
```

## Troubleshooting

| Symptom | Likely Cause | Diagnostic Command | Resolution |
|---------|--------------|-------------------|------------|
| Port in use | Previous instance | `lsof -i :3001` | `kill -9 <PID>` |
| DB connection refused | PostgreSQL down | `pg_isready -h localhost` | `sudo systemctl start postgresql` |
| Module not found | Incomplete install | `npm ls` | `rm -rf node_modules && npm ci` |

---

**Last Updated:** YYYY-MM-DD
```

### 10.5: Troubleshooting Guide

```markdown
# [System/Component] Troubleshooting

## Quick Diagnostics

Run these commands first to assess system state:

```bash
# Check if API is responding
curl -s http://localhost:3001/health | jq .

# Check if database is accessible
psql -h localhost -U postgres -d marnexiidatabase -c "SELECT 1"

# Check process status
ps aux | grep node

# Check recent logs
tail -100 /var/log/app/error.log
```

## Common Issues

### Issue 1: [Symptom Description]

**Symptoms:**
- [Observable symptom 1 - be specific]
- [Observable symptom 2 - be specific]
- Error message: `[exact error text that appears]`

**Affected Components:** [List of affected services/components]

**Severity:** [Critical | High | Medium | Low]

**Diagnosis:**

1. Check [specific thing]:
   ```bash
   [exact diagnostic command]
   ```
   - **Expected if healthy:** [what you should see]
   - **Indicates problem:** [what indicates this issue]

2. Verify [another thing]:
   ```bash
   [exact diagnostic command]
   ```
   - **Expected if healthy:** [what you should see]
   - **Indicates problem:** [what indicates this issue]

**Root Causes:**

| Cause | Likelihood | How to Confirm |
|-------|------------|----------------|
| [Cause A] | High | [Specific confirmation command or check] |
| [Cause B] | Medium | [Specific confirmation command or check] |
| [Cause C] | Low | [Specific confirmation command or check] |

**Resolution:**

**For Cause A:**
```bash
[exact commands to fix, one per line with comments]
# Step 1: Stop the service
pm2 stop api

# Step 2: Fix the issue
[specific fix command]

# Step 3: Restart
pm2 start api

# Step 4: Verify
curl http://localhost:3001/health
```

**For Cause B:**
```bash
[exact commands to fix]
```

**Verification:**
```bash
# Confirm the issue is resolved
[verification command]
# Expected output: [what success looks like]
```

**Prevention:**
- [Specific action to prevent recurrence]
- [Monitoring/alerting to catch early]

---

**Last Updated:** YYYY-MM-DD
```

### 10.6: Event Type Documentation (MarNexii-Specific)

```markdown
# Event Type: [Event Name]

## Overview
[What this event represents and when it fires, in 2-3 sentences]

## Identification

| Property | Value |
|----------|-------|
| UUID | `550e8400-e29b-41d4-a716-446655440000` |
| Name | `event_type_name` |
| Category | [Visibility | Position | Static | System] |

## Trigger Conditions

This event fires when:
1. [Specific condition 1]
2. [Specific condition 2]
3. [Specific condition 3 - be exhaustive]

This event does NOT fire when:
1. [Exception case 1]
2. [Exception case 2]

## Payload Structure

```typescript
interface EventPayload {
  mmsi: number;           // Vessel MMSI
  timestamp: string;      // ISO 8601 timestamp
  field1: string;         // Description of field1
  field2: number | null;  // Description of field2, null when [condition]
}
```

## Example Payload

```json
{
  "mmsi": 311234567,
  "timestamp": "2025-01-10T14:30:00.000Z",
  "field1": "example_value",
  "field2": 42
}
```

## Database Storage

```sql
-- Events are stored in mrnxstation.events
-- The payload is stored in event_payload as JSONB

SELECT
  event_id,
  event_obj_mmsi,
  event_received_at,
  event_payload
FROM mrnxstation.events
WHERE event_type_id = '550e8400-e29b-41d4-a716-446655440000'
ORDER BY event_received_at DESC
LIMIT 10;
```

## Related Events

| Event | Relationship |
|-------|--------------|
| [Other Event Name] | [How they relate - precedes, follows, alternative to] |

## Querying This Event Type

```sql
-- Get event type UUID by name
SELECT event_type_id
FROM mrnxstation.event_types
WHERE event_type_name = 'event_type_name';

-- Count events of this type in last 24 hours
SELECT COUNT(*)
FROM mrnxstation.events
WHERE event_type_id = '550e8400-e29b-41d4-a716-446655440000'
  AND event_received_at > NOW() - INTERVAL '24 hours';
```

---

**Last Updated:** YYYY-MM-DD
```

---

## 11. Documentation Placement Rules

### 11.1: High-Level Documentation → `/documentation/` folder

**What goes here:**
- Architecture and design decisions
- System-wide technical documentation
- Deployment guides and strategies
- Installation and configuration guides
- Database schema documentation
- Integration guides
- Performance tuning documentation
- Operational runbooks

**Examples:**
- `documentation/webapp_deployment_architecture.md`
- `documentation/station_ingestion_paradigm.md`
- `documentation/project_architecture.md`

### 11.2: Code-Specific Documentation → Keep with code

**What stays in code directories:**
- README files for specific modules/packages
- API/library usage documentation
- Code structure guides for specific components
- Development workflow for specific modules

**Examples:**
- `stations/webapp-service/api/PROJECT_STRUCTURE.md`
- `stations/webapp-service/client/README.md`
- `stations/ingress-service/README.md`

### 11.3: Planning and Scratchpads → `/plans/` folder

**What goes here:**
- Work-in-progress planning documents
- Feature implementation plans
- Temporary scratchpads and notes
- Task tracking documents

**Required Suffix:**

All plan files in the `/plans/` folder must end with `_plan.md`. This suffix:
- Clearly identifies the file as a planning document
- Distinguishes from documentation, code, or other artifacts
- Enables consistent file discovery via glob patterns

**Pattern:** `descriptive_name_plan.md`

**Examples:**
- `plans/station_view_implementation_plan.md`
- `plans/typed_3d_marker_system_plan.md`
- `plans/mmolid_migration_plan.md`

**Invalid examples:**
```
❌ plans/station_view_implementation.md    (missing _plan suffix)
❌ plans/notes.md                           (too vague, no suffix)
```

---

## 12. File Naming Conventions

### 12.1: Documentation Folder (`/documentation/`)

**Pattern:** `prefix_descriptive_name_documentation.md`

**Required Prefixes for Bucketing:**

| Prefix | Content Type | Examples |
|--------|--------------|----------|
| `station_` | Station-level features/architecture | `station_ingestion_paradigm_documentation.md` |
| `webapp_` | Web application (API + client) | `webapp_deployment_architecture_documentation.md` |
| `ingress_` | Ingress pipeline specific | `ingress_hot_ingress_documentation.md` |
| `datamodel_` | Database entities/primitives | `datamodel_locations_documentation.md` |
| `general_` | Cross-cutting/meta documentation | `general_documentation_crafting_guidelines.md` |
| `postgres16_` | PostgreSQL operations | `postgres16_installation_documentation.md` |
| `ais_` | AIS protocol/data specific | `ais_ship_types_documentation.md` |
| `central_` | Central server features/architecture | `central_molid_creation_interrogation_documentation.md` |
| `cv_` | Computer Vision models and workflows | `cv_annotation_pipeline_documentation.md` |
| `feeders_` | Data ingestion feeders | `feeders_general_architecture_documentation.md` |
| `utils_` | Supporting utilities | `utils_general_architecture_documentation.md` |
| `reference_` | Reference data documentation | `reference_ships_documentation.md` |

**Required Suffix:**

All documentation files in the `/documentation/` folder must end with `_documentation.md` (or `_guidelines.md` for guideline documents). This suffix:
- Clearly identifies the file as documentation
- Distinguishes from code files, configs, or other artifacts
- Enables consistent file discovery via glob patterns

Skill definition files must end with `_skill.md`. This suffix:
- Identifies the file as an executable prompt/command definition
- Distinguishes from descriptive documentation and prescriptive guidelines
- Contains prompts that can be invoked by AI assistants or tooling

**Rules:**
1. All lowercase letters
2. Words separated by underscores (snake_case)
3. Descriptive names that indicate content
4. `.md` extension (Markdown)
5. No spaces, hyphens, or camelCase
6. Prefix must match content type
7. Suffix must be `_documentation`, `_guidelines` (for guideline docs), or `_skill` (for executable prompt definitions)

**Examples:**
```
✅ webapp_deployment_architecture_documentation.md
✅ station_api_replay_endpoint_documentation.md
✅ datamodel_zones_and_zone_types_documentation.md
✅ general_documentation_crafting_guidelines.md
✅ general_interview_methodology_skill.md

❌ WebApp-Deployment.md         (hyphens, capitals, no prefix, no suffix)
❌ deployment.md                 (no prefix, no suffix, too vague)
❌ station_api_replay.md         (missing _documentation suffix)
❌ misc_notes.md                 (vague, no clear category, no suffix)
```

### 12.2: Code-Specific Documentation

**Pattern:** `UPPERCASE.md` or `README.md`

**Rules:**
1. UPPERCASE for specific guides: `PROJECT_STRUCTURE.md`
2. Standard `README.md` for package/module introductions
3. Can use hyphens for multi-word names: `API-GUIDE.md`

---

## 13. Quality Metrics

Documents must meet these minimum thresholds:

### 13.1: By Document Type

| Doc Type | Min Code Examples | Min Tables | Min Verification Commands | Min Words |
|----------|-------------------|------------|---------------------------|-----------|
| API Reference | 1 per endpoint | 1 (parameters) | 1 curl per endpoint | 200 |
| Architecture/ADR | 2 | 1 (options comparison) | 0 | 500 |
| Deployment Guide | 5 | 2 (prereqs, env vars) | 1 per step | 400 |
| Database Schema | 3 | 3 (schema, constraints, indexes) | 2 | 300 |
| Troubleshooting | 3 | 1 (causes) | 5 | 300 |
| Event Type | 2 | 2 (identification, related) | 1 | 200 |

### 13.2: Universal Requirements

- **Code blocks:** Must have language tags (`sql`, `bash`, `typescript`, etc.)
- **File paths:** Must be absolute from project root or clearly relative
- **Commands:** Must be copy-paste executable (no placeholder brackets like `<value>`)
- **Examples:** Must use realistic values, not "example", "test", "foo", "bar"
- **Tables:** Must have headers and consistent formatting

---

## 14. Examples: Insufficient vs. Compliant

### 14.1: API Documentation

**❌ INSUFFICIENT:**
```markdown
## Create User Endpoint

POST /api/users

Creates a new user. Send user data in the body. Returns the created user or an error.

See the code for details on validation.
```

**✅ COMPLIANT:**
```markdown
## POST /api/v1/vessels/:mmsi

**Description:** Retrieve current state and last known data for a specific vessel.

**Authentication:** None (station-local endpoint)

**Rate Limit:** 100 requests per minute

**Parameters:**

| Name | Location | Type | Required | Description | Example |
|------|----------|------|----------|-------------|---------|
| mmsi | path | integer | Yes | 9-digit Maritime Mobile Service Identity | 311234567 |

**Response 200 OK:**
```json
{
  "mmsi": 311234567,
  "name": "CARIBBEAN TRADER",
  "lat": 10.6544,
  "lon": -61.5186,
  "speed": 12.5,
  "course": 245,
  "heading": 243,
  "shiptype": 70,
  "shiptypeDescription": "Cargo",
  "lastSeen": "2025-01-10T14:30:00.000Z"
}
```

**Response 404 Not Found:**
```json
{
  "error": "VESSEL_NOT_FOUND",
  "message": "No vessel found with MMSI 311234567"
}
```

**Response 400 Bad Request:**
```json
{
  "error": "INVALID_MMSI",
  "message": "MMSI must be a 9-digit integer"
}
```

**Example:**
```bash
curl -s http://localhost:3001/api/v1/vessels/311234567 | jq .
```
```

### 14.2: Deployment Instructions

**❌ INSUFFICIENT:**
```markdown
## Deployment

1. Install dependencies
2. Set up the database
3. Configure environment variables
4. Start the server
5. Verify it's working
```

**✅ COMPLIANT:**
```markdown
## Deployment Steps

### Step 1: Install Dependencies

**Working Directory:** `stations/webapp-service/api`

**Command:**
```bash
npm ci --production
```

**Expected Output:**
```
added 127 packages in 8.2s
```

**Verification:**
```bash
node -e "require('./index.js')" 2>&1 | head -1
# Expected: No output (clean require) or "Server running" if it auto-starts
```

**If This Fails:**
- `Cannot find module`: Run `npm ci` again, check for npm errors
- Permission denied: Check file ownership with `ls -la node_modules`

### Step 2: Configure Database Connection

**Create file:** `stations/webapp-service/api/.env`

```bash
DB_USER=postgres
DB_HOST=localhost
DB_DATABASE=marnexiidatabase
DB_PASSWORD=your_secure_password_here
DB_PORT=5432
API_PORT=3001
```

**Verification:**
```bash
# Test database connection
PGPASSWORD=$DB_PASSWORD psql -h $DB_HOST -U $DB_USER -d $DB_DATABASE -c "SELECT 1"
# Expected: Returns "1" with no errors
```

### Step 3: Start the Server

**Command:**
```bash
cd stations/webapp-service/api && npm start
```

**Expected Output:**
```
Server running on port 3001
Database connected to marnexiidatabase
WebSocket server ready
```

**Verification:**
```bash
curl -s http://localhost:3001/api/v1/station/config | jq '.stationCode'
# Expected: "PTSC001" (or your station code)
```
```

### 14.3: Database Schema

**❌ INSUFFICIENT:**
```markdown
## objects_lk Table

Stores vessel data. Has columns for MMSI, name, and other vessel info.
See the schema file for details.
```

**✅ COMPLIANT:**
```markdown
## Table: mrnxstation.objects_lk

### Overview

The `objects_lk` (objects last known) table stores the current state of every vessel seen by this station. It maintains the most recent values for all vessel attributes, updated each time new AIS data is received. This is the primary table for vessel queries.

### Schema

**Table:** `mrnxstation.objects_lk`

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| obj_mmsi | bigint | NO | - | Maritime Mobile Service Identity (PK) |
| obj_firstseen | timestamptz | NO | - | When vessel was first observed |
| obj_lastseen | timestamptz | NO | - | Most recent observation |
| obj_lk_name | varchar(20) | YES | - | Vessel name from AIS Type 5 |
| obj_lk_callsign | varchar(7) | YES | - | Radio callsign |
| obj_lk_imo | integer | YES | - | IMO number (7 digits) |
| obj_lk_shiptype | smallint | YES | - | AIS ship type code (0-99) |
| obj_lk_length | smallint | YES | - | Length in meters (A+B) |
| obj_lk_breadth | smallint | YES | - | Beam in meters (C+D) |
| obj_lk_draught | numeric(3,1) | YES | - | Draught in meters |
| obj_lk_destination | varchar(20) | YES | - | Reported destination |
| obj_lk_navstatus | smallint | YES | - | Navigation status code (0-15) |
| obj_lk_speed | numeric(4,1) | YES | - | Speed over ground in knots |
| obj_lk_lat | double precision | YES | - | Latitude in decimal degrees |
| obj_lk_lon | double precision | YES | - | Longitude in decimal degrees |
| obj_lk_course | numeric(4,1) | YES | - | Course over ground in degrees |
| obj_lk_heading | smallint | YES | - | True heading in degrees |
| obj_max_speed_observed | numeric(4,1) | YES | - | Maximum speed ever observed |
| obj_max_speed_observed_date | timestamptz | YES | - | When max speed was observed |

### Constraints

| Name | Type | Columns | Details |
|------|------|---------|---------|
| pk_objects_lk | PRIMARY KEY | obj_mmsi | - |
| chk_objects_lk_mmsi | CHECK | obj_mmsi | obj_mmsi > 0 AND obj_mmsi < 999999999 |
| chk_objects_lk_lat | CHECK | obj_lk_lat | obj_lk_lat BETWEEN -90 AND 90 |
| chk_objects_lk_lon | CHECK | obj_lk_lon | obj_lk_lon BETWEEN -180 AND 180 |

### Indexes

| Name | Columns | Type | Purpose |
|------|---------|------|---------|
| pk_objects_lk | obj_mmsi | btree (PK) | Primary key lookups |
| idx_objects_lk_lastseen | obj_lastseen DESC | btree | Recent vessel queries |
| idx_objects_lk_shiptype | obj_lk_shiptype | btree | Filter by vessel type |

### Example Queries

**Get vessel by MMSI:**
```sql
SELECT obj_mmsi, obj_lk_name, obj_lk_lat, obj_lk_lon, obj_lastseen
FROM mrnxstation.objects_lk
WHERE obj_mmsi = 311234567;
```

**Get active vessels (seen in last hour):**
```sql
SELECT obj_mmsi, obj_lk_name, obj_lk_speed, obj_lk_course
FROM mrnxstation.objects_lk
WHERE obj_lastseen > NOW() - INTERVAL '1 hour'
ORDER BY obj_lastseen DESC;
```

**Count vessels by type:**
```sql
SELECT obj_lk_shiptype, COUNT(*) as vessel_count
FROM mrnxstation.objects_lk
WHERE obj_lastseen > NOW() - INTERVAL '24 hours'
GROUP BY obj_lk_shiptype
ORDER BY vessel_count DESC;
```

### Schema Verification

```sql
-- Verify table structure
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_schema = 'mrnxstation' AND table_name = 'objects_lk'
ORDER BY ordinal_position;
-- Expected: 18 rows matching schema above
```

---

**Last Updated:** 2025-01-10
```

---

## 15. Cross-Referencing

### 15.1: Internal Links

**From project root:**
```markdown
See [`webapp_deployment_architecture.md`](./documentation/webapp_deployment_architecture.md)
```

**Within documentation folder:**
```markdown
See also: [`station_ingestion_paradigm.md`](./station_ingestion_paradigm.md)
```

**To specific section:**
```markdown
See [Database Schema Documentation](#103-database-schema-documentation) in Section 10.
```

### 15.2: External Links

Always include:
1. Link text describing the resource
2. URL
3. Summary of what's at the link

```markdown
For PostgreSQL connection pooling options, see
[PostgreSQL 16 Documentation - Connection Pooling](https://www.postgresql.org/docs/16/runtime-config-connection.html),
which covers `max_connections`, `shared_buffers`, and connection limits.
```

---

## 16. AI Assistant Rules

When creating or updating documentation:

1. **Identify document type** → Use appropriate template from Section 10
2. **Apply the Clarifying Question Rule** → Every sentence must be self-explanatory
3. **Check for forbidden phrases** → Run mental grep for Section 9 violations
4. **Verify completeness** → Apply Detail Multiplication rules from Section 4
5. **Use realistic examples** → Project-specific values per Section 6
6. **Ensure self-containment** → No external dependencies per Section 5
7. **Test all commands** → Every code block should be executable
8. **Run self-verification** → Apply all tests from Section 8
9. **Update cross-references** → After moving/renaming, search for old paths
10. **Set correct status:**
    - `Draft` - Work in progress, doesn't meet standards
    - `In Review` - Meets standards, awaiting approval
    - `Complete` - Approved and verified
    - `Deprecated` - Outdated, see replacement

---

## 17. Maintenance

### 17.1: Review Triggers

Documentation must be reviewed when:
- Referenced code/schema changes
- New features affect documented systems
- Bugs reveal documentation gaps
- User questions indicate confusion

### 17.2: Deprecation Process

1. Add deprecation banner at top:
   ```markdown
   > **DEPRECATED:** This document is outdated as of YYYY-MM-DD.
   > See [`replacement_doc.md`](./replacement_doc.md) for current information.
   ```

2. Update status to `**Status:** Deprecated`

3. Move to `archive/documentation/` after 3 months

---

## Quick Reference

### Decision Tree: Where Does This Go?

```
Is it architecture/design/deployment/database?
  ├─ YES → documentation/ (use appropriate prefix)
  └─ NO ↓

Is it specific to a code module?
  ├─ YES → Keep with code (README.md or UPPERCASE.md)
  └─ NO ↓

Is it a plan or work-in-progress?
  ├─ YES → plans/
  └─ NO → Discuss with team
```

### Prefix Quick Reference

| If documenting... | Use prefix... |
|-------------------|---------------|
| Station features | `station_` |
| Web app (API/client) | `webapp_` |
| Ingress pipeline | `ingress_` |
| Database entities | `datamodel_` |
| Cross-cutting/meta | `general_` |
| PostgreSQL ops | `postgres16_` |
| AIS protocol | `ais_` |
| Central server | `central_` |
| Computer Vision | `cv_` |

### Suffix Quick Reference

| Folder | Required Suffix | Example |
|--------|-----------------|---------|
| `/documentation/` | `_documentation`, `_guidelines`, or `_skill` | `webapp_deployment_architecture_documentation.md` |
| `/plans/` | `_plan` | `mmolid_migration_plan.md` |

---

**Last Updated:** 2026-02-01
**Status:** Complete
**Owner:** MarNexii Platform Team

**Note:** This document enforces its own standards. Any violation found in this document should be reported and corrected immediately.
