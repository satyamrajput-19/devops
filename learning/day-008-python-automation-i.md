# Day 8 — Python for Automation I

## Course Alignment

This follows **Session 8: Python for Automation I** from the DevOps & Cloud Engineering course plan.

Course topics:
- Python syntax, data types, functions & error handling
- Working with files & directories (os, pathlib, shutil)
- Rewrite your shell scripts in Python
- Virtual environments & pip

## Learning Objectives

By the end of Day 8, I should be able to:
- Write basic Python scripts using variables, data types and control flow.
- Create reusable functions.
- Handle expected runtime errors safely.
- Read, write and manage files/directories.
- Use `os`, `pathlib` and `shutil` for automation.
- Translate common Bash automation tasks into Python.
- Create an isolated Python environment and install dependencies with pip.

## 1. Python Syntax Fundamentals

Python uses indentation to define code blocks.

Example:

    name = "Satyam"
    print(f"Hello, {name}")

Common building blocks:
- Variables
- Strings
- Integers and floats
- Booleans
- Lists
- Tuples
- Dictionaries
- Sets
- Conditions
- Loops
- Functions

Python is valuable in DevOps because it can automate filesystem operations, cloud APIs, configuration processing, reporting and operational tasks.

## 2. Data Types

### Strings

    service = "nginx"

### Integers

    retry_count = 3

### Lists

    servers = ["web-01", "web-02", "web-03"]

### Dictionaries

    server = {
        "name": "web-01",
        "port": 80
    }

### Booleans

    healthy = True

Practice identifying the correct type for values before writing automation logic.

## 3. Conditions and Loops

Example condition:

    if retry_count > 0:
        print("Retry available")
    else:
        print("No retries left")

Example loop:

    for server in servers:
        print(server)

Automation scripts frequently combine conditions and loops to process many files, servers, records or command results.

## 4. Functions

Functions package reusable logic.

Example:

    def check_file(path):
        return path.exists()

Good automation functions should have:
- A clear purpose
- Meaningful parameters
- Predictable return values
- Small, testable logic

Example:

    from pathlib import Path

    def count_lines(file_path):
        path = Path(file_path)
        with path.open("r", encoding="utf-8") as file:
            return sum(1 for _ in file)

## 5. Error Handling

Automation must handle expected failures.

Example:

    try:
        with open("config.txt", "r", encoding="utf-8") as file:
            data = file.read()
    except FileNotFoundError:
        print("config.txt does not exist")

Useful exception-handling practices:
- Catch specific exceptions where possible.
- Provide useful error messages.
- Avoid silently ignoring failures.
- Exit with an appropriate failure state when automation cannot safely continue.

## 6. Working with Files — pathlib

`pathlib` provides an object-oriented way to work with filesystem paths.

Example:

    from pathlib import Path

    log_dir = Path("logs")

    if not log_dir.exists():
        log_dir.mkdir(parents=True)

    for file in log_dir.iterdir():
        print(file)

Useful operations:
- `Path.exists()`
- `Path.is_file()`
- `Path.is_dir()`
- `Path.mkdir()`
- `Path.iterdir()`
- `Path.read_text()`
- `Path.write_text()`

For new Python automation scripts, `pathlib` is often a clean choice for filesystem path handling.

## 7. os Module

The `os` module provides operating-system interfaces.

Examples:

    import os

    print(os.getcwd())
    print(os.environ.get("HOME"))

Useful areas:
- Current working directory
- Environment variables
- Directory operations
- Operating-system information
- Process/environment integration

DevOps scripts commonly read configuration from environment variables rather than hard-coding sensitive values.

## 8. shutil Module

`shutil` provides high-level file and directory operations.

Examples:

    import shutil

    shutil.copy("source.txt", "backup.txt")

    shutil.copytree("source_dir", "backup_dir", dirs_exist_ok=True)

Useful automation tasks:
- Copy files
- Copy directory trees
- Move files/directories
- Remove directory trees
- Build backup workflows

Always validate source paths and destination paths before destructive operations.

## 9. Rewrite a Bash Script in Python

A useful Day 8 exercise is converting the earlier Bash practice into Python.

### Bash concept

    grep -n "$word" "$file"

### Python approach

    from pathlib import Path

    word = "error"
    file_path = Path("application.log")

    try:
        for line_number, line in enumerate(
            file_path.read_text(encoding="utf-8").splitlines(), start=1
        ):
            if word in line:
                print(f"{line_number}:{line}")
    except FileNotFoundError:
        print("File does not exist")

The goal is not merely to translate syntax. Understand the operational behavior and error handling.

## 10. Bash-to-Python Automation Mapping

