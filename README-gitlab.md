# GitLab CI/CD Workflow for Runtimes Tasks Integrated Action

This GitLab CI/CD workflow runs the Runtimes Tasks Integrated Action with the specified parameters.

## Inputs

The following environment variables must be configured in your GitLab CI/CD settings:

- `TASK_LIST`: List of tasks (required)
- `FEATURES_LEVEL_LOG`: Log Level (required)
- `REPOSITORY_NAME`: Git Repository Name (required)
- `AWS_REGION`: AWS REGION (required)
- `CONTAINER_URL_IAC`: IAC Container url (optional, default: stackspot/runtime-job-iac:latest)
- `CONTAINER_URL_DEPLOY`: Deploy Container url (optional, default: stackspot/runtime-job-deploy:latest)
- `CONTAINER_URL_DESTROY`: Destroy Container url (optional, default: stackspot/runtime-job-destroy:latest)
- `FEATURES_TERRAFORM_MODULES`: Terraform Modules (optional)
- `PATH_TO_MOUNT`: Path to mount inside the docker (required)
- `OUTPUT_FILE`: File name to save outputs: {plugin-alias}_{OUTPUT_FILE} (optional, default: outputs.json)
- `LOCALEXEC_ENABLED`: If Runtimes will allow execution of the local-exec command within terraform (optional, default: false)
- `TF_LOG_PROVIDER`: Level tf log provider - info, debug, warn or trace (optional)
- `CHECKOUT_BRANCH`: Whether or not checkout is enabled (optional, default: false)
- `BASE_PATH_OUTPUT`: Base Path Output (optional)
- `CLIENT_ID`: CLIENT ID (required)
- `CLIENT_KEY`: CLIENT KEY (required)
- `CLIENT_REALM`: CLIENT REALM (required)
- `AWS_ACCESS_KEY_ID`: AWS ACCESS KEY ID from console (optional)
- `AWS_SECRET_ACCESS_KEY`: AWS SECRET ACCESS KEY from console (optional)
- `AWS_SESSION_TOKEN`: AWS SESSION TOKEN from console (optional)
- `AWS_ROLE_ARN`: AWS ROLE ARN (optional)

## Usage

To use this workflow, add the above environment variables to your GitLab CI/CD settings and include the `.gitlab-ci.yml` file in your repository.

```yaml
include:
  - local: '.gitlab-ci.yml'