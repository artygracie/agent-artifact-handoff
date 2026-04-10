# Agent Artifact Handoff (AAH) Specification

**Version:** 0.5.0  
**Status:** Active  
**Authors:** Gracie Redfern, Claude Code  
**Date:** 2026-04-10

---

## Abstract

The Agent Artifact Handoff (AAH) specification defines a standard format for artifacts produced by AI agents. It enables interoperability between agent frameworks, consistent rendering in user interfaces, and reliable handoffs between agents in multi-agent systems.

## Motivation

As agent harnesses proliferate and become more capable, the volume of agent-produced artifacts grows with them. Research documents, analysis, specs, plans, design reviews, experiment logs — agents generate enormous amounts of valuable work product that is *not code*. Code has Git. But the non-code artifacts that inform, justify, and coordinate that code have no standard home.

Today, these artifacts are stored ad-hoc in local filesystems, chat histories, or proprietary formats. This fragmentation creates problems:

- **Inaccessibility**: Artifacts trapped on local machines or ephemeral sessions can't be viewed remotely
- **No provenance**: Who created this? When? As part of what task?
- **No interop**: Agent A's output can't easily flow to Agent B
- **No lifecycle**: Everything is permanent or manually deleted
- **No standards**: Every framework invents its own format

As the number of agent frameworks grows — each with its own harness, orchestration layer, and output conventions — the interoperability problem compounds. Without a shared format, artifacts become siloed within the framework that produced them.

AAH addresses these problems with a minimal, extensible specification.

---

## Terminology

- **Artifact**: A container for one or more sections of content produced by agents
- **Section**: A discrete piece of content within an artifact, with its own type and owner
- **Agent**: An AI system that performs tasks autonomously or semi-autonomously
- **Handoff**: The transfer of an artifact from one agent to another
- **Session**: A bounded execution context for an agent
- **Framework**: Software that orchestrates agent execution (e.g., Clawdbot, CrewAI, AutoGen)

---

## Specification

### 1. Artifact Envelope

Every AAH artifact MUST be wrapped in an envelope with the following structure:

```json
{
  "aah_version": "0.5",
  "artifact": { ... },
  "sections": [ ... ],
  "source": { ... },
  "lifecycle": { ... },
  "handoff": { ... },
  "extensions": { ... }
}
```

An artifact MUST contain at least one section in `sections`. Only `aah_version`, `artifact`, and `sections` are required at the envelope level.

For partial updates, `sections` is replaced by `section_update` (see Section 2.5).

### 2. Core Fields

#### 2.1 `aah_version` (REQUIRED)

The version of the AAH specification this artifact conforms to.

```json
"aah_version": "0.5"
```

#### 2.2 `artifact` (REQUIRED)

Core artifact identification.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique identifier (recommend UUID or prefixed ID) |
| `type` | string | No | Artifact type — the purpose of this artifact (see Section 3) |
| `title` | string | No | Human-readable title |
| `summary` | string | No | Brief description (max 500 chars) |
| `version` | integer | No | Version number (1-indexed, default 1) |
| `version_id` | string | No | Unique ID for this specific version |
| `previous_version_id` | string | No | ID of the previous version (for version chains) |
| `created_at` | ISO 8601 | Yes | Creation timestamp |
| `updated_at` | ISO 8601 | No | Last modification timestamp |

```json
"artifact": {
  "id": "aah_7f3b2a1c-9d4e-4f5a-8b6c-1e2d3f4a5b6c",
  "type": "research",
  "title": "Competitor Analysis: Table Management Software",
  "summary": "Analysis of 5 major competitors in the restaurant table management space",
  "version": 1,
  "created_at": "2026-02-11T23:10:28Z"
}
```

#### 2.3 `source` (OPTIONAL)

Additional provenance context for the artifact. Section authorship is tracked via `agent_id` on each section (see 2.4). Use `source` when you want to record the broader context — which framework, session, or model produced this artifact.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `agent_id` | string | No | Identifier for the creating agent |
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

#### 2.4 `sections` (REQUIRED)

An array of sections that make up the artifact's content. Every artifact MUST have at least one section.

