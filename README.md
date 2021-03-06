# Yet Another Git Strategy

> The Last Git Branching Model You'll Ever Need

# Key Benefits

- Opinionated
- Simplifies CI/CD
- Supports Hotfixes
- Good Tooling Support
- Works with all sizes of teams
- Improves Collaboration

# Table of Contents

- [Overview](#overview)
- [How It Works](#how-it-works)
  - [Feature Branches](#feature-branches)
  - [Commits](#commits)
  - [Releases](#releases)
  - [Pre-Releases](#pre-releases)
  - [Hotfixes](#hotfixes)
  - [CI/CD](#ci-cd)
- [Commit Types](#commit-types-based-of-the-angular-convention)
- [Reccomendations](#reccomendations)
- [Inspiration](#inspiration)

# Overview

![Graph](./graph.svg)

# How It Works

We have one main branch in our Git-Repository called `develop`. This is our current development version, which is always deployed to our staging environment. All other branches are started off of this default branch.
We also have a `production` branch. This is the code that is currently running in production.

## Feature Branches

Generally, new code is written in new, short-living branches, however this model does allow for small and concise changes to be pushed directly into the `develop` branch. This is only acceptable if the changes are small or cosmetic, have already been tested and don't require a code review.

For all other code, you should create a new branch off of develop. These have a naming convention:
`<type>/<branch-name>` (e.g `feat/user-info-api`)

```bash
$ git checkout develop
$ git checkout -b feat/my-new-feature
```

While working, track your progress in a [draft (github)](https://github.blog/2019-02-14-introducing-draft-pull-requests/) or [Work In Progress Pull-Request (gitlab)](https://docs.gitlab.com/ee/user/project/merge_requests/work_in_progress_merge_requests.html) and request a review from at least one teammate when you have finished your feature. When all checks pass (assuming you have set up [CI/CD](#ci-cd)) you [squash and merge](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-request-merges#squash-and-merge-your-pull-request-commits) the branch into develop (It's recommended to [require a linear history](https://help.github.com/en/github/administering-a-repository/requiring-a-linear-commit-history) and [disable other merge types](https://help.github.com/en/github/administering-a-repository/configuring-commit-squashing-for-pull-requests) on your respective git host).

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

Before releasing a version to production, it is recommended to sanitize-check your current staging environment — even if all tests pass. If you are confident, the next step is to (optionally) commit a version bump to `develop`:

```bash
$ git checkout develop
$ # bump your version to v0.1.0
$ git commit -m "chore: bump version to v0.1.0"
$ git tag v0.1.0
$ git push --tags
```

The final step then is merging this to your `production` branch:

```bash
$ git checkout production
$ git merge develop --ff-only
$ git push
```

## Pre-Releases

Pre-Releases are just tagged development versions. To for example automate pushing your library to npm or similar, you need to add a release step to your CI/CD system that runs on every tag `*-{alpha,beta}.*`

```bash
$ git checkout develop
$ # bump your version to v0.1.0-beta.0
$ git commit -m "chore: bump version to v0.1.0-beta.0"
$ git tag v0.1.0-beta.0
$ git push --tags
```

## Hotfixes

Hotfixes should be created as pull-request from a fix-branch directly to production. These can then be reviewed and squash-merged like normal branches.

```bash
$ git checkout production
$ git checkout -b "hotfix/fix-password-hash"
```

After a hotfix lands in production, a more elaborate fix can be created for develop or the fix can be cherry-picked:

```bash
$ git checkout develop
$ git cherry-pick 3af9286 # the commit sha of the hotfix
```

## CI/CD

Continuous Integration and Continuous Delivery are crucial parts of YAGS. Depending on your setup, this can look very different, but the general recommendation is:

- `on every commit, anywhere`: Run all tests and try to build
- `on every commit to develop`: Deploy to your staging environment
- `on every commit to production`: Deploy to your production environment

# Commit/Branch Types

These add human machine-readable meaning to your commit messages and branches (based on the [Angular Commit Convention](https://github.com/angular/angular/blob/22b96b9/CONTRIBUTING.md#type) / [Conventional Commits](https://www.conventionalcommits.org/))

- `feat` - A new feature
- `fix` - A bug fix
- `chore` - Changes that don't affect any code, e.g. releases
- `build` - changes in the build system
- `docs` - Documentation
- `style` - Cosmetic code changes
- `refactor` - A code change that doesn't fix/add anything new
- `perf` - A code change that improves performance
- `test` - Adding or correcting tests

# Recommendedations

## Tools

While this git strategy will work with any combination of build/test tools, these can serve as a good starting point for new projects

### NodeJS/JavaScript/Typescript

- Pre-Commit Hooks with [Husky](https://www.npmjs.com/package/husky)
  - one hook running your tests
  - one hook running [commitlint](https://github.com/conventional-changelog/commitlint)
- ESLint for linting TypeScript/JavaScript code (optionally with my [eslint preset](https://www.npmjs.com/package/@explodingcamera/eslint-config))
- [release-it](https://www.npmjs.com/package/release-it) with @release-it/conventional-changelog and github releases enabled

## Opensource Projetcts

These are some recommendations specifically for opensource-projects. They don't really have a lot to do with this strategy but could be helpful.

- Add a [Code of Conduct](https://www.contributor-covenant.org/) to your project
- Choose a [simple Open-Source License](https://choosealicense.com/)

# Inspiration

I've been heavily inspired by [Trunk Based Development](https://trunkbaseddevelopment.com/), the [GitHub Flow](https://guides.github.com/introduction/flow/) and the [GitLab Flow](https://docs.gitlab.com/ee/topics/gitlab_flow.html).
With YAGS, I've aimed to take the best Parts of all of these and create a lightweight, opinionated and very practical alternative.
