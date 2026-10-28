# Day 21 — Jenkins CI/CD: Jenkinsfile, Agents, Triggers, and Artifacts

This lesson follows Session 21 of the 32-session course: CI/CD fundamentals, Jenkinsfile syntax, artifact versioning, build triggers (SCM polling, webhooks, schedules), agents and executors, and key plugins such as Docker, Git, and Maven.

## Learning outcomes
- Explain how Jenkins controllers, agents, and executors work.
- Write a Declarative Jenkinsfile with stages and post-build actions.
- Understand manual, webhook, polling, and scheduled triggers.
- Explain how plugins extend Jenkins.
- Version a container image using a build number.
- Apply safe credential handling and least privilege.

## 1. CI/CD basics

- **Continuous Integration (CI):** Frequently integrate code and automatically validate it with builds and tests.
- **Continuous Delivery:** Keep validated changes ready for release.
- **Continuous Deployment:** Automatically release changes that pass the required checks.
- **Pipeline:** A version-controlled sequence of build, test, scan, package, and delivery stages.

## 2. Jenkins architecture

- **Controller:** Hosts Jenkins configuration, coordinates jobs, and schedules work.
- **Agent:** A machine or execution environment that runs pipeline steps.
- **Executor:** A slot that lets an agent run one task at a time. Increasing executors may increase concurrency, but also consumes more resources.
- **Plugin:** Adds integration or functionality, such as Git source checkout, Docker, or Maven support.

For a learning lab, a single machine can run Jenkins and execute builds, but production setups should avoid running untrusted builds with broad controller permissions. Keep the controller protected and use appropriately isolated agents.

## 3. Declarative Jenkinsfile

Create a file named `Jenkinsfile` at the root of your application repository:

```groovy
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = 'day21-python-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Python') {
            steps {
                sh 'python3 --version'
                sh 'python3 -m compileall -q .'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t "${IMAGE_NAME}:${IMAGE_TAG}" .'
            }
        }

        stage('Scan Image') {
            steps {
                // The agent must have Trivy installed.
                sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL "${IMAGE_NAME}:${IMAGE_TAG}"'
            }
        }

        stage('Export Image Artifact') {
            steps {
                sh 'docker save "${IMAGE_NAME}:${IMAGE_TAG}" -o "${IMAGE_NAME}-${IMAGE_TAG}.tar"'
                archiveArtifacts artifacts: 'day21-python-app-*.tar', fingerprint: true
            }
        }
    }

    post {
        success {
            echo 'Pipeline passed all configured stages.'
        }
        failure {
            echo 'Pipeline failed. Review the stage logs before retrying.'
        }
        always {
            echo 'Pipeline execution finished.'
        }
    }
}
```

This example expects an application repository with a Dockerfile, and an agent that has Python 3, Docker CLI/daemon access, and Trivy installed. The Python compilation step checks syntax; it is not a substitute for unit tests. Add your project's real test command when tests exist. A Trivy finding at HIGH or CRITICAL severity makes the stage fail; review policy and findings rather than blindly suppressing them.

If Docker access or Trivy is not available on your agent, first run a simpler pipeline with checkout and syntax validation, then add the image build and scan stages once prerequisites are ready. Do not give an agent unrestricted host access just to make Docker work.

## 4. Jenkinsfile syntax

- `pipeline`: declares a Declarative Pipeline.
- `agent`: selects where the pipeline runs.
- `stages`: groups work into named stages.
- `steps`: commands and pipeline actions inside a stage.
- `environment`: defines environment variables.
- `post`: defines actions after the pipeline or a stage completes.
- `options`: sets pipeline behavior, such as timestamps or concurrency restrictions.

A Jenkinsfile is executable pipeline configuration. Review changes to it as carefully as application code.

## 5. Build triggers

| Trigger | How it works | Typical use |
|---|---|---|
| Manual | A person starts a build | Learning, troubleshooting, release approval |
| Webhook | Source-control platform notifies Jenkins of a change | Prompt build after push or pull request |
| SCM polling | Jenkins periodically checks the repository for changes | When webhooks are not available |
| Scheduled | Jenkins runs at a defined time using cron-like syntax | Nightly tests or periodic maintenance |