Each section is a discrete piece of content with its own type, body, and owning agent. This allows a single artifact to contain markdown, code, images, data, and more — each contributed by different agents.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Stable section identifier (e.g., "overview", "api-schema") |
| `heading` | string | Yes | Display title for the section |
| `type` | string | Yes | Content type in `category/subtype` format (see 2.4.1) |
| `body` | string or object | Conditional | Inline content |
| `body_url` | string | Conditional | URL to fetch content (for large or binary content) |
| `media_type` | string | No | MIME type override (derived from `type` if omitted) |
| `encoding` | string | No | Character encoding (default: utf-8) |
| `body_hash` | string | No | SHA-256 hash of content (for integrity + deduplication) |
| `size_bytes` | integer | No | Content size in bytes |
| `token_count` | integer | No | Estimated token count (for LLM context budgeting) |
| `agent_id` | string | Yes | Agent that owns this section |
| `agent_name` | string | No | Human-readable agent name |
| `position` | integer | No | Display order (0-indexed, default: insertion order) |
| `created_at` | ISO 8601 | Yes | When section was created |
| `updated_at` | ISO 8601 | Yes | When section was last modified |
| `version` | integer | No | Section version number |

Either `body` or `body_url` MUST be present. Use `body` for inline text, code, and data. Use `body_url` for large or binary content (images, PDFs, etc.).

```json
"sections": [
  {
    "id": "overview",
    "heading": "Overview",
    "type": "document/markdown",
    "body": "# Competitor Analysis\n\n## Summary\nWe evaluated 5 competitors...",
    "agent_id": "research-agent",
    "agent_name": "Research Agent",
    "position": 0,
    "created_at": "2026-02-16T10:00:00Z",
    "updated_at": "2026-02-17T14:30:00Z",
    "version": 2
  },
  {
    "id": "comparison-data",
    "heading": "Feature Comparison",
    "type": "data/csv",
    "body": "Feature,OpenTable,Resy,Tock\nOnline Booking,Yes,Yes,Yes\nWaitlist,Yes,No,Yes",
    "agent_id": "analytics-agent",
    "position": 1,
    "created_at": "2026-02-17T06:00:00Z",
    "updated_at": "2026-02-17T06:00:00Z",
    "version": 1
  },
  {
    "id": "mockup",
    "heading": "UI Mockup",
    "type": "image/png",
    "body_url": "https://artyfacts.dev/assets/aah_abc123/mockup.png",
    "size_bytes": 245000,
    "agent_id": "design-agent",
    "position": 2,
    "created_at": "2026-02-18T09:00:00Z",
    "updated_at": "2026-02-18T09:00:00Z",
    "version": 1
  }
]
```

##### 2.4.1 Section Content Types

Section `type` follows a `category/subtype` pattern describing the content format:

| Type | Description | Default media type |
|------|-------------|--------------------|
| `document/markdown` | Markdown document | `text/markdown` |
| `document/text` | Plain text | `text/plain` |
| `document/html` | HTML document | `text/html` |
| `code/python` | Python source code | `text/x-python` |
| `code/javascript` | JavaScript source code | `text/javascript` |
| `code/typescript` | TypeScript source code | `text/typescript` |
| `code/generic` | Code in other languages | `text/plain` |
| `data/json` | JSON data | `application/json` |
| `data/yaml` | YAML data | `text/yaml` |
| `data/csv` | CSV tabular data | `text/csv` |
| `data/ndjson` | Newline-delimited JSON | `application/x-ndjson` |
| `image/png` | PNG image | `image/png` |
| `image/jpeg` | JPEG image | `image/jpeg` |
| `image/svg` | SVG image | `image/svg+xml` |
| `structured/json` | Structured data with schema (see Section 5) | `application/json` |

If `media_type` is omitted on a section, it is derived from the `type` using the defaults above. Specify `media_type` explicitly when the default doesn't apply (e.g., `type: "code/generic"` with `media_type: "text/x-rust"`).

Implementations MAY define additional types using the `x-` prefix (e.g., `x-custom/mytype`).

#### 2.5 `section_update` (CONDITIONAL)

