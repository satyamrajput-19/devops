# Day 5 — Mastering Git

## Course Alignment

This follows **Session 5: Mastering Git** from the DevOps & Cloud Engineering course plan.

Course topics:
- Distributed vs centralized version control
- Git's three-stage architecture: working tree, staging area, repository
- Branching, merging and conflict resolution
- Managing large/AI assets with Git-LFS and DVC concepts

Source: uploaded DevOps & Cloud Engineering course plan.

## Learning Objectives

By the end of Day 5, I should be able to:
- Explain distributed vs centralized version control.
- Explain Git's working tree, staging area and repository.
- Create and switch branches safely.
- Commit changes with meaningful messages.
- Merge branches.
- Resolve merge conflicts.
- Understand when large files need Git-LFS or DVC concepts.
- Inspect Git history and understand what changed.
- Use Git as a core DevOps collaboration and automation tool.

## 1. Version Control Systems

Version control records changes to files so that work can be tracked, compared and recovered.

### Centralized Version Control

A centralized system uses a central server as the main repository.

    Developer
       |
       v
    Central Server
       ^
       |
    Developer

### Distributed Version Control

Git is a distributed version control system. Each clone contains repository history locally.

    Developer A <----> Developer B
          \              /
           \            /
              Git Repo
                 |
               GitHub

GitHub provides remote repository hosting and collaboration around Git repositories.

## 2. Git's Three-Stage Architecture

A useful Git mental model is:

    Working Tree
         |
       git add
         v
    Staging Area
         |
      git commit
         v
    Repository

Working Tree:
The files currently checked out on your machine.

    git status

Staging Area:
The changes selected for the next commit.

    git add filename
    git diff --cached

Repository:
The committed Git history.

    git commit -m "docs: add Git fundamentals"
    git log --oneline

## 3. Essential Git Workflow

    git status
    git add <file>
    git diff --cached
    git commit -m "message"
    git log --oneline
    git push

Important distinction:
- commit = save a snapshot in the local Git repository
- push = send commits to a remote repository

## 4. Inspecting Changes

    git status
    git diff
    git diff --cached
    git log --oneline --decorate --graph
    git show <commit>

Ask:
- What changed?
- Which commit introduced it?
- Which files changed?
- Which branch contains the commit?

## 5. Git Branching

Create a branch:

    git branch feature/linux-monitor

Switch:

    git switch feature/linux-monitor

Create and switch:

    git switch -c feature/linux-monitor

List branches:

    git branch

Example:

    main
      |
      +---- feature/linux-monitor
      |
      +---- feature/github-actions

A branch should represent a focused unit of work.

## 6. Merging

    git switch main
    git pull
    git merge feature/linux-monitor

If Git cannot automatically reconcile changes, it reports a merge conflict.

## 7. Merge Conflicts

Typical conflict markers:

    <<<<<<< HEAD
    current branch content
    =======
    incoming branch content
    >>>>>>> feature/linux-monitor

Resolution workflow:
1. Open the conflicted file.
2. Understand both versions.
3. Choose or combine the correct content.
4. Remove conflict markers.
5. Test the result.
6. Stage the resolved file.
7. Complete the merge.

    git status
    git add <resolved-file>
    git commit

Abort an unfinished merge:

    git merge --abort

Never resolve a conflict blindly. Understand what each side was trying to change.

## 8. Meaningful Commit Messages

Examples:

    docs: add Git fundamentals
    feat: add system health monitor
    fix: correct backup script argument validation
    chore: update shellcheck configuration

Useful types:
- feat
- fix
- docs
- refactor
- test
- chore

The goal is a readable project history.

## 9. Remote Repositories

    git remote -v
    git fetch
    git pull
    git push

For a new branch:

    git push -u origin feature/linux-monitor

origin is commonly used as the default name for the main remote.

## 10. Git Diff as a Debugging Tool

Before committing:

    git diff

Ask:
- Did I change only what I intended?
- Did I accidentally add secrets?
- Did I introduce debugging output?
- Did I modify generated files?
- Is the commit focused?

This is especially important for Bash, Terraform, Kubernetes YAML, Dockerfiles, CI/CD workflows and configuration files.

## 11. Git Ignore

A .gitignore file prevents selected files from being tracked.

Typical examples:

    .env
    *.log
    __pycache__/
    .terraform/
    node_modules/

Check whether a file is ignored:

    git check-ignore -v <file>

Important DevOps rule:
Never commit credentials, private keys, access tokens or secrets to a public repository.

If a secret is accidentally committed, deleting the file in a later commit does not automatically remove it from Git history. Treat exposed credentials as compromised and rotate or revoke them.

## 12. Git-LFS Concepts

Git-LFS means Git Large File Storage.

It is designed for large files that do not fit comfortably into normal Git object storage workflows.

Typical candidates:
- Large binaries
- Large datasets
- Media assets
- Model files

Basic workflow concept:

    git lfs install
    git lfs track "*.bin"
    git add .gitattributes
    git add <large-file>
    git commit -m "chore: track large asset with Git LFS"
    git push

Git-LFS stores pointer files in Git while the large file content is handled by LFS storage.

## 13. DVC Concepts

DVC, or Data Version Control, is commonly used when projects need versioning and reproducibility for data and machine-learning artifacts.

