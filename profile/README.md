# Myrgic Labs

AI infrastructure built on substrate-mediated cognition. We treat memory, retrieval, and inference as substrate properties: externalized from the model, content-addressed, and shared across whatever AI tools touch the same workspace.

Brand and longer-form writing: [myrgic.com](https://myrgic.com/).

## Flagship: CogOS

Every modern AI tool has its own memory. Claude Code remembers. Cursor remembers. Your custom agent remembers. But none of them share with each other, none of them follow you across machines, and there's no shape for what's local versus what's shared.

CogOS is the layer underneath. Persistent workspaces any AI tool can plug into. Cross-tool, cross-device, cross-org. Hierarchical like git remotes, but recursive: keep what's local local, promote what matters upstream, sync what's yours across your machines.

## Where to start

| Who you are | Where to go |
|---|---|
| **Contributing to the kernel** | [cogos](https://github.com/myrgic/cogos): kernel daemon, context engine, MCP server |
| **Building voice or modality features** | [mod3](https://github.com/myrgic/mod3): TTS, audio queue, turn-taking |
| **Plugging multiple agents into one kernel** | [cog-sandbox-mcp](https://github.com/myrgic/cog-sandbox-mcp): Python MCP bridge over kernel session/handoff routes |
| **Seeing the kernel run real infrastructure** | [sites](https://github.com/myrgic/sites): monorepo where the kernel reconciles myrgic's own domain deployments |
| **Understanding the why behind the architecture** | [research](https://github.com/myrgic/research): design rationales and theoretical foundations |
| **Writing portable agent skills** | [plugins](https://github.com/myrgic/plugins): SKILL.md plugin format |
| **Deploying to Kubernetes** | [charts](https://github.com/myrgic/charts): Helm and Docker Compose |

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│  Your AI tools                                          │
│  Claude Code · Cursor · custom agents · Ollama · …      │
└────────────────────┬────────────────────────────────────┘
                     │ hooks · MCP · HTTP
                     ▼
┌─────────────────────────────────────────────────────────┐
│  CogOS kernel  (local Go daemon)                        │
│  Owns workspace state. Hosts subsystems. Exposes        │
│  protocol surfaces for whatever AI tool plugs in.       │
└────────────────────┬────────────────────────────────────┘
                     │ reads & writes
                     ▼
┌─────────────────────────────────────────────────────────┐
│  Workspace  (any directory)                             │
│                                                         │
│    your-project/                                        │
│    ├─ src/  docs/  …     ← your stuff, untouched        │
│    ├─ .git/               ← code history (optional)     │
│    └─ .cog/               ← cognitive overlay           │
│       ├─ mem/    cogdocs (memory)                       │
│       ├─ run/    bus events, traces                     │
│       └─ ledger/ hash-chained record                    │
└─────────────────────────────────────────────────────────┘
```

Three pieces:

- **Your AI tools** speak to the kernel through hooks, MCP, and HTTP. Anything that can call a local endpoint or run a hook can plug in.
- **The CogOS kernel** is one local Go daemon. It owns workspace state, hosts subsystems (context assembly, inference routing, reconcilers, event bus, ledger), and exposes the protocol surfaces.
- **The workspace** is any directory you point the kernel at. CogOS adds a `.cog/` overlay alongside whatever else is there. Your codebase. A notes folder. An empty project. Same shape regardless.

## .cog/ and .git/

The cognitive overlay (`.cog/`) and git's overlay (`.git/`) are siblings. Neither owns the other. The cognitive ledger can reference git blob hashes directly when it needs to point at code content, so there's no duplication: git keeps its objects, `.cog/` keeps its cogdocs, and content-addressing lets each refer to the other.

Initialize CogOS in an empty folder and you get a fresh workspace with no code history. Initialize it in an existing repo and the salience scorer reads git's history (recency, edit patterns, blast radius) as input, without taking ownership of any of git's data.

## Workspaces compose

A workspace is just a directory with a `.cog/` overlay. You can have many.

The composition is recursive, modeled on git remotes. Your workspace can have an upstream (your team's workspace, say) and selectively promote knowledge to it the way you push commits to origin. That upstream can have its own upstream. Local stays local until you explicitly promote it.

The result is a topology where personal context lives where it belongs, organizational knowledge is shared deliberately rather than absorbed by default, and nothing about your workflow leaves your machine without you choosing.

## The lab runs on its own kernel

This org's public sites are deployed through the same kernel it publishes. [**myrgic/sites**](https://github.com/myrgic/sites) is a monorepo declaring five domain deployments: myrgic.com (canonical) plus four redirects (.dev / .ai / .net / .org). Each is a per-app `site.yaml` spec consumed by the kernel's `SiteProvider`, a `Reconcilable` provider that diffs declared state against live state in the deploy-target repos, plans, and applies via `cogos reconcile site`. First strategy is `GHPagesStrategy`; the contract supports gitlab-pages, k8s ingress, S3, self-hosted rsync as future implementations.

If you want to see the framework doing real work, look there. Same plan/apply/state shape as Terraform or ArgoCD, executed through the kernel CLI rather than a separate operator.

## What the kernel does

The CogOS kernel (`cogos`) is one Go binary that:

- **Externalized attention.** Intercepts each prompt to a connected AI tool and assembles a focused context window before the model sees it. The substrate decides what's relevant, not the model.
- **Multi-provider inference.** Routes through local and cloud providers behind OpenAI-compatible and Anthropic Messages-compatible endpoints. Local-first, cloud when you want it.
- **Hash-chained ledger.** Persists every routing decision, context assembly, state transition, tool call, and config mutation as a content-addressed CogBlock. Append-only, optionally chain-verified.
- **Live event bus.** In-process broker with SSE streaming. Subsystems and external tools subscribe and see writes in real time.
- **MCP, OpenAI, and Anthropic protocol support.** Speaks the protocols your AI tools already use. Always-on MCP Streamable HTTP server with sessions and JSON-RPC 2.0.
- **Reconcilers.** Autonomic loops that maintain workspace invariants. Plan, apply, snapshot, rollback. Topological ordering.
- **Native agent harness.** Local-model assessment loop running inside the kernel process with kernel-native tools.

Full feature surface: see the [cogos repo](https://github.com/myrgic/cogos).

## Releases

Latest: **v0.13.0**. The kernel ships frequent minor releases; the [Releases page](https://github.com/myrgic/cogos/releases) is the canonical record (notes auto-generated from PR titles since the previous tag). Recent releases have added identity and RBAC-binding primitives (RoleBinding, WorkspaceBinding, NodeBinding, with an IdentityProvider wired into the reconcile registry), cross-node transport over a Syncthing-derived block-exchange protocol, extraction of the reconciler framework toward a standalone `pkg/substrate` library, and a decision-lineage projection surfaced through `cogos spine`. `main` runs ahead of the latest tag.


## Repositories

### Active

| Repo | Description | Language |
|------|-------------|----------|
| [**cogos**](https://github.com/myrgic/cogos) | The kernel. Workspace state, context assembly, multi-provider inference routing, hash-chained ledger, MCP server, agent harness, reconciler framework. | Go |
| [**mod3**](https://github.com/myrgic/mod3) | Voice channel. Multi-model TTS (Kokoro, Voxtral, Chatterbox, Spark) with queue-aware output, barge-in detection, turn-taking. MCP server. | Python |
| [**cog-sandbox-mcp**](https://github.com/myrgic/cog-sandbox-mcp) | MCP bridge that lets multiple agent sessions (Claude Code, Cursor, custom) share one kernel. HTTP streamable transport, 12 `cogos_*` tools layered over the kernel's session and handoff routes. | Python |
| [**constellation**](https://github.com/myrgic/constellation) | L1 trust-node protocol. Git-backed hash-chained event ledger, ECDSA P-256 node identity, signed heartbeats, EMA-weighted peer trust. The kernel's `ConstellationBridge` seam consumes this. | Go |
| [**plugins**](https://github.com/myrgic/plugins) | Portable skill definitions (SKILL.md) for Claude Code and compatible agents. | Markdown |
| [**research**](https://github.com/myrgic/research) | Design rationales and the theoretical foundations behind the kernel's architecture, plus the training pipeline. | Python |
| [**charts**](https://github.com/myrgic/charts) | Helm charts for deploying CogOS nodes to Kubernetes. | Helm |
| [**sites**](https://github.com/myrgic/sites) | Monorepo declaring myrgic's own domain deployments. Per-app `site.yaml` specs reconciled by the kernel's `SiteProvider`. The framework running the lab. | YAML / HTML |
| [**mlx-lm**](https://github.com/myrgic/mlx-lm) | Fork of ml-explore/mlx-lm. Adds `BlockedKVCache`, a block-level KV cache with a content-addressed hash chain, for inference-side experiments related to the cogos substrate. | Python |

*Auto-managed deploy targets (do not commit by hand; produced by the SiteProvider): `myrgic.github.io`, `myrgic.dev`, `myrgic.ai`, `myrgic.net`, `myrgic.org`.*

### Archived

| Repo | Reason |
|------|--------|
| desktop | Functionality absorbed by the kernel's embedded dashboard. |
| openclaw-plugin | OpenClaw context plugin, superseded by the kernel's native context engine. |

## Quick start

```sh
# Build the kernel
git clone https://github.com/myrgic/cogos.git
cd cogos && make build

# Initialize a workspace anywhere (empty folder or existing repo)
./cogos init --workspace ~/my-project

# Start the daemon
./cogos serve --workspace ~/my-project
# http://localhost:6931/health
```

Requirements: Go 1.25+, macOS or Linux.

For voice, add [mod3](https://github.com/myrgic/mod3) as an MCP server in your `.mcp.json`.

## CI

Shared reusable workflows in [`.github`](https://github.com/myrgic/.github):

| Workflow | What it does |
|----------|-------------|
| `go-ci.yml` | `go vet`, build, `go test -race`, golangci-lint |
| `py-ci.yml` | ruff lint + format, pytest |
| `pr-checks.yml` | CHANGELOG reminder, required files validation |

## Contributing

All repositories are MIT licensed. Issues and PRs welcome.
