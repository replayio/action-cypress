# `replayio/action-cypress`

> Record your failed [Cypress](https://cypress.io) tests with [Replay](https://replay.io)

## Usage

1. Log into [app.replay.io](https://app.replay.io)
2. Create a [Team API key](https://docs.replay.io/docs/setting-up-a-team-f5bd9ee853814d6f84e23fb535066199#4913df9eb7384a94a23ccbf335189370) (Personal API keys can be used, but have a limit of 10 recordings)
3. Store the API key as a [GitHub Repository Secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-a-repository) named `RECORD_REPLAY_API_KEY`
4. Add the configuration below to your existing workflow (or start a new one with the [complete example](#complete-workflow-example) below)

```yaml
- uses: replayio/action-cypress@v0.1.0
  with:
    apiKey: ${{ secrets.RECORD_REPLAY_API_KEY }}
    issue-number: ${{ github.event.pull_request.number }}
    project: Replay Firefox
```

## Arguments

Required | Name | Description | Default
-------- | ---- | ----------- | -------
:white_check_mark: | `apiKey` | The Replay API Key used to upload recordings
&nbsp; | `browser` | The Replay browser to use (either `Replay Firefox` or `Replay Chromium`)
&nbsp; | `command` | The command to run your cypress tests | `npx cypress run`
&nbsp; | `issue-number` | The number of the pull request to comment with failed test links | 
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

      - uses: replayio/action-cypress@v0.1.0
        with:
          # An API key (usually a Team API Key) to use to upload replays.
          # Configure this via GitHub repo settings.
          apiKey: ${{ secrets.RECORD_REPLAY_API_KEY }}
          # The Replay browser to use: 'Replay Firefox' or 'Replay Chromium'
          browser: Replay Firefox
          # An optional command to run your tests.
          command: npm run test:e2e -- --
          # When set, the action will comment on the PR with links to
          # replays of any failed tests.
          issue-number: ${{ github.event.pull_request.number }}
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