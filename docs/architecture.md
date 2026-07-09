# Architecture

AgentDeck should be designed as a thin, trustworthy control layer between coding agents and GitHub.

## Initial data flow

```text
GitHub issue or PR
  -> AgentDeck session
  -> Agent adapter
  -> Branch or worktree
  -> Event log
  -> Policy gate
  -> PR evidence packet
  -> Human approval
  -> Merge gate
```

## Core components

### GitHub App

The GitHub App observes issues, pull requests, checks, reviews, labels, and comments. It should post evidence packets and expose status checks that can be used as branch protection rules.

### Session registry

The session registry tracks each agent run:

- session ID
- source issue or PR
- agent adapter
- branch
- current phase
- approved plan
- linked pull request
- current policy state

### Agent adapters

AgentDeck should not require one specific coding agent. Adapters should normalize events from different agents into the same session model.

Initial adapter events:

- `session.started`
- `plan.proposed`
- `plan.approved`
- `plan.rejected`
- `code.started`
- `commit.created`
- `pr.opened`
- `ci.completed`
- `review.completed`
- `approval.requested`
- `approval.granted`
- `approval.denied`
- `evidence.posted`

### Event log

The event log should be append-only. It is the source of truth for the evidence packet and audit trail.

### Policy engine

The policy engine decides whether a session can advance.

Initial policies:

- No code work before plan approval.
- No merge recommendation while CI is failing.
- No merge recommendation when required review is missing.
- No merge recommendation when risk approval is pending.

### Evidence packet generator

The evidence packet summarizes the session for reviewers. It should be posted to the PR and updated as the session changes.

## MVP boundary

The MVP should avoid custom chat, custom CI, and custom merge systems. GitHub should remain the source of truth for pull requests, checks, reviews, and merge.

## Open questions

- Which coding agent adapter should be built first?
- Should the first runtime be a single binary, Docker Compose, or GitHub Action?
- Should the event log begin as SQLite before moving to a server-backed store?
- Which policies should be configurable in version one?
