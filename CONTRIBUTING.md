# Contributing to Myrgic Labs

Thanks for your interest in contributing.

## Where to start

Each repo has its own setup and contribution guidelines:

- **[cogos](https://github.com/myrgic/cogos/blob/main/CONTRIBUTING.md)**: kernel daemon, context engine, MCP server
- **[mod3](https://github.com/myrgic/mod3)**: TTS, audio queue, turn-taking
- **[cog-sandbox-mcp](https://github.com/myrgic/cog-sandbox-mcp)**: Python MCP bridge over kernel session/handoff routes
- **[sites](https://github.com/myrgic/sites)**: monorepo where the kernel reconciles myrgic's own domain deployments
- **[research](https://github.com/myrgic/research)**: design rationales and theoretical foundations
- **[plugins](https://github.com/myrgic/plugins)**: SKILL.md plugin format
- **[charts](https://github.com/myrgic/charts)**: Helm and Docker Compose

See the org README's [Where to start table](https://github.com/myrgic/.github/blob/main/profile/README.md#where-to-start) for the canonical, up-to-date version of this list.

## General guidelines

- Open an issue before starting large changes
- Follow existing code style (Go: gofmt, Python: ruff)
- Include tests for new functionality
- Update CHANGELOG.md in your PR

## Code of Conduct

All participants are expected to follow our [Code of Conduct](CODE_OF_CONDUCT.md).
