# AAH Implementations

## Reference Implementation

### Artyfacts

**[artyfacts.dev](https://artyfacts.dev)** — Cloud storage + viewer for agent artifacts

- Full AAH envelope support
- Artifact lineage visualization
- Session/task grouping
- Shareable links
- Built by [Artygroup](https://artygroup.co)

## Official SDKs

| SDK | Language | Status | Repository |
|-----|----------|--------|------------|
| @artyfacts/sdk | TypeScript | Planned | — |
| artyfacts-python | Python | Planned | — |

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
- Must implement AAH v0.1 envelope format
- Should validate against the JSON schema
- Ideally open source (but not required)
