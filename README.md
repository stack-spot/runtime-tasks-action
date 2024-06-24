# Runtimes Tasks Integrated Action

## runtimes-tasks-action

This Github action aims to integrate seamlessly along with [Runtime Manager Action](https://github.com/stack-spot/runtime-manager-action). Its intention is to in a single action join [Runtime IaC Action](https://github.com/stack-spot/runtime-iac-action), [Runtime Deploy Action](https://github.com/stack-spot/runtime-deploy-action) and [Runtime Destroy Action](https://github.com/stack-spot/runtime-destroy-action) into a single runner. Sometimes users don't have the necessity of doing customizable steps during the deployment process, this action removes this possibility and does the entire process within a single runner instance.

## Usage

### Requirements

To get the account keys (`CLIENT_ID`, `CLIENT_KEY` and `CLIENT_REALM`), please login using a **ADMIN** user on the [StackSpot Portal](https://stackspot.com), and generate new keys [here](https://stackspot.com/en/settings/access-token).
Also you need a runner with access to your AWS account.
You need to pair this action along with [Runtime Manager Action](https://github.com/stack-spot/runtime-manager-action) and [Runtime Manager Cancel Action](https://github.com/stack-spot/runtime-cancel-run-action)

### Use Case

```yaml
name: Stk Self Hosted

on:
  push:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
    orchestration:
        runs-on: ubuntu-latest                        # self-hosted-runner
        outputs:
            tasks: ${{ steps.run.outputs.tasks }}
            run_id: ${{ steps.run.outputs.run_id }}
        steps:
            - name: Authentication + Get Tasks
              id: run
              uses: stack-spot/runtime-manager-action@v2
              with:
                CLIENT_ID: ${{ secrets.CLIENT_ID }}
                CLIENT_KEY: ${{ secrets.CLIENT_KEY }}
                CLIENT_REALM: ${{ secrets.CLIENT_REALM }}
                WORKSPACE: my_workspace
                ENVIRONMENT: my_environment
                VERSION_TAG: my_tag
                TF_STATE_BUCKET_NAME: my_bucket
                TF_STATE_REGION: region
                IAC_BUCKET_NAME: my_bucket
                IAC_REGION: region
                VERBOSE: true                      # not mandatory
                BRANCH: main                       # not mandatory
                OPEN_API_PATH: swagger.yaml        # not mandatory
                DYNAMIC_INPUTS: --key1 value1 --key2 value2

    provision:
        runs-on: ubuntu-latest                   # self-hosted-runner
        needs: [orchestration]
        steps:
            - name: Service Provision
              uses: stack-spot/runtimes-tasks-action@v1
              with:
                TASK_LIST: ${{ needs.orchestration.outputs.tasks }}
                FEATURES_LEVEL_LOG: debug
                AWS_REGION: sa-east-1
                REPOSITORY_NAME: ${{ github.event.repository.name }}
                PATH_TO_MOUNT: /home/runner/_work/${{ github.event.repository.name }}/${{ github.event.repository.name }}
                CHECKOUT_BRANCH: ${{ inputs.checkout-branch }}
                LOCALEXEC_ENABLED: true
                FEATURES_TERRAFORM_MODULES: >-
                    [
                        {
                        "sourceType":"gitHttps",
                        "path":"github.com/stack-spot",// Allow all repositoris on stack-spot org
                        "private": true,
                        "app":"app",
                        "token":"token"
                        },
                        {
                        "sourceType":"terraformRegistry",
                        "path":"hashicorp/stack-spot", // Allow all modules on stack-spot org
                        "private": false
                        }
                    ]
                CLIENT_ID: ${{ secrets.stk-client-id }}
                CLIENT_KEY: ${{ secrets.stk-client-secret }}
                CLIENT_REALM: ${{ secrets.stk-realm }}
                AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN }}

    cancel-run:
        runs-on: ubuntu-latest
        needs: [orchestration, provision]
        if: ${{ always() && needs.orchestration.outputs.run_id != '' }}
        steps:
            - name: Cancel run
              uses: stack-spot/runtime-cancel-run-action@v1
              with:
                CLIENT_ID: ${{ secrets.CLIENT_ID }}
                CLIENT_KEY: ${{ secrets.CLIENT_KEY }}
                CLIENT_REALM: ${{ secrets.CLIENT_REALM }}
                RUN_ID: ${{ needs.orchestration.outputs.run_id }}
```

## Inputs


| Name                         | Description                                                                  | Type    | Required | Default                                |
|------------------------------|------------------------------------------------------------------------------|---------|----------|----------------------------------------|
| `TASK_LIST`                  | List of tasks                                                                | string  | true     |                                        |
| `FEATURES_LEVEL_LOG`         | Log Level                                                                    | string  | true     |                                        |
| `REPOSITORY_NAME`            | Git Repository Name                                                          | string  | true     |                                        |
| `AWS_REGION`                 | AWS REGION                                                                   | string  | true     |                                        |
| `CONTAINER_URL_IAC`          | IAC Container URL                                                            | string  | false    | `stackspot/runtime-job-iac:latest`     |
| `CONTAINER_URL_DEPLOY`       | Deploy Container URL                                                         | string  | false    | `stackspot/runtime-job-deploy:latest`  |
| `CONTAINER_URL_DESTROY`      | Destroy Container URL                                                        | string  | false    | `stackspot/runtime-job-destroy:latest` |
| `FEATURES_TERRAFORM_MODULES` | Terraform Modules                                                            | string  | false    |                                        |
| `PATH_TO_MOUNT`              | Path to mount inside the docker                                              | string  | true     |                                        |
| `OUTPUT_FILE`                | File name to save outputs: {plugin-alias}_{OUTPUT_FILE}                      | string  | false    | `outputs.json`                         |
| `LOCALEXEC_ENABLED`          | If Runtimes will allow execution of the local-exec command within terraform  | boolean | false    | `false`                                |
| `TF_LOG_PROVIDER`            | Level tf log provider - info, debug, warn or trace                           | string  | false    |                                        |
| `CHECKOUT_BRANCH`            | Whether or not checkout is enabled.                                          | boolean | false    | `false`                                |
| `BASE_PATH_OUTPUT`           | Base Path Output                                                             | string  | false    |                                        |
| `CLIENT_ID`                  | CLIENT ID                                                                    | string  | true     |                                        |
| `CLIENT_KEY`                 | CLIENT KEY                                                                   | string  | true     |                                        |
| `CLIENT_REALM`               | CLIENT REALM                                                                 | string  | true     |                                        |
| `AWS_ACCESS_KEY_ID`          | AWS ACCESS KEY ID from console                                               | string  | false    |                                        |
| `AWS_SECRET_ACCESS_KEY`      | AWS SECRET ACCESS KEY from console                                           | string  | false    |                                        |
| `AWS_SESSION_TOKEN`          | AWS SESSION TOKEN from console                                               | string  | false    |                                        |
| `AWS_ROLE_ARN`               | AWS ROLE ARN                                                                 | string  | false    |                                        |

* * *

## License

[Apache License 2.0](https://github.com/stack-spot/runtime-manager-action/blob/main/LICENSE)