# Security and Trust Model

AgentDeck is intended for teams whose coding agents may see private source code, tickets, logs, prompts, and review context. The default posture should be private, least-privilege, and human-supervised.

## Principles

- Self-host by default.
- GitHub remains the source of truth for code review and merge.
- Agents get the minimum permissions needed for a task.
- Humans approve plans and risky actions.
- Every important event is recorded.
- Evidence is summarized, but raw sensitive logs should not be sprayed into PR comments.

## Human approval gates

The first approval gates should be:

1. Plan approval before code changes.
2. Risk approval before touching sensitive files.
3. Merge recommendation only after CI and review checks pass.

Sensitive file examples:

- authentication and authorization code
- payment or billing code
- production deployment configuration
- database migrations
- secrets management
- encryption or key-handling code

## Audit trail

AgentDeck should record:

- who started the session
- which issue or PR triggered it
- which agent adapter ran
- which plan was approved
- which branch and commits were produced
- CI status
- review status
- approval decisions
- evidence packet versions

## Secret handling

AgentDeck should not store provider API keys, GitHub tokens, or repository secrets in plaintext. The first implementation should document where credentials live and how they are scoped before supporting production use.

## Failure modes to design for

- agent goes off-plan
- agent edits sensitive files without approval
- two agents edit the same files
- branch becomes stale
- CI is flaky or incomplete
- webhook delivery is retried
- PR is force-pushed
- evidence packet is outdated
- human approval times out
- untrusted fork opens a PR