For updating a single section without uploading the entire artifact. A section update envelope contains `aah_version`, `artifact` (with at least `id`), and `section_update`:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Section identifier to update |
| `agent_id` | string | Yes | Agent making the update |
| `heading` | string | No | New heading (if changing) |
| `type` | string | No | New content type (if changing) |
| `body` | string or object | Conditional | New inline content |
| `body_url` | string | Conditional | New content URL |
| `change_note` | string | No | Description of what changed |

Either `body` or `body_url` MUST be present.

```json
{
  "aah_version": "0.5",
  "artifact": {
    "id": "aah_abc123"
  },
  "section_update": {
    "id": "comparison-data",
    "agent_id": "analytics-agent",
    "body": "Feature,OpenTable,Resy,Tock\nOnline Booking,Yes,Yes,Yes\nWaitlist,Yes,No,Yes\nRSVP,No,No,Yes",
    "change_note": "Added RSVP feature row"
  }
}
```

#### 2.6 `lifecycle` (OPTIONAL)

Artifact lifecycle management.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `retention` | string | No | Retention policy: `ephemeral`, `7d`, `30d`, `90d`, `permanent` |
| `expires_at` | ISO 8601 | No | Explicit expiration timestamp |
| `visibility` | string | No | Access level: `private`, `team`, `organization`, `public` |
| `status` | string | No | Artifact status: `draft`, `active`, `final`, `superseded`, `archived` |
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

#### 2.7 `handoff` (OPTIONAL)

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

#### 2.8 `extensions` (OPTIONAL)

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

Artifact types describe the artifact's purpose — what it represents as a whole. The `type` field is optional; when provided, it helps with discovery and UI rendering. The content format of individual sections is specified by section `type` (see 2.4.1).

| Type | Description |
|------|-------------|
| `document` | General document or collection of content |
| `spec` | Product or technical specification |
| `research` | Research findings, competitive analysis |
| `analysis` | Data analysis, evaluation, investigation |
| `plan` | Plan, roadmap, or timeline |
| `design` | Design document or architecture |
| `proposal` | Proposal, RFC, or pitch |
| `review` | Code review, QA output, or audit |
| `decision` | Decision record, rationale, outcome |
| `report` | Report, summary, or status update |
| `experiment` | Experiment tracking or A/B test |
| `feature` | Feature development tracking |
| `guide` | Documentation, how-to, or runbook |
| `log` | Activity log, changelog, or journal |

Implementations MAY define additional types using the `x-` prefix (e.g., `x-custom/mytype`).

---

### 4. Working with Sections

#### 4.1 Section Identification

Section IDs MUST be:
- Lowercase alphanumeric with hyphens
- Stable across updates (same ID = same section)
- Unique within an artifact

Recommended section IDs by artifact type:

**Specs:**
- `overview`, `requirements`, `mockup`, `api-schema`, `data-model`, `architecture`

**Experiments:**
- `overview`, `hypothesis`, `measurement-plan`, `baseline`, `rollback`, `day-1`, `day-2`, etc.

**Features:**
- `overview`, `requirements`, `design`, `implementation`, `qa`, `launch`

**Research:**
- `summary`, `methodology`, `findings`, `data`, `recommendations`

**Single-section artifacts:**
- `main` — conventional ID when an artifact has only one section

#### 4.2 Section Ordering

Sections are displayed in order determined by:
1. `position` field (if specified)
2. Insertion order (if no position)

#### 4.3 Section Updates

To update a section, send a `section_update` envelope (see Section 2.5).

The server SHOULD:
1. Archive the previous section content
2. Update the section with new body
3. Increment the section version
4. Update `updated_at` timestamp
5. Record the `agent_id` of the updater

---

### 5. Structured Sections

For sections with well-defined schemas, use type `structured/json`. The `body` is a JSON object with a `schema` field identifying the data format:

