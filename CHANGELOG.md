# Changelog

All notable changes to the AAH specification will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.5.0] - 2026-04-10

### Changed
- **Breaking**: Artifacts are now containers for sections; artifact-level `content` field removed
- **Breaking**: `sections` is required on all artifacts (single-section for simple content)
- **Breaking**: Section `content` field renamed to `body`; sections gain `body_url`, `media_type`, `encoding`, `body_hash`, `size_bytes`, `token_count`
- Artifact `type` is now optional; simplified to purpose-based categories: `document`, `spec`, `research`, `analysis`, `plan`, `design`, `proposal`, `review`, `decision`, `report`, `experiment`, `feature`, `guide`, `log`
- Section `type` now uses content type taxonomy: `document/*`, `code/*`, `data/*`, `image/*`, `structured/*`
- `source` is now optional on the envelope; section `agent_id` is the primary provenance mechanism
- `section_update` gains `agent_id` field; `body` is now conditional (either `body` or `body_url`)

### Removed
- `artifact.initiative` field (use `lifecycle.tags` for grouping)
- `document/sectioned` artifact type (all artifacts contain sections)
- Simple vs. sectioned artifact distinction

## [0.4.0] - 2026-04-10

### Removed
- Task sections and all task-specific fields (`task_status`, `priority`, `assignee_agent_id`, etc.)
- Section approval workflow
- Task reference syntax (`{{task:id}}`)

### Changed
- Retained `section.type` for semantic categorization (without `task` type)

## [0.3.0] - 2026-02-19

### Added
- `section.type` field for semantic categorization
- Task sections, approval workflow, task references (removed in v0.4)

## [0.2.0] - 2026-02-18

### Added
- **Sectioned Artifacts** — Multi-agent collaborative documents
  - New artifact type: `document/sectioned`
  - New `sections` array for multiple named sections
  - Each section has its own `agent_id`, `version`, timestamps
  - `section_update` envelope for partial updates
  - Section-level version history
- **Initiative Grouping**
  - New `artifact.initiative` field for grouping related artifacts
- **New Artifact Types**
  - `structured/experiment` — Experiment tracking documents
  - `structured/feature` — Feature development tracking
- **JSON Schema**
  - `schemas/section.schema.json` — Section validation

### Changed
- Spec version bumped to `0.2.0`
- `lifecycle.status` values expanded: added `active`

## [0.1.0-draft] - 2026-02-11

### Added
- Initial draft specification
- Core envelope structure (`aah_version`, `artifact`, `source`, `content`)
- Lifecycle management (`retention`, `visibility`, `status`, `tags`)
- Handoff support (`target_agent`, `target_role`, `parent_artifact_ids`)
- Extensions mechanism for framework-specific data
- Artifact type taxonomy (`document/*`, `code/*`, `data/*`, `image/*`, `structured/*`)
- Versioning fields (`version`, `version_id`, `previous_version_id`)
- Content deduplication support via `body_hash`
- **HITL Decisions Extension** (`proposals/HITL-DECISIONS.md`)

---

## Version History

| Version | Date | Status |
|---------|------|--------|
| 0.5.0 | 2026-04-10 | **Current** |
| 0.4.0 | 2026-04-10 | Superseded |
| 0.3.0 | 2026-02-19 | Superseded |
| 0.2.0 | 2026-02-18 | Superseded |
| 0.1.0-draft | 2026-02-11 | Superseded |
