# Getting Started with AgentDeck

AgentDeck is currently a docs-first open-source project. The goal of this guide is to help contributors and early users understand the workflow, pilot it manually, and prepare for the first executable MVP.

## The workflow

AgentDeck supervises coding-agent work through a simple loop:

```text
Issue or PR -> Agent session -> Plan gate -> Branch/PR tracking -> CI/review aggregation -> Human approval -> Evidence packet
```

The first product is not a general dashboard. It is a merge-confidence layer for agent-generated pull requests.

## Pilot AgentDeck manually

You can test the workflow today without any server code.

1. Pick a GitHub repository where you already use a coding agent.
2. Pick one issue that is small enough for one pull request.
3. Create an agent plan using [`templates/agent-plan.md`](templates/agent-plan.md).
4. Require a human to approve the plan before the agent changes code.
5. Let the agent work on a dedicated branch.
6. Open a pull request.
7. Fill out [`templates/pr-evidence-packet.md`](templates/pr-evidence-packet.md).
8. Review the evidence packet before merging.

## What a good pilot should prove

A good pilot should answer these questions:

- Did the agent write a clear plan before code?
- Did the agent stay inside the approved scope?
- Can a reviewer see what changed without reading raw chat logs?
- Are CI failures, review comments, and human approvals captured in one place?
- Would this have prevented a risky or confusing merge?

## Suggested first pilot

Use AgentDeck on a low-risk documentation or test-only change first.

Example:

```text
Issue: Add missing setup instructions to README
Agent session: ad-session-001
Plan gate: approved by maintainer
Branch: ad/docs-setup-instructions
PR evidence packet: posted as the first PR comment
Merge rule: no merge until evidence packet says plan, tests, and approval are complete
```

## Contributor setup

Clone the repository:

```sh
git clone https://github.com/ktundwal/agentdeck.git
cd agentdeck
```

There is no application runtime yet. Start by reading:

```sh
open docs/product-brief.md
open docs/architecture.md
open docs/security.md
```

## First implementation target

The first executable milestone should create a GitHub-native evidence packet for one PR.

Minimum behavior:

1. Record an agent session for an issue or PR.
2. Store the approved plan.
3. Track the branch and pull request.
4. Read CI status from GitHub.
5. Record human approval.
6. Generate a PR evidence packet.

## Design principles

- GitHub remains the source of truth for code review and merge.
- AgentDeck owns agent-session state, policy gates, and evidence.
- Humans approve plans and risky actions.
- Agents are replaceable workers behind adapters.
- The audit trail must be durable, append-only, and reviewable.
