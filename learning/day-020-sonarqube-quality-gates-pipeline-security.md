# Day 20 — SonarQube, Quality Gates, and Pipeline Security

This lesson follows Session 20 of the course: code quality in operations, setting up SonarQube, analyzing a project, enforcing Quality Gates in a pipeline, and combining quality checks with vulnerability scanning.

## Learning outcomes
- Explain static analysis, code smells, bugs, vulnerabilities, and technical debt.
- Run SonarQube locally and analyze a small project.
- Understand Quality Gate pass/fail decisions.
- Combine source analysis with tests and container image scanning.
- Store credentials safely and avoid deploying when required checks fail.

## 1. Core concepts

- **Static analysis:** Examines source code without executing the application.
- **Bug:** A defect that may cause incorrect behavior.
- **Code smell:** A maintainability issue or design pattern that may make code harder to change.
- **Vulnerability:** A security weakness that could be exploitable.
- **Quality Gate:** A set of conditions applied to analysis results. A CI workflow can fail when the gate is red.
- **Shift-left security:** Perform quality and security checks earlier in development, before deployment.

SonarQube's exact rules, supported languages, and Quality Gate capabilities depend on the edition, version, and configuration. Do not assume every feature is available in every edition.

## 2. Start SonarQube locally

Use the official SonarQube documentation for the current supported Docker image and system requirements. A local lab can use Docker, but memory and platform requirements vary by version. Do not expose a local SonarQube instance to the public internet.

A common local Docker-based approach is to run the official SonarQube Community Build image with a persistent data volume, following the current official installation instructions. After startup, open `http://localhost:9000` and complete the initial setup. Use a strong local lab password and change any initial credentials immediately.

Do not copy commands from old tutorials without checking current official instructions; image tags, supported runtime requirements, and configuration can change.

## 3. Analyze a small Python project

Use a project you own, such as the Day 18 Python application, and make sure it has a Git repository. Create `sonar-project.properties` in the project root:

```properties
sonar.projectKey=day20-python-app
sonar.projectName=Day 20 Python App
sonar.sources=.
sonar.exclusions=.venv/**,**/__pycache__/**,.git/**
```

This is a starting configuration; adapt it if the project contains generated files, tests, or additional languages. Do not include tokens or passwords in this file.

### Run analysis

Use the scanner method recommended by your installed SonarQube version and edition. For a locally installed SonarScanner CLI, the general pattern is:

```bash
sonar-scanner \
  -Dsonar.projectKey=day20-python-app \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.token="$SONAR_TOKEN"
```

Create a token in the SonarQube user interface and set it in your current shell using the operating system's supported environment-variable syntax. Never commit the token or paste it into public logs. Some current SonarQube workflows use a scanner integrated with the build tool or CI action; follow the official instructions for your environment.

After analysis completes, open the project dashboard and inspect issues, security findings, maintainability indicators, and Quality Gate status.

## 4. Quality Gates

A Quality Gate evaluates analysis results against configured conditions. For example, a team might require no new blocker issues and a minimum percentage of coverage on new code. Exact defaults and supported conditions depend on the SonarQube version and edition.

Good practices:
- Start with the gate supplied by your SonarQube installation.
- Understand which conditions apply to new code vs. all existing code.
- Add tests and coverage reporting when supported by your language/tooling setup.
- Fix genuine issues rather than hiding findings or weakening the gate simply to get a green result.
- A green gate is not proof that code is bug-free or completely secure.

## 5. Combine code quality and image scanning

For a containerized app, use this conceptual sequence:

1. Check out source code.
2. Install dependencies.
3. Run unit tests.
4. Run SonarQube analysis.
5. Wait for and enforce the Quality Gate result using the scanner/CI integration supported by your setup.
6. Build the container image.
7. Scan the image with Trivy.
8. Publish/deploy only if the required checks pass.

Example image scan after building:

```bash
docker build -t day20-python-app:local .
trivy image --severity HIGH,CRITICAL day20-python-app:local
```

This example assumes Docker and Trivy are installed and the Dockerfile is present. Decide the failure policy explicitly: in CI, configure the scanner to return a non-zero exit code for findings that violate your policy. A scan that only prints findings but never changes pipeline status does not enforce a gate.

## 6. Pipeline security basics

- Store SonarQube tokens and registry credentials in CI secrets, not source code.
- Give tokens and pipeline identities the minimum permissions required.
- Do not print secrets in logs.
- Prefer short-lived credentials where supported.
- Pin or review third-party CI actions and scanner versions.
- Restrict deployment permissions to trusted branches/environments.
- Avoid running untrusted pull-request code with access to deployment secrets.
- Keep analysis and scan results available as build evidence.
- Define what happens when a scanner is unavailable; do not silently treat missing security checks as success.

## 7. Troubleshooting

| Problem | What to check |
|---|---|
| SonarQube page does not open | Container status/logs, port mapping, memory/system requirements |
| Scanner cannot connect | Host URL, DNS/network reachability, service readiness |
| Authentication fails | Token validity, permissions, correct environment variable |
| Project has no issues or unexpected files | Project key, source path, exclusions, scanner output |
| Quality Gate is pending | Analysis processing and CI integration that waits for the gate |
| Trivy is missing | Install it from official instructions and verify PATH |
| Scan finds vulnerabilities | Review package, severity, fix version, exploitability, and update strategy |
| CI passes despite failed scan | Check command exit code and ensure failure conditions are configured |

## 8. Interview questions

1. What is static code analysis?
2. What is the difference between a bug, code smell, and vulnerability?
3. What does a Quality Gate do?
4. Why is it useful to check new code separately?
5. How is SonarQube different from Trivy?
6. Why should pipeline tokens be stored as CI secrets?
7. What does shift-left security mean?
8. What should happen when a required security check fails?
9. Why does a green Quality Gate not guarantee completely secure code?
10. How would you troubleshoot a pipeline that ignores a failed scan?

## 9. Completion checklist

- [ ] Read the official setup requirements and start SonarQube locally.
- [ ] Create a project and configure source analysis.
- [ ] Run a scan and inspect the results.
- [ ] Review the Quality Gate status and its actual conditions.
- [ ] Run a Trivy scan on a local image if available.
- [ ] Explain how a failed check should block a pipeline.
- [ ] Confirm tokens and credentials are not committed.
- [ ] Document commands, screenshots, findings, and any blockers in GitHub.

Only state that analysis or scanning succeeded if you ran it and observed the result.

**Next:** Day 21 — Jenkins CI/CD: pipelines, Jenkinsfile, agents, artifacts, and build triggers.
