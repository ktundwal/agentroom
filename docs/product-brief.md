# Product Brief

## One-liner

AgentDeck is the open-source merge-confidence layer for coding agents.

## Problem

AI coding agents can create branches and pull requests faster than humans can supervise them. The work is scattered across GitHub tabs, agent chat logs, Slack threads, CI pages, branch names, and human memory.

That breaks when a maintainer runs multiple agents at once. Reviewers lose track of what was planned, what changed, what failed, what was approved, and what is safe to merge.

## Target audience

The beachhead user is a GitHub maintainer or tech lead running multiple coding-agent tasks per week.

Secondary audiences:

- Platform and DevEx teams standardizing agent workflows
- Staff engineers supervising cross-repo agent work
- Technical founders using agents to ship faster
- OSS maintainers accepting agent-generated contributions

## Wedge

The first product should make one agent-generated PR reviewable and safe to merge.

AgentDeck should not begin as a broad chat product, project-management system, or autonomous agent platform. It should prove trust at the merge boundary.

## Activation moment

A maintainer starts three agent sessions from three issues. AgentDeck approves one plan, blocks one risky plan, tracks CI on the surviving PR, and posts a complete evidence packet so the maintainer can merge without spelunking through chats and tabs.

## Core promise

AgentDeck answers four questions before merge:

1. What was the agent asked to do?
2. What plan did a human approve?
3. What changed, and did CI/review pass?
4. Who approved the risky parts?

## Differentiators

- Self-hosted and private by default
- GitHub-native instead of chat-native
- Adapter-based for existing coding agents
- Evidence packet instead of raw logs
- Plan gate before code
- Human approval before risky actions

## Tagline

Run coding agents like an engineering team, not a pile of chats.
