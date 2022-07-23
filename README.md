# `replayio/action-cypress`

Record your [Cypress](https://cypress.io) tests with [Replay](https://replay.io) and automatically upload replays of failed tests.

**Use with [`@replayio/cypress`](https://github.com/replayio/replay-cli/tree/main/packages/cypress)**

## Overview

Using `action-cypress` in your GitHub Actions workflow is the easiest way to get started recording your Cypress tests in CI.

The action:

* Ensures Replay Browsers are installed on the CI machine
* Runs your tests with the browser specified
* Uploads replays from the test run to the team for the given API key
* Comments on the initiating pull request with links to the test run and replays

## Usage

First, add `@replayio/cypress` and configure using instructions in the [Recording Automated Tests guide](https://docs.replay.io/docs/configuring-cypress-30fd38c1ed8047a2be82ae436e0bbb15).

Then: 

1. Log into [app.replay.io](https://app.replay.io)
2. Create a [Team API key](https://docs.replay.io/docs/setting-up-a-team-f5bd9ee853814d6f84e23fb535066199#4913df9eb7384a94a23ccbf335189370) (Personal API keys can be used, but have a limit of 10 recordings)
3. Store the API key as a [GitHub Repository Secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-a-repository) named `RECORD_REPLAY_API_KEY`
4. Add the configuration below to your existing workflow (or start a new one with the [complete example](#complete-workflow-example) below)
5. Install the [Replay GitHub App](https://github.com/apps/replay-io) to add comments to your Pull Requests with links to replays of your tests

```yaml
- uses: replayio/action-cypress@v0.2.0
  with:
    api-key: ${{ secrets.RECORD_REPLAY_API_KEY }}
    browser: 'Replay Firefox'
```

If no browser is passed, the Cypress default Electron is used.

## Arguments

Required | Name | Description | Default
-------- | ---- | ----------- | -------
:white_check_mark: | `api-key` | The Replay API Key used to upload recordings
&nbsp; | `browser` | The Replay browser to use (either `Replay Firefox` or `Replay Chromium`)
&nbsp; | `command` | The command to run your cypress tests | `npx cypress run`
&nbsp; | `public` | When true, make replays public on upload | `false`
&nbsp; | `upload-all` | Upload all recordings instead of only recordings of failed tests | `false`
&nbsp; | `working-directory` | The relative working directory for the app | `.`

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
  test:
    name: Cypress tests
    runs-on: ubuntu-latest
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

      - uses: replayio/action-cypress@v0.2.0
        with:
          # An API key (usually a Team API Key) to use to upload replays.
          # Configure this via GitHub repo settings.
          api-key: ${{ secrets.RECORD_REPLAY_API_KEY }}
          # The Replay browser to use: 'Replay Firefox' or 'Replay Chromium'
          browser: 'Replay Firefox'
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
```
