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

See [`chatto-message-format.md`](chatto-message-format.md) for the message formats and human response protocol used by the Chatto connector.

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

AgentRoom should not require one specific coding agent. The first adapter should be a generic local event adapter so the merge-confidence loop can be tested before depending on a vendor-specific agent API.

See [`agent-adapter-contract.md`](agent-adapter-contract.md) for the event envelope, MVP event types, readiness mapping, and fixture stream.

Initial adapter events:

- `session.started`
- `plan.proposed`
- `code.started`
- `commit.created`
- `agent.blocked`
- `evidence.note`
- `session.completed`

Chatto approval events and GitHub PR/check/review events are AgentRoom events, but they are not produced by coding-agent adapters.

### GitHub connector events

The GitHub connector owns PR, CI, review, mergeability, and evidence publication facts.

Initial GitHub-derived events:

- `pr.opened`
- `pr.synchronized`
- `ci.completed`
- `review.completed`
- `mergeability.changed`
- `evidence.posted`

### Chatto connector events

The Chatto connector owns human decisions made in Chatto. It must translate explicit Chatto decision commands into internal AgentRoom events in the same event log. Reactions may be recorded as hints, but they are not authoritative approval signals in the MVP.

Initial Chatto-derived events:

- `plan.approved`
- `plan.rejected`
- `approval.requested`
- `approval.granted`
- `approval.denied`

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

See [`github-app-design.md`](github-app-design.md) for the GitHub App permissions, sticky PR comment, readiness check, branch protection behavior, and fork PR handling.
See [`evidence-packet-format.md`](evidence-packet-format.md) for the exact sticky PR comment and check run content.

### Runtime

The MVP runtime should be a single long-running `agentroom server` process with an HTTP webhook receiver, local agent-event ingestion, Chatto connector worker, readiness evaluator, evidence publisher, and SQLite event store.

The first implementation language should be Go. TypeScript/Playwright can be used for the Chatto-backed E2E harness, but the production runtime should ship as a Go binary.

See [`runtime-architecture.md`](runtime-architecture.md) for process boundaries, state sharing, deployment mode, and non-goals.
See [`state-model.md`](state-model.md) for append-only event history, latest-value runtime state, and the evaluator read path.
See [`readiness-evaluator.md`](readiness-evaluator.md) for deterministic readiness decisions.
See [`configuration.md`](configuration.md) for the TOML config schema used by `agentroom init` and `agentroom server`.
See [`test-strategy.md`](test-strategy.md) for the fixture, fake-client, renderer, parser, and evaluator test plan.

## MVP boundary

The MVP should avoid custom CI, custom merge systems, dashboards, and multi-agent orchestration. Chatto should provide the collaboration surface, and GitHub should remain the source of truth for pull requests, checks, reviews, and merge.

## Open questions

- Which Chatto API surface should the first connector use?
- Which agent-specific adapter should follow the generic local event adapter?
- Which policies should be configurable in version one?
