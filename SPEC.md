# Agent Artifact Handoff (AAH) Specification

**Version:** 0.3.0  
**Status:** Active  
**Authors:** Gracie Redfern, Crouton  
**Date:** 2026-02-19

---

## Abstract

The Agent Artifact Handoff (AAH) specification defines a standard format for artifacts produced by AI agents. It enables interoperability between agent frameworks, consistent rendering in user interfaces, and reliable handoffs between agents in multi-agent systems.

## Motivation

As AI agents become more capable, they produce increasing volumes of artifacts: research documents, code, analysis, specs, and more. Today, these artifacts are stored ad-hoc in local filesystems, Git repositories, or proprietary formats. This fragmentation creates problems:

- **Inaccessibility**: Artifacts trapped on local machines can't be viewed remotely
- **No provenance**: Who created this? When? As part of what task?
- **No interop**: Agent A's output can't easily flow to Agent B
- **No lifecycle**: Everything is permanent or manually deleted
- **No standards**: Every framework invents its own format

AAH addresses these problems with a minimal, extensible specification.

---

## Terminology

- **Artifact**: Any discrete output produced by an agent (document, code, image, structured data)
- **Agent**: An AI system that performs tasks autonomously or semi-autonomously
- **Handoff**: The transfer of an artifact from one agent to another
- **Session**: A bounded execution context for an agent
- **Framework**: Software that orchestrates agent execution (e.g., Clawdbot, CrewAI, AutoGen)
- **Section**: A named portion of a sectioned artifact, owned by a specific agent *(v0.2)*
- **Initiative**: A logical grouping for related artifacts (e.g., "free-trial-removal") *(v0.2)*
- **Section Type**: Semantic category of a section (overview, research, spec, roadmap, task, decision) *(v0.3)*
- **Task Section**: A section representing actionable work with status, priority, and assignee *(v0.3)*
- **Task Reference**: Inline reference to a task section using `{{task:id}}` syntax *(v0.3)*

---

## Specification

### 1. Artifact Envelope

Every AAH artifact MUST be wrapped in an envelope with the following structure:

```json
{
  "aah_version": "0.2",
  "artifact": { ... },
  "source": { ... },
  "content": { ... },
  "sections": [ ... ],
  "lifecycle": { ... },
  "handoff": { ... },
  "extensions": { ... }
}
```

**Note:** Simple artifacts use `content`. Sectioned artifacts use `sections`. An artifact MUST have either `content` or `sections`, but not both.

### 2. Core Fields

#### 2.1 `aah_version` (REQUIRED)

The version of the AAH specification this artifact conforms to.

```json
"aah_version": "0.2"
```

#### 2.2 `artifact` (REQUIRED)

Core artifact identification.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique identifier (recommend UUID or prefixed ID) |
| `type` | string | Yes | Artifact type (see Section 3) |
| `title` | string | No | Human-readable title |
| `summary` | string | No | Brief description (max 500 chars) |
| `initiative` | string | No | Initiative identifier for grouping *(v0.2)* |
| `version` | integer | No | Version number (1-indexed, default 1) |
| `version_id` | string | No | Unique ID for this specific version |
| `previous_version_id` | string | No | ID of the previous version (for version chains) |
| `created_at` | ISO 8601 | Yes | Creation timestamp |
| `updated_at` | ISO 8601 | No | Last modification timestamp |

```json
"artifact": {
  "id": "aah_7f3b2a1c-9d4e-4f5a-8b6c-1e2d3f4a5b6c",
  "type": "document/markdown",
  "title": "Competitor Analysis: Table Management Software",
  "summary": "Analysis of 5 major competitors in the restaurant table management space",
  "initiative": "competitor-research-q1",
  "version": 1,
  "created_at": "2026-02-11T23:10:28Z"
}
```

#### 2.3 `source` (REQUIRED)

