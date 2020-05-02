# Yet Another Git Strategy -  The Last git branching model you'll ever need

https://www.conventionalcommits.org/en/v1.0.0/
`<type>[optional scope]: <description>`
`refactor: rename x to y`
`refactor(api): rename x to y`
- Commits AND Branches (e.g `feat/asdfasdf`)
  
# Key Benefits

- Easy Continuous Integration
- Opinionated
- Supports Hotfixes
- Colaboration
- Tooling Support
- Works with all sizes of teams

# How It Works

We have one main branch in our Git-Repository called `develop`. This is our current development version, always deployed as our staging environment. All other branches are started off of this default branch.
We also have a `master` branch. This is the code that is currently running in production.

## "Feature' Branches

Generally, new code is written in new, short-living branches, however this model does allow for small and concise changes to be pushed directly into the `develop` branch. This is only acceptable if the changes are small or cosmetic, have already been tested and don't require a code review.
  
For all other code, you should create a new branch off of develop. These have a naming convention:
`<type>/<branch-name>` (e.g `feat/user-info-api`)

`$ git checkout develop`
`$ git checkout -b feat/my-new-feature`

While working, track your progress in a [draft (github)](https://github.blog/2019-02-14-introducing-draft-pull-requests/) or Work In Progress Pull-Request and request a review from at least one Team-Mate when your finished with your feature. When all checks pass (assuming you have set up [CI/CD](# CI/CD)) you [squash and merge](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-request-merges#squash-and-merge-your-pull-request-commits) the branch into develop (It's recommended to [require a linear history](https://help.github.com/en/github/administering-a-repository/requiring-a-linear-commit-history) and [disable other merge types](https://help.github.com/en/github/administering-a-repository/configuring-commit-squashing-for-pull-requests) on your respective git host).

## Commits
When commiting your code to these branches or `develop`, you also need to follow a naming convention (based on [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)):
```
  <type>[optional scope]: <description>

  [optional body]

  [optional footer(s)]
```

```bash
$ git commit -m "feat: add database user model" \
  -m "BREAKING: This changes some names in our rest-api"
```

## Releases
Before releasing a version to production, it is recomended to sanitize-check your current staging environment - even if all tests pass. If you are confident, the next step is to (optionally) commit a version bump to develop

```bash
$ git checkout develop
$ # bump your version to v0.1.0
$ git commit -m "chore: bump version to v0.1.0"
```

The final step then is merging this to your `master` branch and tagging the version

```bash
$ git checkout master
$ git merge develop -m "release v0.1.0"
$ git tag v0.1.0
$ git push --tags
```

## Pre-Releases

Pre-Releases are just tagged development versions. To automate pushing e.g your library to npm or similar, you need to add a realease step to your CI/CD system that runs on every tag `*-{alpha,beta}.*` 

```bash
$ git checkout develop
$ # bump your version to v0.1.0-beta.0
$ git commit -m "chore: bump version to v0.1.0-beta.0"
$ git tag v0.1.0-beta.0
$ git push --tags
```

## Hotfixes

Hotfixes should be created as pull-request from a fix-branch directly to master. These can then be reviewed and squash-merged like normal branches. 

```bash
$ git checkout master
$ git checkout -b "hotfix/fix-password-hash"
```

After a hotfix lands in production, a more elaborate fix can be created for develop or the fix can be cherry-picked:

```bash
$ git checkout develop
$ git cherry-pick 3af9286 # the commit sha of the hotfix
```

## CI/CD
Continuous Integration and Continuous are crucial parts of YAGS. Depending on your setup, this can look very different, but the general recomendation is:

- `on every commit, anywhere`: Run all tests and try to build
- `on every commit to develop`: Deploy to your staging environment
- `on every commit to master`: Deploy to your production environment

# Commit Types (Based of the [Angular Convention](https://github.com/angular/angular/blob/22b96b9/CONTRIBUTING.md#type))

- `feat` - A new feature
- `fix` - A bug fix
- `chore` - Changes that don't affect any code, e.g releases 
- `build` - changes in the build system
- `docs` - Documentation
- `style` - Cosmetic code changes
- `refactor` - A code change that doesn't fix/add anything new
- `perf` - A code change that imporoves performance
- `test` - Adding or correcting tests

# Recommendedations
## Tools
### NodeJS/JavaScript/Typescript
- Pre-Commit Hooks with [Husky](https://www.npmjs.com/package/husky)
  - one hook running your tests
  - one hook running [commitlint](https://github.com/conventional-changelog/commitlint)
- ESLint for linting TypeScript/JavaScript code (optionally with my [eslint preset](https://www.npmjs.com/package/@explodingcamera/eslint-config))

## Opensource Projetcts
* [https://www.contributor-covenant.org/](A Code of Conduct)
* [A simple Open-Source License like the MIT-License](https://choosealicense.com/)

# Inspiration

https://trunkbaseddevelopment.com/
https://guides.github.com/introduction/flow/
https://docs.gitlab.com/ee/topics/gitlab_flow.html