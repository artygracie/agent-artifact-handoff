# AAH Specification v0.3 Draft

**Proposed Changes from v0.2**

This document outlines the additions for AAH v0.3. These are data format changes only — rendering/UI behavior is implementation-specific.

---

## 1. Section Types

### 1.1 New Field: `section.type`

Sections MAY have a `type` field indicating their semantic purpose.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | string | No | Semantic type of section (see 1.2) |

### 1.2 Standard Section Types

| Type | Description |
|------|-------------|
| `overview` | High-level summary or context |
| `research` | Research findings, competitive analysis, data gathering |
| `spec` | Product specification, requirements, design doc |
| `roadmap` | Prioritized feature list, timeline, milestones |
| `task` | Actionable work item (see Section 2) |
| `decision` | Decision record, rationale, outcome |

Implementations MAY define additional types using the `x-` prefix (e.g., `x-custom/mytype`).

### 1.3 Example

```json
"sections": [
  {
    "id": "overview",
    "type": "overview",
    "heading": "Project Overview",
    "content": "...",
    "agent_id": "product-manager"
  },
  {
    "id": "competitor-analysis",
    "type": "research",
    "heading": "Competitor Analysis",
    "content": "...",
    "agent_id": "research-agent"
  }
]
```

---

## 2. Task Sections

Sections with `type: "task"` represent actionable work items. Task sections support additional fields for tracking execution.

### 2.1 Task-Specific Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `task_status` | string | No | Current status (see 2.2) |
| `priority` | integer | No | Priority level: 1=high, 2=medium, 3=low |
| `assignee_agent_id` | string | No | Agent responsible for execution |
| `source_section_id` | string | No | Section ID where task originated (e.g., roadmap) |
| `started_at` | ISO 8601 | No | When work began |
| `completed_at` | ISO 8601 | No | When work finished |
| `output_url` | string | No | URL to deliverable (PR, doc, deployment) |
| `output_type` | string | No | Type of output (see 2.3) |

### 2.2 Task Status Values

| Status | Description |
|--------|-------------|
| `pending` | Not yet started |
| `in_progress` | Work is underway |
| `blocked` | Waiting on dependency or decision |
| `done` | Completed |

### 2.3 Output Types

| Type | Description |
|------|-------------|
| `pr` | Pull request |
| `doc` | Document (Google Doc, Notion, etc.) |
| `artifact` | Another AAH artifact |
| `deployment` | Deployed URL or environment |

### 2.4 Task Section Example

```json
{
  "id": "task-rsvp-tracking",
  "type": "task",
  "heading": "Implement RSVP Tracking System",
  "content": "## Requirements\n- Track invited/confirmed/declined status\n- Sync with seating chart\n- Dashboard with stats",
  "agent_id": "product-manager",
  "task_status": "in_progress",
  "priority": 2,
  "assignee_agent_id": "engineering-agent",
  "source_section_id": "roadmap",
  "started_at": "2026-02-18T10:00:00Z",
  "output_url": "https://github.com/org/repo/pull/123",
  "output_type": "pr",
  "created_at": "2026-02-17T14:00:00Z",
  "updated_at": "2026-02-19T09:30:00Z",
  "version": 3
}
```

---

## 3. Task References

Content within sections MAY reference task sections using the following syntax:

```
{{task:<section_id>}}
```

### 3.1 Syntax

- `<section_id>` MUST match a section with `type: "task"` in the same artifact
- References are case-sensitive
- References MAY appear anywhere in markdown content

### 3.2 Example

```markdown
## Tier 2 Features

The following features are prioritized for Q2:

### RSVP Tracking
{{task:task-rsvp-tracking}}

### Real-time Collaboration
{{task:task-realtime-collab}}
```

### 3.3 Rendering Guidance

Implementations SHOULD render task references in a way that shows:
- Task heading
- Current status
- A way to navigate to the full task

The specific visual treatment is implementation-defined.

---

## 4. Section Approval Workflow

Sections MAY require human approval before being considered final. This enables human-in-the-loop review of agent-generated content.

### 4.1 Approval Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `status` | string | No | Section status (see 4.2) |
| `approval_requested_at` | ISO 8601 | No | When approval was requested |
| `approval_requested_by` | string | No | Agent that requested approval |
| `approval_note` | string | No | Message explaining what needs review |
| `decided_at` | ISO 8601 | No | When decision was made |
| `decided_by` | string | No | Who made the decision (agent_id or "human") |
| `decision_note` | string | No | Feedback or rationale |

