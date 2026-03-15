# Palmer Action

Run AI-powered tests with [Palmer](https://usepalmer.ai).

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

## Outputs

| Output | Description |
|--------|-------------|
| `run-id` | Palmer run ID |
| `status` | Run status (`passed`, `failed`, `error`) |
| `summary` | Run summary |
| `dashboard-url` | Link to the Palmer dashboard for this run |

## Secret forwarding

Any environment variable prefixed with `PALMER_SECRET_` is forwarded to the test environment. The prefix is stripped:

- `PALMER_SECRET_OPENAI_API_KEY` → `OPENAI_API_KEY` in the sandbox
- `PALMER_SECRET_DATABASE_URL` → `DATABASE_URL` in the sandbox

This uses GitHub's native secret management — Palmer never stores your secrets.
