[![Marketplace](https://img.shields.io/badge/marketplace-Jiminy%20Evaluate-blue?logo=github)](https://github.com/marketplace/actions/jiminy-evaluate)
[![License](https://img.shields.io/badge/license-Apache%202.0-blue)](./LICENSE)
[![Release](https://img.shields.io/github/v/release/christianbelnavis4-chelnok/jiminy-action)](https://github.com/christianbelnavis4-chelnok/jiminy-action/releases)

# Jiminy Evaluate

A GitHub Action that gates CI on an independent accountability check for your AI agent: submit its decision trace to Jiminy on every PR, and fail the build on a bad verdict.

## The problem

An AI agent that calls tools, escalates decisions, and acts on behalf of a user can quietly drift out of scope: a weather bot that starts changing account settings, a support agent that leaks one customer's data into another's answer. Unit tests don't catch this, because the failure mode isn't a wrong return value, it's a wrong *decision*. By the time it's visible in production, it's already shipped.

## What this does

This action evaluates one or more `DecisionTrace` JSON fixtures (a structured log of what your agent did and why) against Jiminy's independent judge, which scores six accountability criteria (scope adherence, tool authorisation, escalation judgement, output traceability, data boundary, differential treatment) and returns a verdict: `approved`, `flagged`, or `rejected`. Set `fail-on: rejected` (or `flagged`, for a stricter gate) and a bad verdict fails the build, the same way a failing test would.

In cross-model calibration against a second, independent judge model, Jiminy's verdicts held **90.9% cross-family concordance**: the two judges agreed on the overall approved/flagged/rejected call in all but a handful of edge cases across a reference trace set.

## Usage

```yaml
name: Jiminy agent accountability check

on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read
  pull-requests: write   # lets the action post a PR comment with the verdict table

jobs:
  jiminy-evaluate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Evaluate agent traces
        uses: christianbelnavis4-chelnok/jiminy-action@v1
        with:
          api-key: ${{ secrets.JIMINY_API_KEY }}
          base-url: ${{ vars.JIMINY_BASE_URL }}
          traces-glob: 'traces/*.json'
          fail-on: rejected   # use 'flagged' to also fail on a flagged verdict
```

Get an API key with no invite code needed via `jiminy auth login`. See the [SDK quickstart](https://github.com/christianbelnavis4-chelnok/jiminy-sdk#try-it-in-30-seconds).

### Two ways to feed it traces

- **Bring your own traces.** Check `traces/*.json` fixtures into your repo (hand-written, or captured once from a real run) and this action evaluates them on every PR, as above.
- **Auto-capture from a live run** (LangChain only for now). Have CI run your agent itself and capture the trace automatically, then hand off to this action, with no fixtures to maintain by hand. See the [jiminy-sdk auto-capture guide](https://github.com/christianbelnavis4-chelnok/jiminy-sdk#cli-style-ci-gating) and worked example.

### Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `api-key` | yes | none | Jiminy `X-API-Key` (store as a repo/org secret) |
| `base-url` | yes | none | Jiminy API base URL |
| `traces-glob` | no | `traces/*.json` | Glob for DecisionTrace JSON fixtures |
| `fail-on` | no | `rejected` | Minimum verdict severity that fails the build: `rejected` or `flagged` |
| `calibrate` | no | `false` | Use calibration mode: no persistence, doesn't count against quota |
| `fail-on-regression` | no | `false` | Advisory-only: also replay active fixtures and warn on regressions |
| `comment-on-pr` | no | `true` | Post/update a PR comment with the verdict table |
| `github-token` | no | `${{ github.token }}` | Token used to post the PR comment |

### Outputs

| Output | Description |
|---|---|
| `exit-code` | `0` if every trace passed the `fail-on` threshold, `1` otherwise |

## Principles

- **Independent judge.** The evaluation runs against a hosted API you don't control the scoring of, the same posture as a compliance audit, not a self-graded test.
- **Fails closed.** A trace that can't be submitted (network error, bad API key) fails the build rather than being silently skipped.
- **No trace, no vendor lock-in on format.** `DecisionTrace` is a plain JSON schema: build it by hand, from an existing log, or via one of the [framework adapters](https://github.com/christianbelnavis4-chelnok/jiminy-sdk#using-the-sdk-in-your-own-code) (LangChain, CrewAI, LangSmith, OpenAI Agents, OpenTelemetry).

## Contributing

See [jiminy-sdk](https://github.com/christianbelnavis4-chelnok/jiminy-sdk): this repo hosts the Action packaging only; the underlying client, adapters, and CI scripts live there.

## License

Apache 2.0. See [LICENSE](./LICENSE).