Provenance information about who/what created this artifact.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `agent_id` | string | Yes | Identifier for the agent |
| `agent_name` | string | No | Human-readable agent name |
| `agent_role` | string | No | Agent's role (e.g., "researcher", "developer") |
| `framework` | string | No | Agent framework used |
| `framework_version` | string | No | Framework version |
| `session_id` | string | No | Session/run identifier |
| `task_id` | string | No | Task or ticket identifier |
| `task_description` | string | No | What the agent was trying to accomplish |
| `model` | string | No | LLM model used (e.g., "claude-sonnet-4") |
| `organization_id` | string | No | Org identifier for multi-tenant systems |

```json
"source": {
  "agent_id": "research-agent-01",
  "agent_name": "Research Agent",
  "agent_role": "researcher",
  "framework": "clawdbot",
  "framework_version": "1.2.3",
  "session_id": "ses_abc123",
  "task_id": "ART-456",
  "task_description": "Research competitor table management features",
  "model": "claude-sonnet-4"
}
```

#### 2.4 `content` (CONDITIONAL)

The actual artifact content. Required for simple artifacts; omit for sectioned artifacts.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `media_type` | string | Yes | MIME type of content |
| `encoding` | string | No | Character encoding (default: utf-8) |
| `body` | string | Conditional | Inline content (for small artifacts) |
| `body_url` | string | Conditional | URL to fetch content (for large artifacts) |
| `body_hash` | string | No | SHA-256 hash of content (for integrity + deduplication) |
| `size_bytes` | integer | No | Content size in bytes |
| `token_count` | integer | No | Estimated token count (for LLM context budgeting) |

Either `body` or `body_url` MUST be present (when using `content`).

```json
"content": {
  "media_type": "text/markdown",
  "encoding": "utf-8",
  "body": "# Competitor Analysis\n\n## Overview\n...",
  "size_bytes": 4523,
  "token_count": 1847
}
```

**Supported media types:**

| Category | Media Types |
|----------|-------------|
| Documents | `text/markdown`, `text/plain`, `text/html` |
| Code | `text/x-python`, `text/javascript`, `text/typescript`, `application/json`, `text/yaml` |
| Data | `application/json`, `text/csv`, `application/x-ndjson` |
| Images | `image/png`, `image/jpeg`, `image/svg+xml` |
| Structured | `application/vnd.aah.structured+json` (see Section 5) |

#### 2.5 `sections` (CONDITIONAL) *(v0.2)*

An array of sections for collaborative artifacts. Required for sectioned artifacts; omit for simple artifacts.

Each section represents a distinct portion of the document, owned by a specific agent.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Stable section identifier (e.g., "baseline", "day-1") |
| `heading` | string | Yes | Display title for the section |
| `content` | string | Yes | Markdown content |
| `agent_id` | string | Yes | Agent that owns this section |
| `agent_name` | string | No | Human-readable agent name |
| `position` | integer | No | Display order (0-indexed, default: insertion order) |
| `created_at` | ISO 8601 | Yes | When section was created |
| `type` | string | No | Section type (see 2.5.1) *(v0.3)* |
| `status` | string | No | Approval status (see 2.5.3) *(v0.3)* |
| `updated_at` | ISO 8601 | Yes | When section was last modified |
| `version` | integer | No | Section version number |

```json
"sections": [
  {
    "id": "overview",
    "heading": "Overview",
    "content": "**Experiment ID:** PS-EXP-001\n**Status:** Running",
    "agent_id": "experiments-manager",
    "agent_name": "Experiments Manager",
    "position": 0,
    "created_at": "2026-02-16T10:00:00Z",
    "updated_at": "2026-02-17T14:30:00Z",
    "version": 2
  },
  {
    "id": "baseline",
    "heading": "Baseline Data",
    "content": "| Metric | Value |\n|--------|-------|\n| Subs | 47 |",
    "agent_id": "analytics-manager",
    "agent_name": "Analytics Manager",
    "position": 1,
    "created_at": "2026-02-17T06:00:00Z",
    "updated_at": "2026-02-17T06:00:00Z",
    "version": 1
  }
]
```

##### 2.5.1 Section Types *(v0.3)*

Sections MAY have a `type` field indicating their semantic purpose.

