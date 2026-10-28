# Day 23 — CI/CD Capstone: End-to-End Pipeline

This follows Session 23 of the course. The goal is to connect the stages you've studied into one documented workflow:

**Git push → tests → SonarQube Quality Gate → Docker build → Trivy scan → deployment promotion**

The course also emphasizes environment promotion (dev → staging → prod), secrets and least privilege, and presenting/defending the pipeline in an interview. This lesson provides a portfolio-ready blueprint; deployment to AWS requires an actual application, cloud configuration, and credentials, so the example does not pretend to deploy without those prerequisites.

## Learning outcomes

- Explain the full CI/CD flow and why stage ordering matters.
- Create a GitHub Actions workflow with test, analysis, image build, and scan stages.
- Fail the workflow when a required check fails.
- Use a unique image tag and preserve build traceability.
- Separate dev, staging, and production promotion.
- Protect secrets and deployment permissions.
- Document the design and defend trade-offs in an interview.

## 1. Pipeline architecture

1. **Source:** A developer pushes a commit or opens a pull request.
2. **Test:** CI installs dependencies and runs automated tests.
3. **Code analysis:** SonarQube analyzes source code and the workflow waits for the Quality Gate.
4. **Build:** Docker builds an image only after earlier required checks pass.
5. **Image scan:** Trivy scans the image for configured vulnerability severities.
6. **Artifact:** The image is tagged with the commit SHA and stored/published.
7. **Promotion:** Deploy to development first, validate it, then promote the same immutable image to staging and production using protected environments and approvals.

A successful CI run is not the same as a successful production deployment. Track each separately.

## 2. Repository structure

A useful structure is:

```text
your-app/
├── .github/
│   └── workflows/
│       └── ci-cd-capstone.yml
├── tests/
├── Dockerfile
├── requirements.txt
├── sonar-project.properties
└── README.md
```

Adapt paths and commands to your actual app. The example below assumes a Python project with `requirements.txt`, tests in `tests/`, and a Dockerfile in the repository root. If your app has a different layout, change the test and build steps accordingly.

## 3. Configure SonarQube

Create or configure a SonarQube project. Set these repository secrets/variables in GitHub:
- `SONAR_TOKEN`: secret containing a project analysis token.
- `SONAR_HOST_URL`: variable or secret containing your SonarQube server URL.

For a local SonarQube server, GitHub-hosted runners cannot normally reach `localhost` on your computer. Use an appropriately reachable, secured SonarQube server or a self-hosted runner with network access. Do not expose an unsecured SonarQube instance to the public internet.

Create `sonar-project.properties`:

```properties
sonar.projectKey=devops-capstone
sonar.projectName=DevOps CI/CD Capstone
sonar.sources=.
sonar.tests=tests
sonar.exclusions=.venv/**,**/__pycache__/**,.git/**,tests/**
```

Adjust language-specific test and coverage settings for your project. The Quality Gate must be configured in SonarQube; a scan by itself does not guarantee the gate is enforced.

## 4. Example GitHub Actions pipeline

Create `.github/workflows/ci-cd-capstone.yml`. This is a reference template; verify action versions and your repository layout before using it. It runs tests, SonarQube analysis, builds an image, and scans it. It deliberately does not deploy to AWS until deployment prerequisites are configured.

```yaml
name: CI/CD Capstone

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read

env:
  IMAGE_NAME: devops-capstone

jobs:
  test:
    name: Test application
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
          if [ -f requirements-dev.txt ]; then pip install -r requirements-dev.txt; fi
      - name: Run tests
        run: python -m pytest -q

  quality:
    name: SonarQube analysis and Quality Gate
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - name: SonarQube scan
        uses: SonarSource/sonarqube-scan-action@v5
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ vars.SONAR_HOST_URL }}
        with:
          args: >
            -Dsonar.qualitygate.wait=true

  image:
    name: Build and scan container
    runs-on: ubuntu-latest
    needs: [test, quality]
    if: github.event_name != 'pull_request'
    steps:
      - uses: actions/checkout@v4
      - name: Build image
        run: docker build -t "${IMAGE_NAME}:${GITHUB_SHA}" .
      - name: Set up Trivy
        uses: aquasecurity/setup-trivy@v0.2.0
        with:
          version: v0.58.1
      - name: Scan image
        run: trivy image --exit-code 1 --severity HIGH,CRITICAL "${IMAGE_NAME}:${GITHUB_SHA}"
      - name: Save image artifact
        run: docker save "${IMAGE_NAME}:${GITHUB_SHA}" -o image.tar
      - uses: actions/upload-artifact@v4
        with:
          name: container-image-${{ github.sha }}
          path: image.tar
          retention-days: 7
```

### Important template notes

