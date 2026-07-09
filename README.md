# AgentRoom

**AgentRoom (`ar`) is the Chatto-native merge desk for agent-authored pull requests.**

AI coding agents can now create branches and pull requests faster than humans can supervise them. The bottleneck is no longer writing code. It is knowing which agent PR is ready, blocked, risky, or waiting on human judgment.

AgentRoom uses **Chatto** as the live human/agent collaboration surface and **GitHub** as the source of truth. Each repo gets a Chatto room. Each agent-authored PR gets a thread. AgentRoom posts intent, plans, branch links, CI state, blockers, evidence summaries, and approval requests through a dedicated Chatto connector account. Humans approve, reject, pause, or redirect from the room. AgentRoom records the decisions, applies readiness policy, tracks GitHub state, and posts evidence packets back to PRs.

The technical bet is simple: combine Chatto's self-hosted collaboration UX and NATS-backed realtime foundation with GitHub's code-review and branch-protection model.

## Chatto integration status

Chatto is pre-1.0 and does not currently appear to expose a dedicated bot/webhook integration API. AgentRoom should treat Chatto integration as an adapter boundary, not a hard-coded internal dependency.

Initial dependency model:

1. Bring your own Chatto instance.
2. Configure AgentRoom with a Chatto URL and a dedicated Chatto user credential.
3. Post messages through Chatto's public ConnectRPC `MessageService`.
4. Detect human decisions through Chatto's realtime WebSocket where possible, with timeline polling as a fallback.
5. Keep GitHub as the source of truth for PRs, checks, reviews, and merge.

See [`docs/chatto-integration.md`](docs/chatto-integration.md) for the current integration assumptions and risks.

> **Tagline:** Every repo gets a room. Every agent PR gets a decision.

## Status

AgentRoom is in early open-source planning. This repository currently contains the product brief, architecture direction, safety model, and pilot workflow templates. The first executable MVP will focus on one GitHub repo connected to one Chatto room, with every agent-authored PR classified as `needs review`, `blocked`, `risky`, or `ready`.

## Who this is for

- GitHub maintainers running multiple coding-agent tasks per week
- Tech leads and staff engineers reviewing agent-generated PRs
- Platform and DevEx teams building internal agent workflows
- Technical founders who want more agent throughput without giving up control

## What AgentRoom is

- A Chatto-native room for each GitHub repo
- A PR thread for each agent-authored pull request
- A merge-confidence state machine for agent work
- A human approval and policy checkpoint
- A PR evidence packet generator
- An append-only audit trail across agents, humans, CI, Chatto, and GitHub

## What AgentRoom is not

- Not a foundation model
- Not an IDE
- Not Jira
- Not a general team chat product
- Not an autonomous merge bot
- Not a replacement for GitHub pull requests

## MVP scope

The first version should prove one narrow outcome: **connect one GitHub repo to one Chatto room and make agent-authored PRs easier to decide on.**

Planned MVP components:

1. GitHub App
2. Experimental Chatto connector
3. Repo room binding
4. PR thread creation
5. Readiness state machine
6. Human approval prompts in Chatto
7. PR evidence packet
8. Append-only event log
9. Generic agent event webhook or one runner adapter

## Get started

Read the getting-started guide:

```sh
open docs/getting-started.md
```

If you want to pilot the workflow before the executable MVP exists, use the templates:

- [`docs/templates/agent-plan.md`](docs/templates/agent-plan.md)
- [`docs/templates/pr-evidence-packet.md`](docs/templates/pr-evidence-packet.md)

## Repository map

| Path | Purpose |
| --- | --- |
| [`docs/getting-started.md`](docs/getting-started.md) | Contributor and pilot workflow guide |
| [`docs/product-brief.md`](docs/product-brief.md) | Product pitch, target user, wedge, and MVP |
| [`docs/architecture.md`](docs/architecture.md) | Initial system architecture and data flow |
| [`docs/agent-adapter-contract.md`](docs/agent-adapter-contract.md) | Generic agent event contract and first adapter choice |
| [`docs/chatto-integration.md`](docs/chatto-integration.md) | Chatto dependency model, connector assumptions, and integration risks |
| [`docs/github-app-design.md`](docs/github-app-design.md) | GitHub App permissions, evidence packets, readiness checks, and fork behavior |
| [`docs/runtime-architecture.md`](docs/runtime-architecture.md) | MVP process model, SQLite state, connectors, workers, and deployment |
| [`docs/state-model.md`](docs/state-model.md) | SQLite event log, latest-value state, evaluator read path, and idempotency |
| [`docs/configuration.md`](docs/configuration.md) | TOML config schema, required fields, secret handling, and init behavior |
| [`docs/security.md`](docs/security.md) | Trust, safety, and approval model |
| [`docs/roadmap.md`](docs/roadmap.md) | Initial development milestones |
| [`docs/templates/`](docs/templates) | Manual pilot templates |

## License

MIT
