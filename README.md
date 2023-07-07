# `replayio/action-cypress`

Record your [Cypress](https://cypress.io) tests with [Replay](https://replay.io) and automatically upload replays of failed tests.

**Use with [`@replayio/cypress`](https://github.com/replayio/replay-cli/tree/main/packages/cypress)**

## Overview

Using `action-cypress` in your GitHub Actions workflow is the easiest way to get started recording your Cypress tests in CI.

The action:

- Ensures Replay Browsers are installed on the CI machine
- Runs your tests with the browser specified
- Uploads replays from the test run to the team for the given API key
- Comments on the initiating pull request with links to the test run and replays

## Usage

First, add `@replayio/cypress` and configure using instructions in the [Recording Automated Tests guide](https://docs.replay.io/docs/configuring-cypress-30fd38c1ed8047a2be82ae436e0bbb15).

Then:

1. Log into [app.replay.io](https://app.replay.io)
2. Create a [Team API key](https://docs.replay.io/docs/setting-up-a-team-f5bd9ee853814d6f84e23fb535066199#4913df9eb7384a94a23ccbf335189370) (Personal API keys can be used, but have a limit of 10 recordings)
3. Store the API key as a [GitHub Repository Secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-a-repository) named `RECORD_REPLAY_API_KEY`
4. Add the configuration below to your existing workflow (or start a new one with the [complete example](#complete-workflow-example) below)
5. Install the [Replay GitHub App](https://github.com/apps/replay-io) to add comments to your Pull Requests with links to replays of your tests

```yaml
- uses: replayio/action-cypress@v0.3.1
  with:
    api-key: ${{ secrets.RECORD_REPLAY_API_KEY }}
    browser: "replay-firefox"
```

If no browser is passed, the Cypress default Electron is used.

> **Note:**
> If you're using `@replayio/cypress` version 0.4.15 or earlier, use `Replay Firefox` or `Replay Chromium` for the browser name

## Arguments

| Required           | Name                | Description                                                              | Default           |
| ------------------ | ------------------- | ------------------------------------------------------------------------ | ----------------- |
| :white_check_mark: | `api-key`           | The Replay API Key used to upload recordings                             |
| &nbsp;             | `browser`           | The Replay browser to use (either `replay-firefox` or `replay-chromium`) |
| &nbsp;             | `command`           | The command to run your cypress tests                                    | `npx cypress run` |
| &nbsp;             | `public`            | When true, make replays public on upload                                 | `false`           |
| &nbsp;             | `upload-all`        | Upload all recordings instead of only recordings of failed tests         | `false`           |
| &nbsp;             | `working-directory` | The relative working directory for the app                               | `.`               |
| &nbsp;             | `test-run-id`       | Uses the provided UUID to group the tests instead of generating its own. | `.`               |

> **Note:** This action appends arguments to your `command` to configure a
> custom reporter. If you're using a command like `npm run cy:run` to run your
> tests, you may need to include `--` at the end to allow those arguments to
> pass to Cypress.

## Complete Workflow Example

```yaml
# .github/workflow/ci.yml
name: Cypress Tests
on:
  pull_request:

jobs:
  test-run-id:
    runs-on: ubuntu-latest
    outputs:
      testRunId: ${{ steps.testRunId.outputs.testRunId }}
    steps:
      - id: testRunId
        run: echo testRunId=$(npx uuid) >> "$GITHUB_OUTPUT"
  test:
    name: Cypress tests
    runs-on: ubuntu-latest
    needs: test-run-id
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Setup node
        uses: actions/setup-node@v3
        with:
          node-version: 16

      - name: Install dependencies
        uses: bahmutov/npm-install@v1

      - uses: actions/cache@v2
        with:
          path: ~/.cache/Cypress
          key: my-cache-${{ runner.os }}-${{ hashFiles('package-lock.json') }}

      - uses: replayio/action-cypress@v0.3.1
        with:
          # An API key (usually a Team API Key) to use to upload replays.
          # Configure this via GitHub repo settings.
          api-key: ${{ secrets.RECORD_REPLAY_API_KEY }}
          # The Replay browser to use: 'replay-firefox' or 'replay-chromium'
          browser: "replay-chromium"
          # An optional command to run your tests.
          command: npm run test:e2e -- --
          # When true, replays will be accessible to anyone with the link.
          # This is useful for open source projects that want to collaborate
          # with external users.
          public: true
          # When true, replays of both passed and failed tests are uploaded
          upload-all: true
          # An optional working directory which is useful if the project being
          # tested resides in a subdirectory of the repository
          working-directory: .
          # An optional UUID used to group tests in replay which may
          # be useful when running across runners or in a matrix
          test-run-id: ${{ needs.test-run-id.outputs.testRunId }}
```