- The template expects pytest to be available. Add it to `requirements-dev.txt` or your project's development dependency file; otherwise the test step will fail.
- If your repository does not yet have tests, add a minimal real test rather than silently skipping test validation.
- Confirm the SonarQube scanner action version and Trivy setup action version against their official documentation before adopting this in a real project. Pin actions to reviewed immutable commit SHAs for stronger supply-chain controls.
- SonarQube needs a reachable host URL and valid token. The Quality Gate wait option asks the scanner to wait for the gate result; validate behavior with your installed SonarQube version and configuration.
- The image job is skipped on pull requests in this sample. This avoids publishing/processing images from untrusted PR contexts; adapt the policy if you want safe PR-only image builds.
- This workflow builds and archives the image tar file; it does not push the image to a registry and does not deploy it to AWS.
- GitHub-hosted runners usually provide Docker support, but local and self-hosted environments need their own setup.

## 5. Deployment promotion: dev → staging → prod

Promotion should move the **same immutable image digest** through environments, not rebuild different images for each environment.

Suggested release flow:
1. Merge validated code to `main`.
2. Build and scan one image; tag it with the commit SHA and record its digest.
3. Deploy that image to dev.
4. Run smoke tests and health checks.
5. Promote the exact same image digest to staging.
6. Require approval and protected environment rules before production.
7. Record the release, image digest, source commit, deployment result, and rollback target.

For an AWS deployment, choose the target first (for example ECS or EKS), then configure the relevant registry, role, network, and deployment steps. Prefer GitHub Actions OIDC federation with a narrowly scoped AWS IAM role instead of long-lived AWS access keys. Do not add a production deploy step with placeholder credentials and claim it is functional.

## 6. Security checklist

- Keep `SONAR_TOKEN`, cloud role settings, and registry credentials out of source code.
- Use minimal `permissions` at workflow and job level.
- Do not expose production secrets to untrusted pull-request workflows.
- Restrict production deployment through GitHub Environments, reviewers, and branch rules.
- Scan the final image and make policy-violating findings fail the job.
- Use unique, traceable tags and record the image digest.
- Review/pin third-party actions and control who can change workflows.
- Store deployment logs and keep a known-good artifact for rollback.
- Define how scanner outages are handled; don't silently treat missing checks as a pass.

## 7. Troubleshooting

| Symptom | What to check |
|---|---|
| Tests fail immediately | Python dependencies, test path, test framework, and test discovery |
| SonarQube cannot connect | Host URL reachability from the runner, TLS, token, project key |
| Quality Gate does not block | Gate configuration, scanner wait option, action logs, and exit status |
| Docker build fails | Dockerfile path, build context, dependencies, image build logs |
| Trivy setup/scan fails | Action/version, network access, PATH, database download, image tag |
| Image job is skipped | Check the `if` condition and whether the event was a pull request |
| Artifact missing | Confirm the save step succeeded and the upload path exists |
| AWS deploy is denied | OIDC trust policy, IAM permissions, account/region, target resource access |
| Production deploy runs too early | Environment protection, job dependencies, branch filters, approvals |

## 8. Present and defend your pipeline

In your README, document:
- Architecture diagram or text flow.
- Required secrets and variables (names only; never values).
- Each stage and why it exists.
- Failure conditions and what blocks promotion.
- Artifact tag/digest and source commit relationship.
- Environment promotion and approval strategy.
- Known limitations and next improvements.
- Evidence of actual runs, including failed-run troubleshooting.

Be transparent about which stages you've actually executed. A documented template is valuable, but do not claim a real deployment until you have completed and verified it.

## 9. Interview questions

1. Walk through your pipeline from Git push to deployment.
2. Why should tests and the SonarQube Quality Gate run before the Docker build?
3. How do you make sure a failed Trivy scan blocks release?
4. Why promote the same image digest between environments?
5. What is the difference between a tag and an image digest?
6. How would you store SonarQube and AWS credentials securely?
7. What is GitHub OIDC and why is it useful for AWS?
8. How would you add a manual approval before production?
9. How do you roll back a bad release?
10. What evidence would you show an interviewer to prove the pipeline works?

## 10. Completion checklist

- [ ] Confirm the application has a Dockerfile and tests.
- [ ] Add the workflow YAML to `.github/workflows/`.
- [ ] Configure SonarQube project settings and GitHub secret/variable names.
- [ ] Run the workflow and inspect each stage.
- [ ] Verify the Quality Gate can fail the workflow.
- [ ] Verify Trivy findings fail the image job under the configured policy.
- [ ] Confirm the image artifact is uploaded.
- [ ] Document dev → staging → prod promotion and approval controls.
- [ ] Record real run results and any limitations in GitHub.
- [ ] Prepare a short verbal walkthrough for an interview.

**Next:** Day 24 — Terraform: providers, resources, variables, outputs, state, and provisioning a VPC.
