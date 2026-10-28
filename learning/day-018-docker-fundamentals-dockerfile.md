# Day 18 — Docker I: Fundamentals and Dockerfile Project

This lesson follows the course's Session 18 topics: virtualization vs. containerization, Docker architecture, images and containers, image lifecycle, a multi-stage Dockerfile, and registries.

## Learning outcomes
- Explain VMs vs containers and Docker's architecture.
- Build an image from a Dockerfile.
- Run, inspect, test, stop, and remove a container.
- Understand image tags and registries.
- Use a multi-stage Dockerfile and basic container security practices.
- Troubleshoot common build and runtime errors.

## 1. Core concepts

- **Image:** Immutable, versioned package used to create containers.
- **Container:** An isolated process environment created from an image.
- **Dockerfile:** Instructions used to build an image.
- **Registry:** Stores and distributes images, such as Docker Hub or Amazon ECR.
- **Docker client:** CLI used to send commands.
- **Docker daemon/engine:** Builds and manages images, containers, networks, and volumes.

VMs typically run separate guest operating systems on a hypervisor. Containers usually share the host OS kernel, so they are often lighter and start faster. Containers still require appropriate isolation and security controls.

## 2. Prerequisites

Install Docker Desktop on Windows/macOS or Docker Engine on a supported Linux distribution. Confirm the Docker daemon is running:

```bash
docker --version
docker info
docker run --rm hello-world
```

If `docker info` cannot connect, start Docker Desktop/Engine and retry. Do not run commands that require elevated privileges unless you understand the permissions involved.

## 3. Build a small Python web application

Create this project structure:

```text
day-18-docker/
├── app.py
├── requirements.txt
├── Dockerfile
├── .dockerignore
└── README.md
```

### `app.py`

```python
from http.server import BaseHTTPRequestHandler, HTTPServer
import os

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == "/health":
            body = b'{"status":"ok"}'
            self.send_response(200)
            self.send_header("Content-Type", "application/json")
        elif self.path == "/":
            body = b"Hello from my Day 18 Docker container!\\n"
            self.send_response(200)
            self.send_header("Content-Type", "text/plain; charset=utf-8")
        else:
            body = b"Not found\\n"
            self.send_response(404)
            self.send_header("Content-Type", "text/plain; charset=utf-8")

        self.send_header("Content-Length", str(len(body)))
        self.end_headers()
        self.wfile.write(body)

    def log_message(self, format, *args):
        print("%s - %s" % (self.address_string(), format % args))

port = int(os.environ.get("PORT", "8080"))
HTTPServer(("0.0.0.0", port), Handler).serve_forever()
```

This tiny standard-library server avoids third-party dependencies so the first lab focuses on Docker. It is a learning app, not a recommendation for a production web service.

### `requirements.txt`

No third-party packages are required for this example. You can leave this file empty, or omit it and adjust the Dockerfile. For simplicity, use the Dockerfile below without a requirements file.

## 4. Multi-stage Dockerfile

Create `Dockerfile`:

```dockerfile
# Stage 1: use a known Python base image as the build stage.
FROM python:3.12-slim AS base
WORKDIR /app
COPY app.py .

# Stage 2: create the runtime image.
FROM python:3.12-slim AS runtime
ENV PYTHONDONTWRITEBYTECODE=1 \\
    PYTHONUNBUFFERED=1 \\
    PORT=8080

RUN useradd --create-home --uid 10001 appuser
WORKDIR /app
COPY --from=base --chown=appuser:appuser /app/app.py ./app.py

USER appuser
EXPOSE 8080
CMD ["python", "app.py"]
```

This demonstrates a multi-stage build and a non-root runtime user. Because this small app has no compiled dependencies or build artifacts, the stages are intentionally simple; multi-stage builds offer a bigger size benefit when a real build stage contains tools or artifacts that are not needed at runtime.

For a real deployment, also pin base images appropriately, scan images, define resource limits at runtime, and establish a patch/update policy.

## 5. Add a .dockerignore

Create `.dockerignore`:

```text
.git
.venv
__pycache__/
*.py[cod]
.env
.env.*
*.pem
*.key
```

This reduces unnecessary build context and helps avoid accidentally copying secrets. Never add credentials to an image or repository.

## 6. Build and run

Run these commands from the project directory:

```bash
docker build -t day18-python-app:1.0 .
docker image ls
docker run -d --name day18-app -p 8080:8080 day18-python-app:1.0
docker ps
docker logs day18-app
```

Test the application:

```bash
curl http://localhost:8080/
curl -i http://localhost:8080/health
curl -i http://localhost:8080/unknown
```

Expected results:
- `/` returns a greeting.
- `/health` returns HTTP 200 and JSON status.
- `/unknown` returns HTTP 404.

Open `http://localhost:8080/` in your browser as another test.

If port 8080 is already occupied, use `-p 8081:8080` and visit `http://localhost:8081/`. The left side is the host port; the right side is the container port.

## 7. Inspect and clean up

```bash
docker inspect day18-app
docker stats --no-stream day18-app
docker stop day18-app
docker rm day18-app
docker image ls
```

Remove the image only if you no longer need it:

```bash
docker image rm day18-python-app:1.0
```

Do not remove unrelated images or containers. Confirm names before deleting anything.

## 8. Troubleshooting guide

| Symptom | Likely cause | What to try |
|---|---|---|
| Cannot connect to Docker daemon | Docker service is stopped | Start Docker Desktop/Engine; rerun `docker info` |
| Port allocation failed | Host port is occupied | Use `-p 8081:8080` |
| Container exits immediately | Application error or wrong command | Run `docker ps -a` and `docker logs day18-app` |
| Image not found | Build failed or wrong tag | Check `docker image ls`; rebuild |
| HTTP connection refused | Container not running or wrong port mapping | Check `docker ps`, logs, and `-p HOST:CONTAINER` |
| Permission denied | Docker socket/daemon permissions | Follow OS-specific Docker setup guidance; avoid blindly using privileged access |

## 9. Image lifecycle and registries

Typical lifecycle:

1. Write or update application code.
2. Build an image from the Dockerfile.
3. Tag the image with a meaningful version.
4. Run and test a container locally.
5. Scan and review the image.
6. Push it to a registry when ready.
7. Deploy the same versioned image to the target environment.

Registry practice (optional): only after you have a registry account and understand the repository visibility, authenticate using the registry's recommended secure method, tag the image with your own registry namespace, and push it. Do not put registry tokens in shell history, Dockerfiles, or Git.

## 10. Interview questions

1. What is the difference between an image and a container?
2. How do containers differ from virtual machines?
3. What is the role of the Docker daemon?
4. What does `docker build -t name:tag .` do?
5. Explain `EXPOSE 8080` vs `-p 8081:8080`.
6. Why use a multi-stage Dockerfile?
7. Why should containers run as non-root when possible?
8. How would you debug a container that exits immediately?
9. What is a container registry?
10. What belongs in a `.dockerignore` file?

## 11. Completion checklist

- [ ] Docker is installed and `docker info` works.
- [ ] `hello-world` runs successfully.
- [ ] Created the application and Dockerfile.
- [ ] Built `day18-python-app:1.0`.
- [ ] Tested the root, health, and unknown routes.
- [ ] Inspected logs and container status.
- [ ] Diagnosed at least one issue or documented what you would check.
- [ ] Stopped and removed the test container.
- [ ] Added README evidence and pushed the lesson to GitHub.

Only report a test as completed if you ran it and observed the result.

**Next:** Day 19 — Docker networking, volumes, Compose, and container security.
