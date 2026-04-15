# CueAPI GitHub Action

Schedule and verify AI agent jobs from your GitHub Actions workflows using [CueAPI](https://cueapi.ai) — the open-source execution accountability primitive for AI agents.

CueAPI lets you schedule agent work on a cue, require evidence-backed outcome reports, and gate execution with write-once verification — making silent agent failure impossible. This action wraps the [`cueapi` Python CLI](https://pypi.org/project/cueapi/) so your CI can create, inspect, and manage cues without bespoke scripts.

## Why use this action

- **Schedule agent runs from CI.** Promote a cue on every release. Pause a cue when a branch is deleted. Rotate payloads on a schedule.
- **Works with any agent runtime.** Claude Code, Codex, Gemini CLI, or your own custom worker — CueAPI is transport-agnostic.
- **Self-hosted or hosted.** Point at `api.cueapi.ai` or your own CueAPI instance via `CUEAPI_API_KEY` / base URL config.
- **Composable.** Compose with any other action — use the `result` output to gate downstream steps, post comments, or trigger Slack notifications.

## Quick start

Store your CueAPI API key as a repository secret named `CUEAPI_API_KEY`, then:

```yaml
name: Schedule daily agent run
on:
  push:
    branches: [main]

jobs:
  create-cue:
    runs-on: ubuntu-latest
    steps:
      - name: Create a daily cue
        uses: cueapi/cueapi-action@v1
        env:
          CUEAPI_API_KEY: ${{ secrets.CUEAPI_API_KEY }}
        with:
          command: create
          name: "daily-report-agent"
          cron: "0 9 * * *"
          timezone: "America/New_York"
          url: "https://my-agent.example.com/run"
          payload: '{"task": "generate_daily_report"}'
          description: "Kicks off the daily report agent at 9am ET"
```

## Supported commands

| Command  | Purpose                                      |
|----------|----------------------------------------------|
| `create` | Create a new cue (recurring or one-time)     |
| `list`   | List cues, optionally filtered by status     |
| `get`    | Fetch details for a specific cue by ID       |
| `update` | Update name, cron, url, payload, description |
| `delete` | Delete a cue                                 |
| `pause`  | Pause a cue                                  |
| `resume` | Resume a paused cue                          |
| `whoami` | Print authenticated identity                 |
| `usage`  | Show current usage stats                     |

## Inputs

| Name          | Required | Description                                                        |
|---------------|----------|--------------------------------------------------------------------|
| `command`     | yes      | CueAPI CLI command to run                                          |
| `name`        | no       | Cue name (create/update)                                           |
| `cron`        | no       | Cron expression, e.g. `0 9 * * *` (create/update)                  |
| `at`          | no       | ISO timestamp for one-time cue (create)                            |
| `url`         | no       | Callback URL to fire (create/update)                               |
| `method`      | no       | HTTP method (default `POST`)                                       |
| `timezone`    | no       | IANA timezone, default `UTC`                                       |
| `payload`     | no       | JSON payload string                                                |
| `description` | no       | Human-readable cue description                                     |
| `worker`      | no       | Set `"true"` to use worker transport (no public URL)               |
| `cue-id`      | no       | Target cue ID (get/update/delete/pause/resume)                     |
| `status`      | no       | Status filter for `list` (`active` / `paused`)                     |
| `limit`       | no       | Max results for `list`                                             |
| `api-key`     | no       | Override API key (prefer `CUEAPI_API_KEY` env from a secret)       |
| `cli-version` | no       | Pin a specific `cueapi` CLI version (default: latest)              |

## Outputs

| Name     | Description                                  |
|----------|----------------------------------------------|
| `result` | Raw stdout from the `cueapi` CLI invocation  |

## Example: full release pipeline

```yaml
name: Release
on:
  release:
    types: [published]

jobs:
  schedule-followup-agent:
    runs-on: ubuntu-latest
    steps:
      - name: Schedule post-release verification agent
        id: cue
        uses: cueapi/cueapi-action@v1
        env:
          CUEAPI_API_KEY: ${{ secrets.CUEAPI_API_KEY }}
        with:
          command: create
          name: "post-release-${{ github.event.release.tag_name }}"
          at: "${{ github.event.release.published_at }}"
          url: "https://agent.example.com/verify-release"
          payload: '{"tag": "${{ github.event.release.tag_name }}"}'
          description: "One-shot verification after ${{ github.event.release.tag_name }} ships"

      - name: Surface the cue ID in the summary
        run: |
          echo "### CueAPI job scheduled" >> $GITHUB_STEP_SUMMARY
          echo '```' >> $GITHUB_STEP_SUMMARY
          echo "${{ steps.cue.outputs.result }}" >> $GITHUB_STEP_SUMMARY
          echo '```' >> $GITHUB_STEP_SUMMARY
```

## Authentication

Pass your API key via the `CUEAPI_API_KEY` environment variable (recommended) or the `api-key` input. Generate a key at [cueapi.ai](https://cueapi.ai) or from your self-hosted dashboard.

```yaml
env:
  CUEAPI_API_KEY: ${{ secrets.CUEAPI_API_KEY }}
```

## Self-hosted CueAPI

Point the CLI at a self-hosted instance by setting `CUEAPI_BASE_URL`:

```yaml
env:
  CUEAPI_API_KEY: ${{ secrets.CUEAPI_API_KEY }}
  CUEAPI_BASE_URL: https://cueapi.internal.example.com
```

## Links

- **Homepage:** https://cueapi.ai
- **Docs:** https://docs.cueapi.ai
- **Core repo:** https://github.com/cueapi/cueapi-core
- **Python CLI:** https://pypi.org/project/cueapi/
- **Issues:** https://github.com/cueapi/cueapi-action/issues

## License

Apache-2.0 © Vector Apps
