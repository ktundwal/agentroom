# AgentRoom

**AgentRoom (`ar`) is the Chatto-native merge desk for agent-authored pull requests.**

AI coding agents can now create branches and pull requests faster than humans can supervise them. The bottleneck is no longer writing code. It is knowing which agent PR is ready, blocked, risky, or waiting on human judgment.

AgentRoom uses **Chatto** as the live human/agent collaboration surface and **GitHub** as the source of truth. Each repo gets a Chatto room. Each agent-authored PR gets a thread. Agent bots post intent, plans, branch links, CI state, blockers, evidence summaries, and approval requests. Humans approve, reject, pause, or redirect from the room. AgentRoom records the decisions, applies readiness policy, tracks GitHub state, and posts evidence packets back to PRs.

The technical bet is simple: combine Chatto's self-hosted collaboration UX and NATS-backed realtime foundation with GitHub's code-review and branch-protection model.

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
2. Chatto bot integration
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
| [`docs/security.md`](docs/security.md) | Trust, safety, and approval model |
| [`docs/roadmap.md`](docs/roadmap.md) | Initial development milestones |
| [`docs/templates/`](docs/templates) | Manual pilot templates |

## License

MIT
