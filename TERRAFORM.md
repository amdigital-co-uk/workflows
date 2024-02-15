# Shared AWS Terraform workflows

## Terraform Plan

This workflow is usually run in the context of a Pull Request, and assumes that the terraform repository is structured as follows:

- `components/{component-name}/*.tf` - the terraform files for the component
- `components/{component-name}/{env}.tfvars` - the terraform variables for a given environment

It will use the terraform CLI to plan the changes that would be made to the infrastructure if the PR were to be merged. It will then upload the plan to S3 and comment on the PR with a summary of the plan.

It will optionally load in secrets from AWS Secrets manager as an additional source of terraform variables.

Finally, it is also possible to configure Checkov to run on the terraform files. The workflow will fail if any Checkov checks fail, and it is also possible to skip certain checks.


```yml
  plan:
    name: "Plan COMPONENT Updates"
    uses: amdigital-co-uk/workflows/.github/workflows/terraform-component-plan.yml@v1
    with:
      component: COMPONENT
      plan-id: ${{ github.event.number }}
      aws-role-arn: ${{ vars.ROLE_ARN }}
      terraform-version: 1.3.7              # Optional, defaults to 1.1.5
      use-aws-secret: true                  # Optional, defaults to false. If set to true, also populate the below
      aws-secret-name: secret/key           # Optional, the name of the secret in AWS Secrets
      append-workspace-to-secret: false     # Optional, defaults to false. If true, workspace name gets appended to the secret name
      s3-bucket: my-s3-bucket               # Optional, defaults to our standardd bucket. S3 bucket to store the plan in
      s3-key: /plans                        # Optional, defaults to /plans. S3 path to store for the plan in
      checkov-enabled: true                 # Optional, defaults to false. Whether to run Checkov on the terraform files
      checkov-skip-checks: "CKV_AWS_247"    # Optional, defaults to empty. A comma-separated list of check IDs to skip
```

### Terraform destroy

If adding the `destroy` label to a Pull Request, the workflow will instead create a plan to destroy the infrastructure instead of updating it. **Use with caution**.

## Terraform Apply

Usually run in the context of a _merged_ Pull Request. It will use the terraform CLI to apply the changes that were planned in the PR. It will then upload the plan to S3 and comment on the PR with a summary of the plan. Finally, it will post any Terraform outputs as a comment on the PR.

```yml
jobs:
  apply:
    if: ${{ github.event.pull_request.merged == true }}
    name: "Apply elasticsearch Changes"
    uses: amdigital-co-uk/workflows/.github/workflows/terraform-component-apply.yml@v1
    with:
      component: COMPONENT
      plan-id: ${{ github.event.number }}
      aws-role-arn: ${{ vars.ROLE_ARN }}
      terraform-version: 1.3.7              # Optional, defaults to 1.1.5
      s3-bucket: my-s3-bucket               # Optional, defaults to our standardd bucket. S3 bucket to store the plan in
      s3-key: /plans                        # Optional, defaults to /plans. S3 path to store for the plan in

```