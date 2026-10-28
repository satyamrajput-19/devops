# Day 19 — Docker II: Networking, Volumes, Compose, and Security

This lesson follows Session 19 of the course: Docker networking modes, port mapping, volumes and bind mounts, Docker Compose, container security, and Trivy image scanning.

## Learning outcomes
- Explain Docker bridge networking and published ports.
- Compare named volumes with bind mounts.
- Run a multi-container stack using Docker Compose.
- Apply basic container hardening.
- Scan an image with Trivy and interpret findings.
- Troubleshoot common networking and persistence problems.

## 1. Networking basics

- **Bridge network:** Common default for standalone containers. A user-defined bridge network provides DNS resolution by container/service name.
- **Host network:** Shares the host network namespace on supported platforms; published-port behavior differs.
- **None:** Disables normal container networking.
- **Overlay:** Connects services across Docker hosts in supported Swarm configurations.

Port mapping uses `HOST_PORT:CONTAINER_PORT`:

```bash
docker run --rm -p 8081:8080 day18-python-app:1.0
```

The application listens on container port 8080; you reach it on the host at port 8081. `EXPOSE 8080` in a Dockerfile is documentation, not host port publishing.

Inspect networks:

```bash
docker network ls
docker network inspect bridge
docker network create day19-net
docker network inspect day19-net
```

Avoid attaching services to the default bridge when a user-defined network provides better name resolution and clearer isolation.

## 2. Volumes vs bind mounts

- **Named volume:** Managed by Docker; usually preferred for persistent application/database data.
- **Bind mount:** Maps a specific host path into a container; useful for development and sharing source/config files, but ties the container to the host path.
- **Writable container layer:** Ephemeral; changes can be lost when the container is removed/recreated.

Named volume example:

```bash
docker volume create day19-data
docker run --rm -v day19-data:/data alpine sh -c 'echo persistent > /data/note.txt'
docker run --rm -v day19-data:/data alpine cat /data/note.txt
docker volume inspect day19-data
```

Expected output: `persistent`. Do not delete a volume containing data you need.

Bind mount example (run from the directory containing a file named `index.html`):

```bash
docker run --rm -p 8082:80 -v "$PWD":/usr/share/nginx/html:ro nginx:alpine
```

The `:ro` suffix makes the mount read-only inside the container. Visit `http://localhost:8082`. On Windows/macOS, make sure the directory is shared/accessible to Docker Desktop.

## 3. Docker Compose lab

Use your Day 18 project files (`app.py` and `Dockerfile`) in this directory:

```text
day-19-compose/
├── app.py
├── Dockerfile
├── compose.yaml
├── .env
└── .gitignore
```

This sample runs the Day 18 web app and a PostgreSQL service on a private Compose network. **The Day 18 app does not yet connect to PostgreSQL**; this lab demonstrates service orchestration, DNS/networking, environment configuration, and persistent storage, not database integration.

### `compose.yaml`

```yaml
services:
  web:
    build: .
    image: day19-python-app:1.0
    ports:
      - "127.0.0.1:8080:8080"
    restart: unless-stopped
    read_only: true
    tmpfs:
      - /tmp
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    networks:
      - app-net

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:?Set POSTGRES_PASSWORD in .env}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-net
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
      interval: 10s
      timeout: 5s
      retries: 5

networks:
  app-net:

volumes:
  postgres-data:
```

Security notes:
- Binding web to `127.0.0.1` makes it reachable from the local host only, rather than every host interface.
- PostgreSQL's port is not published to the host; the service is on the private Compose network.
- Use a strong throwaway lab password in `.env`; never commit `.env` or real secrets. For production, prefer a secrets manager and your platform's supported secret-injection mechanism.
- The sample pins a major Postgres version, not an immutable digest. Production deployments should use a reviewed, patched image and a deliberate update policy.
- Compose hardening options can vary by image and platform. Test them; the read-only root filesystem may require additional writable paths for other apps.

### `.env` (local only)

