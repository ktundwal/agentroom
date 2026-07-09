# Getting Started with AgentRoom

AgentRoom is currently a docs-first open-source project. The goal of this guide is to help contributors and early users understand the workflow, pilot it manually, and prepare for the first executable MVP.

## The workflow

AgentRoom supervises coding-agent work through a Chatto + GitHub loop:

```text
GitHub repo -> Chatto room -> agent PR thread -> readiness state -> human approval -> PR evidence packet
```

The first product is not a general dashboard. It is a Chatto-native merge desk for agent-authored pull requests.

## Pilot AgentRoom manually

You can test the workflow today without any server code.

1. Pick a GitHub repository where you already use a coding agent.
2. Create a Chatto room for that repository.
3. Pick one issue that is small enough for one pull request.
4. Create an agent plan using [`templates/agent-plan.md`](templates/agent-plan.md).
5. Post the plan in a Chatto thread and require a human approval before code changes.
6. Let the agent work on a dedicated branch.
7. Open a pull request.
8. Fill out [`templates/pr-evidence-packet.md`](templates/pr-evidence-packet.md).
9. Post the evidence packet to both the Chatto PR thread and the GitHub PR.
10. Review the evidence packet before merging.

## What a good pilot should prove

A good pilot should answer these questions:

- Did the agent write a clear plan before code?
- Did the agent stay inside the approved scope?
- Can a reviewer see what changed without reading raw chat logs?
- Are CI failures, review comments, Chatto decisions, and human approvals captured in one place?
- Would this have prevented a risky or confusing merge?

## Suggested first pilot

Use AgentRoom on a low-risk documentation or test-only change first.

Example:

```text
Issue: Add missing setup instructions to README
Agent session: ar-session-001
Chatto room: repo-docs
Plan gate: approved by maintainer
Branch: ar/docs-setup-instructions
PR evidence packet: posted as the first PR comment
Merge rule: no merge until evidence packet says plan, tests, and approval are complete
```

## Contributor setup

Clone the repository:

```sh
git clone https://github.com/ktundwal/agentroom.git
cd agentroom
```

There is no application runtime yet. Start by reading:

```sh
open docs/product-brief.md
open docs/architecture.md
open docs/chatto-integration.md
open docs/security.md
```

## Chatto dependency model

The MVP should assume Chatto is already running. AgentRoom should connect to Chatto rather than install or operate it.

Expected configuration:

- Chatto server URL
- dedicated Chatto user for AgentRoom
- room membership for the repos AgentRoom supervises
- credentials for public ConnectRPC calls
- realtime WebSocket access, with polling fallback if needed

A bundled `docker compose` setup that starts AgentRoom and Chatto together can come later, after the connector proves useful against an existing Chatto instance.

## First implementation target

The first executable milestone should connect one GitHub repo to one Chatto room and create a readiness thread plus GitHub evidence packet for one agent-authored PR.

Minimum behavior:

1. Bind one GitHub repo to one Chatto room.
2. Create or update a Chatto thread for an agent-authored PR.
3. Record the approved plan.
4. Track the branch and pull request.
5. Read CI status from GitHub.
6. Record human approval from Chatto.
7. Generate a PR evidence packet.

## Design principles

- GitHub remains the source of truth for code review and merge.
- Chatto is the human/agent collaboration surface.
- AgentRoom owns agent-session state, policy gates, and evidence.
- Humans approve plans and risky actions.
- Agents are replaceable workers behind adapters.
- The audit trail must be durable, append-only, and reviewable.
