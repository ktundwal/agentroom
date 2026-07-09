# Configuration

AgentRoom uses a TOML config file plus environment variables for secrets.

## File location

Default config path:

```text
./agentroom.toml
```

Overrides:

```sh
agentroom server --config /path/to/agentroom.toml
AGENTROOM_CONFIG=/path/to/agentroom.toml agentroom server
```

`agentroom init` should create `./agentroom.toml` by default and should never write secret values into the file.

See [`github-app-setup.md`](github-app-setup.md) for creating the GitHub App values referenced by the `[github]` section.

## Format

Use TOML because it is readable, supports comments, and is common for self-hosted infrastructure tools.

## Example

```toml
[server]
listen_addr = "127.0.0.1:8080"
public_url = "https://agentroom.example.com"

[storage]
sqlite_path = "./data/agentroom.db"

[github]
app_id = 123456
app_slug = "agentroom"
private_key_pem_path = "./secrets/github-app.pem"
webhook_secret_env = "AGENTROOM_GITHUB_WEBHOOK_SECRET"

[chatto]
base_url = "https://chat.example.com"
access_token_env = "AGENTROOM_CHATTO_ACCESS_TOKEN"

[agent_events]
http_enabled = true
shared_secret_env = "AGENTROOM_AGENT_EVENT_SECRET"
file_watch_enabled = true
file_path = ".agentroom/events.ndjson"

[[repos]]
repo = "ktundwal/agentroom"
chatto_room_id = "room_agentroom"
required_checks = ["test", "lint"]
sensitive_paths = [
  ".github/workflows/**",
  "docs/security.md"
]
```

## Required fields

| Field | Required | Purpose |
| --- | --- | --- |
| `server.listen_addr` | yes | Local bind address for HTTP server. |
| `server.public_url` | yes | Public URL GitHub can call for webhooks. |
| `storage.sqlite_path` | yes | SQLite database path. |
| `github.app_id` | yes | GitHub App ID. |
| `github.private_key_pem_path` or `github.private_key_pem_env` | yes | GitHub App private key source. |
| `github.webhook_secret_env` | yes | Environment variable containing webhook secret. |
| `chatto.base_url` | yes | Chatto server URL. |
| `chatto.access_token_env` | yes | Environment variable containing dedicated Chatto user token. |
| `repos[].repo` | yes | GitHub repository binding. |
| `repos[].chatto_room_id` | yes | Chatto room for that repository. |

## Optional fields

| Field | Default | Purpose |
| --- | --- | --- |
| `github.app_slug` | empty | Human-readable GitHub App slug. |
| `agent_events.http_enabled` | `true` | Enable `POST /api/agent-events`. |
| `agent_events.shared_secret_env` | empty | Optional shared secret for event submissions. |
| `agent_events.file_watch_enabled` | `true` | Enable local NDJSON file watcher. |
| `agent_events.file_path` | `.agentroom/events.ndjson` | Local event stream path. |
| `repos[].required_checks` | `[]` | Required check names for readiness. |
| `repos[].sensitive_paths` | `[]` | Glob paths requiring explicit approval. |

## Secret handling

Config files should reference secrets indirectly:

```toml
webhook_secret_env = "AGENTROOM_GITHUB_WEBHOOK_SECRET"
access_token_env = "AGENTROOM_CHATTO_ACCESS_TOKEN"
```

AgentRoom reads the named environment variables at startup. `agentroom doctor` should fail when required secret env vars are missing.

## `agentroom init`

`agentroom init` should:

1. Write a commented `agentroom.toml` template.
2. Create `.agentroom/events.ndjson` if file watching is enabled.
3. Create the SQLite parent directory.
4. Print the environment variables the user must set.
5. Print the next step to create or verify the GitHub App using [`github-app-setup.md`](github-app-setup.md).
6. Refuse to overwrite an existing config unless `--force` is passed.

It should not:

- create GitHub Apps
- create Chatto users
- write plaintext tokens
- start Chatto
- start AgentRoom

## Startup validation

`agentroom server` should validate:

- config parses as TOML
- required fields are present
- secret environment variables are set
- SQLite path is writable
- GitHub private key file exists or env var is set
- at least one repo binding exists
- no duplicate `repos[].repo` entries exist

Invalid config should fail fast before starting webhook or connector workers.
