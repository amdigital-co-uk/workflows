
## Overview

This repository contains workflows used by the software team behind the [Quartex](https://www.quartexcollections.com/) platform, and form part of our CI/CD pipeline. This repository is public (as it is the only way Workflows can be be shared between repositories), although most of our other repositories are not.

This is an experimental replacement for [quartex-workflows](https://github.com/amdigital-co-uk/quartex-workflows), experimenting with a new way of managing and versioning workflows, along with using more consistent naming conventions for inputs etc.

## Built with

- [GitHub Actions](https://docs.github.com/en/actions)
- [bash](https://www.gnu.org/software/bash/)
- [semver CLI tool](https://github.com/fsaintjacques/semver-tool#installation)

## Usage

Note that our secrets are defined at organisation level. So whilst shouldn't need defining within the repository, they do need to be explicitly passed in to the called workflow.

- [Shared .NET workflows](./DOTNET.md)
- [Generic Docker build and push to ECR](./OTHER.md#build-a-docker-application-and-push-to-ecr)
- [Publish an AWS Lambda using Serverless](./OTHER.md#deploy-a-nodejs-lambda-function-using-docker--serverless)
- [Run NodeJS unit tests](./OTHER.md#run-unit-tests-for-a-nodejs-application)

## Contributing

When editing workflows, the use of VSCode is highly encouraged, along with the [GitHub Actions extension](https://marketplace.visualstudio.com/items?itemName=cschleiden.vscode-github-actions) which provides syntax highlighting, validation and intellisense for parameters.

### Naming Conventions and other Style Rules

- Use `kebab-case` for workflow inputs (as well as Job and Step names)
- Use `ALL_CAPS_SNAKE_CASE` for secret names, and for environment variables
- Always give Workflows and Jobs a display name using the `name` property in the YML

### Branches

This repository uses trunk-based development, with short-lived work branches getting merged into main via pull requests. Tags and refs are very important, since that is how calling workflows will reference specific versions of the workflows in this repository.

The `main` branch is protected with a branch protection rule, so all work must be done in a temporary branch.

Create succinct branch names, as that will allow you to reference the work-in-progress versions of your workflows via that git ref. E.g. if working on improvements to the performance of a workflow, you could create a branch called `vPerf`, and you would then be able to reference this WIP version of the workflow by branch name from another repo, e.g:

```yml
jobs:
  xunit:
    uses: amdigital-co-uk/workflows/.github/workflows/xunit.yml@vPerf
    # etc
```

Use this approach to fully test any major changes before attempting to merge your changes into `main` You can then issue a pull request to merge your temp branch into `main`, and finally create a new tag (see below).

### Tagging

This repository uses git tags with [semantic versioning](https://semver.org/). When creating a pull request to merge changes into `main`, you should also add the corresponding label to indicate whether the changes are a MAJOR, MINOR or PATCH version. This will then be used to automatically create the appropriate tags when the pull request is merged.

Assume for instance that the current version is `v1.0.0`. This means there will be a `v1` tag, a `v1.0` tag and a `v1.0.0` tag. Creating a pull request to merge changes that are a MINOR version bump, you should label the pull request as **minor**. Finally, when the pull request is merged, the following tags will be created/updated:

- `v1` will get updated to point to the latest commit on `main`
- `v1.1` will get created to point to the latest commit on `main`
- `v1.1.0` will get created to point to the latest commit on `main`

This allows *calling* workflows from other repositories to reference a MAJOR, MINOR or PATCH version of a workflow file, depending on how specific an iteration of it is needed, or how tolerant of changes the calling workflow is. For instance, one workflow could use `reusable.yml@v1`, which will automatically import any MINOR or PATCH updates, whilst another workflow could use `reusable.yml@v1.0.1` if a specific behaviour is desired.
