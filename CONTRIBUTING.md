# Contributing to AAH

Thank you for your interest in contributing to the Agent Artifact Handoff specification!

## Ways to Contribute

### 1. Report Issues

Found something unclear or incorrect? [Open an issue](https://github.com/artygracie/agent-artifact-handoff/issues/new) describing:

- What you expected
- What you found
- Suggested improvement (if any)

### 2. Propose Changes

For spec changes, please:

1. **Open an issue first** to discuss the change
2. Reference use cases that motivate the change
3. Consider backwards compatibility implications
4. If approved, submit a PR with:
   - Updated SPEC.md
   - Updated JSON schema (if applicable)
   - New/updated examples
   - CHANGELOG entry

### 3. Add Examples

More examples help implementers. To add:

1. Create a new file in `examples/`
2. Ensure it validates against the schema
3. Add a brief description in the file or README

### 4. Build Implementations

Building an AAH implementation? Let us know! We'll add it to the implementations list.

## Pull Request Process

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-change`)
3. Make your changes
4. Run schema validation on examples:
   ```bash
   npx ajv validate -s schemas/v0.1/artifact.schema.json -d examples/*.json
   ```
5. Update CHANGELOG.md
6. Submit PR with clear description

## Spec Change Guidelines

### Backwards Compatibility

- **Minor versions (0.x)**: Breaking changes allowed with justification
- **Major versions (1.0+)**: No breaking changes; new fields must be optional

### Field Naming

- Use `snake_case` for all field names
- Prefer full words over abbreviations
- Be consistent with existing fields

### Documentation

- Every field needs a description
- Include at least one example
- Note any constraints (required, enum values, format)

## Questions?

- Open a [Discussion](https://github.com/artygracie/agent-artifact-handoff/discussions)
- Join our [Discord](https://discord.gg/artygroup)

## Code of Conduct

Be respectful and constructive. We're all here to make agent interoperability better.
