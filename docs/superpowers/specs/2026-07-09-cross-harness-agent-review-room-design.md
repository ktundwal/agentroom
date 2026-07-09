# Cross-Harness Agent Review Room Design

## Summary

Developers already compare multiple agent harnesses manually: one CLI implements or proposes, another reviews, a third verifies, and the human copy-pastes findings between tools. AgentRoom can help later by turning that informal cross-harness loop into a structured review round inside the existing Chatto PR thread.

This is a future extension, not part of the MVP merge-desk loop. The MVP should first prove GitHub-grounded readiness, Chatto approvals, evidence packets, and the source-backed Chatto E2E.

## Product boundary

AgentRoom should not become a universal launcher or fleet orchestrator for Copilot CLI, Mimo, Claude Code, Fleet, or other harnesses.

The extension should instead provide an **agent review room**:

- multiple harnesses participate as named reviewers or contributors
- each finding is captured as structured evidence
- AgentRoom groups convergence, disagreement, risks, unresolved questions, and human decisions
- Chatto remains the shared collaboration surface
- GitHub remains the source of truth for PRs, checks, reviews, and merge
- the human remains the decision-maker

## User problem

The user currently does this by hand:

1. Ask one harness to review or implement.
2. Paste its output into another harness.
3. Paste the second harness's critique back into the first.
4. Repeat until the agents converge or the human decides.
5. Manually summarize the final state for a PR or design decision.

This is useful because different harnesses have different strengths, context windows, tools, models, and review habits. It is also brittle because the durable state lives in the human's clipboard and memory.

## Review round model

A review round is a structured conversation attached to one AgentRoom session and usually one Chatto PR thread.

Fields:

| Field | Purpose |
| --- | --- |
| `round_id` | Stable review-round identifier. |
| `session_id` | AgentRoom session under review. |
| `topic` | What the round is about, such as architecture, risk, tests, or docs. |
| `status` | `open`, `resolved`, or `cancelled`. |
| `participants` | Named harnesses or humans expected to contribute. |
| `opened_by` | Human or AgentRoom policy that started the round. |
| `resolved_by` | Human who resolved the round. |
| `resolution` | Final human decision or summary. |
| `chatto_thread_id` | Thread containing the round. |

## Future event types

These events are future-only and should not be required for MVP readiness:

| Event | Producer | Purpose |
| --- | --- | --- |
| `review_round.started` | human or AgentRoom | Opens a structured agent review round. |
| `agent.finding.posted` | agent/harness | Records a critique, question, concern, or recommendation. |
| `agent.response.posted` | agent/harness | Records a response to a finding. |
| `review_round.summary.posted` | AgentRoom | Records grouped convergence/disagreement/unresolved questions. |
| `review_round.resolved` | human | Records the human's final decision for the round. |

## Chatto interaction

AgentRoom should post a round header into the PR thread:

```markdown
AgentRoom review round: architecture readiness

Round: `ar-round-0001`
Session: `ar-session-001`
Participants: `copilot-cli`, `mimo`

Question:
Does this design have enough detail to start implementation safely?

Reply with findings, or resolve with:
`/ar resolve-round ar-round-0001: <decision>`

ar-meta: kind=review-round session=ar-session-001 round=ar-round-0001
```

Harnesses can contribute through:

- the generic local event adapter
- local HTTP event ingestion
- a copied/imported finding pasted by the human
- a future harness-specific adapter

AgentRoom should not require direct access to each harness's proprietary session state.

## Evidence packet rendering

Evidence packets should include a compact future section only when review rounds exist:

```markdown
## Agent review rounds

| Round | Status | Consensus | Disagreements | Resolved by |
| --- | --- | --- | --- | --- |
| Architecture readiness | resolved | Runtime, state, tests are specified | Chatto auth bootstrap remains soft gap | ktundwal |

Links:
- Chatto review round: https://chat.example.com/...
- Mimo finding: `ar-finding-0007`
- Copilot response: `ar-response-0012`
```

Unresolved review rounds should affect readiness only when policy marks them required. If required and unresolved, they should render as `risky` or `blocked` according to the readiness evaluator's decision-gate rules.

## State model extension

Future latest-value tables:

### `review_round_state`

| Column | Purpose |
| --- | --- |
| `round_id` | Review round ID. |
| `session_id` | AgentRoom session. |
| `topic` | Round topic. |
| `status` | `open`, `resolved`, or `cancelled`. |
| `participants_json` | Expected harness/human participants. |
| `consensus_json` | Current grouped consensus points. |
| `disagreements_json` | Current grouped disagreements. |
| `unresolved_questions_json` | Open questions. |
| `chatto_thread_id` | Chatto thread containing the round. |
| `updated_at` | Last update time. |

### `review_finding_state`

| Column | Purpose |
| --- | --- |
| `finding_id` | Finding ID. |
| `round_id` | Parent review round. |
| `producer` | Harness or human name. |
| `type` | `risk`, `bug`, `question`, `recommendation`, or `verification`. |
| `severity` | Optional severity. |
| `status` | `open`, `answered`, `accepted`, `rejected`, or `resolved`. |
| `evidence_links_json` | File paths, PR links, test logs, or Chatto messages. |
| `updated_at` | Last update time. |

## Safeguards

- AgentRoom summarizes disagreement but does not decide which agent is correct.
- Findings should cite evidence: files, PR comments, tests, logs, or Chatto message links.
- Unresolved disagreements should stay visible in the evidence packet.
- Human resolution is required before a review round can improve readiness.
- No harness gets special trust by default.
- Agent claims remain hints until backed by GitHub state, tests, or human approval.

## Testing strategy

Future tests should include:

- fixture review rounds with conflicting findings
- parser tests for imported agent findings
- renderer golden tests for consensus, disagreement, and unresolved-question sections
- E2E where two named harnesses post findings into Chatto and a human resolves the round
- negative tests where unresolved required rounds keep readiness non-ready

## Milestone placement

This should be considered after:

1. AgentRoom can ingest generic agent events.
2. Chatto-backed approvals work against a real Chatto instance.
3. GitHub evidence packets are stable.
4. At least one real team has used AgentRoom for agent-authored PR review.

It should not block the Go MVP implementation.