| Type | Description |
|------|-------------|
| `overview` | High-level summary or context |
| `research` | Research findings, competitive analysis, data gathering |
| `spec` | Product specification, requirements, design doc |
| `roadmap` | Prioritized feature list, timeline, milestones |
| `task` | Actionable work item (see 2.5.2) |
| `decision` | Decision record, rationale, outcome |

Implementations MAY define additional types using the `x-` prefix (e.g., `x-custom/mytype`).

##### 2.5.2 Task Sections *(v0.3)*

Sections with `type: "task"` represent actionable work items and support additional fields:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `task_status` | string | No | `pending`, `in_progress`, `blocked`, `done` |
| `priority` | integer | No | 1=high, 2=medium, 3=low |
| `assignee_agent_id` | string | No | Agent responsible for execution |
| `source_section_id` | string | No | Section ID where task originated |
| `started_at` | ISO 8601 | No | When work began |
| `completed_at` | ISO 8601 | No | When work finished |
| `output_url` | string | No | URL to deliverable (PR, doc, deployment) |
| `output_type` | string | No | `pr`, `doc`, `artifact`, `deployment` |

**Task section example:**

```json
{
  "id": "task-rsvp-tracking",
  "type": "task",
  "heading": "Implement RSVP Tracking",
  "content": "## Requirements\n- Track invited/confirmed/declined...",
  "agent_id": "product-manager",
  "task_status": "in_progress",
  "priority": 2,
  "assignee_agent_id": "engineering-agent",
  "source_section_id": "roadmap",
  "started_at": "2026-02-18T10:00:00Z",
  "created_at": "2026-02-17T14:00:00Z",
  "updated_at": "2026-02-19T09:30:00Z"
}
```

##### 2.5.3 Section Approval Workflow *(v0.3)*

Sections MAY require human approval. These fields track the approval process:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `status` | string | No | `draft`, `needs_approval`, `approved`, `needs_changes` |
| `approval_requested_at` | ISO 8601 | No | When approval was requested |
| `approval_requested_by` | string | No | Agent that requested approval |
| `approval_note` | string | No | Message explaining what needs review |
| `decided_at` | ISO 8601 | No | When decision was made |
| `decided_by` | string | No | Who decided (agent_id or "human") |
| `decision_note` | string | No | Feedback or rationale |

**Workflow:**
1. Agent creates section with `status: "draft"`
2. Agent sets `status: "needs_approval"` with `approval_note`
3. Human sets `status: "approved"` or `status: "needs_changes"` with `decision_note`

##### 2.5.4 Task References *(v0.3)*

Content MAY reference task sections using this syntax:

**Same-artifact:** `{{task:<section_id>}}`

**Cross-artifact:** `{{task:<artifact_id>:<section_id>}}`

**Example:**
```markdown
## Tier 2 Features
{{task:task-rsvp-tracking}}
{{task:task-realtime-collab}}

## Dependencies
{{task:aah_infra_001:task-auth-system}}
```

Implementations SHOULD render references showing task heading and status. Invalid references SHOULD render as plain text or "not found" indicator.

#### 2.6 `section_update` (CONDITIONAL) *(v0.2)*

For updating a single section without uploading the entire artifact. When present, this indicates a partial update.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Section identifier to update |
| `heading` | string | No | New heading (if changing) |
| `content` | string | Yes | New content |
| `change_note` | string | No | Description of what changed |

```json
{
  "aah_version": "0.2",
  "artifact": {
    "id": "aah_abc123"
  },
  "section_update": {
    "id": "baseline",
    "content": "Updated baseline metrics...",
    "change_note": "Added conversion rate data"
  },
  "source": {
    "agent_id": "analytics-manager"
  }
}
```

#### 2.7 `lifecycle` (OPTIONAL)

Artifact lifecycle management.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `retention` | string | No | Retention policy: `ephemeral`, `7d`, `30d`, `90d`, `permanent` |
| `expires_at` | ISO 8601 | No | Explicit expiration timestamp |
| `visibility` | string | No | Access level: `private`, `team`, `organization`, `public` |
| `status` | string | No | Artifact status: `draft`, `active`, `needs_approval`, `final`, `superseded`, `archived` |
| `superseded_by` | string | No | ID of artifact that replaces this one |
| `tags` | array | No | Freeform tags for categorization |

