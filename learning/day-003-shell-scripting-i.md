# Day 3 — Shell Scripting I

## Source Alignment

This day follows Session 3 of the DevOps & Cloud Engineering course plan:

- Shebang, variables and quoting rules
- Conditionals: if / case and test operators
- Loops: for / while read / until
- Functions, arguments and exit codes

## Objectives

By the end of Day 3, you should be able to:

- Explain what a shell script is
- Use a shebang correctly
- Create and execute Bash scripts
- Work with variables and environment variables
- Understand quoting rules
- Use if, elif, else and case
- Use Bash test operators
- Write for, while read and until loops
- Create functions
- Pass positional arguments to scripts
- Understand and use exit codes

## 1. Shebang

A shebang tells the operating system which interpreter should execute the script.

    #!/bin/bash

Example:

    #!/bin/bash
    echo "Hello from Bash"

Save as hello.sh.

Make it executable:

    chmod +x hello.sh

Run:

    ./hello.sh

## 2. Variables

Create variables without spaces around the equals sign:

    name="Satyam"
    role="DevOps Engineer"

Read a variable with the dollar sign:

    echo "$name"
    echo "$role"

Useful built-in variables:

    echo "$HOME"
    echo "$USER"
    echo "$SHELL"
    echo "$PWD"

Command substitution:

    current_dir="$(pwd)"
    today="$(date)"

## 3. Quoting Rules

Double quotes expand variables:

    name="Satyam"
    echo "Hello $name"

Single quotes do not expand variables:

    echo 'Hello $name'

Prefer quoting variables:

    echo "$name"

instead of:

    echo $name

Quoting helps prevent unintended word splitting and pathname expansion.

## 4. User Input

Use read:

    #!/bin/bash
    read -p "Enter your name: " name
    echo "Hello, $name"

## 5. Conditionals

Basic structure:

    if condition; then
        commands
    elif another_condition; then
        commands
    else
        commands
    fi

Example:

    age=22

    if [ "$age" -ge 18 ]; then
        echo "Adult"
    else
        echo "Minor"
    fi

Numeric operators:

| Operator | Meaning |
|---|---|
| -eq | equal |
| -ne | not equal |
| -gt | greater than |
| -ge | greater than or equal |
| -lt | less than |
| -le | less than or equal |

String operators:

- = equal
- != not equal
- -z empty string
- -n non-empty string

File tests:

- -f regular file
- -d directory
- -e exists
- -r readable
- -w writable
- -x executable

Example:

    if [ -f "$file" ]; then
        echo "Regular file exists"
    fi

## 6. case

Use case when there are multiple possible values.

    #!/bin/bash

    read -p "Enter environment: " env

    case "$env" in
        dev)
            echo "Development environment"
            ;;
        staging)
            echo "Staging environment"
            ;;
        prod)
            echo "Production environment"
            ;;
        *)
            echo "Unknown environment"
            ;;
    esac

This pattern is useful for CLI menus and environment-specific automation.

## 7. for Loop

    for item in one two three; do
        echo "$item"
    done

Example:

    for file in *.log; do
        echo "Found: $file"
    done

## 8. while read

A common DevOps pattern is reading input line by line.

Create servers.txt:

    server-01
    server-02
    server-03

Then:

    while read -r server; do
        echo "Checking $server"
    done < servers.txt

The -r option prevents backslash interpretation.

## 9. until Loop

until keeps running until the condition becomes true.

    count=1

    until [ "$count" -gt 3 ]; do
        echo "Attempt $count"
        count=$((count + 1))
    done

## 10. Functions

Functions allow reusable logic.

    greet() {
        echo "Hello, $1"
    }

Call it:

    greet "Satyam"

A DevOps script should avoid repeating the same operational logic in many places.

## 11. Script Arguments

Bash provides positional parameters:

- $0 — script name
- $1 — first argument
- $2 — second argument
- $# — number of arguments
- $@ — all arguments

Example:

    #!/bin/bash

    echo "Script: $0"
    echo "First argument: $1"
    echo "Second argument: $2"
    echo "Argument count: $#"

Run:

    ./args.sh dev aws

## 12. Exit Codes

Linux commands return an exit status.

    0 = success
    non-zero = failure

Check the previous command:

    echo "$?"

Explicitly exit:

    exit 0

