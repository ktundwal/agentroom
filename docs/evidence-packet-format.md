# Evidence Packet Format

The evidence packet is the primary human-facing output of AgentRoom. It must make the merge decision easier without requiring reviewers to read raw agent logs.

AgentRoom publishes the packet as:

1. A sticky GitHub PR comment.
2. The summary/body of the `AgentRoom readiness` check run.

## Sticky PR comment marker

The sticky comment should contain this hidden marker so AgentRoom can update it in place:

```html
<!-- agentroom:evidence session_id=ar-session-001 pr=42 -->
```

AgentRoom should update the existing marked comment for the same session and PR. It should not create a new comment for every evaluation.

## Version format

Evidence packet versions should be monotonic per session:

```text
ar-evp-v1-000001
ar-evp-v1-000002
```

The version identifies the rendered packet format (`v1`) and the per-session update number.

## Sticky PR comment structure

```markdown
<!-- agentroom:evidence session_id=ar-session-001 pr=42 -->

# AgentRoom evidence: risky

**Verdict:** Needs architecture approval before merge.

| Field | Value |
| --- | --- |
| Session | `ar-session-001` |
| Evidence version | `ar-evp-v1-000003` |
| PR head | `abc1234` |
| Updated | `2026-07-09T07:30:00Z` |
| Chatto thread | [repo-docs / PR #42](https://chat.example.com/...) |

## Blockers and decisions

- [ ] **Risk approval required:** `docs/security.md` changed and matches sensitive path policy.
- [x] **Plan approved:** approved by `ktundwal` in Chatto.
- [x] **Required checks passed:** `test`, `lint`.
- [ ] **Required review missing:** no approving GitHub review for current head SHA.

## Approved plan

> Update getting started docs and open a PR with evidence packet.

Approved by `ktundwal` via Chatto at `2026-07-09T07:02:00Z`.

## Changes

| Type | Files |
| --- | --- |
| Docs | `docs/getting-started.md`, `docs/security.md` |
| Sensitive paths | `docs/security.md` |

## CI and checks

| Check | Required | Status | Conclusion |
| --- | --- | --- | --- |
| `test` | yes | completed | success |
| `lint` | yes | completed | success |

## Reviews

| Reviewer | State | Commit |
| --- | --- | --- |
| _none_ | pending | `abc1234` |

## Agent notes

- Agent reports docs updated and ready for review.

## Audit links

- Chatto room: https://chat.example.com/...
- AgentRoom session: `ar-session-001`
```

## Rendering partial data

Partial data should be explicit, not omitted.

| Missing data | Render as | Readiness effect |
| --- | --- | --- |
| No approved plan | `Plan: missing` | `blocked` |
| No PR linked | `Pull request: not opened` | `blocked` |
| CI not started | `Checks: pending / not reported` | `needs review` unless policy requires completed checks, then `blocked` |
| CI in progress | `Checks: in progress` | `needs review` |
| No reviews | `Reviews: none for current head` | `needs review` |
| Chatto thread missing | `Chatto thread: missing` | `risky` |
| Stale GitHub state | `GitHub state: stale since <timestamp>` | `risky` or `blocked` depending on policy |

## Blockers vs informational items

Use three categories:

1. **Blockers** prevent readiness.
2. **Decisions** require explicit human action.
3. **Notes** provide context but do not affect readiness.

Blockers should always appear before summaries. If there are no blockers, render:

```markdown
## Blockers and decisions

- [x] No blockers detected for current head SHA.
```

## Multiple decision gates

Sessions can have multiple gates:

- plan approval
- sensitive-path approval
- architecture approval
- release approval

Each gate should render independently:

```markdown
## Decision gates

| Gate | Required | State | Decided by | Source |
| --- | --- | --- | --- | --- |
| Plan approval | yes | approved | ktundwal | Chatto |
| Sensitive paths | yes | pending | _none_ | Chatto |
| Architecture approval | no | not required | _n/a_ | policy |
```

Any required pending or denied gate affects readiness according to the readiness evaluator.

## Check run output

The `AgentRoom readiness` check run should contain a compact version:

**Title:**

```text
AgentRoom readiness: risky
```

**Summary:**

```markdown
Needs architecture approval before merge.

- Evidence: ar-evp-v1-000003
- Head SHA: abc1234
- Chatto thread: https://chat.example.com/...
- Blockers: 2
- Required checks: 2/2 passed
- Required reviews: 0/1 approved
```

**Details URL:** link to the sticky PR comment when possible.

## Update rules

AgentRoom should republish the packet when:

- readiness state changes
- PR head SHA changes
- required check state changes
- review state changes
- Chatto decision changes
- evidence note changes

AgentRoom should not republish for duplicate events or unchanged snapshots.
