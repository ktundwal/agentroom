# Runtime Architecture

AgentRoom should start as a single long-running service with a small CLI. It is not a library-only project and not a GitHub Action in the MVP.

## MVP runtime choice

Use a **single binary / single container** runtime for Milestone 1.

Why:

- GitHub webhooks require a long-running HTTP receiver.
- Chatto approvals require a realtime listener or polling worker.
- Agent event ingestion requires either an HTTP endpoint or file watcher.
- Readiness evaluation needs shared durable state.
- A single process keeps the first deployment understandable.

Docker Compose can wrap the binary for local trials, but AgentRoom should not require Compose internally. GitHub Action mode can come later for stateless evidence regeneration, not for the first realtime loop.

## Process model

One `agentroom server` process runs these components:

```text
agentroom server
  ├─ HTTP server
  │  ├─ POST /webhooks/github
  │  ├─ POST /api/agent-events
  │  └─ GET /healthz
  ├─ local event file watcher
  ├─ Chatto connector worker
  ├─ readiness evaluator
  ├─ evidence publisher
  └─ SQLite event store
```

The CLI should provide setup and diagnostics:

```sh
agentroom init
agentroom server
agentroom doctor
agentroom emit-fixture
```

## State store

Use SQLite for the first runtime.

SQLite stores:

- append-only event log
- sessions
- repo bindings
- Chatto room/thread bindings
- GitHub PR bindings
- processed GitHub delivery IDs
- evidence packet versions
- readiness state snapshots

SQLite is enough for a single self-hosted AgentRoom instance and avoids introducing a second infrastructure dependency before the merge-confidence loop is proven.

## Event ingestion

All inputs should normalize into the same internal event log:

| Source | Runtime component | Event owner |
| --- | --- | --- |
| Coding agent or fixture | local file watcher / `POST /api/agent-events` | Agent adapter |
| GitHub webhook | `POST /webhooks/github` | GitHub connector |
| Chatto decision | Chatto realtime listener or polling worker | Chatto connector |
| Manual admin action | CLI or future admin UI | AgentRoom core |

The readiness evaluator consumes the event log and current GitHub state to compute the session state.

## Chatto connector runtime

The Chatto connector should run inside `agentroom server`.

Responsibilities:

- connect to Chatto as the dedicated AgentRoom user
- create or bind repo rooms and PR threads
- post status updates and approval prompts
- listen for or poll human decisions
- translate human decisions into internal AgentRoom events

The connector writes events such as:

- `plan.approved`
- `plan.rejected`
- `approval.granted`
- `approval.denied`

These are AgentRoom events produced from Chatto activity. They are not coding-agent adapter events.

## GitHub connector runtime

The GitHub connector handles webhooks and API writes.

Responsibilities:

- verify webhook signatures
- deduplicate delivery IDs
- fetch current PR/check/review state before evaluating readiness
- write GitHub-derived events to the event log
- update the sticky PR evidence comment
- update the `AgentRoom readiness` check run

The connector writes events such as:

- `pr.opened`
- `pr.synchronized`
- `ci.completed`
- `review.completed`
- `mergeability.changed`
- `evidence.posted`

## Readiness evaluator

The readiness evaluator should be deterministic. Given event log state plus current GitHub facts, it should compute:

- current readiness state
- blockers
- required human decisions
- evidence packet content
- GitHub check conclusion

The evaluator should not call external services directly. Connectors gather facts; the evaluator decides.

## Deployment

MVP deployment should support:

1. User runs or already has Chatto.
2. User creates AgentRoom config with Chatto URL, GitHub App credentials, webhook secret, and SQLite path.
3. User starts `agentroom server`.
4. User exposes `/webhooks/github` to GitHub.
5. User emits fixture agent events or connects a local agent script.

## Non-goals for MVP

- multi-replica AgentRoom
- hosted SaaS
- Kubernetes operator
- distributed queues
- managed Chatto installation
- direct merge automation
- multi-tenant organization admin

## Open questions

- Which language should the first binary use?
- Should the file watcher be enabled by default or only in local/dev mode?
- Should SQLite migrations be embedded in the binary?
- What configuration format should `agentroom init` produce?
