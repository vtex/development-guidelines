# VTEX Development Guideline

## Summary 

- [Intro](#intro)
  - [Objective](#objectives)
- [Concepts](#concepts)
  - [Commits](#commits)
  - [Branches](#branches)
    - [Change Branches](#change-branches)
- [Workflow](#workflow)
  - [Scenario: I want to develop a feature improvement ](#scenario-i-want-to-develop-a-feature-improvement)
  - [Issues and Pull Requests Templates](#issues-and-pull-requests-templates)


## Intro

### Disclaimer
This guide presumes that you know how to use git and the basic concepts of *commit, rebase* and *merge*. 

### Objective
There is no right way to use Git. Therefore, we are defining our way to approach the change management of our codebase. This document is the guidance on using git workflow to simplify our development lifecycle.

## Concepts

### Commits

By simplicity, we have three types of commits:
- __*commits*__: commits made by the user 
- __*merge commits*__: commits through the commend `git merge <branch> --no-ff`
- __*release commits*__: commits using [releasy](https://www.npmjs.com/package/releasy) tool

[Releasy](https://www.npmjs.com/package/releasy) is a tool to simplify the versioning process of a project. It creates a commit with a new version in the `package.json` and creates a *tag* version to this commit.

Commit messages should be written in English following the model: 
```
<Imperative verb> <object>
```

Examples:
- Add PayPal Plus as a new payment method
- Fix profile not showing
- Update Service Router
- Remove unused dependency


What *not* to do: 
- Add dot in the end of text Ex: "Update Service Router."
- Start with lowercase
- Write in Portuguese

### Branches

Every repository has a default branch called `main` (or `master` for old ones) with protected rules enforcing a peer review process. 

The `main` branch must reflect exactly what is in production, it should be treated as __*the single source of truth*__. It is from the `main` that every development branch is based. 

We have projects that also use a `beta` branch as permanent. This approach is adopt when the team wants to release several changes at once in production.
For those who adopt this approach, be aware that the `beta` branch can be constant override by the team. So, don't keep anything saved only in `beta`.

**Important note:** Only *merge commits* should be made on `main` and `beta` branches.

#### Change branches

You must create a branch based on the `main` to start a feature, improvement, or fix. We called this of *change branch*. It must have the following structure name: `<type>/<description>`
The `types` are:
- **feature:** Build a new feature or improvement
  - _feature/add-postal-code_
- **fix:** Fix a bug
  - _fix/_
- **update:** Update project dependencies, docs, runbooks, and so on
  - _update/otel-library_
- **chore:**  Refact to reduce complexity, remove unused code
  - _chore/remove-toil_
 
A description must be short in kebab-case. It should give a basic understanding of what is being developed on the branch. E.g., `git checkout -b feature/paypal-plus`.

**Important note:** Only commits and code reviews should be done in a change branch. Avoid **releasing** to production directly from this branch.

## Workflow

### Scenario: I want to develop a feature improvement 

**Step 1.** Create a *change branch* based on `main`.
```sh
git checkout main
git checkout -b feature/nice-new-thing
```

**Step 2.** Develop the improvement in this *branch* making commits. 
```sh
git commit -m "Add nice new thing"
```

**Step 3.** Do it a *merge commit* of this *change branch* in `beta` with a flag `--no-ff`.
```sh
git checkout beta
git merge feature/nice-new-thing --no-ff
```

**Step 4.** (optional - for those who use beta) Do it a *release commit* on `beta` using releasy.
If it is the only *change branch* in `beta`, use: `releasy`.
If it isn't, use `releasy pre`.

**Important:** Certify that a published version has `-beta` suffix.

**Step 5.** After publishing the version, wait for the build on Pachamama to end and change the version published in Delorean.

**Step 6.** Test your improvement on the *beta* environment (vtexcommercebeta). If you find bugs,  go to your *change branch* back to step 2. Otherwise, continue to step 7. 

**Step 7.** Open a Pull Request and ask for a review. PRs are our friends. They ensure that we share knowledge, raise the bar of our code quality, and eventually find bugs. After your PR is approved, go to Step 8. 

**Step 8.** Do it a *release commit* in the *change branch* using the releasy.
- Add a *minor* version in case of a "feature" or "update": `releasy minor --stable`
- Add a *patch* version in case of a "fix": `releasy patch --stable`

**Important:** Verify that the version doesn't contain the `-beta` suffix. If it has, you forgot to use the *flag* `--stable` in releasy.

**Step 9.** Publish the *change branch* in production: 
Put your commits in `main`, clicking in **Merge Pull Request**. 

**Step 10.** After publishing the new version, wait for the build ends on Pachamama and change the version in Delorean.

**Step 11.** Verify that the changes are in *stable* (vtexcommercestable) and monitor the metrics to certify that your deployment didn't degrade the platform or your service. If you see something wrong, make the rollback in Delorean **immediately**! You can do the troubleshooting later in *beta*.

**Step 12.** Celebrate! 🥳 You published something in production for millions of people. 

### Scenario: Someone update the `main` branch, and I'm developing something on my *change branch* 

Make *rebase* of your *change branch*.

```sh
git checkout main
git pull
git checkout feature/nice-new-thing
git rebase main
git push origin feature/nice-new-thing -f
```

**Important note:** Always maintain your *change branches* rebased in `main`.

### Issues and Pull Requests Templates

A good practice adopted is to use templates for creating Issues and Pull Requests on Github that help you to describe what you are aiming to do with a change proposal. 
To do that, you can copy the `.github` on this repository to your project repository. This folder has [PULL_REQUEST_TEMPLATE](https://github.com/vtex/dev-guidelines/blob/main/.github/PULL_REQUEST_TEMPLATE.md) and [ISSUE_TEMPLATE](https://github.com/vtex/dev-guidelines/blob/main/.github/ISSUE_TEMPLATE.md) files. Feel free to change these files, and also, **please give us feedback to improve these templates**.

Praise for the **order management** and **help-center** projects from which we get these templates. 
