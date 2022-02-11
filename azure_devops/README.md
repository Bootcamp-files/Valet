# Migrating Azure DevOps Pipelines with Valet

## How to run Valet

Valet can be run locally as a CLI, allowing a user to perform an:

- [audit](../README.md#Audit)
- [migration](../README.md#Migrate)
- [dry-run](../README.md#Dry-run)

Valet also requires the user to specify if the target is a `pipeline` or `release` when using the `dry-run` or `migrate` command on Azure DevOps (the `audit` command does not require specifying the type since it will collect information about both). For example:

```bash
valet <dry-run|migrate> azure-devops <pipeline|release>
```

Valet will perform a best attempt at full-fidelity transformation of a Azure DevOps pipeline but each converted workflow should be reviewed and edited as necessary.

### Using environment variables

We recommend maintaining the inputs to Valet with environment variables. To help with this, the distribution files contain a `.env.local.template` file that can be used to generate a `.env.local` file that will be loaded by Valet at run time.

The following are environment variables used by Valet to connect to your Azure DevOps instance.

- `GITHUB_ACCESS_TOKEN`: The personal access token used to create pull requests with a transformed workflow (requires `repo` and `workflow` scopes).
- `GITHUB_INSTANCE_URL`: The url to the GitHub instance. (e.g. `https://github.com`)
- `AZURE_DEVOPS_ACCESS_TOKEN`: The personal access token used to authenticate with your Azure DevOps instance. This token requires the following scopes:
  - Build (Read & Execute)
  - Code (Read)
  - Release (Read)
  - Service Connections (Read)
  - Variable Groups (Read)
- `AZURE_DEVOPS_PROJECT`: The project name or GUID to use when migrating a pipeline. This is optional when running an audit if you'd like to audit all projects.
- `AZURE_DEVOPS_ORGANIZATION`: The organization name of your Azure DevOps instance.
- `AZURE_DEVOPS_INSTANCE_URL`: The url to the Azure DevOps instance (e.g. `https://dev.azure.com`)

These environment variables can be specified in a `.env.local` file that will be loaded by Valet at run time. The distribution archive contains a `.env.local.template` file that can be used to create these files.

## Supported syntax

The following table shows the type of properties we are currently able to transform. For more details about how an Azure DevOps pipeline syntax aligns with GitHub Actions, see [Migrating from Azure Pipelines to GitHub Actions](https://docs.github.com/en/actions/learn-github-actions/migrating-from-azure-pipelines-to-github-actions)

| Azure Pipelines       | Github Actions                        |              Status |
| :-------------------- | :------------------------------------ | ------------------: |
| pool                  | `runners`                             | Partially supported |
|                       | `self hosted runners`                 | Partially supported |
| stage                 | `jobs`                                |           Supported |
| job                   | `jobs.<job_id>`                       |           Supported |
| container             | `jobs.<job_id>.container`             |           Supported |
|                       | `jobs.<job_id>.name`                  |           Supported |
| steps                 | `jobs.<job_id>.steps`                 |           Supported |
| manual deployment     | `jobs.<job_id>.environment`           | Partially supported |
| strategy              | `jobs.<job_id>.strategy`              | Partially supported |
|                       | `jobs.<job_id>.strategy.fail-fast`    |           Supported |
|                       | `jobs.<job_id>.strategy.max-parallel` |           Supported |
|                       | `jobs.<job_id>.strategy.matrix`       |           Supported |
| services              | `jobs.<job_id>.services`              | Partially supported |
| condition             | `jobs.<job_id>.if`                    |           Supported |
|                       | `jobs.<job_id>.steps[*].if`           |           Supported |
| variables             | `env`                                 |           Supported |
|                       | `jobs.<job_id>.env`                   |           Supported |
|                       | `jobs.<job_id>.steps.env`             |           Supported |
| timeoutInMinutes      | `jobs.<job_id>.timeout-minutes`       |           Supported |
| triggers              | `on`                                  | Partially supported |
|                       | `on.<event_name>.types`               |         Unsupported |
| continuousIntegration | `on.<push>.<branches>`                |           Supported |
|                       | `on.<push>.<tags>`                    |           Supported |
|                       | `on.<push>.paths`                     |           Supported |
| pullRequest           | `on.<pull_request>.<branches>`        |           Supported |
|                       | `on.<pull_request>.<tags>`            |         Unsupported |
|                       | `on.<pull_request>.paths`             |           Supported |
| schedules             | `on.schedule`                         |         Unsupported |
|                       | `on.workflow_run`                     |         Unsupported |

The supported Azure DevOps tasks can be seen [here](plugins/index.md).

### Environment variables

Valet will convert default Azure Pipelines environment variables to the closest equivalent in GitHub Actions using the mapping below:

| Azure Pipelines                             | Github Actions                                      |
| :------------------------------------------ | :-------------------------------------------------- |
| `$(Agent.BuildDirectory)`                   | `${{ runner.workspace }}`                           |
| `$(Agent.HomeDirectory)`                    | `${{ env.HOME }}`                                   |
| `$(Agent.JobName)`                          | `${{ github.job }}`                                 |
| `$(Agent.OS)`                               | `${{ runner.os }}`                                  |
| `$(Agent.ReleaseDirectory)`                 | `${{ github.workspace}}`                            |
| `$(Agent.RootDirectory)`                    | `${{ github.workspace }}`                           |
| `$(Agent.ToolsDirectory)`                   | `${{ runner.tool_cache }}`                          |
| `$(Agent.WorkFolder)`                       | `${{ github.workspace }}`                           |
| `$(Build.ArtifactStagingDirectory)`         | `${{ runner.temp }}`                                |
| `$(Build.BinariesDirectory)`                | `${{ github.workspace }}`                           |
| `$(Build.BuildId)`                          | `${{ github.run_id }}`                              |
| `$(Build.BuildNumber)`                      | `${{ github.run_number }}`                          |
| `$(Build.DefinitionId)`                     | `${{ github.workflow }}`                            |
| `$(Build.DefinitionName)`                   | `${{ github.workflow }}`                            |
| `$(Build.PullRequest.TargetBranch)`         | `${{ github.base_ref }}`                            |
| `$(Build.PullRequest.TargetBranch.Name)`    | `${{ github.base_ref }}`                            |
| `$(Build.QueuedBy)`                         | `${{ github.actor }}`                               |
| `$(Build.Reason)`                           | `${{ github.event_name }}`                          |
| `$(Build.Repository.LocalPath)`             | `${{ github.workspace }}`                           |
| `$(Build.Repository.Name)`                  | `${{ github.repository }}`                          |
| `$(Build.Repository.Provider)`              | `GitHub`                                            |
| `$(Build.Repository.Uri)`                   | `${{ github.server.url }}/${{ github.repository }}` |
| `$(Build.RequestedFor)`                     | `${{ github.actor }}`                               |
| `$(Build.SourceBranch)`                     | `${{ github.ref }}`                                 |
| `$(Build.SourceBranchName)`                 | `${{ github.ref }}`                                 |
| `$(Build.SourceVersion)`                    | `${{ github.sha }}`                                 |
| `$(Build.SourcesDirectory)`                 | `${{ github.workspace }}`                           |
| `$(Build.StagingDirectory)`                 | `${{ runner.temp }}`                                |
| `$(Pipeline.Workspace)`                     | `${{ runner.workspace }}`                           |
| `$(Release.DefinitionEnvironmentId)`        | `${{ github.job }}`                                 |
| `$(Release.DefinitionId)`                   | `${{ github.workflow }}`                            |
| `$(Release.DefinitionName)`                 | `${{ github.workflow }}`                            |
| `$(Release.Deployment.RequestedFor)`        | `${{ github.actor }}`                               |
| `$(Release.DeploymentID)`                   | `${{ github.run_id }}`                              |
| `$(Release.EnvironmentId)`                  | `${{ github.job }}`                                 |
| `$(Release.EnvironmentName)`                | `${{ github.job }}`                                 |
| `$(Release.Reason")`                        | `${{ github.event_name }}`                          |
| `$(Release.RequestedFor")`                  | `${{ github.actor }}`                               |
| `$(System.ArtifactsDirectory)`              | `${{ github.workspace }}`                           |
| `$(System.DefaultWorkingDirectory)`         | `${{ github.workspace }}`                           |
| `$(System.HostType)`                        | `build`                                             |
| `$(System.JobId)`                           | `${{ github.job }}`                                 |
| `$(System.JobName)`                         | `${{ github.job }}`                                 |
| `$(System.PullRequest.PullRequestId)`       | `${{ github.event.number }}`                        |
| `$(System.PullRequest.PullRequestNumber)`   | `${{ github.event.number }}`                        |
| `$(System.PullRequest.SourceBranch)`        | `${{ github.ref }}`                                 |
| `$(System.PullRequest.SourceRepositoryUri)` | `${{ github.server.url }}/${{ github.repository }}` |
| `$(System.PullRequest.TargetBranch)`        | `${{ github.event.base.ref }}`                      |
| `$(System.PullRequest.TargetBranchName)`    | `${{ github.event.base.ref }}`                      |
| `$(System.StageAttempt)`                    | `${{ github.run_number }}`                          |
| `$(System.TeamFoundationCollectionUri)`     | `${{ github.server.url }}/${{ github.repository }}` |
| `$(System.WorkFolder)`                      | `${{ github.workspace }}`                           |

## Limitations

Valet requires version 6.0 of the Azure DevOps API, which means Azure DevOps Server 2019 and older are not compatible. See the API version support matrix in the [REST API Versioning documentation](https://docs.microsoft.com/en-us/azure/devops/integrate/concepts/rest-api-versioning#supported-versions) for details.

### Task GUIDs

Tasks that are implicitly added to an Azure DevOps pipeline (e.g. checkout out source code) may be added to a Valet audit as a GUID name. The friendly task name for a GUID can be seen using the following url: `https://dev.azure.com/:organization/_apis/distributedtask/tasks/:guid`

### Manual Tasks

Certain constructs need to be migrated manually. These include:

- Secrets
- Service connections
- Unknown tasks
- Self-hosted runners

### Not Supported

- Variables from KeyVault are not supported
- Some resource triggers are not supported
