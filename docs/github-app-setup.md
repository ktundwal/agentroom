# GitHub App Setup

AgentRoom uses a GitHub App to read pull request state, publish readiness checks, and update the sticky evidence comment.

The MVP should use a manually created GitHub App. `agentroom init` writes local config, but it does not create GitHub Apps, download private keys, or install the App for you.

## Prerequisites

- A public URL that reaches AgentRoom's webhook endpoint.
- A GitHub account or organization where you can create GitHub Apps.
- Permission to install a GitHub App on the repositories AgentRoom will supervise.

The webhook URL is:

```text
<server.public_url>/webhooks/github
```

Example:

```text
https://agentroom.example.com/webhooks/github
```

## Create the GitHub App

1. Open GitHub **Settings**.
2. Go to **Developer settings** -> **GitHub Apps**.
3. Choose **New GitHub App**.
4. Set a clear name, such as `AgentRoom`.
5. Set **Homepage URL** to the AgentRoom public URL or repository URL.
6. Set **Webhook URL** to `<server.public_url>/webhooks/github`.
7. Generate a strong webhook secret and save it in a local secret store.
8. Leave user authorization, OAuth callback URLs, and device flow disabled unless a future feature requires them.

## Permissions

Configure repository permissions:

| Permission | Level |
| --- | --- |
| Contents | Read-only |
| Issues | Read-only |
| Pull requests | Read and write |
| Checks | Read and write |
| Commit statuses | Read-only |
| Metadata | Read-only |

Do not grant write access to contents, deployments, environments, secrets, or administration for the MVP.

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

`issue_comment` is needed only for PR comments such as future `/ar link` or `/ar refresh` commands, but enabling it in the MVP avoids reinstalling the App when those commands land.

## Private key

After creating the App:

1. Open the App settings page.
2. Copy the **App ID**.
3. Generate a private key.
4. Download the `.pem` file.
5. Store it outside the repository, for example:

   ```text
   ./secrets/github-app.pem
   ```

Do not commit the private key.

## Install the App

1. In the GitHub App settings, choose **Install App**.
2. Install it on the account or organization that owns the repository.
3. Prefer **Only select repositories** for the MVP.
4. Select each repository AgentRoom should supervise.

AgentRoom should discover the installation for configured repositories during `agentroom doctor` or startup. Installation webhooks should keep that state fresh when repositories are added or removed.

## Configure AgentRoom

Add the GitHub values to `agentroom.toml`:

```toml
[server]
public_url = "https://agentroom.example.com"

[github]
app_id = 123456
app_slug = "agentroom"
private_key_pem_path = "./secrets/github-app.pem"
webhook_secret_env = "AGENTROOM_GITHUB_WEBHOOK_SECRET"
```

Set the webhook secret through the environment:

```sh
export AGENTROOM_GITHUB_WEBHOOK_SECRET="generated-webhook-secret"
```

For each repo, bind the GitHub repo to its Chatto room:

```toml
[[repos]]
repo = "ktundwal/agentroom"
chatto_room_id = "room_agentroom"
required_checks = ["test", "lint"]
```

## Validate setup

`agentroom doctor` should verify:

- `github.app_id` is present
- the private key file exists and parses
- the webhook secret env var is set
- the App can authenticate to GitHub
- each configured repository has the App installed
- required permissions are available
- the webhook URL matches `<server.public_url>/webhooks/github`

If any check fails, AgentRoom should fail closed before starting webhook or evidence-publishing workers.
