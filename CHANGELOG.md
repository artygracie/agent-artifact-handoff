# Changelog

All notable changes to the AAH specification will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Nothing yet

### Changed
- Nothing yet

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
  - Enables `client.initiative()` SDK pattern
- **New Artifact Types**
  - `structured/experiment` — Experiment tracking documents
  - `structured/feature` — Feature development tracking
- **JSON Schema**
  - `schemas/section.schema.json` — Section validation

### Changed
- Spec version bumped to `0.2.0`
- `lifecycle.status` values expanded: added `active`, `needs_approval`
- Added backward compatibility section
- Added example artifacts appendix

### Proposals Accepted
- `proposals/SECTIONED-ARTIFACTS.md` — Full sectioned artifacts proposal

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
  - Decision types: `approval`, `choice`, `multi_choice`, `text`, `number`, `date`
  - Per-decision approve/reject with comments
  - Deadline and escalation support

### Security
- Defined authentication/authorization recommendations
- Content validation guidelines
- Secrets handling policy

---

## Version History

| Version | Date | Status |
|---------|------|--------|
| 0.2.0 | 2026-02-18 | **Current** |
| 0.1.0-draft | 2026-02-11 | Superseded |
