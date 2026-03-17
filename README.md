# Inflectra/rapiselauncher-node-action

A GitHub Action to install the cross-platform (Node.js) Rapise engine and run Rapise test sets stored in SpiraTest using RapiseLauncher.

Runs on both `ubuntu-latest` and `windows-latest` runners.

## Prerequisites

- A Spira instance (SpiraTest / SpiraTeam / SpiraPlan) with an active user account.
- Test Sets in Spira configured to run automated Rapise tests.
- An Automation Host record created in Spira (e.g., named `GHA`).

## Storing Spira Credentials

Store your Spira API Key as a GitHub Secret so it is never exposed in workflow files:

1. In Spira, go to **My Profile** and copy your **RSS Token** (this is your API Key).
2. In your GitHub repository, go to **Settings > Secrets and variables > Actions**.
3. Click **New repository secret**, set the name to `SPIRA_API_KEY`, paste your token, and save.

## Quick Start

```yaml
- uses: Inflectra/rapiselauncher-node-action@v1
  with:
    spira_url: 'https://myserver.spiraservice.net/9/TestSet/925.aspx'
    spira_username: 'myuser'
    spira_api_key: ${{ secrets.SPIRA_API_KEY }}
    spira_automation_host: GHA
    rapise_params: |
      g_browserLibrary=Selenium - ChromeHeadless
```

## Usage

### Running with RepositoryConnection.xml

If you already have a `RepositoryConnection.xml` file checked into your repository (with Spira server, user, and password pre-configured), point the action to it:

```yaml
name: Run Rapise Tests (Config File)

on:
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: Inflectra/rapiselauncher-node-action@v1
        with:
          spira_config: '${{ github.workspace }}/RepositoryConnection.xml'
          spira_test_set_id: '925,1266'
          rapise_params: |
            g_browserLibrary=Selenium - ChromeHeadless
```

When `spira_config` is provided, the `spira_url`, `spira_username`, `spira_api_key`, and `spira_automation_host` inputs are not needed — everything is read from the XML file.

### Running with Test Set URL and Credentials

Pass Spira connection details directly. The `spira_url` input supports two forms:

- **Short form**: `https://myserver.spiraservice.net/` — requires `spira_project_id` and `spira_test_set_id` to be set separately.
- **Full form**: `https://myserver.spiraservice.net/9/TestSet/925.aspx` — the project ID and test set ID are extracted automatically from the URL.

```yaml
name: Run Rapise Tests (Credentials)

on:
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: Inflectra/rapiselauncher-node-action@v1
        with:
          spira_url: 'https://myserver.spiraservice.net/9/TestSet/925.aspx'
          spira_username: 'myuser'
          spira_api_key: ${{ secrets.SPIRA_API_KEY }}
          spira_automation_host: GHA
          rapise_params: |
            g_browserLibrary=Selenium - ChromeHeadless
```

### Running on Multiple Platforms

Use a matrix strategy to run the same tests on both Linux and Windows:

```yaml
name: Run Rapise Tests (Cross-Platform)

on:
  workflow_dispatch:

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
    steps:
      - uses: actions/checkout@v4

      - uses: Inflectra/rapiselauncher-node-action@v1
        with:
          spira_url: 'https://myserver.spiraservice.net/9/TestSet/925.aspx'
          spira_username: 'myuser'
          spira_api_key: ${{ secrets.SPIRA_API_KEY }}
          spira_automation_host: GHA
          rapise_params: |
            g_browserLibrary=Selenium - ChromeHeadless
```

### Passing Additional Parameters

Use the `rapise_params` input to pass any number of `--param` values to RapiseLauncher. Each line becomes a separate `--param` flag:

```yaml
rapise_params: |
  g_browserLibrary=Selenium - ChromeHeadless
  g_verboseLevel=3
```

> **Note:** On headless Linux runners, always pass `g_browserLibrary=Selenium - ChromeHeadless` to ensure browser tests run without a display.

### Setting a Timeout

Use `timeout_minutes` to kill the launcher if it exceeds the specified duration:

```yaml
- uses: Inflectra/rapiselauncher-node-action@v1
  with:
    spira_url: 'https://myserver.spiraservice.net/9/TestSet/925.aspx'
    spira_username: 'myuser'
    spira_api_key: ${{ secrets.SPIRA_API_KEY }}
    timeout_minutes: 10
    rapise_params: |
      g_browserLibrary=Selenium - ChromeHeadless
```

### Specifying Rapise and Node.js Versions

```yaml
- uses: Inflectra/rapiselauncher-node-action@v1
  with:
    rapise_version: '9.0.35.24'
    node_version: '22.x'
    # ... other inputs
```

### Git Root for Spira Tests Stored in Git

If your Rapise tests are stored in a Git repository connected to Spira, the action automatically sets `GITROOT` to `$GITHUB_WORKSPACE`. Override it with `git_root` if needed:

```yaml
- uses: Inflectra/rapiselauncher-node-action@v1
  with:
    git_root: '${{ github.workspace }}/tests'
    # ... other inputs
```

### Artifacts

By default (`upload_artifacts: 'true'`), the action collects `.trp`, `.tap`, `.log`, and `.xml` files from the workspace and `~/.rapise/temp`, then uploads them as a `rapise-results-<OS>` artifact. Disable with:

```yaml
upload_artifacts: 'false'
```

## Input Reference

| Input | Default | Description |
|---|---|---|
| `spira_config` | | Path to existing `RepositoryConnection.xml`. When set, URL/user/key inputs are ignored. |
| `spira_url` | | Spira server URL. Short form: `https://server/`. Full form: `https://server/9/TestSet/925.aspx`. |
| `spira_username` | | Spira username. |
| `spira_api_key` | | Spira API key (RSS Token). |
| `spira_project_id` | | Spira Project ID. Optional if using full-form URL. |
| `spira_test_set_id` | | Test Set ID(s), comma-separated (e.g. `925,1266`). |
| `spira_automation_host` | *(hostname)* | Automation Host Token. Defaults to runner hostname. |
| `install_rapise` | `true` | Whether to install Rapise. |
| `rapise_version` | `9.0.35.24` | Rapise version to install. |
| `node_version` | `22.x` | Node.js version to set up. |
| `rapise_params` | | Additional `--param` values, one per line. |
| `timeout_minutes` | `0` | Execution timeout in minutes. `0` = no timeout. |
| `git_root` | | Path to Git project root. Defaults to `$GITHUB_WORKSPACE`. |
| `upload_artifacts` | `true` | Upload execution artifacts after run. |

## Reviewing Results

- **In Spira**: Test execution results are automatically uploaded to your Test Run records.
- **In GitHub**: After the workflow completes, go to the workflow run summary page. Under the **Artifacts** section, download the `rapise-results-<OS>` archive containing logs and reports for troubleshooting.

## See Also

- [Inflectra/rapiselauncher-win-action](https://github.com/Inflectra/rapiselauncher-win-action) — native Windows action with video recording and screen resolution support.
