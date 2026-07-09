# AgentDeck

**AgentDeck (`ad`) is the open-source merge-confidence layer for coding agents.**

AI coding agents can now create branches and pull requests faster than humans can supervise them. The bottleneck is no longer writing code. It is knowing which agent did what, whether it followed the plan, whether CI and review passed, and whether a human approved the risky parts before merge.

AgentDeck gives GitHub teams a private control room for supervised agent work. Start from an issue or pull request, track the agent session, require a plan before code, watch the branch, aggregate CI and review status, request explicit human approvals, and publish one evidence packet back to the PR.

> **Tagline:** Run coding agents like an engineering team, not a pile of chats.

## Status

AgentDeck is in early open-source planning. This repository currently contains the product brief, architecture direction, safety model, and pilot workflow templates. The first executable MVP will focus on GitHub-native agent session tracking and PR evidence packets.

## Who this is for

- GitHub maintainers running multiple coding-agent tasks per week
- Tech leads and staff engineers reviewing agent-generated PRs
- Platform and DevEx teams building internal agent workflows
- Technical founders who want more agent throughput without giving up control

## What AgentDeck is

- A self-hosted, GitHub-native trust layer for coding agents
- A session timeline for agent work
- A plan gate before code changes
- A PR evidence packet generator
- A human approval and policy checkpoint
- An append-only audit trail across agents, humans, CI, and reviews

## What AgentDeck is not

- Not a foundation model
- Not an IDE
- Not Jira
- Not team chat
- Not an autonomous merge bot
- Not a replacement for GitHub pull requests

## MVP scope

The first version should prove one narrow outcome: **make one agent-generated PR reviewable and safe to merge.**

Planned MVP components:

1. GitHub App
2. Agent session registry
3. Required plan gate
4. Branch, PR, and CI tracking
5. Human approval check
6. PR evidence packet
7. Append-only event log
8. Basic adapters for existing coding agents

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
