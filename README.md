# delete-branch-package-versions

This action will retrieve the list of versions for a GH Package in the provided organization, then delete any package versions with a name that matches the pre-release format generated by [im-open/git-version-lite] (`major.minor.patch-<sanitized-branch-name>.<formated-date>`).  
  - To sanitize the branch name, the characters `a-z, A-Z, 0-9 and -` are kept and any other character is replaced with `-`.  

If the action runs into an issue deleting a specific package version, it will generate a warning that can be viewed in the Summary section of the workflow rather than failing.  Errors retrieving the package versions will still cause the action to fail though.

## Index

- [delete-branch-package-versions](#delete-branch-package-versions)
  - [Index](#index)
  - [Inputs](#inputs)
  - [Outputs](#outputs)
  - [Usage Examples](#usage-examples)
  - [Contributing](#contributing)
    - [Recompiling](#recompiling)
    - [Incrementing the Version](#incrementing-the-version)
  - [Code of Conduct](#code-of-conduct)
  - [License](#license)
  
## Inputs
| Parameter       | Is Required | Description                                                                                                                                                                                                   |
| --------------- | ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `github-token`  | true        | An access token with the `packages:delete` and `packages:read` scopes that is authorized in the organization where the package versions to be deleted are.                                                    |
| `organization`  | false       | The organization the packages were created in.  Defaults to `github.context.repo.org` if not provided.                                                                                                        |
| `branch-name`   | true        | The branch name the packages were created with.  This is how package versions to delete are identified.                                                                                                       |
| `package-type`  | true        | The type of package where versions will be deleted.  Can be one of npm, maven, rubygems, nuget, docker or container.                                                                                          |
| `package-names` | false       | The names of the packages that versions will be deleted from. Expects one value or a comma separated list (e.g. package1, package2). If ommitted, it will default to all of the packages in the current repo. |

## Outputs
No Outputs

## Usage Examples

*Delete versions of one package*
```yml
on:
  pull_request:
    types: [closed]
jobs:
  cleanup:
    runs-on: ubuntu-latest
    
    steps:
      - name: Clean up the GitHub package versions that were created for this branch
        uses: im-open/delete-branch-package-versions@v2.0.1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          organization: ${{ github.repository_owner }}
          branch-name: ${{ github.head_ref }}
          package-type: 'npm'
          package-names: 'my-pkg'
```

*Delete versions of multiple packages*
```yml
on:
  pull_request:
    types: [closed]
jobs:
  cleanup:
    runs-on: ubuntu-latest
    
    steps:
      - name: Clean up the GitHub package versions that were created for this branch
        uses: im-open/delete-branch-package-versions@v2.0.1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          organization: ${{ github.repository_owner }}
          branch-name: ${{ github.head_ref }}
          package-type: 'nuget'
          package-names: 'Some.Package.Name, Another.Package'

*Delete versions of every package in the repository in which the workflow is run*
```yml
on:
  pull_request:
    types: [closed]
jobs:
  cleanup:
    runs-on: ubuntu-latest
    
    steps:
      - name: Clean up the GitHub package versions that were created for this branch
        uses: im-open/delete-branch-package-versions@v2.0.1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          organization: 'myOrg'
          branch-name: ${{ github.head_ref }}
          package-type: 'npm'
```

## Contributing

When creating new PRs please ensure:
1. The action has been recompiled.  See the [Recompiling](#recompiling) section below for more details.
2. For major or minor changes, at least one of the commit messages contains the appropriate `+semver:` keywords listed under [Incrementing the Version](#incrementing-the-version).
3. The `README.md` example has been updated with the new version.  See [Incrementing the Version](#incrementing-the-version).
4. The action code does not contain sensitive information.

### Recompiling

If changes are made to the action's code in this repository, or its dependencies, you will need to re-compile the action.

```sh
# Installs dependencies and bundles the code
npm run build

# Bundle the code (if dependencies are already installed)
npm run bundle
```

These commands utilize [esbuild](https://esbuild.github.io/getting-started/#bundling-for-node) to bundle the action and
its dependencies into a single file located in the `dist` folder.

### Incrementing the Version

This action uses [git-version-lite] to examine commit messages to determine whether to perform a major, minor or patch increment on merge.  The following table provides the fragment that should be included in a commit message to active different increment strategies.
| Increment Type | Commit Message Fragment                     |
| -------------- | ------------------------------------------- |
| major          | +semver:breaking                            |
| major          | +semver:major                               |
| minor          | +semver:feature                             |
| minor          | +semver:minor                               |
| patch          | *default increment type, no comment needed* |

## Code of Conduct

This project has adopted the [im-open's Code of Conduct](https://github.com/im-open/.github/blob/master/CODE_OF_CONDUCT.md).

## License

Copyright &copy; 2021, Extend Health, LLC. Code released under the [MIT license](LICENSE).

[im-open/git-version-lite]: https://github.com/im-open/git-version-lite
[git-version-lite]: https://github.com/im-open/git-version-lite
