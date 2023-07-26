# Shared .NET workflows

## Run .NET Core automated tests via XUnit

### Check branch coverage against Static Threshold

Default behaviour when setting `branch-threshold` is to report test coverage and fail the workflow if it does not meet the threshold.

```yml
jobs:
  xunit:
    uses: amdigital-co-uk/quartex-workflows/.github/workflows/xunit.yml@v1
    with:
      docker-compose: Quartex.Sample.Service.IntegrationTests/docker-compose.yml
      branch-threshold: 80
    secrets:
      PKG_TOKEN: ${{ secrets.PKG_TOKEN }}
```

### Check branch coverage against historic data

Alternatively, when omitting `branch-threshold` you can instead set the following parameters and secret values, to have the workflow check current test coverage against the last recorded test coverage for the same repository (test coverage values are stored in a CSV in S3 at the configured location). This is why we need to pass AWS authentication details to this workflow.

If coverage goes down, you see a failure; if it is maintained or improved, the new value will be persisted back to S3 as the new threshold for future runs.

```yml
jobs:
  xunit:
    uses: amdigital-co-uk/quartex-workflows/.github/workflows/xunit.yml@v1
    with:
      docker-compose: Quartex.Sample.Service.IntegrationTests/docker-compose.yml
      coverage-s3-path: s3://my-s3-bucket/code-coverage-reporting/my-repo.csv # this path should be a unique CSV file for each repo
      aws-region: us-west-2   # default is us-east-1
      aws-role-arn: ${{ vars.PIPELINE_TESTS_ARN }}
      never-fail-at: 99     # default is 95
    secrets:
      PKG_TOKEN: ${{ secrets.PKG_TOKEN }}
```

## Create and push NuGet packages to private GitHub packages feed

```yml
jobs:
  nuget:
    uses: amdigital-co-uk/quartex-workflows/.github/workflows/nuget.yml@v1
    with:
      pkg-src: https://nuget.pkg.github.com/amdigital-co-uk/index.json
      projects: "Quartex.Sample.Common Quartex.Sample.Interfaces"
      skip-duplicate: false
    secrets:
      PKG_TOKEN: ${{ secrets.PKG_TOKEN }}
```

## Package .NET Core application as Docker image and push to AWS ECR

Note this assumes our standard format of `Dockerfile`. This allows all our .NET applications to use an identical `Dockerfile` by passing through a number of parameters into the `docker build` command.

```yml
jobs:
  dotnet:
    if: ${{ github.event.pull_request.merged == true || github.event_name == 'workflow_dispatch' }}
    uses: amdigital-co-uk/quartex-workflows/.github/workflows/dotnet.yml@v1
    with:
      ref: ${{ github.ref }}
      project: Quartex.Sample.Service
      component: sample-service
      docker-namespace: samples
      docker-image: sample-docker-image
      docker-tag: latest
      ecr-region-1: us-east-1
      ecr-region-1: us-east-2
      aws-role-arn: ${{ vars.PIPELINE_TESTS_ARN }}
      configs: org/config-repo # Optional: specify a repo to checkout and retrieve JSON configs from
    secrets:
      PKG_TOKEN: ${{ secrets.PKG_TOKEN }}
      SRC_TOKEN: ${{ github.token }} # Must be specified if CONFIGS is set; must be a token that has read access to the repo
```