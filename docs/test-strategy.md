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
- validates messages against `docs/chatto-message-format.md`
- ignores ambiguous replies
- ignores valid-looking commands in the wrong room or thread
- ignores replies from the AgentRoom connector user
- translates accepted commands into AgentRoom events
- records whether an event was observed through realtime or polling
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

The offline fixture E2E should run without live Chatto or GitHub:

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

### Required Chatto-backed E2E

The full integration gate must include a real Chatto instance. It should start Chatto from a local source checkout, drive the human approval through Chatto's browser UI, and require AgentRoom to observe that real Chatto message before writing the approval event.

Required provider:

```sh
CHATTO_E2E_PROVIDER=source
CHATTO_REPO=/Users/kapil/github/chatto
```

Required flow:

1. Build or reuse a bootstrap-enabled Chatto binary from the local Chatto checkout.
2. Start Chatto with a fresh data directory and wait for `/readyz`.
3. Start AgentRoom with a temporary config and SQLite database.
4. Ingest fixture agent events and deterministic GitHub fixture events.
5. AgentRoom posts the Chatto thread root and approval request.
6. Playwright logs in as a maintainer and replies `/ar approve <decision-id>` through the real Chatto UI.
7. AgentRoom observes the Chatto reply through a forced connector observation mode.
8. AgentRoom writes the Chatto-derived approval event, recomputes readiness, and republishes evidence.

This test must not synthesize the Chatto approval event. The approval event is valid only if it includes the real Chatto room ID, thread ID, message ID, actor, gate, decision ID, and parsed command.

The happy path must run in both modes:

```sh
CHATTO_CONNECTOR_OBSERVATION_MODE=realtime
CHATTO_CONNECTOR_OBSERVATION_MODE=polling
```

The event log must record the mode that observed the message, and the test must assert it.

Required negative Chatto-backed E2E scenarios:

- malformed human command in the correct thread is ignored, produces the documented parse-error clarification, and does not create an approval event
- valid-looking command in the wrong room or thread is ignored and does not create an approval event
- invalid or expired Chatto connector credentials mark the connector unhealthy, fail closed, and do not create approval events
- connector disconnect/reconnect coverage runs separately for forced realtime and forced polling, and each mode creates exactly one approval event with the matching observation mode

See [`superpowers/specs/2026-07-09-chatto-e2e-design.md`](superpowers/specs/2026-07-09-chatto-e2e-design.md) for the full design.

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

The source-backed Chatto E2E is the required live integration test for the Chatto connector. Live GitHub tests should remain optional until the Chatto connector loop is proven.

Required local-source Chatto settings:

```sh
CHATTO_E2E_PROVIDER=source
CHATTO_REPO=/Users/kapil/github/chatto
```

Optional live-GitHub smoke setting:

```sh
AGENTROOM_LIVE_GITHUB=1
```

Default CI can keep fake-client and recorded-webhook tests for fast feedback, but the Chatto connector milestone is not complete until the source-backed Chatto E2E passes.

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
- source-backed Chatto E2E passes for Chatto connector changes

No live GitHub service should be required for the default gate.
