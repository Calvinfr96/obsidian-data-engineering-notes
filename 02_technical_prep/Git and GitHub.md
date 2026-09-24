---
aliases:
  - "github_basics"
tags:
  - de-study
source_notes:
  - "/Users/Apple/dea-workspace/dea-general-learning/Git and GitHub/github_basics.md"
imported: 2026-09-24
---

# Git and GitHub

## Contents

- [[#Introduction to Version Control]]
- [[#Creating a Git Repository]]
- [[#Git Workflow]]
- [[#Git Branching]]
- [[#Git Rebasing & Squashing]]
- [[#Repository vocabulary]]

## Introduction to Version Control
- Git is widely used as a powerful and modern version control system. Git offers several advantages over other common version control systems, including:
  - Distributed Versioning
  - Ability to store repositories locally
  - Limited need for network access
  - Hashed content using SHA-1 hashing algorithm
- Git Lifecycle:
  - Clone Repository --> Make Updates Locally --> Review Changes --> Commit and Push Changes.
- Git vs. GitHub:
  - Git:
    - Distributed version control system used to track changes and versions of different files.
    - Command-based tool used to work with files. Requires network connection to push and pull files.
    - Repositories are stored locally, but can be shared on a centralized server.
    - Can work with github and other web-based hosting services.
  - GitHub:
    - Web-based hosting service for hit repositories.
    - GUI platform made for developers.
    - Files are stored in centralized servers that everyone can be granted access to.
    - Has alternatives, such as GitLab and BitBucket.
- Version control is important because it allows you to see history of a project to identify when bugs were introduced to help with troubleshooting. This also allows you to revert to previous versions.
- Version control acts as a time machine for software developers and data engineers. It tracks changes in code/files over time.

## Creating a Git Repository
- A repository can either be made in a folder on your local machine using `git init`, or it can be created in the github UI, then cloned to your local machine. A git repo can be cloned using the HTTPS link from the UI, along with the `git clone {URL}` command.

## Git Workflow
- `git add` adds files from the working directory, where the local git repo resides, to the "staging area", where files will be committed, reviewed, and pushed to the remote git repo.
- `git commit` bundles **new or modified** files in the staging area and prepares them to be pushed to the remote git repo. A message can be added to the commit to summarize the changes that were made to the repo.
- `git push` pushes the committed changes from the local repo to the remote repo.
- `git status` tells you what changes have and haven't been staged for commit.
- `git log` will give you a list of previously committed changes.
- `git diff` will show the changes that have been made to a file since its most recent commit.

### Undoing Changes
- `git restore {filename}` restores the last committed version of a file.
- `git restore --staged {filename}` removes a file from the staging area, but keeps the changes.
- `git reset --soft HEAD~1` moves committed changes back to the staging area.
- `git reset --hard HEAD~1` permanently deletes the previous commit.
- `git checkout {commit_id}` temporarily moves a commit to `HEAD` (the most recent commit).

## Git Branching
- A branch is a separate line of development in a git repo. Branches allow you to isolate experimental features or bug fixes without affecting the main piece of software.
- `git branch` lists all of the branches currently in the repository.
  - `git branch -u {branch_name}` will configure the current branch to track the branch called `branch_name`.
  - `git branch -vv` will show all existing branches along with their tracking details.
- `git checkout {branch_name}` switches from one branch to another.
- `git checkout -b {branch_name}` creates a new branch and switches to that branch.
  - `git checkout -b {branch_name} -t` will configure the new branch to track the changes from the current branch.
  - Once a branch is created and checked out, `git push` will publish the branch from the local repo to the remote repo.
    - `git push -u origin {branch_name}` will push the local branch to the remote repo and configure the local branch to track its remote counterpart.
- `git merge {branch_name}` merges changes from the branch `branch_name` with changes from the branch currently checked out.
- A pull request is used to move changes from one branch to another branch.

## Git Rebasing & Squashing
- A git rebase moves or reapplies commits on top of another branch's latest commits, creating a clean, straight timeline. Rebasing is useful when you want to keep commit history linear and when a feature branch is behind the `main` branch.
- `git log --oneline --graph` displays the commit history of the various branches in your repo.
- `git rebase {branch_name}` merges commits from `branch_name` to the current branch.
  - `git pull --rebase` is a shortcut that performs a `git fetch` followed by a `git rebase` instead of the default `git merge`. The command fetches the latest commits from the remote tracking branch, temporarily removes local commits that haven't been pushed yet, applies the new remote commits to your local branch, then replays your local commits one by one on top of the newly updated branch.
  - Using `git pull --rebase` is better than using `git merge` because it keeps commit history clean and linear and allows you to resolve merge conflicts on a commit-by-commit basis, instead of all at once.
  - Before using `git pull --rebase`, it's important to make sure your branches are properly configured to track the appropriate remote branch, which is where the command fetches changes from.
  - `git pull --rebase` is best used on local branches that haven't been pushed yet, while `git merge` is best used for shared/public branches.
- Squashing combines multiple, small commits into one, meaningful commit. This helps keep commit history clear and concise.
  - One common method used to squash commits is the `git rebase -i` command. For example, `git rebase -i HEAD~3` would be used to squash the most recent commit, along with the two previous commits. In the text editor that opens, change the word `pick` to `squash` (or just `s`) for the commits you want to merge into the one above them.
  - Another common method used to squash commits is the `git merge --squash` command. When merging a feature branch into a target branch (like main), this command takes all the changes from the feature branch and stages them as a single, new commit on the target branch without preserving the individual commit history of the feature branch.

## Repository vocabulary

- Git is a distributed version control system, meaning there is no singular main server and full history of a project can be made available through a cloned copy of a remote repository.
- A repository is a collection of commits and branches saved in a .git directory.
- A commit is a snapshot of a project and the changes made to it at a certain point in time. HEAD is the name of the most recent commit in a particular branch of the repository. It can also refer to the commit you currently have checked out.
- A branch is a collection of commits within a repository. The primary branch of a repository is the 'main' branch.
