# delete-branch-package-versions

This action will retrieve the list of versions for a GH Package in the provided organization and repository, then delete any package versions with a name that matches the pre-release format generated by [im-open/git-version-lite] (`major.minor.patch-<sanitized-branch-name>.<formated-date>`).

- To sanitize the branch name, the characters `a-z, A-Z, 0-9 and -` are kept and any other character is replaced with `-`.

If the action runs into an issue deleting a specific package version, it will generate a warning that can be viewed in the Summary section of the workflow rather than failing.  Errors retrieving the package versions will still cause the action to fail though.

## Index <!-- omit in toc -->

- [delete-branch-package-versions](#delete-branch-package-versions)
  - [Inputs](#inputs)
  - [Outputs](#outputs)
  - [Usage Examples](#usage-examples)
  - [Contributing](#contributing)
    - [Incrementing the Version](#incrementing-the-version)
    - [Source Code Changes](#source-code-changes)
    - [Recompiling Manually](#recompiling-manually)
    - [Updating the README.md](#updating-the-readmemd)
    - [Tests](#tests)
  - [Code of Conduct](#code-of-conduct)
  - [License](#license)

## Inputs

| Parameter       | Is Required | Description                                                                                                                                                                                                  |
|-----------------|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `github-token`  | true        | An access token with the `packages:delete` and `packages:read` scopes that is authorized in the organization where the package versions to be deleted are.                                                   |
| `organization`  | false       | The organization the packages were created in.  Defaults to `github.context.repo.org` if not provided.                                                                                                       |
| `repository`    | false       | The repository the packages are in.  Defaults to `github.context.repo.repo` if not provided.                                                                                                                 |
| `branch-name`   | true        | The branch name the packages were created with.  This is how package versions to delete are identified.                                                                                                      |
| `package-type`  | true        | The type of package where versions will be deleted.  Can be one of npm, maven, rubygems, nuget, docker or container.                                                                                         |
| `package-names` | false**     | The names of the packages that versions will be deleted from. Expects one value or a comma separated list (e.g. package1, package2). If omitted, it will default to all of the packages in the current repo. |
| `strict-match-mode` | false       | Flag that determines the pattern the action will use to identify matches in the release name and tag.  Defaults to `true`.<br/>• `true: -<sanitized-branch-name>.` Releases created with [git-version-lite] tags follow this pattern.<br/>• `false: <sanitized-branch-name>` |

** *Note: When using versions prior to 3.0.0, `package-names` must be provided if the `package-type` is not `debian` or `pypi`.  GitHub officially dropped support for querying `npm`, `rubygems`, `maven`, `docker` and `nuget` packages associated with a repo through the GraphAPI on June 01, 2023, which previous versions of the action used.  Version 3.0.0 and later use the REST API to find packages associated with a repo when the `package-names` input is not provided.*

## Outputs

No Outputs

## Usage Examples

*Delete versions that match the specified branch for one package in the org and repo where the workflow is run*

```yml
on:
  pull_request:
    types: [closed]
jobs:
  cleanup:
    runs-on: ubuntu-latest

    steps:
      - name: Clean up the GitHub package versions that were created for this branch
        uses: im-open/delete-branch-package-versions@v3.2.0
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          branch-name: ${{ github.head_ref }}
          package-type: 'npm'
          package-names: 'my-pkg'
```

*Delete versions that match the specified branch for multiple packages in the org and repo where the workflow is run*

```yml
on:
  pull_request:
    types: [closed]
jobs:
  cleanup:
    runs-on: ubuntu-latest

    steps:
      - name: Clean up the GitHub package versions that were created for this branch
        # You may also reference just the major or major.minor version
        uses: im-open/delete-branch-package-versions@v3.2.0
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          branch-name: ${{ github.head_ref }}
          package-type: 'nuget'
          package-names: 'Some.Package.Name,Another.Package'
```

*Delete versions that match the specified branch for every package in the specified org and repository*

```yml
on:
  pull_request:
    types: [closed]
jobs:
  cleanup:
    runs-on: ubuntu-latest

    steps:
      - name: Clean up the GitHub package versions that were created for this branch
        uses: im-open/delete-branch-package-versions@v3.2.0
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          organization: 'mySpecifiedOrg'
          repository: 'mySpecifiedRepo'
          branch-name: ${{ github.head_ref }}
          package-type: 'npm'
```

## Contributing

When creating PRs, please review the following guidelines:

- [ ] The action code does not contain sensitive information.
- [ ] At least one of the commit messages contains the appropriate `+semver:` keywords listed under [Incrementing the Version] for major and minor increments.
- [ ] The action has been recompiled.  See [Recompiling Manually] for details.
- [ ] The README.md has been updated with the latest version of the action.  See [Updating the README.md] for details.
- [ ] Any tests in the [build-and-review-pr] workflow are passing

### Incrementing the Version

This repo uses [git-version-lite] in its workflows to examine commit messages to determine whether to perform a major, minor or patch increment on merge if [source code] changes have been made.  The following table provides the fragment that should be included in a commit message to active different increment strategies.

| Increment Type | Commit Message Fragment                     |
|----------------|---------------------------------------------|
| major          | +semver:breaking                            |
| major          | +semver:major                               |
| minor          | +semver:feature                             |
| minor          | +semver:minor                               |
| patch          | *default increment type, no comment needed* |

### Source Code Changes

The files and directories that are considered source code are listed in the `files-with-code` and `dirs-with-code` arguments in both the [build-and-review-pr] and [increment-version-on-merge] workflows.

If a PR contains source code changes, the README.md should be updated with the latest action version and the action should be recompiled.  The [build-and-review-pr] workflow will ensure these steps are performed when they are required.  The workflow will provide instructions for completing these steps if the PR Author does not initially complete them.

If a PR consists solely of non-source code changes like changes to the `README.md` or workflows under `./.github/workflows`, version updates and recompiles do not need to be performed.

### Recompiling Manually

This command utilizes [esbuild] to bundle the action and its dependencies into a single file located in the `dist` folder.  If changes are made to the action's [source code], the action must be recompiled by running the following command:

```sh
# Installs dependencies and bundles the code
npm run build
```

### Updating the README.md

If changes are made to the action's [source code], the [usage examples] section of this file should be updated with the next version of the action.  Each instance of this action should be updated.  This helps users know what the latest tag is without having to navigate to the Tags page of the repository.  See [Incrementing the Version] for details on how to determine what the next version will be or consult the first workflow run for the PR which will also calculate the next version.

### Tests

The [build-and-review-pr] workflow includes tests which are linked to a status check. That status check needs to succeed before a PR is merged to the default branch.  When a PR comes from a branch, the workflow has access to secrets which are required to run the tests successfully.

When a PR comes from a fork, the workflow cannot access any secrets, so the tests won't have the necessary permissions to run. When a PR comes from a fork, the changes should be reviewed, then merged into an intermediate branch by repository owners so tests can be run against the PR changes.  Once the tests have passed, changes can be merged into the default branch.

## Code of Conduct

This project has adopted the [im-open's Code of Conduct](https://github.com/im-open/.github/blob/main/CODE_OF_CONDUCT.md).

## License

Copyright &copy; 2023, Extend Health, LLC. Code released under the [MIT license](LICENSE).

<!-- Links -->
[Incrementing the Version]: #incrementing-the-version
[Recompiling Manually]: #recompiling-manually
[Updating the README.md]: #updating-the-readmemd
[source code]: #source-code-changes
[usage examples]: #usage-examples
[build-and-review-pr]: ./.github/workflows/build-and-review-pr.yml
[increment-version-on-merge]: ./.github/workflows/increment-version-on-merge.yml
[esbuild]: https://esbuild.github.io/getting-started/#bundling-for-node
[git-version-lite]: <<https://github.com/im-open/git-version-lite>
[im-open/git-version-lite>]: <https://github.com/im-open/git-version-lite>
