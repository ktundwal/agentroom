# Chatto Message Format

Chatto is AgentRoom's primary collaboration surface. Messages must be structured enough for humans to act quickly and for the Chatto connector to parse decisions reliably.

The MVP should use plain text / Markdown messages posted by the dedicated AgentRoom Chatto user. It should not depend on native Chatto slash commands, custom cards, or first-class bot interactions.

This document is the product contract for Chatto-side human interaction. The Chatto connector, parser tests, Chatto-backed E2E tests, and evidence links should conform to this format rather than redefining commands elsewhere.

## Thread model

Each agent-authored PR gets one Chatto thread.

Thread root message:

```markdown
AgentRoom: PR #42 is linked to session `ar-session-001`

Goal: Update getting started docs.
Branch: `ar/docs-example`
GitHub PR: https://github.com/ktundwal/agentroom/pull/42

Current state: needs review

Reply in this thread to approve, reject, pause, or redirect AgentRoom decisions.

ar-meta: kind=thread-root session=ar-session-001 pr=42
```

The `ar-meta:` footer is machine-readable. The connector should parse it only from messages authored by the AgentRoom Chatto user.

## Approval request message

Approval requests must name the gate and provide exact response commands.

```markdown
AgentRoom approval required: plan

Session: `ar-session-001`
Decision: `ar-decision-0001`
Requested by: `copilot-cli`

Plan:
1. Edit `docs/getting-started.md`
2. Open a PR
3. Request review

Risk: low
Sensitive paths: none

Reply with one exact command:
- `/ar approve ar-decision-0001`
- `/ar reject ar-decision-0001: <reason>`
- `/ar pause ar-decision-0001: <reason>`
- `/ar redirect ar-decision-0001: <new instruction>`

ar-meta: kind=approval-request session=ar-session-001 decision=ar-decision-0001 gate=plan
```

## Gate names

Supported MVP gates:

| Gate | Meaning |
| --- | --- |
| `plan` | Approve the plan before code work proceeds. |
| `sensitive-paths` | Approve changes to configured sensitive paths. |
| `architecture` | Approve a design or architecture decision. |

The command includes the decision ID, not just the gate name, so multiple gates can be open in the same thread.

## Human response protocol

The Chatto connector should accept only thread replies that match one of these command forms:

```text
/ar approve <decision-id>
/ar reject <decision-id>: <reason>
/ar pause <decision-id>: <reason>
/ar redirect <decision-id>: <new instruction>
```

Rules:

- Ignore messages authored by the AgentRoom Chatto user.
- Ignore replies outside the bound PR thread.
- Ignore commands for unknown or already-closed decision IDs.
- Require a reason for `reject`, `pause`, and `redirect`.
- Treat reactions as non-authoritative hints in the MVP.
- Accept decisions only from configured approvers when policy requires approvers.

## Ambiguous responses

If a human reply mentions AgentRoom but does not match the command grammar, the connector should post a clarification:

```markdown
AgentRoom could not parse that decision.

Please reply with one exact command:
- `/ar approve ar-decision-0001`
- `/ar reject ar-decision-0001: <reason>`
- `/ar pause ar-decision-0001: <reason>`
- `/ar redirect ar-decision-0001: <new instruction>`

ar-meta: kind=parse-error session=ar-session-001 decision=ar-decision-0001
```

Ambiguous replies should not create approval or rejection events.

## Readiness summary message

The connector should post or update a concise readiness summary in the thread:

```markdown
AgentRoom readiness: risky

PR: https://github.com/ktundwal/agentroom/pull/42
Head: `abc1234`
Evidence: `ar-evp-v1-000003`

Blockers and decisions:
- [ ] Architecture approval required
- [x] Plan approved by `ktundwal`
- [x] Required checks passed: `test`, `lint`
- [ ] Required review missing for current head

Next action: `/ar approve ar-decision-0002` or review the GitHub PR.

ar-meta: kind=readiness-summary session=ar-session-001 pr=42 evidence=ar-evp-v1-000003
```

For MVP, AgentRoom may post new summaries instead of editing prior messages if Chatto update semantics are not reliable. Once message update is stable, AgentRoom should update its own latest summary message in place.

## Event translation

Parsed Chatto commands create AgentRoom events:

| Command | Event |
| --- | --- |
| `/ar approve <decision-id>` | `plan.approved` or `approval.granted` based on gate |
| `/ar reject <decision-id>: <reason>` | `plan.rejected` or `approval.denied` based on gate |
| `/ar pause <decision-id>: <reason>` | `approval.denied` with `decision=paused` |
| `/ar redirect <decision-id>: <instruction>` | `approval.denied` with `decision=redirected` and a follow-up `evidence.note` |

The event payload should include:

- decision ID
- gate
- Chatto room ID
- Chatto thread ID
- Chatto message ID
- human Chatto user
- parsed command
- reason or instruction when present
- observation mode: `realtime` or `polling`

## Security notes

- Do not parse decisions from unbound rooms.
- Do not parse decisions from the AgentRoom connector account.
- Do not accept approval commands from users that cannot be mapped to approved Chatto/GitHub identities when policy requires identity checks.
- Keep the original Chatto message ID in the event payload for auditability.