| Bash | Python |
|---|---|
| `$1`, `$2` | Function arguments / CLI arguments |
| `if` | `if` |
| `for` | `for` |
| `grep` | String/file processing |
| `mkdir` | `Path.mkdir()` |
| `cp` | `shutil.copy()` |
| `mv` | `shutil.move()` |
| `rm -r` | `shutil.rmtree()` |
| Environment variables | `os.environ` / `os.getenv()` |
| Exit status | Return values / `sys.exit()` |

## 11. Virtual Environments

A virtual environment isolates Python dependencies for a project.

Create one:

    python3 -m venv .venv

Activate on Linux/macOS:

    source .venv/bin/activate

Verify Python:

    python --version

Deactivate:

    deactivate

A project should avoid relying on random system-wide packages.

## 12. pip

`pip` is used to install Python packages.

Example:

    python -m pip install requests

Check installed packages:

    python -m pip list

Save dependencies:

    python -m pip freeze > requirements.txt

Install from a requirements file:

    python -m pip install -r requirements.txt

For automation projects, keep dependencies explicit and reproducible.

## 13. Hands-On Labs

### Lab 1 — Python Basics

Create a script that:
- Defines variables using multiple data types.
- Uses a condition.
- Iterates through a list.
- Calls a function.
- Handles an invalid input case.

### Lab 2 — File and Directory Automation

Build a script that:
- Accepts a directory path.
- Checks whether it exists.
- Lists files.
- Creates a backup directory.
- Copies selected files into the backup directory.

### Lab 3 — Bash to Python

Rewrite one of the earlier Bash practice scripts in Python.

Recommended targets:
- Word search
- Multiple directory creation
- Backup automation

Document what changed and why.

### Lab 4 — Virtual Environment

Create:

    python3 -m venv .venv

Then:
- Activate it.
- Upgrade/install a required package with pip.
- Record the dependency.
- Deactivate the environment.

## 14. Failure Engineering

Intentionally test:
- Missing file
- Missing directory
- Invalid path
- Permission failure
- Existing destination
- Empty input
- Unexpected file type

For every failure, answer:
1. What failed?
2. Which exception or condition identified it?
3. Did the script fail safely?
4. What useful diagnostic message was produced?
5. How would you prevent the failure in production automation?

## 15. DevOps Connection

Python becomes increasingly useful as automation complexity grows.

    Linux / Bash
         |
         v
    Python Automation
         |
         +---- Files / Logs
         |
         +---- APIs
         |
         +---- Cloud Automation
         |
         +---- CLI Tools
         |
         v
    DevOps Platforms

Day 8 focuses on the foundation needed before API-driven and CLI-based automation.

## 16. Interview Questions

### Q1. Why use Python in DevOps?

Python is useful for reusable automation, filesystem operations, API integration, reporting and tooling.

### Q2. What is the difference between a list and a dictionary?

A list is an ordered collection accessed by position, while a dictionary stores key-value mappings.

### Q3. Why use functions?

Functions make automation logic reusable, testable and easier to maintain.

### Q4. Why use exception handling?

To handle expected runtime failures without allowing automation to fail unpredictably.

### Q5. What is pathlib?

A Python module that provides object-oriented filesystem path handling.

### Q6. What are os and shutil used for?

`os` provides operating-system interfaces, while `shutil` provides high-level file and directory operations.

### Q7. Why use virtual environments?

To isolate project dependencies and reduce conflicts between Python projects.

### Q8. What is pip?

A package installer used to install Python packages and manage project dependencies.

## 17. DSA / CS Fundamentals

### Problem — Reverse a String

Given a string, return the string in reverse order.

Example:

    Input:  "devops"
    Output: "spoved"

Practice:
- Understand the two-pointer approach.
- Compare iterative and slicing-based solutions.
- Consider time and space complexity.

Target:
- Time: O(n)
- Extra space: O(1) for an in-place character-array approach.

## 18. Day 8 Checkpoint

Before moving on, explain without notes:
- Python basic syntax
- Core data types
- Functions
- Error handling
- pathlib
- os
- shutil
- Bash-to-Python automation
- Virtual environments
- pip

Practical checkpoint:

Build a small Python automation script that accepts a directory, validates it, processes files, handles errors and creates a backup/output directory.

## Learning Report

Record:
- One Python concept I understood well
- One filesystem operation I automated
- One Bash script I rewrote in Python
- One exception I handled
- One virtual-environment command I practiced
- One Python automation interview question I can now answer confidently

## Next

**Day 9 — Python for Automation II**

Course topics:
- Modules & project structure; argparse CLIs
- Calling APIs with requests; parsing JSON & YAML
- Logging & scheduling Python jobs
- Build a real automation CLI tool
