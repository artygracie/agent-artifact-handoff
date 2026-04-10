# Proposal: Sectioned Artifacts

**Status:** Accepted (incorporated into AAH v0.2, evolved through v0.5)  
**Author:** Crouton (Clawdbot)  
**Date:** 2026-02-18  
**Target Version:** AAH 0.2

> **Historical Note:** This proposal was accepted and incorporated into AAH v0.2. The concepts have since evolved significantly through v0.5, where sections became the primary content model for all artifacts (not just a special `document/sectioned` type). The `document/sectioned` type was removed, `content` was renamed to `body`, sections gained their own content type system (`document/markdown`, `code/python`, `data/csv`, etc.), and the `initiative` field was removed. See the [current specification](../SPEC.md) for the latest design.

---

## Summary

Extend AAH to support **sectioned artifacts** — single artifacts containing multiple named sections, each owned by a different agent. This enables collaborative documents where multiple agents contribute without creating separate artifact files.

## Motivation

### Problem

In multi-agent systems, related work often produces many disconnected artifacts:

| Current Behavior | Result |
|-----------------|--------|
| Experiments Manager creates measurement plan | Artifact #1 |
| Analytics Manager creates baseline data | Artifact #2 |
| Engineering Manager creates rollback plan | Artifact #3 |
| Experiments Manager creates daily log | Artifact #4 |

Users must navigate 4+ documents to understand one initiative.

### Solution

Allow multiple agents to contribute **sections** to a single artifact:

```
📄 Paywall Experiment (1 artifact)
├── Overview (experiments-manager)
├── Baseline Data (analytics-manager)
├── Rollback Plan (engineering-manager)
└── Daily Log (experiments-manager)
```

## Specification Changes

### 1. New Artifact Type

Add `document/sectioned` to supported types:

```json
{
  "artifact": {
    "type": "document/sectioned"
  }
}
```

### 2. Sections Array

Sectioned artifacts replace `content.body` with a `sections` array:

```json
{
  "aah_version": "0.2",
  "artifact": {
    "id": "aah_abc123",
    "type": "document/sectioned",
    "title": "Paywall Experiment (PS-EXP-001)"
  },
  "sections": [
    {
      "id": "overview",
      "heading": "Overview",
      "content": "**Experiment ID:** PS-EXP-001\n...",
      "agent_id": "experiments-manager",
      "agent_name": "Experiments Manager",
      "created_at": "2026-02-16T10:00:00Z",
      "updated_at": "2026-02-17T14:30:00Z",
      "version": 2
    },
    {
      "id": "baseline",
      "heading": "Baseline Data",
      "content": "| Metric | Value |...",
      "agent_id": "analytics-manager",
      "agent_name": "Analytics Manager",
      "created_at": "2026-02-17T06:00:00Z",
      "updated_at": "2026-02-17T06:00:00Z",
      "version": 1
    }
  ],
  "lifecycle": { ... }
}
```

### 3. Section Schema

Each section has:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Stable identifier (e.g., "baseline", "day-1") |
| `heading` | string | Yes | Display title |
| `content` | string | Yes | Markdown content |
| `agent_id` | string | Yes | Agent that owns this section |
| `agent_name` | string | No | Human-readable agent name |
| `position` | integer | No | Display order (default: insertion order) |
| `created_at` | ISO 8601 | Yes | When section was created |
| `updated_at` | ISO 8601 | Yes | When section was last modified |
| `version` | integer | No | Section version number |

### 4. Section Update Envelope

To update a single section without uploading the entire artifact:

```json
{
  "aah_version": "0.2",
  "artifact": {
    "id": "aah_abc123"
  },
  "section_update": {
    "id": "baseline",
    "heading": "Baseline Data",
    "content": "Updated content...",
    "change_note": "Added conversion rate metric"
  },
  "source": {
    "agent_id": "analytics-manager"
  }
}
```

The `section_update` field indicates this is a partial update, not a full artifact upload.

### 5. Initiative Linking

Sectioned artifacts SHOULD include an `initiative` field for grouping:

```json
{
  "artifact": {
    "id": "aah_abc123",
    "type": "document/sectioned",
    "title": "Paywall Experiment",
    "initiative": "free-trial-removal"
  }
}
```

This enables:
- Finding existing artifacts by initiative
- Grouping related artifacts in UIs
- SDK helper: `client.initiative("free-trial-removal")`

## Backward Compatibility

- Simple artifacts (with `content.body`) remain valid
- Servers MUST support both simple and sectioned artifacts
- Clients MAY support only simple artifacts (graceful degradation)

Detection logic:
```python
if "sections" in envelope:
    # Sectioned artifact
else:
    # Simple artifact (use content.body)
```

## SDK Interface

### Creating a Sectioned Artifact

```typescript
const artifact = await client.createSectioned({
  title: "Paywall Experiment (PS-EXP-001)",
  initiative: "free-trial-removal",
  sections: [
    {
      id: "overview",
      heading: "Overview",
      content: "...",
    }
  ]
});
```

### Finding or Creating by Initiative

```typescript
// Returns existing artifact or creates new one
const artifact = await client.initiative("free-trial-removal", {
  title: "Paywall Experiment (PS-EXP-001)",
});
```

### Adding/Updating a Section

```typescript
// Upsert: creates if doesn't exist, updates if it does
await artifact.section({
  id: "baseline",
  heading: "Baseline Data",
  content: "...",
});
```

### Getting Section History

```typescript
const history = await artifact.sectionHistory("baseline");
// Returns array of previous versions
```

## Database Schema

### `sections` table

```sql
CREATE TABLE sections (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  artifact_id UUID REFERENCES artifacts(id) ON DELETE CASCADE,
  section_id TEXT NOT NULL,  -- stable ID like "baseline"
  heading TEXT NOT NULL,
  content TEXT NOT NULL,
  agent_id TEXT NOT NULL,
  agent_name TEXT,
  position INTEGER DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  version INTEGER DEFAULT 1,
  UNIQUE(artifact_id, section_id)
);
```

### `section_versions` table

```sql
CREATE TABLE section_versions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  section_id UUID REFERENCES sections(id) ON DELETE CASCADE,
  version INTEGER NOT NULL,
  content TEXT NOT NULL,
  agent_id TEXT NOT NULL,
  change_note TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/artifacts` | Create artifact (simple or sectioned) |
| POST | `/artifacts/:id/sections` | Add/update section |
| GET | `/artifacts/:id/sections` | List all sections |
| GET | `/artifacts/:id/sections/:sectionId` | Get single section |
| GET | `/artifacts/:id/sections/:sectionId/history` | Section version history |
| DELETE | `/artifacts/:id/sections/:sectionId` | Remove section |

## Migration Path

1. **Add schema** — New tables, no breaking changes
2. **Update API** — Add section endpoints
3. **Update SDK** — Add section methods
4. **Update skills** — Agents use `client.initiative()` pattern
5. **UI update** — Render sectioned artifacts with attribution

## Open Questions

1. **Section ordering** — Use `position` field or alphabetical by ID?
2. **Section deletion** — Soft delete (mark removed) or hard delete?
3. **Concurrent edits** — What if two agents update the same section simultaneously?
4. **Section types** — Should sections have types (markdown, table, checklist)?

## References

- [Current AAH Spec (v0.1)](../SPEC.md)
- [Sectioned Artifacts Design Doc](../../artyfacts/docs/SECTIONS-SPEC.md)
- [UI Mockup: Experiment](https://artyfacts.dev/sectioned-artifact.html)
- [UI Mockup: Feature](https://artyfacts.dev/sectioned-feature.html)
