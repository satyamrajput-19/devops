# Day 22 — GitHub Actions Deep Dive

This lesson follows Session 22 of the course: workflows, jobs, runners, secrets management, matrix builds, reusable workflows, self-hosted runners, custom actions, and a GitLab CI/CD comparison.

## Learning outcomes
- Explain workflows, events, jobs, steps, actions, and runners.
- Create a workflow triggered by pushes, pull requests, and manual dispatch.
- Use least-privilege workflow permissions.
- Understand secrets, matrix builds, reusable workflows, and self-hosted runners.
- Compare GitHub Actions with Jenkins and GitLab CI/CD.

## 1. Core terminology

- **Workflow:** YAML automation file stored under `.github/workflows/`.
- **Event:** Something that triggers a workflow, such as a push, pull request, or manual dispatch.
- **Job:** A set of steps executed on a runner.
- **Step:** A shell command or action.
- **Action:** A reusable unit of automation.
- **Runner:** A machine that executes the job. GitHub-hosted runners are managed by GitHub; self-hosted runners are managed by you or your organization.

Jobs run independently by default and may run in parallel. Use `needs` to express dependencies.

## 2. Create your first workflow

In your repository, create `.github/workflows/day22-ci.yml`:

```yaml
name: Day 22 - Python CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read

jobs:
  validate:
    name: Validate Python
    runs-on: ubuntu-latest

    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Show Python version
        run: python --version

      - name: Compile Python files
        run: python -m compileall -q .
```

This is a starter learning example. Review and pin third-party actions according to your organization's supply-chain policy; major-version tags are convenient but are not immutable. The compile step checks syntax only. Add unit tests and project-specific paths as appropriate.

**Important YAML note:** the key `on` is valid GitHub Actions syntax. Some generic YAML parsers may interpret it differently depending on YAML version.

### Verify the workflow
1. Commit the file to your repository.
2. Open the repository's **Actions** tab.
3. Select **Day 22 - Python CI**.
4. Inspect the run, job, and step logs.
5. Make a small harmless change and push again to observe the trigger.

Do not commit credentials or tokens to test secret handling.

## 3. Add Docker and Trivy (optional extension)

Once the basic workflow passes, extend it only if your repository contains a Dockerfile and the runner supports Docker and Trivy. Example steps inside the same job:

```yaml
      - name: Build image
        run: docker build -t day22-python-app:${{ github.sha }} .

      - name: Scan image
        run: trivy image --exit-code 1 --severity HIGH,CRITICAL day22-python-app:${{ github.sha }}
```

These steps intentionally fail the job when Trivy finds configured HIGH or CRITICAL findings. Ensure Trivy is installed before using the scan step; do not assume it is preinstalled on every runner. This example builds locally on the runner and does not push the image to a registry.

## 4. Secrets and permissions

Store sensitive values in GitHub Actions secrets, using repository, environment, or organization scope as appropriate. Reference a secret only when needed, for example:

```yaml
env:
  SERVICE_TOKEN: ${{ secrets.SERVICE_TOKEN }}
```

Never print secret values, put them in committed files, or expose them to untrusted pull-request code. Use environment protection for deployments when available.

Recommended security practices:
- Set the narrowest practical `permissions` block.
- Use `contents: read` when checkout is all the job needs.
- Grant write permissions only to jobs that genuinely require them.
- Prefer short-lived cloud credentials and OIDC federation over long-lived cloud keys where supported.
- Pin and review third-party actions; review their maintainers and requested permissions.
- Avoid running untrusted code on a self-hosted runner that has access to sensitive systems.
- Separate build/test jobs from deployment jobs and protect production environments.

## 5. Matrix builds

A matrix runs the same job across multiple configurations, such as Python versions:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        python-version: ['3.11', '3.12']

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      - run: python --version
```

This creates a separate job run for each matrix value. Add real tests before treating it as test coverage.

## 6. Job dependencies and artifacts

Example dependency:

```yaml
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Validation stage"

  package:
    needs: validate
    runs-on: ubuntu-latest
    steps:
      - run: echo "Runs only after validate succeeds"
```

For build outputs, use the official artifact upload/download actions where appropriate. Artifacts are useful for passing files between jobs and preserving build outputs. They are not the same as a container registry, and artifact retention is configured separately.

## 7. Reusable workflows and custom actions

- **Reusable workflow:** a workflow called by another workflow, commonly using `workflow_call`. Useful for standardizing CI across repositories.
- **Custom action:** reusable logic packaged as a JavaScript action, Docker action, or composite action. Useful for repeated steps.
- **Matrix:** repeats a job across multiple combinations of inputs.
- **Self-hosted runner:** gives you more control over hardware, software, and network access, but also makes patching, isolation, capacity, and security your responsibility.

Do not confuse a reusable workflow with a custom action: workflows compose jobs; actions are used within steps.

## 8. GitHub Actions vs. Jenkins vs. GitLab CI/CD

| Area | GitHub Actions | Jenkins | GitLab CI/CD |
|---|---|---|---|
| Typical configuration | YAML workflows in GitHub repository | Jenkinsfile (often Groovy-based pipeline DSL) | YAML in `.gitlab-ci.yml` |
| Execution | GitHub-hosted or self-hosted runners | Agents managed by Jenkins | GitLab-hosted or self-managed runners |
| Ecosystem | GitHub marketplace actions and GitHub integration | Large plugin ecosystem and flexible integrations | Integrated GitLab source control, CI, and DevSecOps features |
| Operations | Hosted runner reduces infrastructure management | Often requires you to manage Jenkins controller/plugins/agents | Depends on GitLab offering and runner model |
| Best fit | Repositories already using GitHub | Existing Jenkins environments or specialized custom pipelines | Teams using GitLab's integrated platform |

No tool is universally best. Consider existing source control, compliance, runner needs, team skills, cost, plugin/action supply-chain risk, and operational ownership.

## 9. Troubleshooting

| Problem | What to check |
|---|---|
| Workflow does not trigger | File path, YAML syntax, branch filters, event type |
| YAML parse error | Indentation, quoting, list structure, expression syntax |
| Action cannot access repository | Token permissions and checkout configuration |
| Secret is empty | Secret name, scope, event context, environment protection |
| Matrix job fails on one version | Runtime compatibility and version-specific dependencies |
| Docker command fails | Runner Docker support, daemon availability, permissions |
| Trivy command not found | Install/setup step and PATH |
| Deployment job runs too early | Add correct `needs` dependencies and environment protection |

## 10. Interview questions

1. What is the difference between a workflow, job, step, action, and runner?
2. How can a GitHub Actions workflow be triggered?
3. What does `needs` do?
4. What is a matrix build?
5. How should secrets be handled?
6. What is the difference between a reusable workflow and a custom action?
7. When would you choose a self-hosted runner?
8. Why should workflow permissions be restricted?
9. Compare GitHub Actions with Jenkins.
10. What is the difference between an uploaded workflow artifact and a container registry image?

## 11. Completion checklist

- [ ] Create `.github/workflows/day22-ci.yml`.
- [ ] Commit it to your repository.
- [ ] Verify a run in the Actions tab.
- [ ] Inspect logs for each step.
- [ ] Review least-privilege permissions and secret handling.
- [ ] Try a Python matrix job.
- [ ] Understand reusable workflows and custom actions.
- [ ] Add Docker/Trivy only if prerequisites are met.
- [ ] Document the result and troubleshooting notes.

Only claim the workflow passed after you have observed a successful run.

**Next:** Day 23 — CI/CD Capstone: connect Git push, tests, SonarQube, Trivy, Docker build, and deployment promotion.
