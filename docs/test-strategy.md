# Test Strategy

AgentRoom's first implementation should be test-first around the merge-confidence loop.

The core invariant:

> Given normalized agent, GitHub, and Chatto inputs, AgentRoom deterministically produces the expected readiness state, evidence packet, and GitHub check conclusion.

## Test layers

### Unit tests

Unit tests should cover pure logic without network calls:

| Component | Test style |
| --- | --- |
| Readiness evaluator | table-driven input states -> readiness, blockers, check conclusion |
| Evidence packet renderer | golden Markdown snapshots for complete, partial, risky, and blocked cases |
| Chatto command parser | command strings -> decision events or parse errors |
| Config loader | TOML examples -> validated config or exact validation errors |
| State upsert logic | event + prior state -> latest-value table changes |

### Connector tests

Connector tests should use fake clients, not live services.

GitHub connector:

- verifies webhook signature handling
- deduplicates delivery IDs
- handles `pull_request`, `check_run`, `check_suite`, `status`, and review payload fixtures
- updates latest-value GitHub tables
- updates sticky evidence comment instead of creating duplicates
- updates `AgentRoom readiness` for the current head SHA

Chatto connector:

- posts thread root, approval request, parse-error, and readiness summary messages through a fake Chatto client
- parses accepted `/ar ...` commands
- ignores ambiguous replies
- ignores replies from the AgentRoom connector user
- translates accepted commands into AgentRoom events
- keeps room/thread/message IDs in event payloads

Agent event ingestion:

- accepts valid NDJSON fixture events
- rejects unknown event versions
- rejects malformed required fields
- deduplicates by `event_id`
- handles local file and HTTP ingestion paths with the same validation

### Renderer golden tests

Evidence packet renderer tests should store expected Markdown fixtures for:

- `ready` PR with full data
- `needs review` PR with missing review
- `risky` PR with sensitive paths and pending approval
- `blocked` PR with failed required check
- stale GitHub state
- missing Chatto thread

Golden tests should normalize timestamps and evidence versions so diffs stay readable.

### End-to-end fixture test

The first E2E test should run without live Chatto or GitHub:

1. Create a temporary SQLite database.
2. Load a test `agentroom.toml`.
3. Ingest fixture agent events.
4. Ingest fixture Chatto approval event.
5. Ingest fixture GitHub PR/check/review events.
6. Run readiness evaluation.
7. Render evidence packet.
8. Assert final readiness, check conclusion, and Markdown output.

Expected result:

```text
state=ready
check_conclusion=success
evidence_contains=AgentRoom evidence: ready
```

### Head-SHA consistency tests

Tests should cover:

- PR head changes after checks pass -> old checks do not apply
- review on old head does not satisfy required review for new head
- evaluator discards stale snapshot and re-evaluates current head

### Idempotency tests

Tests should cover duplicate:

- agent event IDs
- GitHub delivery IDs
- Chatto command messages
- evidence packet publication attempts

Duplicates must not create duplicate event rows, duplicate PR comments, duplicate Chatto messages, or duplicate readiness snapshots.

## Live integration tests

Live Chatto and GitHub tests should be optional and excluded from default CI.

Suggested opt-in flags:

```sh
AGENTROOM_LIVE_GITHUB=1
AGENTROOM_LIVE_CHATTO=1
```

Default CI should use fake clients and recorded webhook fixtures.

## Test fixtures

Recommended layout once code exists:

```text
testdata/
  agent-events/
  github-webhooks/
  chatto-messages/
  evidence-golden/
  configs/
```

## Minimum pre-merge test gate

Before merging implementation changes, the default test command should prove:

- readiness evaluator table tests pass
- evidence renderer golden tests pass
- Chatto parser tests pass
- GitHub webhook fixture tests pass
- config validation tests pass
- E2E fixture test passes

No live service should be required for the default gate.
