# Migrating GitLab Pipelines with Valet

## How to run Valet

Valet can be run locally as a CLI, allowing a user to perform an:

- [audit](../README.md#Audit)
- [migration](../README.md#Migrate)
- [dry-run](../README.md#Dry-run)

Valet will perform a best attempt at full-fidelity transformation of a GitLab pipeline but each converted workflow should be reviewed and edited as necessary.

### Using environment variables

We recommend maintaining the inputs to Valet with environment variables. To help with this, the distribution files contain a `.env.local.template` file that can be used to generate a `.env.local` file that will be loaded by Valet at run time.

The following are environment variables used by Valet to connect to your GitLab CI instance.

- `GITHUB_ACCESS_TOKEN`: The personal access token used to create pull requests with a transformed workflow (requires `repo` and `workflow` scopes).
- `GITHUB_INSTANCE_URL`: The url to the target GitHub instance. (e.g. `https://github.com`)
- `GITLAB_ACCESS_TOKEN`: The private token used to authenticate with your GitLab instance (requires `read_api` scope).
- `GITLAB_INSTANCE_URL`: The url to the GitLab instance (e.g. `https://gitlab.com`)
- `NAMESPACE`: The namespace(s) (i.e. group) that contain the GitLab pipelines.

These environment variables can be specified in a `.env.local` file that will be loaded by Valet at run time. The distribution archive contains a `.env.local.template` file that can be used to create these files.

### Optional arguments

#### Source file path

This parameter can be set to feed a single source file path into Valet for a dry run or migration. Valet will use the source file specified instead of fetching the pipeline contents from source control.

```bash
valet dry-run gitlab --output-dir /data/path/to/output/ --namespace <namespace> --project <project-name> --source-file-path /data/path/to/.gitlab-ci.yml
```

#### Config file path

This parameter can be set to feed a list of source files into Valet for an audit. This will tell Valet to ingest the source files instead of fetching pipeline source code from source control.

```bash
valet audit gitlab --output-dir /data/path/to/output/ --config-file-path /data/path/to/.gitlab-config.yml
```

For auditing a GitLab instance using a config file, the config file should be in this format:

```yaml
pipelines:
  - project_slug: namespace/project-name
    source_file_path: path/to/.gitlab-ci.yml
  - project_slug: namespace/some-other-project-name
    source_file_path: path/to/.gitlab-ci.yml
```

Each pipeline name must be unique.

## Supported syntax

The following table shows the type of properties we are currently able to transform. For more details about how an GitLab pipeline syntax aligns with GitHub Actions, see [Migrating from GitLab CI/CD to GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/migrating-from-gitlab-cicd-to-github-actions)

| GitLab Pipelines                        | GitHub Actions                  |              Status |
| :-------------------------------------- | :------------------------------ | ------------------: |
| `stages`                                | `jobs`                          |           Supported |
| `workflow`                              | `if`                            | Partially supported |
| `include`                               |                                 |         Unsupported |
| `default`                               |                                 |           Supported |
| `auto_cancel_pending_pipelines`         | `concurrency`                   |           Supported |
| `job`                                   | `jobs.<job_id>`                 |           Supported |
| `tags`                                  | `jobs.<job_id>.runs-on`         |           Supported |
| `environment`                           | `jobs.<job_id>.environment`     | Partially Supported |
| `services`                              | `jobs.<job_id>.services`        | Partially Supported |
| `resource_group`                        | `jobs.<job_id>.concurrency`     |           Supported |
| `build_timeout` or `timeout`            | `jobs.<job_id>.timeout-minutes` |           Supported |
| `image`                                 | `jobs.<job_id>.container`       |           Supported |
| `parallel`                              | `jobs.<job_id>.strategy`        | Partially supported |
| `before_script`                         | `jobs.<job_id>.steps`           |           Supported |
| `script`                                | `jobs.<job_id>.steps`           |           Supported |
| `after_script`                          | `jobs.<job_id>.steps`           |           Supported |
| `needs`                                 | `jobs.<job_id>.needs`           |           Supported |
| `rules`                                 | `jobs.<job_id>.if`              | Partially supported |
| `only` or `except`                      | `jobs.<job_id>.if`              | Partially supported |
| `variables`                             | `env`                           |         Unsupported |
|                                         | `jobs.<job_id>.env`             |         Unsupported |
| `only_allow_merge_if_pipeline_succeeds` | `on.pull_request`               |           Supported |
| Run pipelines for new commits           | `on.push`                       |           Supported |
| `schedule`                              | `on.schedule`                   |           Supported |
| Run pipelines manually                  | `on.workflow_dispatch`          |           Supported |

The supported GitLab constructs can be seen [here](transformers/index.md).

### Environment variables

Valet will convert predefined GitLab environment variables to the closest equivalent in GitHub Actions using the mapping below:

| Travis CI                                     | Github Actions                                                                        |
| :-------------------------------------------- | :------------------------------------------------------------------------------------ |
| `CI_API_V4_URL`                               | `${{ github.api_url }}`                                                               |
| `CI_BUILDS_DIR`                               | `${{ github.workspace }}`                                                             |
| `CI_COMMIT_BRANCH`                            | `${{ github.ref }}`                                                                   |
| `CI_COMMIT_REF_NAME`                          | `${{ github.ref }}`                                                                   |
| `CI_COMMIT_REF_SLUG`                          | `${{ github.ref }}`                                                                   |
| `CI_COMMIT_SHA`                               | `${{ github.sha }}`                                                                   |
| `CI_COMMIT_SHORT_SHA`                         | `${{ github.sha }}`                                                                   |
| `CI_COMMIT_TAG`                               | `${{ github.ref }}`                                                                   |
| `CI_JOB_ID`                                   | `${{ github.job }}`                                                                   |
| `CI_JOB_MANUAL`                               | `${{ github.event_name == 'workflow_dispatch' }}`                                     |
| `CI_JOB_NAME`                                 | `${{ github.job }}`                                                                   |
| `CI_JOB_STATUS`                               | `${{ job.status }}`                                                                   |
| `CI_JOB_URL`                                  | `${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}` |
| `CI_JOB_TOKEN`                                | `${{ github.token }}`                                                                 |
| `CI_NODE_INDEX`                               | `${{ strategy.job-index }}`                                                           |
| `CI_NODE_TOTAL`                               | `${{ strategy.job-total }}`                                                           |
| `CI_PIPELINE_ID`                              | `${{ github.repository}}/${{ github.workflow }}`                                      |
| `CI_PIPELINE_IID`                             | `${{ github.workflow }}`                                                              |
| `CI_PIPELINE_SOURCE`                          | `${{ github.event_name }}`                                                            |
| `CI_PIPELINE_TRIGGERED`                       | `${{ github.actions }}`                                                               |
| `CI_PIPELINE_URL`                             | `${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}` |
| `CI_PROJECT_DIR`                              | `${{ github.workspace }}`                                                             |
| `CI_PROJECT_ID`                               | `${{ github.repository }}`                                                            |
| `CI_PROJECT_NAME`                             | `${{ github.event.repository.name }}`                                                 |
| `CI_PROJECT_NAMESPACE`                        | `${{ github.repository_owner }}`                                                      |
| `CI_PROJECT_PATH_SLUG`                        | `${{ github.repository }}`                                                            |
| `CI_PROJECT_PATH`                             | `${{ github.repository }}`                                                            |
| `CI_PROJECT_ROOT_NAMESPACE`                   | `${{ github.repository_owner }}`                                                      |
| `CI_PROJECT_TITLE`                            | `${{ github.event.repository.full_name }}`                                            |
| `CI_PROJECT_URL`                              | `${{ github.server_url }}/${{ github.repository }}`                                   |
| `CI_REPOSITORY_URL`                           | `${{ github.event.repository.clone_url }}`                                            |
| `CI_RUNNER_EXECUTABLE_ARCH`                   | `${{ runner.os }}`                                                                    |
| `CI_SERVER_HOST`                              | `${{ github.server_url }}`                                                            |
| `CI_SERVER_URL`                               | `${{ github.server_url }}`                                                            |
| `CI_SERVER`                                   | `${{ github.actions }}`                                                               |
| `GITLAB_CI`                                   | `${{ github.actions }}`                                                               |
| `GITLAB_USER_EMAIL`                           | `${{ github.actor }}`                                                                 |
| `GITLAB_USER_ID`                              | `${{ github.actor }}`                                                                 |
| `GITLAB_USER_LOGIN`                           | `${{ github.actor }}`                                                                 |
| `GITLAB_USER_NAME`                            | `${{ github.actor }}`                                                                 |
| `TRIGGER_PAYLOAD`                             | `${{ github.event_path }}`                                                            |
| `CI_MERGE_REQUEST_ASSIGNEES`                  | `${{ github.event.pull_request.assignees }}`                                          |
| `CI_MERGE_REQUEST_ID`                         | `${{ github.event.pull_request.number }}`                                             |
| `CI_MERGE_REQUEST_IID`                        | `${{ github.event.pull_request.number }}`                                             |
| `CI_MERGE_REQUEST_LABELS`                     | `${{ github.event.pull_request.labels }}`                                             |
| `CI_MERGE_REQUEST_MILESTONE`                  | `${{ github.event.pull_request.milestone }}`                                          |
| `CI_MERGE_REQUEST_PROJECT_ID`                 | `${{ github.repository }}`                                                            |
| `CI_MERGE_REQUEST_PROJECT_PATH`               | `${{ github.repository }}`                                                            |
| `CI_MERGE_REQUEST_PROJECT_URL`                | `${{ github.server_url }}/${{ github.repository }}`                                   |
| `CI_MERGE_REQUEST_REF_PATH`                   | `${{ github.ref }}`                                                                   |
| `CI_MERGE_REQUEST_SOURCE_BRANCH_NAME`         | `${{ github.event.pull_request.head.ref }}`                                           |
| `CI_MERGE_REQUEST_SOURCE_BRANCH_SHA`          | `${{ github.event.pull_request.head.sha}}`                                            |
| `CI_MERGE_REQUEST_SOURCE_PROJECT_ID`          | `${{ github.event.pull_request.head.repo.full_name }}`                                |
| `CI_MERGE_REQUEST_SOURCE_PROJECT_PATH`        | `${{ github.event.pull_request.head.repo.full_name }}`                                |
| `CI_MERGE_REQUEST_SOURCE_PROJECT_URL`         | `${{ github.event.pull_request.head.repo.url }}`                                      |
| `CI_MERGE_REQUEST_TARGET_BRANCH_NAME`         | `${{ github.event.pull_request.base.ref }}`                                           |
| `CI_MERGE_REQUEST_TARGET_BRANCH_SHA`          | `${{ github.event.pull_request.base.sha }}`                                           |
| `CI_MERGE_REQUEST_TITLE`                      | `${{ github.event.pull_request.title }}`                                              |
| `CI_EXTERNAL_PULL_REQUEST_IID`                | `${{ github.event.pull_request.number }}`                                             |
| `CI_EXTERNAL_PULL_REQUEST_SOURCE_REPOSITORY`  | `${{ github.event.pull_request.head.repo.full_name }}`                                |
| `CI_EXTERNAL_PULL_REQUEST_TARGET_REPOSITORY`  | `${{ github.event.pull_request.base.repo.full_name }}`                                |
| `CI_EXTERNAL_PULL_REQUEST_SOURCE_BRANCH_NAME` | `${{ github.event.pull_request.head.ref }}`                                           |
| `CI_EXTERNAL_PULL_REQUEST_SOURCE_BRANCH_SHA`  | `${{ github.event.pull_request.head.sha }}`                                           |
| `CI_EXTERNAL_PULL_REQUEST_TARGET_BRANCH_NAME` | `${{ github.event.pull_request.base.ref }}`                                           |
| `CI_EXTERNAL_PULL_REQUEST_TARGET_BRANCH_SHA`  | `${{ github.event.pull_request.base.sha }}`                                           |