Conceptually:

    Git
      |
      +-- Code
      +-- Configuration
      +-- Metadata
             |
             v
            DVC
             |
             +-- Large datasets
             +-- Model artifacts

Git-LFS and DVC solve related large-asset problems in different workflows.

For DevOps and cloud engineering, the important concept is choosing an appropriate versioning strategy rather than putting every large artifact directly into normal Git history.

## 14. Hands-On Labs

### Lab 1 — Three-Stage Workflow

    mkdir git-day5-lab
    cd git-day5-lab
    git init
    echo "# Git Day 5" > README.md
    git status
    git add README.md
    git diff --cached
    git commit -m "docs: initialize Git Day 5 lab"
    git log --oneline

### Lab 2 — Branching and Merge

    git switch -c feature/git-notes
    git add .
    git commit -m "docs: add Git workflow notes"
    git switch main
    git merge feature/git-notes
    git log --oneline --graph --decorate --all

### Lab 3 — Conflict Resolution

Create two branches that modify the same lines.

When Git reports a conflict:

    git status

Resolve the file, then:

    git add <file>
    git commit

Verify:

    git log --oneline --graph --decorate --all

### Lab 4 — Git Investigation

Use the DevOps repository and answer:
1. What is the current branch?
2. What is the latest commit?
3. Which files changed in the latest commit?
4. What remote is configured?
5. How many commits are visible in the current history?
6. Which files are ignored?

Useful commands:

    git branch --show-current
    git log -1 --stat
    git remote -v
    git log --oneline
    git status
    git check-ignore -v <file>

## 15. Failure Engineering

Intentionally practice:
- Staging the wrong file and correcting it.
- Creating a merge conflict.
- Aborting a merge.
- Resolving a conflict incorrectly, then recovering.
- Creating an unwanted local commit and inspecting it.
- Working with an ignored file.
- Testing what happens when a remote branch has changed.

For every failure, answer:
1. What happened?
2. Which Git state was I in?
3. Which command revealed the problem?
4. How did I recover?
5. How can I avoid the same mistake?

## 16. DevOps Connection

    Git
      |
      v
    Source Control
      |
      v
    Pull Request / Code Review
      |
      v
    CI Pipeline
      |
      v
    Build + Test + Security Scan
      |
      v
    Artifact / Container Image
      |
      v
    Deployment
      |
      v
    Monitoring + Operations

Git becomes the foundation for reproducible DevOps workflows.

The later course sessions build toward GitHub workflows, CI/CD, infrastructure automation and GitOps.

## 17. Interview Questions

### Q1. What is the difference between Git and GitHub?

Git is a distributed version control system. GitHub is a platform for hosting Git repositories and collaborating around them.

### Q2. What are Git's three stages?

The working tree contains current files, the staging area contains selected changes for the next commit, and the repository contains committed history.

### Q3. What is the difference between git fetch and git pull?

git fetch downloads remote updates without integrating them into the current branch. git pull fetches and then integrates the changes.

### Q4. What is a Git branch?

A branch is a movable reference to a line of commits, allowing work to proceed independently.

### Q5. What causes a merge conflict?

A conflict occurs when Git cannot automatically reconcile incompatible changes, commonly when the same lines or nearby parts of a file were changed differently.

### Q6. How do you resolve a merge conflict?

Inspect the conflict, decide the correct content, remove conflict markers, test the result, stage the resolved files and complete the merge.

### Q7. Why should secrets not be committed?

A Git repository preserves history. Even if a secret is deleted later, the credential may remain accessible in previous commits or other copies. Exposed credentials should be revoked or rotated.

### Q8. What is Git-LFS?

Git-LFS is an extension for handling large files by storing lightweight pointer files in Git while the large content is handled through LFS storage.

### Q9. What is DVC?

DVC is a data versioning tool used to manage datasets and other large artifacts alongside Git-based project workflows.

## 18. DSA / CS Fundamentals

### Problem — Two Sum

Given an array of integers and a target value, find two elements whose sum equals the target.

    Input:  [2, 7, 11, 15]
    Target: 9
    Output: indices [0, 1]

Target:
- Time: O(n)
- Space: O(n)

Core idea:
Use a hash map to remember previously seen values. For each number, calculate:

    complement = target - current

If the complement already exists in the map, the pair has been found.

This introduces hash-table lookup, useful CS knowledge for technical interviews.

## 19. Day 5 Checkpoint

Before moving on, explain without notes:
- Centralized vs distributed version control
- Working tree
- Staging area
- Repository
- git add
- git commit
- git push
- git fetch
- git pull
- Branching
- Merging
- Merge conflicts
- git merge --abort
- .gitignore
- Git-LFS concepts
- DVC concepts

Practical checkpoint:
Create a repository, create a branch, make a commit, merge it into main, intentionally create a conflict, resolve it, and inspect the resulting history.

## Learning Report

Record:
- What I built
- What Git command I learned
- What failed
- How I resolved the failure
- One Git mistake I will avoid in production
- One interview question I can now answer confidently

## Next

Day 6 — Advanced Git & GitHub

Course topics:
- Rebase, squash, cherry-pick, stash and reflog
- GitHub authentication
- Pull request workflows
- Code review etiquette
- Branching strategies: GitHub Flow, Git Flow and trunk-based development
- Branch protection
- GitHub Actions introduction
