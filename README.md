# Valet

A tool to help migrate to [GitHub Actions](https://github.com/features/actions).

Valet currently has support for pipelines from Azure DevOps, Jenkins, Travis CI, Circle CI, and GitLab CI. The documentation described below applies regardless of the source CI platform. Documentation that is specific to each platform can be seen here:

- [Azure DevOps](./azure_devops/README.md)
- [Jenkins](./jenkins/README.md)
- [Travis CI](./travis_ci/README.md)
- [Circle CI](./circle_ci/README.md)
- [GitLab CI](./gitlab_ci/README.md)

## How to install Valet

Valet is distributed as a Docker container and a set of scripts to interact it.

### Prerequisites

The following are requirements to running Valet:

- Docker running on the machine
- `valet` and `valet-update` [scripts](#Valet-scripts)
- Username and password to authenticate with our container registry.

### Valet scripts

#### `valet-update`

The `valet-update` script is used to download the latest version of the Valet. To use, run the script in a command prompt. If you are prompted to login, use your GitHub handle as the username and a Personal Access Token with the `read:packages` scope as the password. If you have provided credentials previously, you will not have to authenticate again.

To verify the container was retrieved successfully, run `docker image ls ghcr.io/valet-customers/valet-cli` from the command line. You should see `ghcr.io/valet-customers/valet-cli` listed, with the tag of `latest`.

#### `valet`

The `valet` script is used to execute Valet in the container.

## How to run Valet

Valet can be run locally as a CLI, allowing a user to perform an:

- [audit](#Audit)
- [migration](#Migrate)
- [dry-run](#Dry-run)

Valet will perform a best attempt at full-fidelity transformation of a pipeline but each converted workflow should be reviewed and edited as necessary.

### Audit

The `audit` command can be used to scan a CI instance output a summary of the data used. This summary can then be used to plan timelines for migrating to GitHub Actions. To run an audit, use the following command:

```bash
valet audit <jenkins|azure-devops|travis-ci> --output-dir /data/path/to/output/
```

In the above example, Valet uses predefined environment variables to connect to your CI instance. You can optionally provide these inputs explictly using options specific for each platform. These options can be discovered in the [platform's documentation](#valet). The `help` command will describe all required and optional arguments that can be used with the `audit` command.

```bash
valet help audit
```

#### Uploading audit reports

The `upload-audit` command can be used to upload the output of an audit to the Valet engineering team. This is especially useful to make the team aware of any plugins or tasks that we should prioritize adding support for. To upload an audit report, use the following command:

```bash
valet upload-audit --audit-dir /data/path/to/output
```

There will be a prompt to complete an authorization request with GitHub. Once authenticated, the audit report will be uploaded to a private storage that only the GitHub team can access.

### Migrate

The `migrate` command will perform a migration of a pipeline and open a pull request with the transformed GitHub Actions workflow. To run a migration, use one of the following command:

```bash
valet migrate <jenkins|azure-devops|travis-ci> --target-url "https://github.com/org/repo" <platform-specific input>
```

The input to the `migrate` command varies slightly between Valet's supported platforms and can be discovered in the [platform's documentation](#valet). The `help` command will describe all required and optional arguments that can be used with the `migrate` command.

```bash
valet help migrate
```

### Dry run

The `dry-run` command will transform a pipeline to a GitHub Actions workflow and output its contents to a file inside of the folder specified. If the pipeline converts to multiple workflows - for example, a Jenkins Multibranch pipeline - Valet will output file for each transformed workflow. To run a dry-run migration, use the following command:

```bash
valet dry-run <jenkins|azure-devops|travis-ci> --output-dir /data/path/to/output/
```

The input to the `dry-run` command varies slightly between Valet's supported platforms and can be discovered in the [platform's documentation](#valet). The `help` command will describe all required and optional arguments that can be used with the `dry-run` command.

```bash
valet help dry-run
```

### Using environment variables

Environment variables can optionally be used to specify the following arguments:

We recommend maintaining the inputs to Valet with environment variables. To help with this, the distribution files contain a `.env.local.template` file that can be used to generate a `.env.local` file that will be loaded by Valet at run time.

The following are environment variables used regardless of the CI platform you are migrating from.

- `GITHUB_ACCESS_TOKEN`: The personal access token used to create pull requests with a transformed workflow (requires `repo` and `workflow` scopes).
- `GITHUB_INSTANCE_URL`: The url to the GitHub instance. (e.g. `https://github.com`)

Platform-specific environment variables can be found in the [platform's documentation](#valet).

### Optional arguments

#### Limit allowed actions

The following parameters can be used to limit which actions are allowed in converted workflows:

- `--allowed-actions` takes a list of actions to allow (wildcards are supported) and any other actions will be disallowed. Provide an empty list to disallow all actions (i.e `--allowed-actions=`).

- `--allow-verified-actions` will disallow any actions that are not from a verified publisher.

- `--allow-github-created-actions` will allow only actions published from the **github** or **actions** organizations (equivalent to `--allowed-actions actions/* github/*`).

Any of these these options can be combined to expand the list of allowed actions. If none of these options are supplied, all actions will be allowed by default.

#### Credentials file

The `--credentials-file` parameter can be used to specify the path to a file containing credentials to use for different servers that Valet needs to authenticate with. One scenario where this is useful is when build scripts (e.g. a `.travis.yml` or `jenkinsfile`) is stored in multiple GitHub Enterprise Server instances.

The format of this file must be a yaml file with a list of server and access token combinations. Valet will use the url that most closely matches the network request being made. For example, given the following contents of a credentials file:

```yml
- url: https://github.com
  access_token: hunter12
- url: https://github.com/specific_org/
  access_token: specific_token
- url: https://jenkins.org
  access_token: abc123
  username: marty_mcfly
```

The access token `hunter12` will be used by Valet to authenticate all network requests to <https://github.com> _unless_ the network request is for a repository in the organization `specific_org` - for which `specific_token` will be used to authenticate.

##### Alternative source code providers

It is possible to have Valet automatically fetch source code from non-GitHub repositories. To do so, a credentials file must be used to specify the credentials needed to retrieve the source code and the `provider` must be specified for these credentials. For example:

```yml
- url: https://gitlab.com
  access_token: super_secret_token
  provider: gitlab
```

Valet will use the token `super_secret_token` to retrieve any source code that is hosted in GitLab. The supported values for `provider` are:

- `github` (default value)
- `gitlab`
- `bitbucket_server`

#### Specify GitHub features to include in workflows

It is possible to limit the features used in workflows created by Valet. This can often be useful to exclude newer Actions syntax from any workflows if migrating to an older GitHub Enterprise Server instance. To do so, the `--features` command line option can be used. For example:

```bash
valet dry-run ... --features ghes-3.1
```

The supported values for `features` are:

- `all` (default value)
- `ghes-latest`
- `ghes-3.2`
- `ghes-3.1`
- `ghes-3.0`

### Path arguments

When running Valet, path arguments are relative to the container's disk. This means absolute paths relative to the host machine are not supported. When Valet is executed, the container's `/data` directory is mounted to the directory in which Valet is executed (this can be overridden using the `VALET_LOCAL_FOLDER` environment variable).

Path arguments require you to explicitly add `/data/` to the filename so it's written on the host disk. To simply see the output, `/dev/stdout` can be used as the output file.

For example, the following will output Valet's audit summary to the `/Users/mona/out` directory:

```bash
# /Users/mona
valet audit --output-dir /data/out
```

### Using a proxy

In order to access servers that are configured with a HTTP proxy, the following environment variables must be set with the proxy's URL:

- `OCTOKIT_PROXY`: for any GitHub server
- `HTTP_PROXY` (or `HTTPS_PROXY`): for any other servers

For example:

```sh
export OCTOKIT_PROXY=https://proxy.example.com:port
export HTTPS_PROXY=$OCTOKIT_PROXY
```

If the proxy requires authentication, a username and password must be embedded in the proxy URL (eg: `https://username:password@proxy.url:port`)

### SSL certificate verification

By default, Valet will verify ssl certificates prior to making network requests. This behavior can be disabled with the `--no-ssl-verify` option. For example:

```bash
valet audit --output-dir /data/out --no-ssl-verify
```

## Custom transformations

Valet offers the ability to extend its built-in mapping in 2 different locations:

- Plugin -> Action mapping
- Agent label -> Action runner labels

### Custom plugin mapping

Valet offers the ability to build custom transformation logic that will be used when converting pipelines to an Actions workflow. There are a number of scenarios where this can be useful:

- Valet doesn't provide a mapping for a plugin or task to Github Actions.
- You want to override the default mapping that Valet provide for your specific needs.
- Pipelines reference proprietary build logic (e.g. Jenkins shared libraries) which are custom to your CI instance.

A custom transformer is a ruby file that contains one or many `transform` methods. These methods must return one or many `Hash`es that correspond to the Action defined in yaml. The options used to configure the plugin can be inspected by using the `item` variable passed to the transformer. As an example, the following file could be used to convert a shared library's method named "buildJavascriptLibrary":

```ruby
transform "buildJavascriptLibrary" do |item|
  step_arguments = item["arguments"]
  run_command = []

  %w[install package deploy].each do |script|
    run_command << "npm run #{script}" if step_arguments.detect { |arg| arg.key == script }["value"]["value"] == "true"
  end

  {
    name: "build javascript app",
    shell: "bash",
    run: run_command.join("\n")
  }
end
```

The first argument (e.g. "buildJavascriptApp") to the `transform` method will be the identifier or name the CI instance uses for the plugin or task.

**Note**: The data structure of `item` will be different depending on the CI platform and the type of item it is transforming. The above example shows a custom transformer for a build step in Jenkins job using a Jenkinsfile.

This transformer can be specified to Valet using the `custom-transformers` cli option. A few examples can be seen below:

```bash
valet migrate jenkins ... --target-url "https://github.com/org/repo" --custom-transformers /data/path/to/custom/transformers.rb

valet migrate azure-devops ... --target-url "https://github.com/org/repo" --custom-transformers /data/path/to/custom/transformers.rb /data/path/to/other/custom/transformers.rb

valet audit azure-devops --output-dir /data/output --custom-transformers /data/path/to/**/transformers.rb
```

The same [path argument](#Path-arguments) limitation applies to where the manifest and mapping files can reside. This means these files must reside within the directory that `valet` is executed from. For example, if the manifest is located at `$(pwd)/transformers/manifest.yml`, the cli option would need to be `--custom-transformer-manifest /data/transformers/manifest.yml`.

### Custom runner mapping

Valet offers the ability to provide customized mapping between build agents in the source CI instance and their equivalent GitHub Actions runner labels. The syntax for providing a agent label mapping is similar to a plugin transformer.

A `runner` method is used with the first parameter being the Jenkins agent label and the second parameter being the GitHub Actions runner label(s) to map to. As an example, the following could be used:

```ruby
# If a job uses the agent label "jenkins-main", use "ubuntu-latest" in the converted workflow
runner "jenkins-main", "ubuntu-latest"

# If a job uses the agent label "mega-desk", use ["self-hosted", "mega", "dunder", "mifflin"] in the converted workflow
runner "mega-desk", ["self-hosted", "mega", "dunder", "mifflin"]
```

These transformers are specified to Valet using the `custom-transformers` cli option - just as you would specify a plugin transformer.

#### Configuring the default runner

Valet will attempt to map the build agent as best as it can. In cases where the tool cannot do this, the "ubuntu-latest" runner is used as a default. This default value can be controlled using a special keyword with the `runner` method. For example, the following command will use "windows-latest" as the default runner instead.

```ruby
runner :default, "windows-latest"
```

## Limitations

Valet will perform a best-effort attempt of a migration with a goal of converting 80% of the constructs of a pipeline. However, the conversion rate of any given pipeline will vary depending on how the pipeline are configured. Certain constructs are unsupported for any migration with Valet. These items include:

- Secrets are not migrated
- Permissions are not migrated
- Deployment gates are not migrated
- Build agents are not migrated
- Published packages (e.g. NuGet packages) are not migrated

Any platform-specific limitations can be found in the [platform's documentation](#valet).
