# Migrating Travis CI Pipelines with Valet

## How to run Valet

Valet can be run locally as a CLI, allowing a user to perform an:

- [audit](../README.md#Audit)
- [migration](../README.md#Migrate)
- [dry-run](../README.md#Dry-run)

Valet will perform a best attempt at full-fidelity transformation of a Travis CI pipeline but each converted workflow should be reviewed and edited as necessary.

### Using environment variables

We recommend maintaining the inputs to Valet with environment variables. To help with this, the distribution files contain a `.env.local.template` file that can be used to generate a `.env.local` file that will be loaded by Valet at run time.

The following are environment variables used by Valet to connect to your Travis CI instance.

- `GITHUB_ACCESS_TOKEN`: The personal access token used to create pull requests with a transformed workflow (requires `repo` and `workflow` scopes).
- `GITHUB_INSTANCE_URL`: The url to the target GitHub instance. (e.g. `https://github.com`)
- `TRAVIS_CI_ACCESS_TOKEN`: The personal access token used to authenticate with your Travis CI instance. Use the `API authentication` token found in the settings of your Travis instance.
- `TRAVIS_CI_ORGANIZATION`: The organization name of your Travis CI instance. For an audit, optionally input multiple organizations (e.g. `--travis_ci_organization "org1" "org2"`)
- `TRAVIS_CI_INSTANCE_URL`: The url to the Travis CI instance (e.g. `https://travis-ci.com`)
- `TRAVIS_CI_SOURCE_GITHUB_ACCESS_TOKEN`: The personal access token used to authenticate with your source GitHub instance (requires `repo` scope).
- `TRAVIS_CI_SOURCE_GITHUB_INSTANCE_URL`: The url to the source GitHub instance. (e.g. `https://github.com`)

These environment variables can be specified in a `.env.local` file that will be loaded by Valet at run time. The distribution archive contains a `.env.local.template` file that can be used to create these files.

### Optional arguments

#### Source file path

This parameter can be set to feed a single source file path into Valet for a dry run or migration. Valet will use the source file specified instead of fetching the pipeline contents from source control.

```bash
valet dry-run travis-ci --output-dir /data/path/to/output/ --travis-ci-repository <repository-name> --source-file-path /data/path/to/.travis.yml
```

#### Allow inactive repositories

This parameter can be set to specify if Valet should include inactive repositories in the audit. Inactive repositories are not included in the audit if this option is not set.

```bash
valet dry-run travis-ci --output-dir /data/path/to/output/ --travis-ci-repository <repository-name> --allow-inactive-repositories
```

#### Config file path

This parameter can be set to feed a list of source files into Valet for an audit. This will alert Valet to ingest the source files instead of fetching pipeline source code from source control.

```bash
valet audit --output-dir /data/path/to/output/ --config-file-path /data/path/to/.travis.yml
```

For auditing a Travis instance using a config file, the config file should be in this format:

```yaml
pipelines:
  - repository_slug: travis-org-name/travis-repo-name
    source_file_path: path/to/.travis.yml
  - repo_identifier: travis-org-name/travis-repo-name
    source_file_path: path/to/.travis.yml
```

Each pipeline name must be unique.

## Supported syntax

The following table shows the type of properties we are currently able to transform. For more details about how an Travis CI pipeline syntax aligns with GitHub Actions, see [Migrating from Travis CI to GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/migrating-from-travis-ci-to-github-actions)

| Travis Pipelines    | Github Actions                     |              Status |
| :------------------ | :--------------------------------- | ------------------: |
| os & dist           | `runners`                          |           Supported |
|                     | `self hosted runners`              |         Unsupported |
| stages              | `jobs`                             |           Supported |
| job                 | `jobs.<job_id>`                    |           Supported |
|                     | `jobs.<job_id>.container`          |         Unsupported |
|                     | `jobs.<job_id>.name`               |           Supported |
| scripts             | `jobs.<job_id>.steps`              |           Supported |
| matrix              | `jobs.<job_id>.strategy`           |           Supported |
|                     | `jobs.<job_id>.strategy.fail-fast` |           Supported |
|                     | `jobs.<job_id>.strategy.matrix`    |           Supported |
| if                  | `jobs.<job_id>.if`                 |           Supported |
| env                 | `env`                              |           Supported |
|                     | `jobs.<job_id>.env`                |           Supported |
|                     | `jobs.<job_id>.steps.env`          |           Supported |
|                     | `jobs.<job_id>.timeout-minutes`    |         Unsupported |
|                     | `on`                               | Partially supported |
|                     | `on.<event_name>.types`            |         Unsupported |
| branches            | `on.<push>.<branches>`             |           Supported |
|                     | `on.<push>.<tags>`                 |         Unsupported |
|                     | `on.<push>.paths`                  |         Unsupported |
| build_pull_requests | `on.<pull_request>`                |           Supported |
|                     | `on.<pull_request>.<branches>`     |         Unsupported |
|                     | `on.<pull_request>.<tags>`         |         Unsupported |
|                     | `on.<pull_request>.paths`          |         Unsupported |
| cron triggers       | `on.schedule`                      |         Unsupported |
|                     | `on.workflow_run`                  |         Unsupported |

The supported Travis CI tasks can be seen [here](plugins/index.md).

### Environment variables

Valet will convert default Travis CI environment variables to the closest equivalent in GitHub Actions using the mapping below:

| Travis CI                     | Github Actions                                                                        |
| :---------------------------- | :------------------------------------------------------------------------------------ |
| `$CONTINUOUS_INTEGRATION`     | `$CI`                                                                                 |
| `$USER`                       | `${{ github.actor }}`                                                                 |
| `$HOME`                       | `${{ github.workspace }}`                                                             |
| `$TRAVIS_BRANCH`              | `${{ github.ref }}`                                                                   |
| `$TRAVIS_BUILD_DIR`           | `${{ github.workspace }}`                                                             |
| `$TRAVIS_BUILD_ID`            | `${{ github.run_number }}`                                                            |
| `$TRAVIS_BUILD_NUMBER`        | `${{ github.run_id }}`                                                                |
| `$TRAVIS_COMMIT`              | `${{ github.sha }}`                                                                   |
| `$TRAVIS_EVENT_TYPE`          | `${{ github.event_name }}`                                                            |
| `$TRAVIS_PULL_REQUEST_BRANCH` | `${{ github.base_ref }}`                                                              |
| `$TRAVIS_PULL_REQUEST`        | `${{ github.event.number }}`                                                          |
| `$TRAVIS_PULL_REQUEST_SHA`    | `${{ github.head.sha }}`                                                              |
| `$TRAVIS_PULL_REQUEST_SLUG`   | `${{ github.repository }}`                                                            |
| `$TRAVIS_TAG`                 | `${{ github.ref }}`                                                                   |
| `$TRAVIS_OS_NAME`             | `${{ runner.os }}`                                                                    |
| `$TRAVIS_JOB_ID`              | `${{ github.job }}`                                                                   |
| `$TRAVIS_REPO_SLUG`           | `${{ github.repository_owner/github.repository }}`                                    |
| `$TRAVIS_BUILD_WEB_URL`       | `${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}` |

## Limitations

### Manual Tasks

Certain constructs need to be migrated manually. These include:

- Secrets
- Unknown job properties

### Project Languages

Travis project languages are transformed by adding a set of preconfigured build tools (e.g. `ruby/setup-ruby@v1`) and a default build script (e.g. `run: bundle install --jobs=3 --retry=3`) to the transformed workflow. Valet will assume a Travis pipeline's project language is `ruby` if none is explicitly declared.

The following project languages are supported by Valet:

- `android`
- `bash`
- `c`
- `clojure`
- `c++`
- `crystal`
- `csharp`
- `d`
- `dart`
- `elixir`
- `erlang`
- `generic`
- `go`
- `groovy`
- `haskell`
- `haxe`
- `java`
- `julia`
- `matlab`
- `minimal`
- `nix`
- `node_js`
- `objective-c`
- `perl`
- `perl6`
- `php`
- `python`
- `r`
- `ruby`
- `rust`
- `scala`
- `sh`
- `shell`
- `smalltalk`
- `swift`

### Configure self service migration workflow

To configure the workflow, perform the following steps:

1. Copy the following files into the root of the migration repository while preserving their relative path:
   - [.github/workflows/migration.yml](.github/workflows/migration.yml)
   - [.github/ISSUE_TEMPLATE/migration.md](.github/ISSUE_TEMPLATE/migration.md)
1. Configure the following [secrets](https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets):
   - `GHCR_LOGIN`: The username to access the `valet` container.
   - `GHCR_VALET_PASSWORD`: The password to access the `valet` container (requires `read:packages` scope).
   - `TRAVIS_CI_ACCESS_TOKEN`: The personal access token used to access the TravisCI instance.
   - `TRAVIS_CI_INSTANCE_URL`: The base url for the TravisCI instance (eg. `https://travis-ci.com`)
   - `TRAVIS_CI_SOURCE_GITHUB_ACCESS_TOKEN`: The personal access token used to authenticate with your source GitHub instance (requires `repo` scope).
   - `TRAVIS_CI_SOURCE_GITHUB_INSTANCE_URL`: The url to the source GitHub instance. (e.g. `https://github.com`)
   - `GH_INSTANCE_URL`: GitHub base url (eg. `https://github.com`)
   - `GH_ACCESS_TOKEN`: GitHub personal access token to create pull requests. The [token](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token) must have permission to create pull requests in all target repos and requires the following [scopes](https://docs.github.com/en/free-pro-team@latest/developers/apps/scopes-for-oauth-apps):
     - `repo`
     - `workflow`
1. If you don't want to run the migration on [self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners) you need to change the `runs-on` value. (in case you are using GitHub Enterprise Server or the job need to access jenkins inside your network).
1. Configure any [custom transformers](../README.md#custom-transformers) you may need.