### 4.2 Section Status Values

| Status | Description |
|--------|-------------|
| `draft` | Initial state, work in progress |
| `needs_approval` | Agent requests human review |
| `approved` | Human approved the content |
| `needs_changes` | Human requested revisions |

### 4.3 Workflow

1. Agent creates section with `status: "draft"`
2. Agent sets `status: "needs_approval"` with `approval_note`
3. Human reviews and sets either:
   - `status: "approved"` with `decision_note`
   - `status: "needs_changes"` with `decision_note`
4. If changes needed, agent revises and re-requests approval

### 4.4 Example

```json
{
  "id": "pricing-strategy",
  "type": "spec",
  "heading": "Pricing Strategy",
  "content": "## Proposed Pricing Tiers\n...",
  "agent_id": "product-manager",
  "status": "needs_approval",
  "approval_requested_at": "2026-02-19T10:00:00Z",
  "approval_requested_by": "product-manager",
  "approval_note": "Please review pricing tiers before we implement",
  "created_at": "2026-02-19T09:00:00Z",
  "updated_at": "2026-02-19T10:00:00Z"
}
```

After approval:

```json
{
  "id": "pricing-strategy",
  "status": "approved",
  "decided_at": "2026-02-19T11:30:00Z",
  "decided_by": "human",
  "decision_note": "Approved. Consider adding enterprise tier later."
}
```

---

## 5. Updated Section Schema

Complete section object with all v0.3 fields:

```json
{
  "id": "string (required)",
  "heading": "string (required)",
  "content": "string (required)",
  "agent_id": "string (required)",
  "agent_name": "string (optional)",
  "position": "integer (optional)",
  "version": "integer (optional)",
  "created_at": "ISO 8601 (required)",
  "updated_at": "ISO 8601 (required)",
  
  "type": "string (optional, v0.3)",
  
  "task_status": "string (optional, v0.3, task only)",
  "priority": "integer (optional, v0.3, task only)",
  "assignee_agent_id": "string (optional, v0.3, task only)",
  "source_section_id": "string (optional, v0.3, task only)",
  "started_at": "ISO 8601 (optional, v0.3, task only)",
  "completed_at": "ISO 8601 (optional, v0.3, task only)",
  "output_url": "string (optional, v0.3, task only)",
  "output_type": "string (optional, v0.3, task only)",
  
  "status": "string (optional, v0.3, approval)",
  "approval_requested_at": "ISO 8601 (optional, v0.3, approval)",
  "approval_requested_by": "string (optional, v0.3, approval)",
  "approval_note": "string (optional, v0.3, approval)",
  "decided_at": "ISO 8601 (optional, v0.3, approval)",
  "decided_by": "string (optional, v0.3, approval)",
  "decision_note": "string (optional, v0.3, approval)"
}
```

---

## 6. Backward Compatibility

### v0.2 → v0.3

- All v0.2 artifacts remain valid
- `section.type` is optional; untyped sections behave as before
- Task fields only apply when `type: "task"`
- Approval fields are optional on all sections
- `{{task:id}}` references are plain text in non-aware implementations

Detection logic:
```python
if section.get("type") == "task":
    # Task section with extended fields
elif section.get("status") in ["needs_approval", "approved", "needs_changes"]:
    # Section with approval workflow
else:
    # Standard section (v0.2 compatible)
```

---

## 7. Changelog Entry

### v0.3.0 (2026-02-19)

- Added `section.type` field for semantic categorization
- Added standard section types: overview, research, spec, roadmap, task, decision
- Added task section fields: task_status, priority, assignee_agent_id, source_section_id, started_at, completed_at, output_url, output_type
- Added task reference syntax: `{{task:<section_id>}}`
- Added section approval workflow: status, approval_requested_at, approval_requested_by, approval_note, decided_at, decided_by, decision_note

---

## Review Questions

1. Should `priority` be an integer (1/2/3) or string ("high"/"medium"/"low")?
2. Should task references support cross-artifact linking? `{{task:artifact_id:section_id}}`
3. Should we add `event_id` for external system linkage, or leave that to extensions?
4. Should approval workflow be a separate sub-object or flat fields?
