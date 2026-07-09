# Agent Adapter Contract

AgentRoom should start with a generic event contract before committing to a deep integration with any one coding agent.

## First adapter choice

The first implementation should be the **generic local event adapter**.

Why:

- It lets AgentRoom test the merge-confidence loop before depending on a vendor-specific agent API.
- It works with Copilot CLI, Claude Code, OpenHands, shell scripts, CI jobs, and manual pilots.
- It gives future agent-specific adapters a concrete contract to target.

Agent-specific adapters can come later, after the event contract proves sufficient against real PRs.

## Transport

The generic adapter should accept newline-delimited JSON events through one or both of these inputs:

1. Local file tailing:

```text
.agentroom/events.ndjson
```

2. Local HTTP ingestion:

```http
POST /api/agent-events
```

The file transport is simplest for local agents and scripts. The HTTP transport is better for remote runners and CI jobs.

## Event envelope

Every event should use this envelope:

```json
{
  "version": "ar.agent_event.v1",
  "event_id": "evt_01JABCDEF00000000000000000",
  "session_id": "ar-session-001",
  "repo": "ktundwal/agentroom",
  "type": "plan.proposed",
  "occurred_at": "2026-07-09T07:00:00Z",
  "producer": {
    "kind": "agent",
    "name": "copilot-cli",
    "run_id": "run_123"
  },
  "links": {
    "issue_url": "https://github.com/ktundwal/agentroom/issues/1",
    "pull_request_url": null,
    "branch": "ar/docs-example"
  },
  "payload": {}
}
```

Rules:

- `event_id` must be globally unique and idempotency-safe.
- `session_id` groups events into one AgentRoom session.
- `repo` must be an `owner/name` GitHub repository.
- `type` must be one of the supported event types.
- `occurred_at` must be an ISO-8601 UTC timestamp.
- `payload` must match the event-specific schema.

## MVP event types

Start with the smallest set needed to create a useful PR thread and evidence packet:

| Event type | Required payload | Purpose |
| --- | --- | --- |
| `session.started` | `goal`, `requested_by` | Opens or links an AgentRoom session. |
| `plan.proposed` | `plan_markdown`, `risk_level`, `sensitive_paths` | Creates the plan gate. |
| `plan.approved` | `approved_by`, `approval_source` | Allows work to proceed. |
| `plan.rejected` | `rejected_by`, `reason` | Blocks work before code. |
| `code.started` | `branch` | Marks implementation as started. |
| `commit.created` | `sha`, `message` | Records code progress. |
| `pr.opened` | `number`, `url`, `head_sha` | Links the GitHub PR. |
| `agent.blocked` | `reason`, `needs_human` | Requests human help. |
| `evidence.note` | `markdown` | Adds agent-supplied context to the evidence packet. |
| `session.completed` | `result` | Marks the agent side done. |

GitHub supplies CI, review, and mergeability events through the GitHub App. Those should not be duplicated by the agent adapter unless the agent is reporting local pre-PR checks.

## Readiness mapping

Agent events update readiness state as follows:

| State | Inputs |
| --- | --- |
| `needs review` | PR exists, plan approved, no blocking evidence, human review pending |
| `blocked` | `plan.rejected`, `agent.blocked`, failing required CI, or merge conflict |
| `risky` | sensitive paths touched, high-risk plan, unresolved human decision, or missing required approval |
| `ready` | plan approved, required CI passed, required review present, no unresolved risk |

The state machine must treat agent claims as hints, not proof. Proof comes from GitHub checks, PR diffs, review state, and recorded human approvals.

## Evidence packet inputs

The evidence packet should include:

- approved plan summary
- branch and PR links
- commit list
- sensitive paths
- agent notes
- GitHub CI status
- GitHub review status
- Chatto approval decisions
- final readiness state

## Validation fixture

The first implementation should include a fixture event stream that exercises the full loop without a real coding agent:

```json
{"version":"ar.agent_event.v1","event_id":"evt_001","session_id":"ar-session-001","repo":"ktundwal/agentroom","type":"session.started","occurred_at":"2026-07-09T07:00:00Z","producer":{"kind":"agent","name":"fixture","run_id":"fixture-1"},"links":{"issue_url":"https://github.com/ktundwal/agentroom/issues/1","pull_request_url":null,"branch":"ar/docs-example"},"payload":{"goal":"Update getting started docs","requested_by":"ktundwal"}}
{"version":"ar.agent_event.v1","event_id":"evt_002","session_id":"ar-session-001","repo":"ktundwal/agentroom","type":"plan.proposed","occurred_at":"2026-07-09T07:01:00Z","producer":{"kind":"agent","name":"fixture","run_id":"fixture-1"},"links":{"issue_url":"https://github.com/ktundwal/agentroom/issues/1","pull_request_url":null,"branch":"ar/docs-example"},"payload":{"plan_markdown":"1. Edit docs/getting-started.md\n2. Open PR\n3. Request review","risk_level":"low","sensitive_paths":[]}}
{"version":"ar.agent_event.v1","event_id":"evt_003","session_id":"ar-session-001","repo":"ktundwal/agentroom","type":"plan.approved","occurred_at":"2026-07-09T07:02:00Z","producer":{"kind":"human","name":"ktundwal","run_id":"chatto-thread-1"},"links":{"issue_url":"https://github.com/ktundwal/agentroom/issues/1","pull_request_url":null,"branch":"ar/docs-example"},"payload":{"approved_by":"ktundwal","approval_source":"chatto"}}
```

## Open questions

- Which agent-specific adapter should follow the generic adapter?
- Should AgentRoom provide wrapper scripts for common CLIs?
- Should local file ingestion be enough for the first public demo?
- How should AgentRoom authenticate remote runner event submissions?
