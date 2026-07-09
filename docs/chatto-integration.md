# Chatto Integration

AgentRoom uses Chatto as the human/agent collaboration surface, but Chatto should be treated as an external dependency with an evolving integration surface.

## Current assumption

The MVP should use **bring your own Chatto**:

1. The user installs or already runs Chatto.
2. The user creates a dedicated Chatto account for AgentRoom.
3. The user adds that account to the repo rooms AgentRoom should supervise.
4. AgentRoom connects with Chatto URL + user credentials.
5. AgentRoom posts status and evidence through public Chatto APIs.

AgentRoom should not install, upgrade, or operate Chatto in the first MVP.

## Why not bundle Chatto first?

Bundling Chatto would make trials easier, but it would also expand AgentRoom into deployment and operations before the merge-confidence loop is proven.

The first milestone should prove this narrower claim:

> A Chatto room plus GitHub evidence makes agent-authored PRs easier to decide on.

## Integration surface

Chatto currently has a public protobuf/ConnectRPC API and realtime WebSocket direction. Useful surfaces include:

- `MessageService.CreateMessage` for posting room messages and thread replies as the current user
- `MessageService.AddReaction` and `RemoveReaction` for reaction-based signals
- `MessageService.GetMessage` and `BatchGetMessages` for message reads
- `/api/realtime` for visible event delivery to an authenticated user

Chatto does not currently appear to expose a dedicated bot API or outgoing webhook system. AgentRoom should therefore call this integration an **experimental Chatto connector**, not a first-class bot integration.

## Authentication model

The practical starting point is a dedicated Chatto user account:

- The account represents AgentRoom inside Chatto.
- It should be a member only of rooms AgentRoom supervises.
- It should use the least privileges needed to post status, read relevant threads, and add reactions.
- It should not be a server owner or admin for normal operation.

The Chatto Operator API may be useful for local bootstrap, such as creating the dedicated user during a self-hosted demo. It is Unix-socket-only and root-equivalent, so it must not become a normal runtime dependency.

## Human approval capture

Until Chatto exposes outgoing webhooks or a formal bot interaction API, AgentRoom has two options:

1. **Realtime listener:** connect to `/api/realtime` as the dedicated Chatto user and watch visible room/thread events.
2. **Polling fallback:** periodically read the relevant room or thread timeline for structured approval messages or reactions.

The realtime listener is the preferred product experience. Polling is simpler and should remain a fallback for early compatibility.

## Chat-agnostic boundary

AgentRoom should keep a chat-surface adapter boundary even if Chatto is the default and differentiating integration.

The boundary should support:

- create or bind repo room
- create or bind PR thread
- post status update
- post approval request
- read human decision
- link back to GitHub PR evidence

This keeps AgentRoom from depending on unofficial Chatto internals and leaves room for future Slack, Discord, Matrix, or GitHub-only adapters if the market demands them.

## Risks

- Chatto is pre-1.0 and APIs may change.
- Dedicated user accounts are not the same as formal bot identities.
- Realtime WebSocket behavior may not cover every interaction AgentRoom needs.
- Polling can be slower and noisier than webhooks.
- Operator API access is powerful and local-only.
- Chatto adoption may be friction for teams already standardized on Slack or GitHub.

These risks are acceptable only if the Chatto-native experience materially improves maintainer decisions on agent-authored PRs.
