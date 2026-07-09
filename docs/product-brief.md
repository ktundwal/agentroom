# Product Brief

## One-liner

AgentRoom is the Chatto-native merge desk for agent-authored pull requests.

## Problem

AI coding agents can create branches and pull requests faster than humans can supervise them. The work is scattered across GitHub tabs, agent chat logs, Slack threads, chat rooms, CI pages, branch names, and human memory.

That breaks when a maintainer runs multiple agents at once. Reviewers lose track of what was planned, what changed, what failed, what was approved, and what is safe to merge.

## Target audience

The beachhead user is a GitHub maintainer or tech lead running multiple coding-agent tasks per week.

Secondary audiences:

- Platform and DevEx teams standardizing agent workflows
- Staff engineers supervising cross-repo agent work
- Technical founders using agents to ship faster
- OSS maintainers accepting agent-generated contributions

## Wedge

The first product should connect one GitHub repo to one Chatto room and make every agent-authored PR easier to decide on.

AgentRoom should not begin as a broad chat product, project-management system, or autonomous agent platform. It should prove that a Chatto room plus GitHub evidence makes agent PR review faster and safer.

## Activation moment

A maintainer opens one Chatto repo room and sees three agent PR threads. One is ready, one is blocked by CI, and one needs architecture approval. The maintainer can decide without opening ten tabs or reading raw agent logs.

## Core promise

AgentRoom answers four questions before merge:

1. What was the agent asked to do?
2. What plan did a human approve?
3. What changed, and did CI/review pass?
4. Who approved the risky parts?

## Differentiators

- Chatto-native collaboration with GitHub-grounded merge decisions
- Self-hosted and private by default
- Adapter-based for existing coding agents
- Evidence packet instead of raw logs
- Repo room and PR thread instead of scattered chat
- Human approval before risky actions

## Tagline

Every repo gets a room. Every agent PR gets a decision.