```json
{
  "id": "competitor-analysis",
  "heading": "Competitor Analysis",
  "type": "structured/json",
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
  },
  "agent_id": "research-agent",
  "created_at": "2026-02-17T10:00:00Z",
  "updated_at": "2026-02-17T10:00:00Z"
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
- `artifact.type` — by artifact purpose
- `source.agent_id` — artifacts from a specific agent
- `source.session_id` — artifacts from a specific session
- `source.task_id` — artifacts related to a task
- `sections.type` — by section content type
- `lifecycle.status` — by status
- `lifecycle.tags` — by tags
- `created_at` — date range

---

### 8. Rendering Hints

To assist UI rendering, artifacts MAY include rendering hints in extensions:

```json
"extensions": {
  "rendering": {
    "table_of_contents": true,
    "collapse_sections": ["data", "appendix"],
    "preview_section": "mockup"
  }
}
```

---

## Security Considerations

- **Authentication**: Artifact storage systems SHOULD require authentication for upload and access
- **Authorization**: Respect `visibility` field; enforce organization boundaries
- **Content validation**: Validate section content matches declared `type` and `media_type`
- **Size limits**: Implementations SHOULD enforce maximum artifact and section sizes
- **Secrets**: Artifacts MUST NOT contain credentials, API keys, or other secrets
- **Section permissions**: Implementations MAY restrict which agents can update which sections

---

## Backward Compatibility

### v0.5 (Breaking Change)

v0.5 is a breaking change from previous versions:

- Artifact-level `content` field has been removed; all content lives in `sections`
- `sections` is now required on all artifacts
- Section `content` field renamed to `body`
- Section `type` now uses content type taxonomy (e.g., `document/markdown`) instead of semantic categories
- Artifact `type` simplified to purpose-based categories (e.g., `spec`, `research`, `plan`)
- `source` is now optional; section `agent_id` is the primary provenance mechanism
- `section_update` now includes `agent_id`; `body` is conditional (either `body` or `body_url`)
- `document/sectioned` artifact type removed (all artifacts contain sections)

### Previous Versions

- v0.4: Removed task sections, approval workflow, task references
- v0.3: Added section types, task sections, approval workflow
- v0.2: Added sectioned artifacts, section_update, initiative grouping
- v0.1: Initial specification

---

## Future Considerations

The following are under consideration for future versions:

- **Signing**: Cryptographic signatures for artifact authenticity
- **Streaming**: Support for streaming large artifacts
- **Relationships**: Richer artifact relationship types (derives-from, supersedes, relates-to)
- **Schemas registry**: Central registry for structured section schemas
- **Reactions/Comments**: Human feedback on artifacts and sections
- **Concurrent editing**: Conflict resolution for simultaneous section updates

---

## Changelog

### v0.5.0 (2026-04-10)

- **Breaking**: Artifacts are now containers for sections; artifact-level `content` field removed
- **Breaking**: `sections` is required on all artifacts (single-section for simple content)
- **Breaking**: Section `content` field renamed to `body`; sections gain `body_url`, `media_type`, `encoding`, `body_hash`, `size_bytes`, `token_count`
- Artifact `type` is now optional; simplified to purpose-based categories: `document`, `spec`, `research`, `analysis`, `plan`, `design`, `proposal`, `review`, `decision`, `report`, `experiment`, `feature`, `guide`, `log`
- Removed `artifact.initiative` field (use `lifecycle.tags` for grouping)
- `source` is now optional on the envelope; section `agent_id` is the primary provenance mechanism
- `section_update` gains `agent_id` field; `body` is now conditional (either `body` or `body_url`)
- Section `type` now uses content type taxonomy: `document/*`, `code/*`, `data/*`, `image/*`, `structured/*`
- Removed `document/sectioned` artifact type
- Removed simple vs. sectioned artifact distinction

### v0.4.0 (2026-04-10)

- Removed task sections and all task-specific fields
- Removed section approval workflow
- Removed task reference syntax
- Retained `section.type` for semantic categorization

### v0.3.0 (2026-02-19)

- Added `section.type` field for semantic categorization
- Added task sections, approval workflow, task references (removed in v0.4)

### v0.2.0 (2026-02-18)

- Added sectioned artifacts (`document/sectioned` type)
- Added `sections` array for multi-agent collaboration
- Added `section_update` for partial updates
- Added `artifact.initiative` for grouping
- Added section-level versioning

### v0.1.0 (2026-02-11)

- Initial specification
- Core envelope structure
- Simple artifact support
- Lifecycle and handoff fields

---

## Appendix A: JSON Schema

Formal JSON Schemas for AAH are available at:

- [`schemas/artifact.schema.json`](./schemas/artifact.schema.json)
- [`schemas/section.schema.json`](./schemas/section.schema.json)

---

## Appendix B: Reference Implementations

| Implementation | Type | Status |
|---------------|------|--------|
| [Artyfacts](https://artyfacts.dev) | The GitHub for agent work that's not code | Reference |
| [`@artyfacts/mcp-server`](https://github.com/artygracie/artyfacts/tree/main/packages/mcp-server) | MCP Server | Active |

---

## Appendix C: Example Artifacts

### Single-Section Artifact

```json
{
  "aah_version": "0.5",
  "artifact": {
    "id": "aah_research_001",
    "type": "research",
    "title": "Competitor Analysis",
    "created_at": "2026-02-18T10:00:00Z"
  },
  "source": {
    "agent_id": "research-agent",
    "framework": "clawdbot"
  },
  "sections": [
    {
      "id": "main",
      "heading": "Competitor Analysis",
      "type": "document/markdown",
      "body": "# Competitor Analysis\n\n## Overview\n...",
      "agent_id": "research-agent",
      "created_at": "2026-02-18T10:00:00Z",
      "updated_at": "2026-02-18T10:00:00Z"
    }
  ],
  "lifecycle": {
    "status": "final",
    "tags": ["research", "competitors"]
  }
}
```

### Multi-Section Artifact (mixed content types)

```json
{
  "aah_version": "0.5",
  "artifact": {
    "id": "aah_spec_001",
    "type": "spec",
    "title": "RSVP Feature Spec",
    "created_at": "2026-03-01T10:00:00Z"
  },
  "sections": [
    {
      "id": "overview",
      "heading": "Overview",
      "type": "document/markdown",
      "body": "## RSVP Tracking\n\nAllow hosts to track guest RSVPs with real-time status updates.\n\n### Goals\n- Track invited/confirmed/declined counts\n- Send automated reminders\n- Provide host dashboard",
      "agent_id": "product-manager",
      "position": 0,
      "created_at": "2026-03-01T10:00:00Z",
      "updated_at": "2026-03-01T10:00:00Z"
    },
    {
      "id": "competitor-data",
      "heading": "Competitor Comparison",
      "type": "data/csv",
      "body": "Feature,PlanSeats,OpenTable,Resy,Tock\nRSVP Tracking,Planned,Yes,No,Yes\nReminders,Planned,Yes,No,No\nDashboard,Planned,Yes,Yes,Yes",
      "agent_id": "research-agent",
      "position": 1,
      "created_at": "2026-03-01T12:00:00Z",
      "updated_at": "2026-03-01T12:00:00Z"
    },
    {
      "id": "mockup",
      "heading": "UI Mockup",
      "type": "image/png",
      "body_url": "https://artyfacts.dev/assets/aah_spec_001/rsvp-mockup.png",
      "size_bytes": 245000,
      "agent_id": "design-agent",
      "position": 2,
      "created_at": "2026-03-02T09:00:00Z",
      "updated_at": "2026-03-02T09:00:00Z"
    },
    {
      "id": "api-schema",
      "heading": "API Schema",
      "type": "code/typescript",
      "body": "interface RSVPResponse {\n  guestId: string;\n  status: 'invited' | 'confirmed' | 'declined';\n  respondedAt?: string;\n}\n\ninterface RSVPSummary {\n  eventId: string;\n  invited: number;\n  confirmed: number;\n  declined: number;\n  pending: number;\n}",
      "agent_id": "engineering-agent",
      "position": 3,
      "created_at": "2026-03-02T14:00:00Z",
      "updated_at": "2026-03-02T14:00:00Z"
    }
  ],
  "lifecycle": {
    "status": "active",
    "tags": ["feature", "rsvp", "planseats"]
  }
}
```

### Section Update

```json
{
  "aah_version": "0.5",
  "artifact": {
    "id": "aah_spec_001"
  },
  "section_update": {
    "id": "competitor-data",
    "agent_id": "research-agent",
    "body": "Feature,PlanSeats,OpenTable,Resy,Tock,Yelp\nRSVP Tracking,Planned,Yes,No,Yes,No\nReminders,Planned,Yes,No,No,No\nDashboard,Planned,Yes,Yes,Yes,Yes",
    "change_note": "Added Yelp to competitor comparison"
  }
}
```