```dotenv
POSTGRES_PASSWORD=replace-with-a-strong-local-lab-password
```

Add these to `.gitignore`:

```text
.env
.env.*
!.env.example
```

Never commit the actual local `.env`. You may create an `.env.example` containing only a placeholder.

### Run and inspect

```bash
docker compose config
docker compose up --build -d
docker compose ps
docker compose logs --tail=100
docker compose exec db pg_isready -U appuser -d appdb
curl -i http://localhost:8080/health
```

The web app and database can resolve each other using service names `web` and `db` on the Compose network. The Python app in this lab does not use the database yet.

Stop the services:

```bash
docker compose down
```

This preserves named volumes. To delete the database volume as well, use `docker compose down -v` **only if you intentionally want to permanently remove this lab's stored database data**.

## 4. Container security essentials

- Run as a non-root user where possible.
- Use minimal, trusted, regularly updated base images.
- Never bake passwords, API keys, or private keys into images or Git.
- Use read-only filesystems where compatible, drop Linux capabilities, and prevent privilege escalation when feasible.
- Publish only required ports; avoid exposing databases publicly.
- Set CPU/memory limits appropriately for your deployment platform.
- Scan images and fix vulnerabilities based on severity, exploitability, and available patches.
- Pin and review image versions; keep a rebuild/patch process.
- Treat images and registries as part of the software supply chain.

## 5. Scan with Trivy

Install Trivy using the official installation instructions for your operating system. Then scan the image:

```bash
trivy image day19-python-app:1.0
```

For a focused vulnerability report:

```bash
trivy image --severity HIGH,CRITICAL day19-python-app:1.0
```

A scan can report vulnerabilities in OS packages and language dependencies, depending on image contents and scanner capabilities. Review the package, installed version, fixed version, severity, and whether a fix is available. A clean scan does not prove an image is completely secure.

## 6. Troubleshooting guide

| Symptom | Likely cause | What to check |
|---|---|---|
| Port is already allocated | Host port in use | Change host side, e.g. `127.0.0.1:8081:8080` |
| App cannot resolve database | Services not on same network or wrong hostname | Check `docker compose ps`, network, and use service name `db` |
| Data disappears after recreation | Data stored only in container layer | Mount a named volume at the database data path |
| Compose refuses to start | YAML/variable/interpolation error | Run `docker compose config` |
| Database is unhealthy | Bad credentials, startup delay, or configuration | Inspect `docker compose logs db` and health status |
| Trivy command not found | Trivy not installed or not on PATH | Verify installation and PATH |
| App fails with read-only filesystem | App needs a writable path | Provide a narrowly scoped tmpfs/volume or adjust app paths |

## 7. Interview questions

1. What is the difference between a named volume and a bind mount?
2. What is the difference between container port and published host port?
3. Why use a user-defined bridge network?
4. What problem does Docker Compose solve?
5. Why should a database port generally not be published publicly?
6. Why run a container as non-root?
7. What does Trivy scan, and what does a clean scan not guarantee?
8. What is the difference between `docker compose down` and `docker compose down -v`?
9. How do you troubleshoot a container that cannot reach another service?
10. Why should secrets not be baked into Docker images?

## 8. Completion checklist

- [ ] Inspect Docker networks and create a user-defined bridge.
- [ ] Demonstrate a named volume survives container removal.
- [ ] Demonstrate a read-only bind mount.
- [ ] Start the Compose stack and validate configuration.
- [ ] Check the web health endpoint and database health.
- [ ] Inspect logs and diagnose one issue, or document the diagnostic process.
- [ ] Scan the image with Trivy if installed; document findings accurately.
- [ ] Verify `.env` is ignored by Git.
- [ ] Stop the stack; preserve data unless intentionally cleaning it up.
- [ ] Document observed results in GitHub.

Do not claim that a command succeeded or an image is vulnerability-free unless you ran and checked it.

**Next:** Day 20 — SonarQube, code quality gates, and pipeline security.
