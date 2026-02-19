# Agent Artifact Handoff (AAH)

**A standard format for artifacts produced by AI agents.**

[![Version](https://img.shields.io/badge/version-0.2.0-blue)](./SPEC.md)
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

**Simple artifact:**
```json
{
  "aah_version": "0.2",
  "artifact": {
    "id": "aah_abc123",
    "type": "document/markdown",
    "title": "Competitor Analysis"
  },
  "source": {
    "agent_id": "research-agent",
    "framework": "crewai"
  },
  "content": {
    "media_type": "text/markdown",
    "body": "# Competitor Analysis\n\n..."
  },
  "lifecycle": {
    "retention": "30d",
    "visibility": "team"
  }
}
```

**Sectioned artifact (v0.2):**
```json
{
  "aah_version": "0.2",
  "artifact": {
    "id": "exp-001",
    "type": "document/sectioned",
    "title": "Paywall Experiment"
  },
  "source": { "agent_id": "experiments-manager" },
  "sections": [
    {
      "id": "overview",
      "heading": "Overview",
      "content": "14-day experiment...",
      "agent_id": "experiments-manager",
      "position": 0
    },
    {
      "id": "day-1",
      "heading": "Day 1",
      "content": "| Signups | 12 |",
      "agent_id": "analytics-manager",
      "position": 1
    }
  ]
}
```

## Specification

📖 **[Read the full specification →](./SPEC.md)**

Key sections:
- [Artifact Envelope](./SPEC.md#1-artifact-envelope)
- [Core Fields](./SPEC.md#2-core-fields)
- [Artifact Types](./SPEC.md#3-artifact-types)
- [Lineage & Handoffs](./SPEC.md#6-handoff)
- [Transport](./SPEC.md#5-transport)

## Examples

See the [`examples/`](./examples) directory for complete AAH envelopes:

**Basic artifacts (v0.1):**
- [Research document](./examples/document.json)
- [Code artifact](./examples/code.json)
- [Artifact with lineage](./examples/with-lineage.json)
- [Ephemeral scratch artifact](./examples/ephemeral.json)

**Sectioned artifacts (v0.2):**
- [Multi-day experiment](./examples/sectioned-experiment.json) — 14-day experiment with daily sections
- [Implementation plan](./examples/sectioned-implementation.json) — Phased implementation with multiple agents
- [Section update](./examples/section-update.json) — Partial update to add/modify a section

## JSON Schema

Validate your artifacts against the official schema:

```bash
# Using ajv-cli
npx ajv validate -s schemas/v0.1/artifact.schema.json -d your-artifact.json
```

## Implementations

| Name | Type | Language | Status |
|------|------|----------|--------|
| [Artyfacts](https://artyfacts.dev) | Storage + Viewer | TypeScript | Reference |
| `@artyfacts/sdk` | SDK | TypeScript | Planned |
| `artyfacts-python` | SDK | Python | Planned |

## Framework Integration

AAH is designed to work with any agent framework:

| Framework | Integration Status |
|-----------|-------------------|
| [Clawdbot](https://github.com/clawdbot/clawdbot) | ✅ Native |
| LangChain | Planned |
| CrewAI | Planned |
| AutoGen | Planned |

## Contributing

We welcome contributions! See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines.

**Ways to contribute:**
- 🐛 Report issues or inconsistencies in the spec
- 💡 Propose new artifact types or fields
- 📖 Improve documentation and examples
- 🔧 Build framework integrations

## Versioning

AAH follows semantic versioning:
- **0.x.x** — Draft/experimental (breaking changes allowed)
- **1.0.0** — Stable (backwards-compatible changes only)

Current version: **0.2.0**

See [CHANGELOG.md](./CHANGELOG.md) for version history.

## License

MIT License — see [LICENSE](./LICENSE)

---

**Created by [Artygroup](https://artygroup.co)**

*Have questions? Open an issue or reach out on [Discord](https://discord.gg/artygroup).*
