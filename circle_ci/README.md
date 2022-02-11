# Migrating Circle CI Pipelines with Valet

## How to run Valet

Valet can be run locally as a CLI, allowing a user to perform an:

- [audit](../README.md#Audit)
- [migration](../README.md#Migrate)
- [dry-run](../README.md#Dry-run)

Valet will perform a best attempt at full-fidelity transformation of a Circle CI pipeline but each converted workflow should be reviewed and edited as necessary.

### Using environment variables

We recommend maintaining the inputs to Valet with environment variables. To help with this, the distribution files contain a `.env.local.template` file that can be used to generate a `.env.local` file that will be loaded by Valet at run time.

The following are environment variables used by Valet to connect to your Circle CI instance.

- `GITHUB_ACCESS_TOKEN`: The personal access token used to create pull requests with a transformed workflow (requires `repo` and `workflow` scopes).
- `GITHUB_INSTANCE_URL`: The url to the target GitHub instance. (e.g. `https://github.com`)
- `CIRCLE_CI_ACCESS_TOKEN`: The personal access token used to authenticate with your Circle CI instance.
- `CIRCLE_CI_INSTANCE_URL`: The url to the Circle CI instance (e.g. `https://circleci.com`). If the variable left unset, Valet will use `https://circleci.com` as default value.
- `CIRCLE_CI_ORGANIZATION`: The organization name of your Circle CI instance.
- `CIRCLE_CI_PROVIDER`: The location where your pipeline's source file is stored (e.g. `github`). Currently Valet only supports GitHub.

These environment variables can be specified in a `.env.local` file that will be loaded by Valet at run time. The distribution archive contains a `.env.local.template` file that can be used to create these files.

### Optional arguments

#### Source file path

This parameter can be set to feed a single source file path into Valet for a dry run or migration. Valet will use the source file specified instead of fetching the pipeline contents from source control.

```bash
valet dry-run circle-ci --output-dir /data/path/to/output/ --circle-ci-repository <repository-name> --source-file-path /data/path/to/./.circleci/config.yml
```

#### Config file path

This parameter can be set to feed a list of source files into Valet for an audit. This will alert Valet to ingest the source files instead of fetching pipeline source code from source control.

```bash
valet audit circle-ci --output-dir /data/path/to/output/ --config-file-path /data/path/to/.circle-config.yml
```

For auditing a Circle CI instance using a config file, the config file should be in this format:

```yaml
pipelines:
  - project_slug: circle-org-name/circle-project-name
    source_file_path: path/to/.circleci/config.yml
  - project_slug: circle-org-name/circle-project-name
    source_file_path: path/to/.circleci/config.yml
```

Each pipeline name must be unique.

## Supported syntax

The following table shows the type of properties we are currently able to transform. For more details about how an Circle CI pipeline syntax aligns with GitHub Actions, see [Migrating from Circle CI to GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/migrating-from-circleci-to-github-actions)

| Circle CI Pipelines | Github Actions                     |              Status |
| :------------------ | :--------------------------------- | ------------------: |
| setup               |                                    |         Unsupported |
| version             |                                    |         Unsupported |
| orbs                | `actions`                          | Partially Supported |
| parameters          | `env`                              |           Supported |
|                     | `workflow-dispatch.inputs`         |           Supported |
| executors           | `container`                        | Partially Supported |
|                     | `runs-on`                          |           Supported |
|                     | `self hosted runners`              |         Unsupported |
| jobs                | `jobs`                             |           Supported |
| job                 | `jobs.<job_id>`                    |           Supported |
|                     | `jobs.<job_id>.name`               |           Supported |
| steps               | `jobs.<job_id>.steps`              |           Supported |
| matrix              | `jobs.<job_id>.strategy`           |           Supported |
|                     | `jobs.<job_id>.strategy.matrix`    |           Supported |
| when, unless        | `jobs.<job_id>.if`                 |           Supported |
| environment         | `env`                              |           Supported |
|                     | `jobs.<job_id>.env`                |           Supported |
|                     | `jobs.<job_id>.steps.env`          |           Supported |
| triggers            | `on`                               |           Supported |
| cron triggers       | `on.schedule`                      |           Supported |

The supported Circle CI tasks can be seen [here](index.md).

### Environment variables

Valet will convert default Circle CI environment variables to the closest equivalent in GitHub Actions using the mapping below:

| Circle CI                             | Github Actions                                                                        |
| :------------------------------------ | :------------------------------------------------------------------------------------ |
| `CI`                                  | `$CI`                                                                                 |
| `CIRCLE_BRANCH`                       | `${{ github.ref }}`                                                                   |
| `CIRCLE_JOB`                          | `${{ github.job }}`                                                                   |
| `CIRCLE_PR_NUMBER`                    | `${{ github.event.number }}`                                                          |
| `CIRCLE_PR_REPONAME`                  | `${{ github.repository }}`                                                            |
| `CIRCLE_PROJECT_REPONAME`             | `${{ github.repository }}`                                                            |
| `CIRCLE_SHA1`                         | `${{ github.sha }}`                                                                   |
| `CIRCLE_TAG`                          | `${{ github.ref }}`                                                                   |
| `CIRCLE_USERNAME`                     | `{{ github.actor }}`                                                                  |
| `CIRCLE_WORKFLOW_ID`                  | `${{ github.run_number }}`                                                            |
| `CIRCLE_WORKING_DIRECTORY`            | `${{ github.workspace }}`                                                             |
| `<< pipeline.id >>`                   | `${{ github.workflow }}`                                                              |
| `<< pipeline.number >>`               | `${{ github.run_number }}`                                                            |
| `<< pipeline.project.git_url >>`      | `$GITHUB_SERVER_URL/$GITHUB_REPOSITORY`                                               |
| `<< pipeline.project.type >>`         | `github`                                                                              |
| `<< pipeline.git.tag >>`              | `${{ github.ref }}`                                                                   |
| `<< pipeline.git.branch >>`           | `${{ github.ref }}`                                                                   |
| `<< pipeline.git.revision >>`         | `${{ github.event.pull_request.head.sha }}`                                           |
| `<< pipeline.git.base_revision >>`    | `${{ github.event.pull_request.base.sha }}`                                           |

## Limitations

- Automatic caching in between jobs of different workflows is not supported

### Manual Tasks

Certain constructs need to be migrated manually. These include:

- Contexts
- Project environment variables
- Unknown job properties
- Unknown orbs
