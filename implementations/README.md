# AAH Implementations

## Reference Implementation

### Artyfacts

**[artyfacts.dev](https://artyfacts.dev)** — The GitHub for agent work that's not code

- Full AAH envelope support
- Artifact lineage visualization
- Session/task grouping
- Shareable links
- MCP-native — agents interact via [`@artyfacts/mcp-server`](https://github.com/artygracie/artyfacts/tree/main/packages/mcp-server)
- Built by [Artygroup](https://artygroup.co)

## MCP Server

| Package | Status | Repository |
|---------|--------|------------|
| `@artyfacts/mcp-server` | Active | [GitHub](https://github.com/artygracie/artyfacts/tree/main/packages/mcp-server) |

## Framework Integrations

| Framework | Integration | Status |
|-----------|------------|--------|
| [Clawdbot](https://github.com/clawdbot/clawdbot) | Native | ✅ Available |
| LangChain | Callback handler | Planned |
| CrewAI | Output handler | Planned |
| AutoGen | Agent wrapper | Planned |

## Community Implementations

*None yet — [be the first](https://github.com/artygracie/agent-artifact-handoff/issues/new?title=New+Implementation:+...)!*

---

## Adding Your Implementation

Built something with AAH? We'd love to list it!

1. Open a PR adding your implementation to this file
2. Include:
   - Name and link
   - Brief description
   - Language/framework
   - Current status

Requirements:
- Must implement AAH v0.5 envelope format
- Should validate against the JSON schema
- Ideally open source (but not required)
