# Day 9 — Python for Automation II

## Course Alignment

This follows **Session 9: Python for Automation II** from the DevOps & Cloud Engineering course plan.

Course topics:
- Modules & project structure; `argparse` CLIs
- Calling APIs with `requests`; parsing JSON & YAML
- Logging & scheduling Python jobs
- Build a real automation CLI tool

## Learning Objectives

By the end of Day 9, I should be able to:
- Organize Python automation into reusable modules.
- Build command-line interfaces with `argparse`.
- Call HTTP APIs with `requests`.
- Parse JSON and YAML configuration/data.
- Add structured logging to automation.
- Understand scheduling patterns for Python jobs.
- Build a practical automation CLI tool.

## 1. Modules & Project Structure

A module is a Python file containing reusable code.

Example structure:

    automation-tool/
    ├── app.py
    ├── cli.py
    ├── api.py
    ├── config.py
    ├── utils.py
    ├── requirements.txt
    └── README.md

Keep responsibilities separated:
- `cli.py` — command-line interface
- `api.py` — API communication
- `config.py` — configuration loading
- `utils.py` — reusable helpers
- `app.py` — application flow

The goal is maintainable automation rather than one large script.

## 2. Imports

Example:

    from pathlib import Path
    import json

Use imports to reuse standard-library modules and project modules.

Avoid unnecessary duplication and keep module responsibilities clear.

## 3. argparse CLI

`argparse` provides a standard way to build command-line interfaces.

Example:

    import argparse

    parser = argparse.ArgumentParser(
        description="DevOps automation tool"
    )

    parser.add_argument(
        "--file",
        required=True,
        help="Input file"
    )

    args = parser.parse_args()

    print(args.file)

A good CLI should:
- Provide useful help text.
- Validate required arguments.
- Use meaningful option names.
- Return a useful exit status on failure.

Example usage:

    python app.py --file application.log

## 4. Positional and Optional Arguments

Positional argument:

    parser.add_argument("filename")

Optional argument:

    parser.add_argument("--output", default="report.txt")

Boolean flag:

    parser.add_argument(
        "--verbose",
        action="store_true"
    )

Practice designing commands that are predictable and easy to use.

## 5. Calling APIs with requests

The `requests` library can make HTTP requests from Python.

Example:

    import requests

    response = requests.get(
        "https://example.com",
        timeout=10
    )

    response.raise_for_status()

    print(response.status_code)

Important practices:
- Set a timeout.
- Check the response status.
- Handle request failures.
- Do not hard-code credentials or tokens.

For automation, API calls are a major step beyond local filesystem scripting.

## 6. Working with JSON

JSON is commonly used by APIs.

Example:

    import json

    data = '{"service": "nginx", "port": 80}'
    parsed = json.loads(data)

    print(parsed["service"])

Writing JSON:

    with open("config.json", "w", encoding="utf-8") as file:
        json.dump(parsed, file, indent=2)

Common operations:
- `json.loads()` — JSON string to Python object
- `json.dumps()` — Python object to JSON string
- `json.load()` — JSON file to Python object
- `json.dump()` — Python object to JSON file

## 7. Parsing YAML

YAML is commonly used for configuration.

Example:

    import yaml

    with open("config.yaml", "r", encoding="utf-8") as file:
        config = yaml.safe_load(file)

Use `safe_load()` when loading YAML configuration.

A typical configuration might contain:

    service:
      name: nginx
      port: 80

    monitoring:
      enabled: true

Keep configuration separate from application logic where practical.

## 8. Logging

Use Python's `logging` module instead of relying only on `print()` for operational automation.

Example:

    import logging

    logging.basicConfig(
        level=logging.INFO,
        format="%(asctime)s %(levelname)s %(message)s"
    )

    logging.info("Automation started")
    logging.warning("Configuration is incomplete")

Useful levels:
- DEBUG
- INFO
- WARNING
- ERROR
- CRITICAL

Logs should help answer:
- What happened?
- When did it happen?
- What operation failed?
- What should an operator investigate?

## 9. Error Handling for API Automation

Automation should distinguish expected failures.

Example:

    import requests

    try:
        response = requests.get(
            "https://example.com",
            timeout=10
        )
        response.raise_for_status()
    except requests.RequestException as exc:
        print(f"API request failed: {exc}")

Combine API error handling with logging and meaningful exit codes.

## 10. Scheduling Python Jobs

Python automation can be executed on a schedule.

A common Linux approach is cron.

Example:

    */15 * * * * /path/to/.venv/bin/python /path/to/app.py

Scheduling considerations:
- Use an explicit Python interpreter.
- Use absolute paths where appropriate.
- Capture useful logs.
- Ensure the job is safe to run repeatedly.
- Test the command manually before scheduling it.

The Day 4 Bash work on cron and automation provides the operational foundation for this topic.

## 11. Real Automation CLI Tool

### Project — DevOps Automation CLI

Build a small Python CLI tool that can perform useful operational work.

