# Using Git(Hub) collaboratively

## Introduction

As a distributed version control system, Git allows multiple people to work on the same project at the same time.

In this module, we illustrate multiple ways Git(Hub) can be used collaboratively.
The concepts we will be discussing are applicable to other Git hosting services such as GitLab and Codeberg as well.

## Adding collaborators to your GitHub repository

In the previous module, we introduced ***forking*** on GitHub, which allows users to create and work on an independent copy of another user's repository under their own accounts. This allows users to contribute to the forked repository with ***pull requests*** (we will come to this later).
However,it is sometimes convenient to have someone whom you can trust to have direct access to your repository, so that they can help with development and management. In this scenario, you can add them as a collaborator.

To add a user as collaborator to a GitHub repository, go to the main page of the repository and click on the "Settings" tab, then click "Collaborators" under "Access" in the left panel. In the right panel, click the "Add people" button, and then enter or search your collaborator's GitHub account.

The invited user can then accept the invitation (before it expires) and gain direct access to this repository. You can always revoke an invitation or remove an existing collaborator.

## Branches

Using branches allows one user or more collaborators to work on different perspectives of a project in parallel.

Each branch is an independent line of development. A Git repository can have one or multiple branches.
The first and default branch of a Git repository is usually `main`.
A new branch is created on the basis of an existing branch.

### Branch creation - Web interface

Creating a branch in the GitHub web interface is straightforward and requires no local setup.

1. Go to the main page of the repository on GitHub
2. Locate the dropdown branch selector near the top-left of the file list, which displays the current branch name (usually `main`).
3. Click the dropdown to open the branch menu. Optionally, here you can change the current branch, based on which the new branch will be created.
4. In the search/input field, type the name of the new branch you want to create.
5. GitHub will show an option like "Create branch <branch-name> from main". Select it to create the branch instantly.

The current branch is switched to the newly created one in the web interface. You can always switch to a branch with the dropdown branch selector.

### Working with branches locally

To work with the newly created branch on your machine,...

It is a good practice to start clean: This means there is no uncommited changes in the current branch.

First, fetch all up-to-date branches using

```sh
git fetch
```

The new branch will be fetched. Using

```sh
git branch
```

will show all branches, with the current one highlighted. Next, use

```sh
git switch <branch-name>
```
to switch to the newly created branch. All subsequent Git operations (e.g., commit, push) will be applied on the new branch until you switch to another branch.

### Branch creation - Local

A branch can also be created locally. Using the following commands to create, and switch to a new branch based on the current branch.

```sh
git branch <branch-name>
git switch <branch-name>
```

By default, the new branch has all commits of the current branch and initially points to the same latest commit `HEAD`. Alternatively, use `git switch -c` to create and switch to a new branch in one step.

```sh
git switch -c <branch-name>
```

Now, you can work on the new branch and commit your changes locally using `git commit` for instance. However, at this stage, other collaborators cannot see these commits yet, since they are not yet pushed to the remote repository on GitHub. To do so, use the `git push` command:

```sh
git push -u origin <branch-name>
```

Now you and collaborators can check that the new branch is pushed to GitHub. At the same time, Git remembers that the local branch `<branch-name>' corresponds to the remote branch `<branch-name>` on GitHub, such that subsequent synchronization commands can be simplified to what we have learned in the last module

```sh
git push
git pull
```
without explicitly specifying the remote repository and branch name each time.

## Integrating contributions

In collaborative Git workflows, developments are usually done on separate branches and later integrated together. Proper integration workflows are essential for maintaining code quality, avoiding conflicts, and preserving a clean project history.

Typical integration scenarios include:

merging a feature branch into main
updating a feature branch with changes from main
reviewing contributions through pull requests
resolving merge conflicts
choosing between fast-forward merges, merge commits, or rebasing

### Creating a pull request

A pull request (PR) is used to propose changes from one branch into another.

Suppose we have created a branch called `new-branch` from `main` at some commit `C` and made some commits there. At the same time, no change has been made in the `main` branch. Now we want to integrate the new commits `D` and `E` in `new-branch` into `main`.

Assuming both branches are synchronized with the remote repository on GitHub, the situation can be represented as follows:

```
main            A - B - C
                        |
new-branch               - D - E
```

The diagram shows that `new-branch` has diverged from `main` at commit `C`, and contains two additional commits (`D` and `E`) that are not yet present in main.
To propose a pull request on GitHub, go to the repository page and click the "Pull Request" tab, and click the "Create pull request".

Next, select the base branch, where changes will be merged (in our case `main`); and the "compare branch" with the new commits (in our case `new-branch`).
GitHub will automatically show the differences between the branches. Review the changes, check especially

* file diffs
* added/removed lines
* commit history

Once everything looks good, click the "Create pull request" button. Now you can add a title and description for the pull request. It is a good practice to provide a clear summary of what was changed and why it was changed. Finally, click the "Create pull request" button to submit the pull request.

Once created, the pull request will appear in the "Pull Request" tab of the repository. Collaborators can review the pull request, leave comments, request changes, and
approve the pull request.
After approval, the pull request can be merged into the base branch.
