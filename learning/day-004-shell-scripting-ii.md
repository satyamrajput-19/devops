# Day 4 — Shell Scripting II: Filesystems & Performance

## Course Alignment

This follows Session 4 of the uploaded DevOps & Cloud Engineering course plan.

Course topics:
- Defensive scripting: set -euo pipefail, trap, validation
- Logging and debugging: log(), bash -x, ShellCheck
- Monitoring CPU, memory and disk with alerting
- Scheduling with cron and filesystem automation
- Portfolio project: System Health Monitor — filesystem automation + performance reporting

Source: uploaded DevOps & Cloud Engineering course plan.

## Learning Objectives

By the end of Day 4, I should be able to:
- Write safer Bash scripts with defensive scripting patterns.
- Validate arguments and expected inputs.
- Use trap for cleanup and interruption handling.
- Add structured logging.
- Debug with bash -x and ShellCheck.
- Monitor CPU, memory and disk.
- Trigger threshold-based alerts.
- Schedule scripts with cron.
- Automate safe filesystem maintenance.

## 1. Defensive Bash

A common defensive baseline is:

    #!/usr/bin/env bash
    set -euo pipefail

- set -e: stop when a command fails in contexts where Bash treats the failure as fatal.
- set -u: treat unset variables as errors.
- pipefail: make a pipeline report failure when an earlier command fails.

Defensive scripting is a discipline; set -e has Bash-specific edge cases around conditionals, &&, || and command substitutions.

## 2. Input Validation

