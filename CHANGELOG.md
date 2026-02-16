# Changelog

All notable changes to the AAH specification will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- JSON Schema for validation
- **HITL Decisions Extension** (`proposals/HITL-DECISIONS.md`)
  - New artifact type: `decision/request` — Structured decision requests from agents to humans
  - New artifact type: `decision/response` — Human responses with granular approvals
  - Decision types: `approval`, `choice`, `multi_choice`, `text`, `number`, `date`
  - Per-decision approve/reject with comments for audit trail
  - Blocking task tracking
  - Deadline and escalation support
  - JSON Schemas: `decision-request.schema.json`, `decision-response.schema.json`

### Changed
- Nothing yet

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

### Security
- Defined authentication/authorization recommendations
- Content validation guidelines
- Secrets handling policy

---

## Version History

| Version | Date | Status |
|---------|------|--------|
| 0.1.0-draft | 2026-02-11 | Current |