```json
"lifecycle": {
  "retention": "30d",
  "visibility": "team",
  "status": "active",
  "tags": ["experiment", "planseats", "free-trial-removal"]
}
```

#### 2.8 `handoff` (OPTIONAL)

Information for agent-to-agent handoffs.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `target_agent` | string | No | Intended recipient agent ID |
| `target_role` | string | No | Intended recipient role (if agent unknown) |
| `expects_response` | boolean | No | Whether a response artifact is expected |
| `response_deadline` | ISO 8601 | No | Deadline for response |
| `parent_artifact_ids` | array | No | IDs of artifacts this one derives from |
| `context` | object | No | Additional context for recipient agent |
| `priority` | string | No | `low`, `normal`, `high`, `urgent` |

```json
"handoff": {
  "target_role": "pm",
  "expects_response": true,
  "response_deadline": "2026-02-12T12:00:00Z",
  "parent_artifact_ids": ["aah_parent123"],
  "priority": "normal",
  "context": {
    "next_step": "Create feature tickets based on this research",
    "constraints": "Focus on features we can ship in Q1"
  }
}
```

#### 2.9 `extensions` (OPTIONAL)

Framework-specific or custom extensions. Extensions SHOULD be namespaced.

```json
"extensions": {
  "clawdbot": {
    "gateway_id": "gw_abc123",
    "channel": "slack"
  },
  "linear": {
    "issue_id": "ART-456",
    "project_id": "proj_xyz"
  }
}
```

---

### 3. Artifact Types

Artifact types follow a `category/subtype` pattern:

| Type | Description |
|------|-------------|
| `document/markdown` | Markdown document |
| `document/text` | Plain text document |
| `document/html` | HTML document |
| `document/sectioned` | Multi-section collaborative document *(v0.2)* |
| `code/python` | Python source code |
| `code/javascript` | JavaScript source code |
| `code/typescript` | TypeScript source code |
| `code/generic` | Code in other languages |
| `data/json` | JSON data |
| `data/yaml` | YAML data |
| `data/csv` | CSV tabular data |
| `image/screenshot` | Screenshot image |
| `image/diagram` | Diagram or chart |
| `image/generated` | AI-generated image |
| `structured/spec` | Product/technical specification |
| `structured/analysis` | Analysis or research output |
| `structured/plan` | Plan or roadmap |
| `structured/review` | Code review or QA output |
| `structured/experiment` | Experiment tracking *(v0.2)* |
| `structured/feature` | Feature development tracking *(v0.2)* |

Implementations MAY define additional types using the `x-` prefix (e.g., `x-custom/mytype`).

---

### 4. Sectioned Artifacts *(v0.2)*

Sectioned artifacts enable multiple agents to collaborate on a single document.

#### 4.1 When to Use Sectioned Artifacts

Use sectioned artifacts when:
- Multiple agents contribute to the same initiative
- Content has distinct logical sections with different owners
- You want to track who wrote what
- You want section-level version history

