# Shared NodeJS and TypeScript related workflows

## Build a Docker application and push to ECR

This is very similar to the [equivalent .NET workflow](./DOTNET.md#package-net-core-application-as-docker-image-and-push-to-aws-ecr) but wihtout the additional templated parameters that make our .NET applications able to use identical `Dockerfile`s.

This currently assumes a branching strategy of `stage/{name}` >> tagname `{name}`, with anything else mapping to the default `docker-tag` input.

```yml
jobs:
  build:
    if: ${{ github.event.pull_request.merged == true || github.event_name == 'workflow_dispatch' }}
    uses: amdigital-co-uk/quartex-workflows/.github/workflows/docker.yml@v1
    with:
      ref: ${{ github.ref }}
      docker-namespace: samples
      docker-image: sample-docker-image
      docker-tag: live # default tag to use when there
      workdir: api # working directory (if not the root of the repo), which should contain the Dockerfile
      build-npmrc: true
      ecr-region-1: us-east-1
      ecr-region-2: us-east-2
      aws-role-arn: ${{ vars.PIPELINE_TESTS_ARN }}
    secrets:
      PKG_TOKEN: ${{ secrets.PKG_TOKEN }}
```

## Deploy a NodeJS Lambda function using Docker & Serverless

```yml
jobs:
  build:
    if: ${{ github.event.pull_request.merged == true || github.event_name == 'workflow_dispatch' }}
    uses: amdigital-co-uk/quartex-workflows/.github/workflows/node-lambda.yml@v1
    with:
      ref: ${{ github.ref }}
      docker-namespace: qtfn
      docker-image: sample-docker-image
      stage: live
      ecr-region-1: us-east-1
      ecr-region-2: us-east-2
      aws-role-arn: ${{ vars.PIPELINE_TESTS_ARN }}
```

## Run unit tests for a NodeJS application

```yml
jobs:
  mocha:
    if: ${{ github.event.pull_request.merged == true || github.event_name == 'workflow_dispatch' }}
    uses: amdigital-co-uk/quartex-workflows/.github/workflows/node-tests.yml@v1
    with:
      coverage-s3-path: s3://my-s3-bucket/code-coverage-reporting/my-repo.csv # this path should be a unique CSV file for each repo
      debug: TRUE
      aws-region: us-east-1
      aws-role-arn: ${{ vars.PIPELINE_TESTS_ARN }}
    secrets:
      PKG_TOKEN: ${{ secrets.PKG_TOKEN }}
```