Failure example:

    exit 1

This becomes important in CI/CD because pipelines use exit codes to determine whether a step succeeded or failed.

# Hands-on Lab

Create the workspace:

    mkdir -p ~/devops-day3
    cd ~/devops-day3

## Lab 1 — System Information

Create system-info.sh:

    #!/bin/bash

    echo "User: $USER"
    echo "Home: $HOME"
    echo "Shell: $SHELL"
    echo "Hostname: $(hostname)"
    echo "Current directory: $(pwd)"
    echo "Date: $(date)"

Run:

    chmod +x system-info.sh
    ./system-info.sh

## Lab 2 — Environment Classifier

Create environment.sh:

    #!/bin/bash

    if [ "$#" -ne 1 ]; then
        echo "Usage: $0 <dev|staging|prod>"
        exit 1
    fi

    case "$1" in
        dev)
            echo "Development environment selected"
            ;;
        staging)
            echo "Staging environment selected"
            ;;
        prod)
            echo "Production environment selected"
            ;;
        *)
            echo "Invalid environment"
            exit 1
            ;;
    esac

    exit 0

Test:

    ./environment.sh dev
    ./environment.sh staging
    ./environment.sh prod
    ./environment.sh test

Observe the exit status:

    echo "$?"

## Lab 3 — Server List

Create servers.txt:

    server-01
    server-02
    server-03

Create check-servers.sh:

    #!/bin/bash

    while read -r server; do
        echo "Checking $server"
    done < servers.txt

Run:

    chmod +x check-servers.sh
    ./check-servers.sh

## Lab 4 — Reusable Function

Create greet.sh:

    #!/bin/bash

    greet() {
        local name="$1"
        echo "Hello, $name"
    }

    if [ "$#" -lt 1 ]; then
        echo "Usage: $0 <name>"
        exit 1
    fi

    greet "$1"

Run:

    chmod +x greet.sh
    ./greet.sh Satyam

# Mini Project — System Health Monitor Foundation

The course plan includes a portfolio project called System Health Monitor involving filesystem automation and performance reporting.

Today, build only the scripting foundation.

Create health-check.sh.

Requirements:

1. Print hostname.
2. Print current user.
3. Print current date.
4. Check whether / exists and is a directory.
5. Print disk usage using df -h /.
6. Return exit code 0 on success.
7. Return a non-zero exit code if a required check fails.
8. Use at least one function and one conditional.

Do not build the complete monitoring system today. We will extend it in later sessions.

# DSA — Day 3

## Problem: Check if an Array Is Sorted

Given:

    [1, 2, 3, 4, 5]

Expected:

    true

Given:

    [1, 3, 2, 4]

Expected:

    false

Tasks:

1. Write the algorithm.
2. Explain the invariant.
3. Give time complexity.
4. Give space complexity.
5. Implement it in Java.

Target complexity:

    Time: O(n)
    Space: O(1)

# Interview Questions

1. What is a shebang?
2. Why do we use #!/bin/bash?
3. Difference between single and double quotes in Bash?
4. What is command substitution?
5. What are $0, $1, $#, and $@?
6. What is an exit code?
7. Why is exit code 0 normally treated as success?
8. Difference between for, while, and until?
9. When would you use case instead of multiple if statements?
10. Why should shell-script variables usually be quoted?
11. What is the purpose of a function?
12. How can a Bash script signal failure to a CI/CD pipeline?

# DevOps Connection

Shell scripting becomes useful for:

- server automation
- deployment scripts
- CI/CD jobs
- log processing
- health checks
- backup automation
- environment validation
- Docker entrypoints
- troubleshooting
- cloud operations

The important mindset is:

Don't manually repeat an operational task when a reliable script can perform it consistently.

# Day 3 Checkpoint

Before moving to Day 4, complete:

- Shebang explanation
- Variables and quoting
- if and case
- Numeric, string and file tests
- for loop
- while read
- until
- Functions
- Script arguments
- Exit codes
- All four hands-on labs
- System Health Monitor foundation
- Sorted-array Java solution
- Interview questions
- Learning report

## Learning Report

At the end of the day, record:

- What I learned
- Commands/scripts I practiced
- One mistake I made
- How I fixed it
- One DevOps use case
- One interview question I can now answer
- Git commit SHA