Webhooks are often more responsive and efficient than frequent polling. Use only triggers that your repository and Jenkins configuration actually support.

Example scheduled trigger:

```groovy
triggers {
    cron('H 2 * * 1-5')
}
```

This schedules builds on weekdays around 02:00 according to Jenkins' scheduling semantics and controller timezone; `H` distributes the minute value. Configure a trigger inside the pipeline and confirm it is enabled in your Jenkins job.

## 6. Agents, executors, and plugins

- Label agents according to their capabilities, for example `linux`, `docker`, or `java`.
- Use `agent { label 'docker' }` only if a matching agent exists and is configured correctly.
- Use Git integration for source control, Docker tooling for image builds, and Maven tooling for Java projects.
- Install only necessary plugins, keep them updated, and review their maintainers and permissions.
- Keep build tools and runtimes consistent across agents.

## 7. Artifact versioning

An artifact is a packaged output of a build, such as a JAR, ZIP, or container image.

Useful identifiers include:
- Jenkins build number, e.g. `day21-python-app:42`
- Git commit SHA, for traceability to source
- Release version, e.g. `1.2.0`

Avoid relying only on a mutable tag such as `latest` for releases. Preserve the relationship between source commit, build, scan results, and artifact. Jenkins' `archiveArtifacts` stores files with the build; a container registry stores images for broader distribution. The sample pipeline exports an image tar file as a Jenkins build artifact for learning, not as the preferred production registry workflow.

## 8. Security practices

- Store credentials in Jenkins Credentials and reference them by ID.
- Never hardcode passwords, tokens, or registry credentials in a Jenkinsfile.
- Grant jobs and agents only the permissions they need.
- Avoid exposing secrets to untrusted pull-request builds.
- Use isolated agents for untrusted code and restrict Docker daemon access.
- Review plugin permissions and update Jenkins/plugins deliberately.
- Require tests and security checks before publishing or deploying.
- Keep artifact tags traceable to the source revision.

## 9. Troubleshooting

| Problem | Checks |
|---|---|
| Pipeline syntax error | Check braces, quotes, Declarative syntax, and Jenkins logs |
| `python3: not found` | Confirm Python is installed on the selected agent |
| Docker daemon permission denied | Check daemon availability and agent permissions; don't blindly run everything as root |
| Trivy command not found | Install Trivy on the agent and verify PATH |
| Artifact not archived | Check filename glob, working directory, and whether the file was generated |
| Webhook does not trigger | Check webhook URL, connectivity, Jenkins job configuration, and webhook delivery logs |
| Build queues indefinitely | Check executor availability, labels, and offline agents |
| Wrong image version | Verify build number/tag and the commit used by the build |

## 10. Interview questions

1. What is the difference between CI, Continuous Delivery, and Continuous Deployment?
2. What is a Jenkinsfile, and why should it be stored in Git?
3. What is the difference between a controller, agent, and executor?
4. Compare a webhook, SCM polling, and a scheduled trigger.
5. What is a Jenkins plugin? Give examples.
6. What is a build artifact?
7. Why is artifact versioning important?
8. Why should credentials be stored in Jenkins Credentials?
9. Why can unrestricted Docker daemon access be dangerous?
10. What should happen if a required test or vulnerability scan fails?

## 11. Completion checklist

- [ ] Review CI/CD fundamentals and Jenkins architecture.
- [ ] Open Jenkins and create a Pipeline job linked to your repository, or use a local test repository.
- [ ] Add a Jenkinsfile and run checkout plus Python syntax validation.
- [ ] Add Docker build and Trivy scan stages only after verifying agent prerequisites.
- [ ] Verify that a failed required stage marks the build as failed.
- [ ] Record the image tag and build number.
- [ ] Review triggers and plugin requirements.
- [ ] Ensure no credentials are committed.
- [ ] Document logs, issues, and fixes in GitHub.

Do not claim that the pipeline passed until you have actually run it and inspected the result.

**Next:** Day 22 — GitHub Actions Deep Dive: workflows, jobs, runners, secrets, matrices, and reusable workflows.
