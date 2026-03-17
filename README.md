# Palmer Action

Run AI-powered tests with [Palmer](https://palmer.generalvolition.com).

## Usage

### Minimal (recommended)

Set `PALMER_API_KEY` as an org or repo secret, then:

```yaml
name: Palmer Tests
on:
  push:
    branches: [main]
  schedule:
    - cron: '0 6 * * *' # daily at 6am

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: tabtabtabai/palmer-action@v1
        with:
          prompt: "Run all e2e tests and verify the checkout flow"
          browser-enabled: true
```

### With secret forwarding

Pass secrets to the test environment using the `PALMER_SECRET_` prefix:

```yaml
- uses: tabtabtabai/palmer-action@v1
  with:
    prompt: "Run all e2e tests"
    browser-enabled: true
  env:
    PALMER_SECRET_OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
    PALMER_SECRET_DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

### Private repositories

If Palmer needs to clone a private GitHub repository, forward a GitHub token so
ripatorium can authenticate the sandbox clone step:

```yaml
name: Palmer Tests
on:
  pull_request:

permissions:
  contents: read
  issues: write

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: tabtabtabai/palmer-action@v1
        with:
          prompt: "Verify the dashboard loads correctly"
          browser-enabled: true
          branch: ${{ github.event.pull_request.head.ref }}
          github-token: ${{ github.token }}
        env:
          PALMER_API_KEY: ${{ secrets.PALMER_API_KEY }}
          PALMER_SECRET_GITHUB_TOKEN: ${{ github.token }}
          # Optional when self-hosting Palmer / ripatorium:
          PALMER_API_URL: https://api.generalvolition.com
          PALMER_APP_URL: https://palmer.generalvolition.com
```

This pattern is intended for trusted GitHub Actions runs where the forwarded
token has `contents: read` access to the target repository under test.
If you want Palmer to leave a PR comment with the run link, also grant
`issues: write`.

If your ripatorium backend also clones a private Palmer source repository,
configure a backend `GH_TOKEN` separately in your server environment. That
server-side token is not forwarded by the action.

### Explicit API key

Override the default `PALMER_API_KEY` env var:

```yaml
- uses: tabtabtabai/palmer-action@v1
  with:
    api-key: ${{ secrets.CUSTOM_PALMER_KEY }}
    prompt: "Run smoke tests"
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `api-key` | No | `PALMER_API_KEY` env var | Palmer API key |
| `prompt` | **Yes** | — | Test prompt describing what to test |
| `browser-enabled` | No | `false` | Enable browser for tests |
| `branch` | No | Current branch | Branch to test |
| `wait` | No | `true` | Wait for run to complete |
| `timeout` | No | `600` | Timeout in seconds when waiting |
| `github-token` | No | — | GitHub token used to comment the Palmer run link on pull requests/issues |

## Outputs

| Output | Description |
|--------|-------------|
| `run-id` | Palmer run ID |
| `status` | Run status (`passed`, `failed`, `error`) |
| `summary` | Run summary |
| `dashboard-url` | Link to the Palmer dashboard for this run |

## Secret forwarding

Any environment variable prefixed with `PALMER_SECRET_` is made available to the
Palmer run. The prefix is stripped:

- `PALMER_SECRET_OPENAI_API_KEY` → `OPENAI_API_KEY` in the sandbox
- `PALMER_SECRET_DATABASE_URL` → `DATABASE_URL` in the sandbox
- `PALMER_SECRET_GITHUB_TOKEN` → `GITHUB_TOKEN` for private GitHub repo cloning

This uses GitHub's native secret management — Palmer never stores your secrets.

## Run link visibility

The action always writes the Palmer dashboard URL into:

- the workflow logs as a notice
- the GitHub Actions step summary

If you also want that link posted back onto the pull request, pass
`github-token: ${{ github.token }}` and grant `issues: write`.
