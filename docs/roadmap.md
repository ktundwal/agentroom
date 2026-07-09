# Roadmap

## Milestone 0: Docs-first pilot

- Product brief
- Architecture direction
- Security model
- Manual Chatto repo room workflow
- Manual agent plan template
- Manual PR evidence packet template

## Milestone 1: Chatto repo room and GitHub evidence packet

- Experimental Chatto connector
- GitHub App scaffold
- GitHub App permissions from `docs/github-app-design.md`
- Repo room binding
- PR thread creation
- Approved plan storage
- PR evidence packet generation
- CI status aggregation
- `AgentRoom readiness` check run

## Milestone 2: Readiness and policy gates

- Required plan gate
- Required CI pass gate
- Required review gate
- Sensitive-file approval gate
- GitHub status check integration
- Readiness states: `needs review`, `blocked`, `risky`, `ready`

## Milestone 3: Agent event ingestion

- Generic agent event webhook
- Generic local event adapter from `docs/agent-adapter-contract.md`
- Event ingestion
- Chatto status updates
- Realtime Chatto event listener or timeline polling fallback

## Milestone 4: Self-hosted runtime

- Docker Compose deployment
- Local database
- Chatto connection configuration
- GitHub webhook receiver
- Admin configuration
- Backup and restore guidance

## Milestone 5: Team adoption

- Repository-level policy config
- Organization-level defaults
- Evidence packet customization
- Audit export
- Public dogfooding examples
