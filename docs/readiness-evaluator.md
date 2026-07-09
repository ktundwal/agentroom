# Readiness Evaluator

The readiness evaluator is the deterministic decision engine for AgentRoom.

Given one session, the event log, and latest-value SQLite state, it returns:

- readiness state
- blockers
- decision gates
- evidence packet inputs
- GitHub check conclusion

The evaluator must not call GitHub or Chatto directly.

## State precedence

Readiness states are ordered by severity:

```text
blocked > risky > needs review > ready
```

The evaluator should compute all applicable findings, then choose the highest-severity state.

## Decision tree

Evaluate in this order:

1. **Session validity**
   - Missing session -> `blocked`
   - Session closed/cancelled -> `blocked`
2. **Plan gate**
   - Plan rejected -> `blocked`
   - Plan missing -> `blocked`
   - Plan proposed but not approved -> `blocked`
3. **PR linkage**
   - PR missing -> `blocked`
   - PR closed without merge -> `blocked`
   - PR head SHA missing -> `blocked`
4. **Freshness**
   - GitHub PR/check/review state stale beyond policy threshold -> `risky`
   - Chatto decision state stale beyond policy threshold -> `risky`
5. **Mergeability**
   - Merge conflict or GitHub non-mergeable state -> `blocked`
   - Draft PR -> `needs review`
6. **Agent blockers**
   - Open `agent.blocked` event with unresolved human need -> `blocked`
7. **Required checks**
   - Required check failed, cancelled, timed out, or action required -> `blocked`
   - Required check missing or in progress -> `needs review`
8. **Sensitive paths and risk gates**
   - Sensitive paths touched without required approval -> `risky`
   - High-risk plan without risk approval -> `risky`
   - Required decision denied -> `blocked`
9. **Reviews**
   - Changes requested on current head -> `blocked`
   - Required review missing for current head -> `needs review`
10. **Ready**
   - Plan approved
   - PR linked
   - current GitHub state fresh
   - required checks passed for current head
   - required reviews present for current head
   - required decision gates approved
   - no unresolved blockers

Return `ready` only when every ready condition is true.

## Conflict examples

| Inputs | State | Reason |
| --- | --- | --- |
| CI passes, required review missing | `needs review` | Work may be correct but still needs human review. |
| Plan approved, sensitive paths touched, no risk approval | `risky` | Human decision required before readiness. |
| Agent says `session.completed`, CI in progress | `needs review` | Agent completion is not proof of mergeability. |
| Required check failed and review approved | `blocked` | Failed required checks outrank approval. |
| Review changes requested and CI passing | `blocked` | Requested changes block readiness. |
| GitHub state stale | `risky` | AgentRoom cannot prove current readiness. |
| PR head SHA changed during evaluation | re-evaluate | Snapshot is stale; recompute on the new head SHA. |

## Head SHA consistency

The evaluator should read all GitHub latest-value state for one `head_sha`.

If `github_pr_state.head_sha` changes while evaluation is running:

1. Discard the computed result.
2. Reload PR, check, and review state.
3. Re-run evaluation for the new head SHA.

Readiness snapshots must record the evaluated `head_sha`.

## Decision gates

Decision gates should be normalized before readiness evaluation:

| Gate | Required when | Blocking behavior |
| --- | --- | --- |
| Plan approval | Always | Missing/rejected plan -> `blocked` |
| Sensitive path approval | Changed files match `sensitive_paths` | Missing approval -> `risky`; denied -> `blocked` |
| Architecture approval | Policy or human request requires it | Missing approval -> `risky`; denied -> `blocked` |
| Release approval | Future deployment automation | Missing approval -> `risky`; denied -> `blocked` |

## Check run conclusion mapping

| Readiness state | Check conclusion |
| --- | --- |
| `ready` | `success` |
| `needs review` | `neutral` |
| `risky` | `action_required` |
| `blocked` | `failure` |

## Determinism requirements

- Evaluate against one SQLite transaction snapshot where possible.
- Produce stable blocker ordering.
- Include every applicable blocker/decision, not just the first one.
- Never let agent-supplied claims override GitHub facts or human decisions.
- Treat unknown required state as non-ready.
