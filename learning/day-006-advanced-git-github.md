# Day 6 — Advanced Git & GitHub

## Course Alignment

This follows **Session 6: Advanced Git & GitHub** from the DevOps & Cloud Engineering course plan.

Course topics:
- Rebase, squash, cherry-pick, stash and reflog
- GitHub authentication
- Pull request workflows
- Code review etiquette
- Branching strategies: GitHub Flow, Git Flow and trunk-based development
- Branch protection
- GitHub Actions introduction

## Learning Objectives

By the end of Day 6, I should be able to:
- Rebase a branch and understand when to use it.
- Squash commits into a clean history.
- Cherry-pick a specific commit.
- Temporarily save work with stash.
- Recover references and commits with reflog.
- Understand GitHub authentication and secure remote access.
- Create and work with pull requests.
- Follow practical code-review etiquette.
- Explain GitHub Flow, Git Flow and trunk-based development.
- Understand the purpose of branch protection.
- Understand the role of GitHub Actions in CI/CD.

## 1. Rebase

Rebase moves or reapplies commits on top of another base.

    git switch feature/linux-monitor
    git fetch origin
    git rebase origin/main

Conceptually:

    Before:
    main:    A---B---C
                  \
    feature:       D---E

    After rebase:
    main:    A---B---C
                      \
    feature:           D'---E'

Rebase can create a cleaner linear history, but it rewrites commit history.

Important rule:
Do not casually rebase commits that other people are already depending on.

If a rebase becomes problematic:

    git rebase --abort

Continue after resolving a conflict:

    git add <resolved-file>
    git rebase --continue

## 2. Squashing Commits

Squashing combines multiple commits into fewer commits.

Interactive rebase:

    git rebase -i HEAD~3

Example:

    pick   commit-1
    squash commit-2
    squash commit-3

Use squashing when several small development commits should become one meaningful unit before integration.

Example final history:

    feat: add system health monitor

instead of:

    test
    fix
    update
    final-final

## 3. Cherry-Pick

Cherry-pick applies one existing commit onto the current branch.

    git switch main
    git cherry-pick <commit-sha>

Useful when a specific fix is needed without merging an entire branch.

If conflicts occur:

    git status
    git add <resolved-file>
    git cherry-pick --continue

Abort:

    git cherry-pick --abort

Use cherry-pick deliberately because it copies a change into a new history context.

## 4. Git Stash

Stash temporarily stores local changes so the working tree can be cleaned.

    git status
    git stash
    git status

List stashes:

    git stash list

Restore the latest stash:

    git stash pop

Apply without removing it:

    git stash apply

Include untracked files when needed:

    git stash -u

Useful scenario:

    Working on feature A
           |
           v
    urgent fix required
           |
           v
    git stash
           |
           v
    switch to fix branch
           |
           v
    complete fix
           |
           v
    return to feature A
           |
           v
    git stash pop

Do not use stash as a substitute for meaningful commits when work is ready to be saved.

## 5. Git Reflog

Reflog records movements of local references such as HEAD.

    git reflog

It can help locate commits after operations such as:
- reset
- rebase
- branch movement
- accidental history changes

Investigation example:

    git reflog
    git show <recovered-commit>

Reflog is a local recovery mechanism and should be treated as a troubleshooting tool.

## 6. GitHub Authentication

GitHub repositories can be accessed through authenticated Git operations.

Common approaches include:
- HTTPS with appropriate authentication
- SSH keys

Check the configured remote:

    git remote -v

SSH remote example:

    git@github.com:USERNAME/REPOSITORY.git

Security rules:
- Never commit passwords, access tokens or private keys.
- Do not paste credentials into scripts or public issues.
- Use appropriate credential-management mechanisms.
- Rotate/revoke credentials if they are exposed.

## 7. Pull Request Workflow

A pull request provides a structured workflow for proposing and reviewing changes.

Typical flow:

    Create branch
        |
        v
    Make changes
        |
        v
    Commit
        |
        v
    Push branch
        |
        v
    Open Pull Request
        |
        v
    Review + CI checks
        |
        v
    Address feedback
        |
        v
    Merge
        |
        v
    Delete branch

Useful PR contents:
- What changed?
- Why was it changed?
- How was it tested?
- Any known limitations?
- Any deployment or rollback considerations?

## 8. Code Review Etiquette

A useful review focuses on the code and its impact.

Review for:
- Correctness
- Security
- Maintainability
- Test coverage
- Configuration mistakes
- Reliability
- Performance where relevant
- Unnecessary complexity

Good review comment:

    "Could this validation happen before the file operation?
     That would make the failure path explicit."

Avoid personal or vague comments.

As the project grows, code review becomes part of the DevOps quality gate before CI/CD and deployment.

## 9. Branching Strategies

### GitHub Flow

A lightweight workflow centered around short-lived feature branches and pull requests.

    main
      |
      +-- feature
             |
             v
           PR
             |
             v
           main

### Git Flow

A more structured branching model with branches commonly associated with features, releases and production fixes.

### Trunk-Based Development

Developers integrate small changes frequently into a shared trunk/main branch, generally using short-lived branches or direct integration depending on the team's controls.

