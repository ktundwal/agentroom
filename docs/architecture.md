# Architecture

AgentRoom should be designed as a thin, trustworthy merge desk between Chatto, coding agents, and GitHub.

## Initial data flow

```text
GitHub webhook or agent event
  -> AgentRoom event log
  -> Readiness state machine
  -> Chatto repo room / PR thread update
  -> Human action in Chatto
  -> Audit record
  -> GitHub PR evidence packet / check
```

## Core components

### Chatto connector

The Chatto connector creates or binds repo rooms and updates PR threads. It posts agent intent, branch links, PR links, CI state, blockers, approval prompts, and readiness summaries through a dedicated Chatto user account.

Chatto is the live collaboration surface. It is not the source of truth for code, checks, reviews, or merge.

Chatto does not currently appear to expose a dedicated bot/webhook API. The initial connector should use public user-facing APIs only:

- post messages with ConnectRPC `MessageService.CreateMessage`
- read message/reaction state with public ConnectRPC APIs
- subscribe to visible events through `/api/realtime` when possible
- fall back to polling room/thread timelines when realtime coverage is insufficient

Operator API access is optional bootstrap plumbing only. It is root-equivalent, Unix-socket-only, and should not be required for normal AgentRoom operation.

### GitHub App

The GitHub App observes issues, pull requests, checks, reviews, labels, and comments. It should post evidence packets and expose status checks that can be used as branch protection rules.

### Session registry and state machine

The session registry tracks each agent run and PR thread:

- session ID
- Chatto room ID
- Chatto thread ID
- source issue or PR
- agent adapter
- branch
- readiness state
- approved plan
- linked pull request
- current policy state

### Agent adapters

AgentRoom should not require one specific coding agent. Adapters should normalize events from different agents into the same session model.

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

The evidence packet summarizes the session for reviewers. It should be posted to the PR and updated as Chatto, GitHub, and agent events change.

## MVP boundary

The MVP should avoid custom CI, custom merge systems, dashboards, and multi-agent orchestration. Chatto should provide the collaboration surface, and GitHub should remain the source of truth for pull requests, checks, reviews, and merge.

## Open questions

- Which Chatto API surface should the first connector use?
- Which coding agent adapter should be built first?
- Should the first runtime be a single binary, Docker Compose, or GitHub Action?
- Should the event log begin as SQLite before moving to a server-backed store?
- Which policies should be configurable in version one?
