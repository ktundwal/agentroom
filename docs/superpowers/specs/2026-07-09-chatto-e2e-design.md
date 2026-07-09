# Chatto-Backed E2E Design

## Summary

AgentRoom needs a required end-to-end test that starts a real Chatto instance from the local Chatto source checkout and drives the human decision through Chatto the way a maintainer would. The test must prove the Chatto connector loop, not only the pure evaluator loop.

The first required provider is a black-box local-source harness:

- build/start Chatto from a local checkout, such as `/Users/kapil/github/chatto`
- interact with Chatto through public UI, public ConnectRPC APIs, and realtime/polling behavior
- avoid importing Chatto test helpers or source internals into AgentRoom tests
- drive the human approval through a browser with Playwright
- preserve logs, database, traces, and message IDs on failure

Dockerized Chatto remains a later production-conformance target. The local-source provider is the development gate because it is deterministic, debuggable, and can be pinned to the exact Chatto checkout under test.

The Chatto message and command protocol is defined in [`../../chatto-message-format.md`](../../chatto-message-format.md). This E2E must assert that AgentRoom renders that product format; it must not redefine the human interface inside the test harness.

## Goals

- Prove that AgentRoom can operate against a real Chatto server.
- Prove that AgentRoom posts a real approval request into Chatto.
- Prove that a maintainer can approve by typing `/ar approve <decision-id>` in the Chatto UI.
- Prove that AgentRoom observes the real Chatto reply and writes a Chatto-derived event.
- Prove that readiness and evidence update from that observed event.
- Make failures actionable before manual testing by preserving complete diagnostics.

## Non-goals

- Do not require a production Chatto deployment for the required E2E gate.
- Do not use Docker as the first required E2E provider.
- Do not import Chatto Playwright fixtures, page objects, or internal packages into AgentRoom.
- Do not use Chatto test-only endpoints to create the human approval.
- Do not require live GitHub for the first Chatto connector E2E. GitHub facts should be supplied through deterministic webhook/check fixtures for this gate.

## Required scenario

The required E2E scenario is:

1. Build or reuse a bootstrap-enabled Chatto binary from the local Chatto source checkout.
2. Start one isolated Chatto process with a fresh data directory and deterministic test config.
3. Start AgentRoom with a temporary SQLite database and config pointed at the Chatto base URL.
4. Ingest fixture agent events that create a session and plan gate.
5. Ingest deterministic GitHub fixture events for the linked PR, current head SHA, checks, reviews, and mergeability.
6. AgentRoom posts a Chatto thread root and plan approval request into the repo room.
7. Playwright logs in as a maintainer, opens Chatto, navigates to the repo room/thread, verifies the approval request, types `/ar approve <decision-id>`, and sends the reply.
8. AgentRoom observes the Chatto reply through a forced connector observation mode: realtime or polling.
9. AgentRoom writes a Chatto-derived approval event containing the Chatto room, thread, message, actor, and parsed command.
10. AgentRoom reevaluates readiness and republishes evidence/check output.
11. The test asserts that the readiness state, evidence packet, and event log all reflect the real Chatto approval.

The happy-path scenario must run twice:

- `CHATTO_CONNECTOR_OBSERVATION_MODE=realtime`
- `CHATTO_CONNECTOR_OBSERVATION_MODE=polling`

Each run must assert which mode observed the message. If a mode is unavailable, the test must fail with a clear connector capability error rather than silently exercising the other mode.

## Harness components

### `ChattoSourceHarness`

Owns the Chatto process lifecycle for the test.

Responsibilities:

- read `CHATTO_REPO`, defaulting to `../chatto` from the AgentRoom repo root
- verify the checkout contains `mise.toml`, `apps/frontend/e2e/fixtures/chatto.toml`, and `apps/frontend/e2e/fixtures/bin/chatto` or the sources needed to build it
- run Chatto setup/build commands when the binary is missing or stale
- write an isolated Chatto config and data directory for the test
- start Chatto on an available localhost port
- wait for `GET /readyz` to return ready
- expose `base_url`, `room_id`, connector credentials, maintainer credentials, and log paths to the AgentRoom harness
- terminate Chatto and copy artifacts before cleanup

The implementation may reuse Chatto's existing build task:

```sh
cd "$CHATTO_REPO"
mise run build-e2e-server
```

The AgentRoom harness should treat that as an external build command, not as a library dependency.

### `AgentRoomTestRuntime`

Owns the AgentRoom test process and state.

Responsibilities:

- create a temporary `agentroom.toml`
- create a temporary SQLite path
- set secret environment variables for fixture GitHub and Chatto credentials
- start AgentRoom in test mode
- expose health/readiness endpoints for the harness
- ingest fixture agent events and GitHub events
- expose event-log, readiness, evidence, and connector-status assertions

AgentRoom test mode may use deterministic GitHub fixtures, but it must not synthesize the Chatto approval event. The approval event must be produced only by the Chatto connector after it observes the browser-authored Chatto reply.

### `ChattoBrowserDriver`

Owns the human path.

Responsibilities:

- launch Playwright against the real Chatto base URL
- log in as the maintainer using test credentials
- open the repo room
- find the AgentRoom approval request by the visible text and decision ID defined in `docs/chatto-message-format.md`
- open the thread when needed
- type the exact command into Chatto's thread reply composer
- send the message with the same UI path a human would use
- assert the reply appears in the Chatto thread

The browser driver may use stable accessibility selectors and visible text. It should not call Chatto test-only endpoints for approval.

### `ChattoConnector`

The product connector is part of the system under test.

Responsibilities exercised by this E2E:

- authenticate as the dedicated AgentRoom Chatto user
- post the thread root
- post the approval request
- observe room/thread activity through the forced realtime or polling mode
- parse `/ar approve <decision-id>`
- write a Chatto-derived event to AgentRoom's event log
- ignore messages authored by the AgentRoom connector user

## Chatto setup

The harness should use a generated test config derived from Chatto's E2E fixture model:

- embedded NATS enabled
- video disabled unless a future test needs it
- one fresh data directory per test run
- bootstrap server enabled
- default repo room available, or created by the connector through public APIs
- dedicated AgentRoom connector identity available
- maintainer identity available

The test should prefer public integration credentials supported by Chatto, such as an opaque bearer token for the connector. If the first implementation uses a session-cookie login flow for the source harness, that must be contained in the harness and replaced by token auth once AgentRoom's production connector supports it end to end.

## Required assertions

The test must assert all of the following before passing:

- Chatto `/readyz` returned ready before AgentRoom started.
- AgentRoom reported Chatto connector ready before fixture events were ingested.
- Chatto room contains the AgentRoom thread root.
- Chatto thread contains the AgentRoom approval request from `docs/chatto-message-format.md` with the expected decision ID.
- The maintainer's browser-authored `/ar approve <decision-id>` reply is visible in Chatto.
- AgentRoom event log contains a Chatto-derived approval event.
- The approval event includes Chatto room ID, thread ID, message ID, actor, gate, decision ID, and parsed command.
- The approval event includes the forced observation mode, `realtime` or `polling`.
- The approval event was not inserted before the browser reply existed.
- Readiness reaches the expected post-approval state for the fixture.
- Evidence packet includes the approval and the Chatto thread link.
- The AgentRoom readiness check output matches the computed readiness state.

## Required negative scenarios

The source-backed E2E suite must include negative coverage, not only the happy path.

### Malformed command ignored

1. AgentRoom posts the plan approval request.
2. Playwright replies in the correct Chatto thread with `approve` or `looks good`.
3. The connector observes the reply.
4. AgentRoom must not write `plan.approved` or `approval.granted`.
5. AgentRoom should post the parse-error clarification from `docs/chatto-message-format.md`.
6. Readiness must remain blocked on the pending plan gate.

### Wrong thread ignored

1. AgentRoom posts the plan approval request in the bound PR thread.
2. Playwright replies `/ar approve <decision-id>` in another room or unrelated thread.
3. The connector may observe the message, but it must not create an approval event.
4. Readiness must remain blocked on the pending plan gate.

### Auth failure fails closed

1. Start Chatto normally.
2. Start AgentRoom with an invalid or expired Chatto connector credential.
3. AgentRoom must mark the Chatto connector unhealthy and fail readiness with a clear Chatto-auth blocker.
4. AgentRoom must not create Chatto-derived approval events.
5. AgentRoom must not treat missing Chatto observation as an approval or success-shaped fallback.

### Connector reconnect preserves correctness

Run reconnect coverage separately for each observation mode:

- `realtime`: disconnect or restart the connector after the approval request is posted, wait for the realtime subscription to reconnect, then have Playwright post `/ar approve <decision-id>`. The approval event must record `observation_mode=realtime`.
- `polling`: stop the connector after the approval request is posted, have Playwright post `/ar approve <decision-id>`, restart the connector, and require polling catch-up from the stored cursor. The approval event must record `observation_mode=polling`.

Both variants must write exactly one approval event.

## Failure handling

The harness should fail loudly and preserve artifacts.

On any failure, copy these into the E2E artifact directory:

- Chatto stdout/stderr logs
- AgentRoom stdout/stderr logs
- generated Chatto config with secrets redacted
- generated AgentRoom config with secrets redacted
- AgentRoom SQLite database
- Chatto data directory metadata needed to inspect message state
- Playwright trace
- Playwright screenshot
- Playwright video when enabled
- JSON dump of relevant AgentRoom event-log rows
- JSON dump of readiness snapshots
- the Chatto room ID, thread ID, message ID, and decision ID under test
- the forced observation mode and actual observation mode recorded on the event

Cleanup should run only after artifact capture.

## Isolation and determinism

The full E2E should run serially at first.

Each run must use:

- fresh ports
- fresh Chatto data directory
- fresh AgentRoom SQLite database
- deterministic session ID
- deterministic decision ID
- deterministic PR number and head SHA
- deterministic fixture timestamps where possible

The test should not depend on external GitHub network calls. Live GitHub can be added as a separate smoke test after the Chatto connector loop is proven.

## Provider model

The first provider is:

```sh
CHATTO_E2E_PROVIDER=source
CHATTO_REPO=/Users/kapil/github/chatto
```

Future provider:

```sh
CHATTO_E2E_PROVIDER=docker
```

Both providers should eventually run the same AgentRoom scenario. The source provider is required first. The Docker provider validates production packaging later.

## Done criteria

The Chatto connector implementation is not done until this E2E passes from a clean AgentRoom checkout against the local Chatto source checkout.

Passing means:

- the test starts real Chatto
- the test starts real AgentRoom
- the approval is typed through the real Chatto UI
- AgentRoom observes that real Chatto message
- realtime and polling observation-mode variants both pass
- malformed-command and wrong-thread negative scenarios do not create approval events
- Chatto auth failure fails closed without synthetic approvals
- readiness/evidence update from the observed message
- no manual pre-seeding of approval events is used
- all artifacts needed to debug a failure are available

The expected manual-testing state is zero known bugs. If the E2E exposes a connector, parser, timing, auth, or evidence bug, fix the product or harness before asking a human to test the flow manually.