The important skill is understanding the trade-offs and following the strategy used by the engineering team.

## 10. Branch Protection

Branch protection helps enforce repository quality and collaboration controls.

Common controls can include:
- Required pull-request reviews
- Required status checks
- Restrictions on direct pushes
- Required conversation resolution
- Rules around force pushes and branch deletion

For a DevOps repository, protection helps prevent an uncontrolled change from bypassing the normal review and CI process.

## 11. GitHub Actions Introduction

GitHub Actions provides repository automation through workflows.

A workflow can:
- Run tests
- Run linters
- Perform security checks
- Build artifacts
- Build container images
- Publish artifacts
- Trigger deployment workflows

Basic workflow structure:

    .github/
      workflows/
        ci.yml

Conceptual flow:

    Git push / Pull Request
             |
             v
       GitHub Actions
             |
       +-----+-----+
       |           |
      Test       Lint
       |           |
       +-----+-----+
             |
             v
        Build / Scan
             |
             v
          Artifact

A later course session goes deeper into CI/CD. Day 6 introduces the GitHub Actions foundation.

## 12. Hands-On Labs

### Lab 1 — Rebase

Create a feature branch, make commits, update main, then rebase the feature branch.

Practice:

    git fetch origin
    git rebase origin/main

Inspect:

    git log --oneline --graph --decorate --all

### Lab 2 — Squash

Create three small commits and combine them with:

    git rebase -i HEAD~3

Verify the final history.

### Lab 3 — Cherry-Pick

Create a small fix commit on one branch and apply it to another branch:

    git cherry-pick <commit-sha>

### Lab 4 — Stash + Reflog

Create local changes, stash them, restore them, and inspect:

    git stash list
    git stash pop
    git reflog

### Lab 5 — Pull Request Workflow

Create:

    feature/day-6-git

Push it to GitHub and open a PR against main.

Document:
- What changed
- How it was tested
- Review feedback
- Final merge result

## 13. Failure Engineering

Intentionally practice:
- Rebase conflict and recovery.
- Aborting a rebase.
- Cherry-pick conflict and recovery.
- Stashing work before switching branches.
- Recovering a reference with reflog.
- Identifying an accidental direct change to main.
- Reviewing a PR with a deliberate configuration mistake.

For every failure, answer:
1. What happened?
2. Which Git state was I in?
3. Which command revealed the problem?
4. How did I recover?
5. How can I avoid the same mistake?

## 14. DevOps Connection

    GitHub
       |
       v
    Pull Request
       |
       v
    Code Review
       |
       v
    GitHub Actions
       |
       v
    Test + Lint + Security
       |
       v
    Build Artifact
       |
       v
    Deployment
       |
       v
    Monitoring

This connects advanced Git/GitHub practices with the CI/CD and automation stages covered later in the course.

## 15. Interview Questions

### Q1. What is the difference between merge and rebase?

Merge combines histories while preserving the branch structure. Rebase reapplies commits onto a new base and rewrites the rebased commit history.

### Q2. When would you use cherry-pick?

When a specific existing commit needs to be applied to another branch without merging the entire source branch.

### Q3. What is git stash?

It temporarily stores local changes so the working tree can be cleaned for another task.

### Q4. What is reflog?

A local record of reference movements that can help recover commits or previous states after history-changing operations.

### Q5. Why should you avoid rebasing shared history?

Because rebase rewrites commit identities and can make collaborators' existing history diverge.

### Q6. What is a pull request?

A GitHub collaboration mechanism for proposing changes, running checks, receiving review and integrating changes into a target branch.

### Q7. What is branch protection?

Repository controls that can require reviews, status checks or other conditions before changes can be merged into protected branches.

### Q8. What is GitHub Actions?

A GitHub automation platform used to run workflows in response to repository events or other configured triggers.

## 16. DSA / CS Fundamentals

### Problem — Valid Parentheses

Given a string containing brackets such as (), {}, and [], determine whether the brackets are correctly balanced.

Example:

    Input:  "({[]})"
    Output: true

Target:
- Time: O(n)
- Space: O(n)

Core idea:
Use a stack. Push opening brackets and, for each closing bracket, verify that the top of the stack contains the matching opening bracket.

This reinforces stack operations, a core data-structure concept used in technical interviews.

## 17. Day 6 Checkpoint

Before moving on, explain without notes:
- git rebase
- git rebase --abort
- interactive rebase
- squash
- git cherry-pick
- git stash
- git reflog
- GitHub authentication
- Pull request workflow
- Code review
- GitHub Flow
- Git Flow
- Trunk-based development
- Branch protection
- GitHub Actions

Practical checkpoint:
Create a feature branch, make multiple commits, squash them, rebase onto main, open a pull request, review the changes, and understand the CI workflow that runs against it.

## Learning Report

Record:
- What advanced Git operation I practiced
- What failed
- How I recovered
- One GitHub collaboration concept I learned
- One security practice I followed
- One interview question I can now answer confidently

## Next

**Day 7 — Networking Fundamentals**

Course topics:
- Networking fundamentals
- Core networking concepts needed for DevOps and cloud engineering