Use simple artifacts when:
- Single agent, single output
- Content is atomic (can't be meaningfully divided)
- Historical simplicity is preferred

#### 4.2 Section Identification

Section IDs MUST be:
- Lowercase alphanumeric with hyphens
- Stable across updates (same ID = same section)
- Unique within an artifact

Recommended section IDs by artifact type:

**Experiments:**
- `overview`, `hypothesis`, `measurement-plan`, `baseline`, `rollback`, `day-1`, `day-2`, etc.

**Features:**
- `overview`, `requirements`, `design`, `implementation`, `qa`, `launch`

**Research:**
- `summary`, `methodology`, `findings`, `recommendations`, `appendix`

#### 4.3 Section Ordering

Sections are displayed in order determined by:
1. `position` field (if specified)
2. Insertion order (if no position)

#### 4.4 Section Updates

To update a section, send a `section_update` envelope:

```json
{
  "aah_version": "0.2",
  "artifact": { "id": "aah_abc123" },
  "section_update": {
    "id": "baseline",
    "content": "New content..."
  },
  "source": { "agent_id": "analytics-manager" }
}
```

The server SHOULD:
1. Archive the previous section content
2. Update the section with new content
3. Increment the section version
4. Update `updated_at` timestamp
5. Record the `agent_id` of the updater

#### 4.5 Finding Existing Artifacts

Before creating a new artifact, agents SHOULD check if one exists for the same initiative:

```
GET /artifacts?initiative=free-trial-removal
```

If found, add sections to the existing artifact. If not found, create a new sectioned artifact.

---

### 5. Structured Artifacts

For artifacts with well-defined schemas, use `application/vnd.aah.structured+json` with a `schema` field:

```json
"content": {
  "media_type": "application/vnd.aah.structured+json",
  "body": {
    "schema": "aah:analysis/competitor",
    "data": {
      "competitors": [
        {
          "name": "OpenTable",
          "strengths": ["Brand recognition", "Large network"],
          "weaknesses": ["High fees", "Complex onboarding"]
        }
      ],
      "recommendation": "Focus on SMB segment underserved by OpenTable"
    }
  }
}
```

---

### 6. Transport

AAH is transport-agnostic. Artifacts MAY be transmitted via:

- **HTTP API**: POST/PUT to artifact storage endpoints
- **Message queues**: Kafka, RabbitMQ, SQS
- **File system**: Written as `.aah.json` files
- **Embedded**: Inline in agent framework messages

Implementations SHOULD support JSON serialization. Implementations MAY additionally support YAML or MessagePack.

---

### 7. Discovery & Querying

Artifact storage systems implementing AAH SHOULD support querying by:

- `artifact.id` — exact match
- `artifact.initiative` — artifacts for an initiative *(v0.2)*
- `source.agent_id` — artifacts from a specific agent
- `source.session_id` — artifacts from a specific session
- `source.task_id` — artifacts related to a task
- `artifact.type` — by artifact type
- `lifecycle.status` — by status
- `lifecycle.tags` — by tags
- `created_at` — date range

---

### 8. Rendering Hints

To assist UI rendering, artifacts MAY include rendering hints in extensions:

```json
"extensions": {
  "rendering": {
    "preferred_view": "split",
    "syntax_highlight": "python",
    "collapse_sections": ["methodology", "appendix"],
    "table_of_contents": true
  }
}
```

---

## Security Considerations

- **Authentication**: Artifact storage systems SHOULD require authentication for upload and access
- **Authorization**: Respect `visibility` field; enforce organization boundaries
- **Content validation**: Validate content matches declared `media_type`
- **Size limits**: Implementations SHOULD enforce maximum artifact sizes
- **Secrets**: Artifacts MUST NOT contain credentials, API keys, or other secrets
- **Section permissions**: Implementations MAY restrict which agents can update which sections

---

## Backward Compatibility

### v0.2 → v0.3

- All v0.2 artifacts remain valid
- `section.type` is optional; untyped sections behave as before
- Task fields only apply when `type: "task"`
- Approval fields are optional on all sections
- `{{task:id}}` references are plain text in non-aware implementations

### v0.1 → v0.2

- All v0.1 artifacts remain valid
- `sections` and `section_update` are additive
- `artifact.initiative` is optional
- Servers MUST accept both simple and sectioned artifacts
- Clients MAY support only simple artifacts

Detection logic:
```python
if section.get("type") == "task":
    # Task section with extended fields (v0.3)
elif section.get("status") in ["needs_approval", "approved", "needs_changes"]:
    # Section with approval workflow (v0.3)
elif "sections" in envelope or "section_update" in envelope:
    # Sectioned artifact (v0.2)
elif "content" in envelope:
    # Simple artifact (v0.1 compatible)
```

---

## Future Considerations

The following are under consideration for future versions:

- **Signing**: Cryptographic signatures for artifact authenticity
- **Streaming**: Support for streaming large artifacts
- **Relationships**: Richer artifact relationship types (derives-from, supersedes, relates-to)
- **Schemas registry**: Central registry for structured artifact schemas
- **Reactions/Comments**: Human feedback on artifacts
- **Concurrent editing**: Conflict resolution for simultaneous section updates
- **Task dependencies**: Formal dependency graph between tasks

---

## Changelog

### v0.3.0 (2026-02-19)

- Added `section.type` field for semantic categorization (overview, research, spec, roadmap, task, decision)
- Added task section fields: task_status, priority, assignee_agent_id, source_section_id, started_at, completed_at, output_url, output_type
- Added task reference syntax: `{{task:<section_id>}}` and `{{task:<artifact_id>:<section_id>}}`
- Added section approval workflow: status, approval_requested_at, approval_requested_by, approval_note, decided_at, decided_by, decision_note

### v0.2.0 (2026-02-18)

- Added sectioned artifacts (`document/sectioned` type)
- Added `sections` array for multi-agent collaboration
- Added `section_update` for partial updates
- Added `artifact.initiative` for grouping
- Added new artifact types: `structured/experiment`, `structured/feature`
- Added section-level versioning

### v0.1.0 (2026-02-11)

- Initial specification
- Core envelope structure
- Simple artifact support
- Lifecycle and handoff fields

---

## Appendix A: JSON Schema

Formal JSON Schemas for AAH v0.2 are available at:

- [`schemas/artifact.schema.json`](./schemas/artifact.schema.json)
- [`schemas/section.schema.json`](./schemas/section.schema.json)

---

## Appendix B: Reference Implementations

| Implementation | Type | Status |
|---------------|------|--------|
| [Artyfacts](https://artyfacts.dev) | Storage + Viewer | Reference |
| [@artyfacts/sdk](https://github.com/artygracie/artyfacts/tree/main/packages/sdk) | TypeScript SDK | Active |
| artyfacts-python | Python SDK | Planned |

---

## Appendix C: Example Artifacts

### Simple Artifact (v0.1 compatible)

```json
{
  "aah_version": "0.2",
  "artifact": {
    "id": "aah_research_001",
    "type": "document/markdown",
    "title": "Competitor Analysis",
    "created_at": "2026-02-18T10:00:00Z"
  },
  "source": {
    "agent_id": "research-agent",
    "framework": "clawdbot"
  },
  "content": {
    "media_type": "text/markdown",
    "body": "# Competitor Analysis\n\n## Overview\n..."
  },
  "lifecycle": {
    "status": "final",
    "tags": ["research", "competitors"]
  }
}
```

### Sectioned Artifact (v0.2)

```json
{
  "aah_version": "0.2",
  "artifact": {
    "id": "aah_experiment_001",
    "type": "document/sectioned",
    "title": "Paywall Experiment (PS-EXP-001)",
    "initiative": "free-trial-removal",
    "created_at": "2026-02-16T10:00:00Z"
  },
  "source": {
    "agent_id": "experiments-manager",
    "framework": "clawdbot"
  },
  "sections": [
    {
      "id": "overview",
      "heading": "Overview",
      "content": "**Experiment ID:** PS-EXP-001\n**Status:** Running",
      "agent_id": "experiments-manager",
      "created_at": "2026-02-16T10:00:00Z",
      "updated_at": "2026-02-16T10:00:00Z",
      "version": 1
    },
    {
      "id": "baseline",
      "heading": "Baseline Data",
      "content": "| Metric | Value |\n|--------|-------|\n| Subs | 47 |",
      "agent_id": "analytics-manager",
      "created_at": "2026-02-17T06:00:00Z",
      "updated_at": "2026-02-17T06:00:00Z",
      "version": 1
    }
  ],
  "lifecycle": {
    "status": "active",
    "tags": ["experiment", "planseats"]
  }
}
```

### Section Update (v0.2)

```json
{
  "aah_version": "0.2",
  "artifact": {
    "id": "aah_experiment_001"
  },
  "section_update": {
    "id": "baseline",
    "content": "| Metric | Value |\n|--------|-------|\n| Subs | 47 |\n| Revenue | $269 |",
    "change_note": "Added revenue metric"
  },
  "source": {
    "agent_id": "analytics-manager"
  }
}
```
