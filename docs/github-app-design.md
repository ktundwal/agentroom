# GitHub App Design

AgentRoom should use GitHub as the source of truth for pull requests, checks, reviews, branch protection, and final merge.

## MVP responsibility

The GitHub App should do four things:

1. Observe PR state.
2. Publish AgentRoom readiness as a check.
3. Attach evidence packets to PRs.
4. Link GitHub activity to Chatto PR threads.

It should not merge PRs in the MVP.

## Permissions

Initial GitHub App permissions:

| Permission | Level | Why |
| --- | --- | --- |
| Contents | Read | Read changed files and commits for evidence and risk classification. |
| Issues | Read | Link sessions started from issues. |
| Pull requests | Read & write | Read PR metadata, changed files, reviews, and post PR comments. |
| Checks | Read & write | Read existing checks and create the AgentRoom readiness check run. |
| Commit statuses | Read | Read legacy statuses when repos still use them. |
| Metadata | Read | Required baseline GitHub App permission. |

Avoid requesting write access to contents, deployments, environments, secrets, or administration in the MVP.

## Webhook events

Subscribe to:

- `pull_request`
- `pull_request_review`
- `pull_request_review_comment`
- `check_run`
- `check_suite`
- `status`
- `issue_comment`
- `installation`
- `installation_repositories`

`issue_comment` is needed only if AgentRoom supports PR commands such as `/ar link` or `/ar refresh`.

## Evidence packet attachment

AgentRoom should attach evidence in two places:

1. **Sticky PR comment** for human-readable evidence.
2. **Check run named `AgentRoom readiness`** for branch protection and machine-readable readiness.

The sticky PR comment should be updated in place. It should include:

- current readiness state
- approved plan summary
- links to Chatto room/thread
- changed files summary
- CI/check summary
- review summary
- approval decisions
- unresolved blockers
- timestamp and evidence packet version

The check run should use:

| Readiness state | GitHub check conclusion |
| --- | --- |
| `ready` | `success` |
| `needs review` | `neutral` |
| `risky` | `action_required` |
| `blocked` | `failure` |

If GitHub branch protection cannot require `neutral` or `action_required` semantics consistently, repositories should require the `success` state for merge and treat every other state as non-ready.

## Branch protection

AgentRoom should support branch protection by exposing `AgentRoom readiness` as a required check.

Recommended rule:

```text
Require status checks to pass before merging:
- AgentRoom readiness
- existing CI checks
```

AgentRoom should not bypass branch protection. It should only report readiness.

## Fork PRs

Fork PRs need stricter handling:

- Do not trust agent events from fork-controlled branches.
- Do not expose Chatto credentials, GitHub write tokens, or repository secrets to fork code.
- Do not mark fork PRs `ready` until a trusted maintainer links or approves the session.
- Treat fork PRs as `risky` when provenance is unclear.

For MVP, AgentRoom can support same-repo PRs first and classify fork PRs as `risky` with a clear explanation.

## Idempotency and updates

GitHub webhooks can be retried and delivered out of order. AgentRoom should:

- store GitHub delivery IDs
- deduplicate webhook events
- fetch current PR state before publishing readiness
- update the sticky PR comment rather than creating new comments
- update the readiness check run for the current head SHA

## Merge authority

The MVP should not merge PRs. Humans merge in GitHub after branch protection and review requirements pass.

Future merge automation must require a separate design review and should never grant agent runners direct merge authority.

## Open questions

- Should AgentRoom use check runs only, or also commit statuses for older workflows?
- What exact PR comment marker should identify the sticky evidence packet?
- Should `/ar refresh` and `/ar link` commands be supported through PR comments in Milestone 1?
- Should fork PR support be deferred entirely or included as `risky` read-only mode?
