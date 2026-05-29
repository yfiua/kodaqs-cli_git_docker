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

To work with the newly created branch on your machine, first make sure your current branch is clean: This means there is no uncommited change in the current branch.

Then, fetch all up-to-date branches using

```sh
git fetch
```

The new branch will be fetched. Using

```sh
git branch
```

will show all local branches, with the current one highlighted. Next, use

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

* merging a feature branch into main
* reviewing contributions through pull requests
* resolving merge conflicts

### Creating a pull request

A pull request (PR) is used to propose changes from one branch into another.

Suppose we have created a branch called `new-branch` from `main` at some commit `C` and made some commits there. At the same time, no change has been made in the `main` branch. Now we want to integrate the new commits `D` and `E` from `new-branch` into `main`.

Assuming both branches are synchronized with the remote repository on GitHub, the situation can be represented as follows:

```
main            A - B - C
                        |
new-branch              ^ - D - E
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

### Integrating diverged branches

In the scenario above, integrating changes from `new-branch` is straight forward, since the two branches have not diverged. In fact, when the pull request is accepted, the two commits `D` and `E` are added in a "fast-forward" manner. In reality, however, things might get more complicated. For example, when `D` and `E` are made in `new-branch`, a collaborator may add another commit `F` in the `main` branch.

```
main            A - B - C - F
                        |
new-branch              ^ - D - E
```

In this situation, the two branches have ***diverged***, because both branches now contain commits that are absent from the other branch.
It is important to note that divergence does not necessarily imply conflict branches. For example, commit `F` may modify different files or different parts of the codebase than commits `D` and `E`. In such cases, Git can still integrate the changes automatically. Conflicts arise only when incompatible changes are made to the same portions of the same files.

When the pull request is ***merged***, Git creates a new ***merge commit*** (`M` in the diagram) that combines the histories of both branches:

```
main            A - B - C - F --- M
                        |        /
new-branch              ^ - D - E
```

### Resolving conflicts

Conflicts occur if incompatible changes are made to the same portions of the same files. In that case, Git cannot automatically determine which version should be kept, and manual conflict resolution becomes necessary.

Using the example scenario above, suppose at commit `C` we have a file `print.py`:

```python
print('Hello!')
```

In commit `F` in the `main` branch, the file is changed to

```python
print('Hello from main!')
```

In commit `E` in `new-branch`, the file is changed to

```python
print('Hello from new branch!')
```

We have prepared a sample repository demonstrating diverged branches [here](https://github.com/yfiua/github-diverged-branches). Fork it (including all branches) and play around with it.

Now, when trying to merge `new-branch` into `main`, Git cannot decide automatically which change should be kept, and we have to manually resolve the conflict. When a pull request from `new-branch` to `main` is created, Git will show that merging is not possible until the conflict is resolved.
The conflict can be resolved either with the web editor or locally (see the "View command line instructions" link).

For example, in the web editor, the file `print.py` is displayed with conflict markers that indicate the two versions of the files.

```python
<<<<<<< new-branch
print('Hello from new branch!')
=======
print('Hello from main!')
>>>>>>> main
```

we may decide to combine both changes and modify `print.py` as follows:

```python
print('Hello from main and new branch!')
```

After removing the conflict markers and saving the file, the conflict is considered resolved, and the pull request can be completed successfully.

## References and resources

We have covered the basics of collaborative Git(Hub) usage in this module. There are many more advanced features and best practices that you can learn, for example merging vs rebasing pull requests, and squash pull requests. Here are some resources to help you learn more:

* [Collaborative coding with GitHub and RStudio](https://github.com/lmu-osc/Collaborative-RStudio-GitHub) by Anna Krystalli from RSE-Sheffield
* [Collaborating with pull requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/) by GitHub Docs