Example:

    #!/usr/bin/env bash
    set -euo pipefail

    if [[ $# -ne 1 ]]; then
        echo "Usage: $0 <directory>"
        exit 1
    fi

    directory="$1"
    if [[ ! -d "$directory" ]]; then
        echo "Error: directory does not exist: $directory" >&2
        exit 1
    fi

Production habits:
- Check argument count.
- Quote variables.
- Validate files and directories before operating on them.
- Send errors to stderr when appropriate.
- Return non-zero status for failed operations.

## 3. trap and Cleanup

trap lets a script respond to signals or shell events.

Example:

    tmp_file="$(mktemp)"

    cleanup() {
        rm -f "$tmp_file"
    }

    trap cleanup EXIT

Useful for temporary files, lock files, background processes and partial automation state.

## 4. Logging

A simple logging function:

    log() {
        printf '[%s] %s\n' "$(date '+%Y-%m-%d %H:%M:%S')" "$*"
    }

    log "Starting system health check"

Useful levels:

    info()  { log "[INFO] $*"; }
    warn()  { log "[WARN] $*"; }
    error() { log "[ERROR] $*" >&2; }

Operational scripts should make it clear what they are doing and what failed.

## 5. Debugging Bash

Run a script with tracing:

    bash -x ./health-check.sh

Use this when variables, conditionals, loops or command arguments behave unexpectedly.

ShellCheck provides static analysis:

    shellcheck health-check.sh

Workflow:
Write → Run → ShellCheck → Fix → Test again

## 6. CPU Monitoring

Linux provides CPU information through tools such as top and uptime. One common approach is:

    cpu_usage="$(top -bn1 | awk '/Cpu\\(s\\)/ {print 100 - $8}')"
    printf 'CPU usage: %.2f%%\n' "$cpu_usage"

Output formats can differ between Linux environments, so test monitoring commands on the target distribution.

## 7. Memory Monitoring

free can be used to inspect memory:

    memory_usage="$(free | awk '/Mem:/ {printf "%.0f", ($3/$2)*100}')"
    echo "Memory usage: ${memory_usage}%"

Threshold example:

    MEMORY_THRESHOLD=80
    if (( memory_usage >= MEMORY_THRESHOLD )); then
        echo "WARNING: Memory usage is ${memory_usage}%"
    fi

## 8. Disk Monitoring

df can report filesystem usage:

    disk_usage="$(df -P / | awk 'NR==2 {gsub("%","",$5); print $5}')"
    echo "Root filesystem usage: ${disk_usage}%"

Threshold example:

    DISK_THRESHOLD=80
    if (( disk_usage >= DISK_THRESHOLD )); then
        echo "WARNING: Disk usage is ${disk_usage}%"
    fi

## 9. System Health Monitor

Today we extend the course portfolio project: System Health Monitor — filesystem automation + performance reporting.

Target report:

    System Health Report
    --------------------
    Hostname:
    Timestamp:
    CPU Usage:
    Memory Usage:
    Disk Usage:
    Status:

Recommended structure:

    system-health-monitor/
    ├── health-check.sh
    ├── logs/
    └── README.md

Requirements:
1. Validate prerequisites.
2. Collect metrics.
3. Compare metrics with thresholds.
4. Log results.
5. Return meaningful exit codes.
6. Generate a report.
7. Make the script schedulable with cron.

## 10. Cron Scheduling

Inspect current cron jobs:

    crontab -l

Edit them:

    crontab -e

Example: run every 5 minutes:

    */5 * * * * /path/to/health-check.sh >> /path/to/health.log 2>&1

Cron fields:

    minute hour day-of-month month day-of-week

Use absolute paths and explicitly redirect stdout/stderr.

## 11. Filesystem Automation

Useful operations include:
- Find files older than a defined age.
- Create backup directories.
- Archive logs.
- Remove temporary files.
- Check filesystem capacity.
- Detect unexpected files.
- Generate reports.

Example inspection command:

    find /tmp -type f -mtime +7 -print

Safety workflow:

    find → review output → test command → automate cleanup

Never combine destructive operations with an unverified find expression.

## 12. Hands-On Labs

### Lab 1 — Defensive Script
Create defensive-demo.sh that:
- Uses set -euo pipefail.
- Accepts a directory argument.
- Validates the argument.
- Uses cleanup with trap.
- Returns meaningful status codes.

Test with a valid directory, no argument, and a nonexistent directory.

### Lab 2 — Logging + Debugging
Create logger-demo.sh that:
- Defines info, warn and error functions.
- Writes timestamps.
- Contains an intentional small bug.
- Is debugged with bash -x.
- Is checked with ShellCheck.

### Lab 3 — System Health Monitor
Create system-health-monitor/ with health-check.sh, logs/, and README.md.
Implement CPU, memory and disk thresholds, timestamp, hostname, warnings, exit status and logging.

### Lab 4 — Cron + Filesystem Automation
Schedule the health monitor with cron.
Create a test-directory cleanup/report task that finds old files but does not delete anything until the output is reviewed.

## 13. Failure Engineering

Intentionally test:
- Missing arguments.
- Invalid directories.
- Unset variables.
- Failed commands.
- Interrupted scripts.
- High resource conditions where practical.
- Broken cron paths.
- Permission failures.

For every failure, answer:
1. What failed?
2. How did the script detect it?
3. What did the logs show?
4. What exit code was returned?
5. How would an operator know what to do next?

## 14. DevOps Connection

    Bash
      ↓
    Linux automation
      ↓
    CI/CD scripts
      ↓
    Docker entrypoints
      ↓
    Cloud automation
      ↓
    Terraform / Ansible workflows
      ↓
    Kubernetes operational tooling
      ↓
    Production incident response

The goal is safe, observable, repeatable and debuggable automation.

## 15. Interview Questions

### Q1. Why use set -euo pipefail?
To make common scripting failures more visible and reduce silent errors from failed commands, unset variables and pipelines.

### Q2. What is trap used for?
To execute actions when the shell receives signals or exits, commonly for cleanup.

### Q3. How do you debug a Bash script?
Use targeted logging, bash -x and static analysis such as ShellCheck.

### Q4. How would you monitor disk usage?
Use df, extract the usage percentage, compare it with a threshold, and log or alert when the threshold is exceeded.

### Q5. How do you schedule a script?
Use cron through crontab -e, with absolute paths and explicit output/error redirection.

### Q6. Why test destructive filesystem automation first?
An incorrect path, pattern or permission can cause unintended data loss. Inspecting the target set first reduces that risk.

## 16. DSA / CS Fundamentals

Problem: Check if an array is sorted in non-decreasing order.

Target:
- Time: O(n)
- Space: O(1)

Core idea: compare each element with the previous element. If any current element is smaller, the array is not sorted.

## 17. Day 4 Checkpoint

Before Day 5, explain without notes:
- set -e
- set -u
- pipefail
- trap
- Input validation
- Bash logging
- bash -x
- ShellCheck
- CPU/memory/disk monitoring
- Threshold-based alerting
- Cron scheduling
- Safe filesystem automation

## Learning Report

Record:
- What I built
- What failed
- What I fixed
- What command helped me debug it
- One Bash mistake I will avoid in production
- One interview question I can now answer confidently

## Next

Day 5 — Mastering Git

Course topics:
- Distributed vs centralized version control
- Git's three-stage architecture
- Working tree, staging area, repository
- Branching
- Merging
- Conflict resolution
- Git-LFS and DVC concepts