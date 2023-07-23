
## Overview

This repository contains workflows used by the software team behind the [Quartex](https://www.quartexcollections.com/) platform, and form part of our CI/CD pipeline. This repository is public (as it is the only way Workflows can be be shared between repositories), although most of our other repositories are not.

This is an experimental replacement for [quartex-workflows](https://github.com/amdigital-co-uk/quartex-workflows), experimenting with a new way of managing and versioning workflows, along with using more consistent naming conventions for inputs etc.

## Built with

- [GitHub Actions](https://docs.github.com/en/actions)
- [bash](https://www.gnu.org/software/bash/)

## Contributing

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

This repository uses git tags with [semantic versioning](https://semver.org/).

After merging changes into main, you can create a new set of tags by using the [Create Tags](https://github.com/amdigital-co-uk/workflows/actions/workflows/tag.yml) workflow. For instance when tagging using `v1.0.0`, the workflow will create individual tags called `v1`, `v1.0` and `v1.0.0`. Then, when subsequently tagging using `v1.0.1`, the `v1` and `v1.0` tags will get updated to the latest commit, and the new `v1.0.1` tag will get created.

This allows calling workflows to reference a MAJOR, MINOR or PATCH version of a workflow file, depending on how specific an iteration of it is required. For instance, one workflow could use `v1`, which will automatically import any MINOR or PATCH updates, whilst another workflow could use `v1.0.1` if a specific behaviour is desired.
