# Migrating Jenkins Jobs with Valet

## How to run Valet

Valet can be run locally as a CLI, allowing a user to perform an:

- [audit](../README.md#Audit)
- [migration](../README.md#Migrate)
- [dry-run](../README.md#Dry-run)

Valet will perform a best attempt at full-fidelity transformation of a Jenkins job but each converted workflow should be reviewed and edited as necessary.

### Using environment variables

We recommend maintaining the inputs to Valet with environment variables. To help with this, the distribution files contain a `.env.local.template` file that can be used to generate a `.env.local` file that will be loaded by Valet at run time.

The following are environment variables used by Valet to connect to your Jenkins instance.

- `GITHUB_ACCESS_TOKEN`: The personal access token used to create pull requests with a transformed workflow (requires `repo` and `workflow` scopes).
- `GITHUB_INSTANCE_URL`: The url to the GitHub instance. (e.g. `https://github.com`)
- `JENKINSFILE_ACCESS_TOKEN`: The personal access token used to retrieve the contents of a `Jenkinsfile` stored in the build repository (requires `repo` scope).
- `JENKINS_ACCESS_TOKEN`: The access token used to view Jenkins resources. **Note**: this token requires access to all jobs that you want to migrate or audit. In cases where a folder or job do not inherit parent ACLs, you must grant explicit permissions or full admin privileges.
- `JENKINS_USERNAME`: The username of the user's access token.
- `JENKINS_INSTANCE_URL`: The url to the Jenkins instance.

These environment variables can be specified in a `.env.local` file that will be loaded by Valet at run time. The distribution archive contains a `.env.local.template` file that can be used to create these files.

### Optional arguments

#### Source file path

This parameter can be set to feed a single source file path into Valet for a dry run or migration. Valet will use the source file specified instead of fetching the pipeline contents from source control. This option can be used for Jenkinsfile and multibranch pipelines.

##### Jenkinsfile pipeline

```bash
valet dry-run --output-dir /data/path/to/output/ --source-file-path /data/path/to/Jenkinsfile
```

##### Multibranch pipeline

```bash
valet dry-run --output-dir /data/path/to/output/ --source-file-path /data/path/to/multibranch.yml
```

Where `multibranch.yml` is a yaml file in this format:

```yaml
pipeline-name:
  branch-name: path/to/Jenkinsfile
  branch-name: path/to/Jenkinsfile
```

#### Config file path

This parameter can be set to feed a list of source files into Valet for an audit. This will alert Valet to ingest the source files instead of fetching pipeline source code from source control.

For auditing a Jenkins instance using a config file, the config file should be in this format:

```yaml
pipelines:
  pipeline-name: path/to/Jenkinsfile
  multi-branch-pipeline-name:
    branch-name: path/to/Jenkinsfile
    branch-name: path/to/Jenkinsfile
```

## Supported syntax

The following table shows the type of properties we are currently able to transform. For more details about how a Jenkins pipeline syntax aligns with GitHub Actions, see [Migrating from Jenkins to GitHub Actions
](https://docs.github.com/en/actions/learn-github-actions/migrating-from-jenkins-to-github-actions)

| Jenkins Designer Pipeline | Jenkinsfile | Github Actions                     |              Status |
| :------------------------ | :---------- | :--------------------------------- | ------------------: |
| general                   | agent       | `runners`                          | Partially supported |
|                           |             | `self hosted runners`              | Partially supported |
| build                     | stages      | `jobs`                             | Partially supported |
|                           | stage       | `jobs.<job_id>`                    |           Supported |
| docker template           | docker      | `jobs.<job_id>.container`          |           Supported |
|                           |             | `jobs.<job_id>.name`               |           Supported |
|                           | steps       | `jobs.<job_id>.steps`              | Partially supported |
|                           | options     | `jobs.<job_id>.strategy`           |         Unsupported |
|                           |             | `jobs.<job_id>.strategy.fail-fast` |         Unsupported |
|                           | matrix      | `jobs.<job_id>.strategy.matrix`    |         Unsupported |
|                           |             | `jobs.<job_id>.timeout-minutes`    |         Unsupported |
|                           | when        | `jobs.<job_id>.if`                 |         Unsupported |
|                           | parameters  | `inputs`                           |         Unsupported |
|                           |             | `output`                           |         Unsupported |
|                           | inputs      | `inputs`                           |         Unsupported |
| build environment         | environment | `env`                              | Partially supported |
|                           |             | `jobs.<job_id>.env`                |         Unsupported |
|                           |             | `jobs.<job_id>.steps.env`          |         Unsupported |
|                           |             | `jobs.<job_id>.timeout-minutes`    | Partially supported |
| build triggers            | triggers    | `on`                               | Partially supported |
|                           |             | `on.<event_name>.types`            | Partially supported |
|                           |             | `on.<push>.<branches>`             |           Supported |
|                           |             | `on.<push>.<tags>`                 |           Supported |
|                           |             | `on.<push>.paths`                  |           Supported |
|                           |             | `on.<pull_request>.<branches>`     |           Supported |
|                           |             | `on.<pull_request>.<tags>`         |         Unsupported |
|                           |             | `on.<pull_request>.paths`          |           Supported |
|                           |             | `on.schedule`                      | Partially supported |
|                           |             | `on.workflow_run`                  |           Supported |

The supported Jenkins plugins can be seen [here](plugins/index.md).

### Environment variables

Valet will convert default Jenkins environment variables to the closest equivalent in GitHub Actions using the mapping below:

| Jenkins           | Github Actions                                                                        |
| :---------------- | :------------------------------------------------------------------------------------ |
| `${BUILD_ID}`     | `${{ github.run_id }}`                                                                |
| `${BUILD_NUMBER}` | `${{ github.run_id }}`                                                                |
| `${BUILD_TAG}`    | `${{ github.workflow }}-${{ github.run_id }}`                                         |
| `${BUILD_URL}`    | `${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}` |
| `${JENKINS_URL}`  | `${{ github.server_url }}`                                                            |
| `${JOB_NAME}`     | `${{ github.workflow }}`                                                              |
| `${WORKSPACE}`    | `${{ github.workspace }}`                                                             |

## Self service migration workflow

Optionally, Valet can be orchestrated using GitHub Actions and Issues. A template of this workflow has been will be provided to you.

To configure the workflow, add the template files to a repo in which users that can execute migrations have the `triage` [permission](https://docs.github.com/en/free-pro-team@latest/github/setting-up-and-managing-organizations-and-teams/repository-permission-levels-for-an-organization) at a minimum.

To interact with Valet, a user creates an issue using the `migration` Issue template and enters the following data:

- Jenkins Job URL
- GitHub Target repository

Once the Issue is created, the workflow will respond to the issue with further instructions. If both parameters have been correctly filled, it will automatically start a dry run and will add the transformed workflow as a comment on the Issue.

The user can continue to respond to the Issue to run more dry runs or perform a actual migrating by commenting `/run-migration`.

### Configure self service migration workflow

To configure the workflow, perform the following steps:

1. Copy the following files into the root of the migration repository while preserving their relative path:
   - [.github/workflows/migration.yml](.github/workflows/migration.yml)
   - [.github/ISSUE_TEMPLATE/migration.md](.github/ISSUE_TEMPLATE/migration.md)
1. Configure the following [secrets](https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets):
   - `GHCR_LOGIN`: The username to access the `valet` container.
   - `GHCR_VALET_PASSWORD`: The password to access the `valet` container (requires `read:packages` scope).
   - `JENKINSFILE_ACCESS_TOKEN`: The personal access token used to retrieve the contents of a `Jenkinsfile` stored in the build repository (requires `repo` scope).
   - `JENKINS_ACCESS_TOKEN`: The access token used to view Jenkins resources.
   - `JENKINS_USERNAME`: The username of the user's access token.
   - `JENKINS_INSTANCE_URL`: The url to the Jenkins instance.
   - `GH_INSTANCE_URL`: GitHub base url (eg. `https://github.com`)
   - `GH_ACCESS_TOKEN`: GitHub personal access token to create pull requests. The [token](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token) must have permission to create pull requests in all target repos and requires the following [scopes](https://docs.github.com/en/free-pro-team@latest/developers/apps/scopes-for-oauth-apps):
     - `repo`
     - `workflow`
1. If you don't want to run the the migration on [self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners) you need to change the `runs-on` value. (in case you are using GitHub Enterprise Server or the job need to access jenkins inside your network).
1. Configure any [custom transformers](../README.md#custom-transformers) you may need.

## Limitations

### Manual Tasks

Certain constructs need to be migrated manually. These include:

- Secrets
- Mandatory build tools
- Unknown plugins
- Self-hosted runners

### Not Supported

- [Scripted pipelines](https://www.jenkins.io/doc/book/pipeline/syntax/#scripted-pipeline) (e.g. pure Groovy) are not supported
