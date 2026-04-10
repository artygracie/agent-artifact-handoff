# Agent Artifact Handoff (AAH)

**A standard format for artifacts produced by AI agents.**

[![Version](https://img.shields.io/badge/version-0.5.0-blue)](./SPEC.md)
[![License](https://img.shields.io/badge/license-MIT-green)](./LICENSE)

---

## The Problem

AI agents produce artifacts constantly: research documents, code, analysis, specs, plans. Today these artifacts end up:

- **Dumped in codebases** — polluting repos with generated `.md` and `.json` files
- **Trapped on local machines** — inaccessible when you switch devices
- **Unconnected** — no way to trace which artifact led to which decision
- **Unmanaged** — no lifecycle, no cleanup, everything permanent by default
- **Non-standard** — every framework invents its own format

## The Solution

AAH defines a minimal, extensible envelope format for agent artifacts that enables:

- **Interoperability** between agent frameworks
- **Provenance tracking** (who created what, when, as part of what task)
- **Lineage** (which artifacts derive from which)
- **Lifecycle management** (retention, visibility, status)
- **Clean handoffs** between agents in multi-agent systems

## Quick Example

Artifacts are containers for sections. Each section has its own content type and owning agent:

```json
{
  "aah_version": "0.5",
  "artifact": {
    "id": "aah_spec_001",
    "type": "spec",
    "title": "RSVP Feature Spec"
  },
  "sections": [
    {
      "id": "overview",
      "heading": "Overview",
      "type": "document/markdown",
      "body": "## RSVP Tracking\n\nAllow hosts to track guest RSVPs...",
      "agent_id": "product-manager",
      "created_at": "2026-03-01T10:00:00Z",
      "updated_at": "2026-03-01T10:00:00Z"
    },
    {
      "id": "competitor-data",
      "heading": "Competitor Comparison",
      "type": "data/csv",
      "body": "Feature,Us,OpenTable,Resy\nRSVP,Planned,Yes,No",
      "agent_id": "research-agent",
      "created_at": "2026-03-01T12:00:00Z",
      "updated_at": "2026-03-01T12:00:00Z"
    },
    {
      "id": "mockup",
      "heading": "UI Mockup",
      "type": "image/png",
      "body_url": "https://artyfacts.dev/assets/mockup.png",
      "agent_id": "design-agent",
      "created_at": "2026-03-02T09:00:00Z",
      "updated_at": "2026-03-02T09:00:00Z"
    }
  ],
  "lifecycle": {
    "status": "active",
    "tags": ["feature", "rsvp"]
  }
}
```

A single artifact can contain markdown, code, images, data, and more — each contributed by different agents.

## Specification

**[Read the full specification →](./SPEC.md)**

Key sections:
- [Artifact Envelope](./SPEC.md#1-artifact-envelope)
- [Core Fields](./SPEC.md#2-core-fields)
- [Artifact Types](./SPEC.md#3-artifact-types)
- [Working with Sections](./SPEC.md#4-working-with-sections)
- [Transport](./SPEC.md#6-transport)

## Examples

See the [`examples/`](./examples) directory for complete AAH envelopes:

- [Research document](./examples/document.json) — Single-section markdown artifact
- [Code artifact](./examples/code.json) — Single-section TypeScript component
- [Artifact with lineage](./examples/with-lineage.json) — Artifact with parent linkage and handoff
- [Ephemeral scratch data](./examples/ephemeral.json) — Temporary data with short retention
- [Multi-section experiment](./examples/sectioned-experiment.json) — Experiment with daily data sections
- [Implementation plan](./examples/sectioned-implementation.json) — Multi-phase plan from multiple agents
- [Section update](./examples/section-update.json) — Partial update to a single section

## JSON Schema

Validate your artifacts against the schema:

- [`schemas/section.schema.json`](./schemas/section.schema.json)

## Implementations

| Name | Type | Status |
|------|------|--------|
| [Artyfacts](https://artyfacts.dev) | The GitHub for agent work that's not code | Reference |
| [`@artyfacts/mcp-server`](https://github.com/artygracie/artyfacts/tree/main/packages/mcp-server) | MCP Server | Active |

## Framework Integration

AAH is designed to work with any agent framework:

| Framework | Integration Status |
|-----------|-------------------|
| [Clawdbot](https://github.com/clawdbot/clawdbot) | Native |
| LangChain | Planned |
| CrewAI | Planned |
| AutoGen | Planned |

## Contributing

We welcome contributions! See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines.

## Versioning

AAH follows semantic versioning:
- **0.x.x** — Draft/experimental (breaking changes allowed)
- **1.0.0** — Stable (backwards-compatible changes only)

Current version: **0.5.0**

See [CHANGELOG.md](./CHANGELOG.md) for version history.

## License

MIT License — see [LICENSE](./LICENSE)

---

**Created by [Artygroup](https://artygroup.co)**

*Have questions? Open an issue or reach out on [Discord](https://discord.gg/artygroup).*
