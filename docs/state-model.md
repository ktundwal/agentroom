# State Model

AgentRoom uses SQLite for both append-only history and latest-value runtime state.

The design deliberately separates:

- **event log:** immutable facts received from agents, GitHub, Chatto, and AgentRoom itself
- **runtime state:** current snapshots used by the readiness evaluator

This mirrors the event-vs-latest-value split used by evented systems such as Chatto. The evaluator should not replay the full event log on every decision.

## Write model

Every connector write should happen in one transaction:

1. Insert the immutable event into `event_log`.
2. Upsert any latest-value rows affected by that event.
3. Enqueue or trigger readiness evaluation for the affected session.

If the event is a duplicate, the transaction should no-op by `event_id` or source delivery ID.

## Core tables

### `event_log`

Append-only record of all normalized events.

| Column | Purpose |
| --- | --- |
| `event_id` | Unique idempotency key. |
| `session_id` | AgentRoom session. |
| `repo` | GitHub `owner/name`. |
| `source` | `agent`, `github`, `chatto`, or `agentroom`. |
| `type` | Event type. |
| `occurred_at` | Source timestamp. |
| `received_at` | AgentRoom timestamp. |
| `payload_json` | Original normalized payload. |

### `sessions`

Latest session metadata.

| Column | Purpose |
| --- | --- |
| `session_id` | Primary key. |
| `repo` | GitHub `owner/name`. |
| `goal` | Human-readable goal. |
| `status` | `planned`, `running`, `blocked`, `completed`, `closed`. |
| `current_readiness` | `needs review`, `blocked`, `risky`, `ready`. |
| `created_at` | Session creation time. |
| `updated_at` | Last state change. |

### `github_pr_state`

Latest PR snapshot for evaluator reads.

| Column | Purpose |
| --- | --- |
| `repo` | GitHub `owner/name`. |
| `pr_number` | Pull request number. |
| `session_id` | Linked AgentRoom session. |
| `head_sha` | Current PR head SHA. |
| `base_ref` | Target branch. |
| `state` | GitHub PR state. |
| `draft` | Whether PR is draft. |
| `mergeable_state` | Latest known mergeability state. |
| `changed_files_json` | Latest changed-file summary. |
| `updated_at` | Last GitHub refresh time. |

### `github_check_state`

Latest check/status state per PR head SHA.

| Column | Purpose |
| --- | --- |
| `repo` | GitHub `owner/name`. |
| `pr_number` | Pull request number. |
| `head_sha` | Commit SHA the check applies to. |
| `name` | Check run or status context. |
| `source` | `check_run`, `check_suite`, or `status`. |
| `status` | queued, in_progress, completed, or provider-specific equivalent. |
| `conclusion` | success, failure, neutral, cancelled, timed_out, action_required, or null. |
| `required` | Whether this check is required by AgentRoom policy. |
| `details_url` | Link to provider details. |
| `updated_at` | Last GitHub refresh time. |

### `github_review_state`

Latest review state per reviewer.

| Column | Purpose |
| --- | --- |
| `repo` | GitHub `owner/name`. |
| `pr_number` | Pull request number. |
| `reviewer` | GitHub login. |
| `state` | approved, changes_requested, commented, dismissed, pending. |
| `submitted_at` | Review submission time. |
| `commit_sha` | SHA reviewed when available. |

### `chatto_thread_state`

Latest Chatto binding and decision state.

| Column | Purpose |
| --- | --- |
| `session_id` | AgentRoom session. |
| `chatto_room_id` | Bound Chatto room. |
| `chatto_thread_id` | Bound Chatto thread. |
| `last_seen_event_id` | Realtime or polling cursor. |
| `latest_decision` | approved, rejected, paused, redirected, or null. |
| `latest_decision_by` | Human who made the decision. |
| `updated_at` | Last Chatto sync time. |

### `readiness_snapshots`

Historical readiness decisions.

| Column | Purpose |
| --- | --- |
| `snapshot_id` | Unique snapshot ID. |
| `session_id` | AgentRoom session. |
| `repo` | GitHub `owner/name`. |
| `pr_number` | Pull request number. |
| `head_sha` | PR head SHA evaluated. |
| `state` | `needs review`, `blocked`, `risky`, or `ready`. |
| `blockers_json` | Structured blockers. |
| `decision_gates_json` | Normalized gate states used for packet rendering. |
| `evidence_version` | Evidence packet version produced from this snapshot. |
| `created_at` | Evaluation time. |

## Evaluator read path

The readiness evaluator should read:

1. `sessions`
2. `github_pr_state`
3. `github_check_state`
4. `github_review_state`
5. `chatto_thread_state`
6. relevant recent `event_log` entries for provenance and evidence notes

It should not call GitHub or Chatto directly. If state is stale, the evaluator should return `blocked` or `risky` with a stale-data reason and let connectors refresh facts.

## Required check handling

Required checks should come from AgentRoom repo policy, not only GitHub branch protection. GitHub branch protection can be complex and sometimes unavailable to app permissions.

For MVP, each repo binding should declare required check names in config. The GitHub connector marks matching rows in `github_check_state.required = true`, and the evaluator requires all required checks for the current `head_sha` to conclude successfully before returning `ready`.

## Idempotency

Deduplication keys:

- agent events: `event_id`
- GitHub webhooks: delivery ID plus event type
- Chatto decisions: Chatto event ID or room/thread/message/reaction tuple
- evidence packets: session ID plus PR head SHA plus evidence version

Duplicate events must not create duplicate Chatto messages, duplicate PR comments, or duplicate readiness snapshots.