Suggested commands:

    python devops_cli.py health
    python devops_cli.py logs --file application.log
    python devops_cli.py report --input data.json
    python devops_cli.py config --file config.yaml

Possible capabilities:
- Analyze log files.
- Generate a report.
- Read JSON/YAML configuration.
- Call an API.
- Write structured logs.
- Return appropriate exit codes.

The objective is to combine the Day 8 and Day 9 Python foundations into one reusable automation tool.

## 12. Suggested Project Structure

    devops-cli/
    ├── devops_cli/
    │   ├── __init__.py
    │   ├── cli.py
    │   ├── api.py
    │   ├── config.py
    │   ├── logs.py
    │   └── report.py
    ├── tests/
    ├── requirements.txt
    ├── README.md
    └── .gitignore

Keep the CLI layer separate from business logic so individual components can be tested and reused.

## 13. Hands-On Labs

### Lab 1 — Build an argparse CLI

Create a CLI that accepts:
- Input file
- Output file
- Optional verbose flag

Test:
- Missing required arguments
- Invalid paths
- Help output
- Successful execution

### Lab 2 — API Automation

Build a script that:
- Calls a public test API.
- Uses a timeout.
- Checks the HTTP response.
- Parses JSON.
- Logs success/failure.

### Lab 3 — JSON/YAML Configuration

Create:
- `config.json`
- `config.yaml`

Write a Python script that loads configuration and prints selected values.

### Lab 4 — Logging and Scheduling

Build a small Python job that:
- Logs when it starts.
- Performs an automation task.
- Logs success/failure.
- Can be executed manually.
- Can be scheduled with cron.

### Lab 5 — Real Automation CLI

Combine the previous labs into one small DevOps CLI tool.

Document:
- Installation
- CLI commands
- Configuration
- Examples
- Failure cases

## 14. Failure Engineering

Intentionally test:
- Missing CLI argument
- Invalid file path
- Malformed JSON
- Malformed YAML
- API timeout
- HTTP error response
- Missing configuration key
- Permission failure
- Invalid command
- Repeated scheduled execution

For every failure, answer:
1. What failed?
2. How did the program detect it?
3. Was the failure logged clearly?
4. Was a useful exit status returned?
5. Could the failure be retried safely?
6. How would you prevent it in production?

## 15. DevOps Connection

Day 9 moves Python from local scripting toward operational tooling:

    Python
       |
       +---- CLI
       |
       +---- APIs
       |
       +---- JSON / YAML
       |
       +---- Logging
       |
       +---- Scheduling
       |
       v
    DevOps Automation Tools

This foundation will later support cloud automation, CI/CD tooling, infrastructure workflows and platform engineering tasks.

## 16. Interview Questions

### Q1. Why use argparse?

It provides a standard way to build command-line interfaces and parse user-supplied arguments.

### Q2. Why should API requests use a timeout?

Without a timeout, automation can wait indefinitely when a remote service does not respond.

### Q3. What is the difference between JSON and YAML?

Both can represent structured data, but JSON is widely used for APIs while YAML is commonly used for human-readable configuration.

### Q4. Why use logging instead of print?

Logging provides levels, timestamps and operational diagnostics that are more suitable for automation and troubleshooting.

### Q5. What should an automation script do when an API call fails?

Detect the failure, log useful context, handle the expected exception and return an appropriate failure status.

### Q6. Why separate CLI code from business logic?

It improves maintainability, testing and reuse.

### Q7. What is cron used for?

Cron schedules commands or jobs to run at specified times or intervals on Unix-like systems.

### Q8. Why should automation be safe to run repeatedly?

Scheduled and operational tooling may execute multiple times. Repeated execution should avoid unintended duplication or destructive side effects.

## 17. DSA / CS Fundamentals

### Problem — Valid Anagram

Given two strings, determine whether they contain the same characters with the same frequencies.

Example:

    Input:  "listen", "silent"
    Output: true

Practice:
- Character frequency counting.
- Hash-map/dictionary usage.
- Compare an O(n log n) sorting approach with an O(n) frequency approach.

Target:
- Time: O(n)
- Extra space: O(k), where k is the number of distinct characters.

## 18. Day 9 Checkpoint

Before moving on, explain without notes:
- Python modules
- Project structure
- argparse
- requests
- JSON parsing
- YAML parsing
- logging
- cron scheduling
- API error handling
- CLI design

### Practical checkpoint

Build a Python CLI that:
1. Accepts a file/config argument.
2. Validates the input.
3. Reads JSON or YAML.
4. Calls an API.
5. Logs the result.
6. Handles failures.
7. Returns a meaningful exit status.

## Learning Report

Record:
- One module/project-structure concept I understood well
- One CLI command I built
- One API call I automated
- One JSON/YAML parsing task I completed
- One logging technique I practiced
- One scheduling task I tested
- One Python automation interview question I can now answer confidently

## Next

**Day 10 — AWS Fundamentals / Cloud Foundations**

The next phase begins cloud-focused learning after the Python automation foundation.
