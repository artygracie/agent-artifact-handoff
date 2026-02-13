# Agent Artifact Handoff (AAH)

**A standard format for artifacts produced by AI agents.**

[![Version](https://img.shields.io/badge/version-0.1.0--draft-blue)](./SPEC.md)
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

```json
{
  "aah_version": "0.1",
  "artifact": {
    "id": "aah_abc123",
    "type": "document/markdown",
    "title": "Competitor Analysis",
    "created_at": "2026-02-12T10:30:00Z"
  },
  "source": {
    "agent_id": "research-agent",
    "framework": "crewai",
    "session_id": "ses_xyz",
    "task_id": "TASK-456"
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

- [Research document](./examples/document.json)
- [Code artifact](./examples/code.json)
- [Artifact with lineage](./examples/with-lineage.json)
- [Ephemeral scratch artifact](./examples/ephemeral.json)

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

Current version: **0.1.0-draft**

See [CHANGELOG.md](./CHANGELOG.md) for version history.

## License

MIT License — see [LICENSE](./LICENSE)

---

**Created by [Artygroup](https://artygroup.co)**

*Have questions? Open an issue or reach out on [Discord](https://discord.gg/artygroup).*
