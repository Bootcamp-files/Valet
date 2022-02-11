# Concept Mappings

| CircleCI                                                       | GitHub                       |
| :------------------------------------------------------------- | :--------------------------- |
| [AddSshKeys](Transformers/Steps/AddSshKeys.md)                 | run                          |
| [AttachWorkspace](Transformers/Steps/AttachWorkspace.md)       | actions/download-artifact@v2 |
| [Checkout](Transformers/Steps/Checkout.md)                     | actions/checkout@v2          |
| [Deploy](Transformers/Executors/Deploy.md)                     | run                          |
| [Docker](Transformers/Executors/Docker.md)                     | container                    |
| [Machine](Transformers/Executors/Machine.md)                   | runs-on                      |
| [Macos](Transformers/Executors/Macos.md)                       | runs-on                      |
| [PersistToWorkspace](Transformers/Steps/PersistToWorkspace.md) | actions/upload-artifact@v2   |
| [Run](Transformers/Steps/Run.md)                               | run                          |
| [Schedule](Transformers/Triggers/Schedule.md)                  | on                           |
| [Store Test Results](Transformers/Steps/StoreTestResults.md)   | actions/upload-artifact@v2   |
| [Store Artifacts](Transformers/Steps/StoreArtifacts.md)        | actions/upload-artifact@v2   |
| [Unless](Transformers/Steps/Unless.md)                         | if                           |
| [When](Transformers/Steps/When.md)                             | if                           |

Any jobs not listed above will not be mapped to an action and will be left as a comment in the converted workflow.

## Unsupported

The following steps do not have any equivalent in GitHub Actions:

- setup_remote_docker

The following job properties do not have any equivalent in GitHub Actions:

- resource_class
- parallelism
- branches (deprecated in CircleCI)
- circleci_ip_ranges

The following concepts are not supported by Valet:

- executor type parameters
- dynamic configuration pipelines (`setup` key)

## Orb Mappings

| CircleCI                                                    | GitHub                                                  |
| :---------------------------------------------------------- | :------------------------------------------------------ |
| [circleci/aws-cli](Transformers/Orbs/AwsCli)                | aws-actions/configure-aws-credentials@v1                |
| [circleci/aws-ecr](Transformers/Orbs/AwsEcr)                | aws-actions/amazon-ecr-login@v1                         |
| [circleci/aws-ecs](Transformers/Orbs/AwsEcs)                | aws-actions/amazon-ecs-render-task-definition@v1        |
| [circleci/aws-s3](Transformers/Orbs/AwsS3)                  | aws-actions/configure-aws-credentials@v1, run           |
| [circleci/browser-tools](Transformers/Orbs/BrowserTools)    | -                                                       |
| [cypress-io/cypress](Transformers/Orbs/Cypress)             | cypress-io/github-action@v2, run                        |
| [circleci/docker](Transformers/Orbs/Docker)                 | hadolint/hadolint-action@v1.6.0, run                    |
| [circleci/heroku](Transformers/Orbs/Heroku)                 | run                                                     |
| [circleci/node](Transformers/Orbs/Node)                     | actions/setup-node@v2, actions/cache@v2, run            |
| [circleci/python](Transformers/Orbs/Python)                 | actions/cache@v2, run                                   |
| [circleci/ruby](Transformers/Orbs/Ruby)                     | ruby/setup-ruby@v1, run                                 |
| [circleci/slack](Transformers/Orbs/Slack)                   | rtCamp/action-slack-notify@v2                           |
| [circleci/windows](Transformers/Orbs/Windows)               | runs-on                                                 |
