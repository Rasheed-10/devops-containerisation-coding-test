# DevOps Interview Task (Part 1): Containerize

## Containerisation Implementation and Results

### Overview

This project demonstrates the containerisation and optimisation of the Python Flask application provided by DWP. The original Dockerfile was refactored to improve image size, build efficiency, security and maintainability.

The `app/` directory remains unchanged, and **Gunicorn** is retained as the application server.

### 1. Dockerfile Improvements

The following improvements were implemented:

- **Smaller base image:** Used `python:3.12-slim`, pinned by digest, to reduce image size and improve reproducibility.
- **Layer caching:** Copied and installed Python dependencies before copying application files, allowing dependency layers to be reused when only application code changes.
- **Non-root execution:** Created a dedicated application user to avoid running the container as root.
- **Optimised dependencies:** Used `pip --no-cache-dir` to avoid retaining unnecessary package caches.
- **Runtime configuration:** Configured Python environment variables, exposed port `8080`, and retained Gunicorn as the application server.
- **Build context:** Added `.dockerignore` to exclude unnecessary files from Docker builds.

**Final image size:** Approximately **46.43 MB**, verified using `docker image inspect`.

### 2. Published Container Image

The final optimised image is publicly available on GitHub Container Registry (GHCR).

**Final image:** `ghcr.io/rasheed-10/dwp-app:v1.0.1`

Pull the image:

```bash
docker pull ghcr.io/rasheed-10/dwp-app:v1.0.1
```

The earlier `v1.0.0` image is superseded and is not intended for assessment.

### 3. Build and Run

Build locally:

```bash
docker build -t dwp-app:local .
```

Run the published image:

```bash
docker run -d --name dwp-app -p 8080:8080 ghcr.io/rasheed-10/dwp-app:v1.0.1
```

Verify application endpoints:

```bash
curl http://localhost:8080/
curl http://localhost:8080/healthz
curl http://localhost:8080/info
```

Expected health response:

```json
{"status":"ok"}
```

The published image was successfully pulled from GHCR and tested against all three endpoints.

### 4. GitHub Actions CI/CD

The `.github/workflows/ci.yml` workflow implements:

| Job | Purpose |
|---|---|
| Gitleaks | Detects exposed secrets in Git history |
| Snyk | Scans Python dependencies for known vulnerabilities |
| Trivy | Scans the built Docker image for HIGH and CRITICAL vulnerabilities |
| GHCR Publishing | Publishes a versioned container image using `GITHUB_TOKEN` |

The workflow uses independent security jobs, an image artifact to reuse the built image, and a manually triggered assessment publishing process.

### 5. Security Findings

Security scanning identified outstanding vulnerabilities:

- **Snyk:** A HIGH-severity vulnerability affecting the provided `gunicorn==22.0.0` dependency.
- **Trivy:** 44 HIGH-severity findings in Debian OS packages at the time of scanning, with no CRITICAL findings.

The `app/` directory, including its dependency specification, was preserved as required by the assessment.

Security findings remain visible in CI. The assessment-specific publishing workflow permits delivery of the requested image despite these findings.

The image is **not claimed to be vulnerability-free**. Findings should be reviewed and remediated or formally risk-accepted before production deployment.

### 6. Verification

The final `v1.0.1` image was verified to:

- Pull successfully from GHCR.
- Start and serve the Flask application using Gunicorn.
- Respond successfully on `/`, `/healthz` and `/info`.
- Run using the configured non-root application user.

### 7. Submission

**GitHub Repository:** https://github.com/Rasheed-10/devops-containerisation-coding-test

**Public Container Image:** `ghcr.io/rasheed-10/dwp-app:v1.0.1`

**GHCR Package:** https://github.com/Rasheed-10?tab=packages

---



**This is a take-home task.** Please submit by the date given when the task was
issued.

## Overview

You are given a small web application (`app/`). A working `Dockerfile` is
provided, but it was written quickly and does **not** follow container best
practices. Your job is to refactor it into a production-quality image and push
it to a container registry.

You do **not** need to modify the application code.

## What we provide

Clone this repository into your local account.

> **Do Not** fork this repository. Clone it and re-push it into your local account.

The following is provided as part of this repository:

```text
Dockerfile            # a working but non-production Dockerfile — improve it
app/
  app.py              # a minimal Python Flask web app (do not modify)
  requirements.txt    # the app's dependencies (do not modify)
```

The application:

-   Listens on the port given by the `PORT` environment variable (default `8080`).
-   Reads a `GREETING` environment variable used in its response.
-   Exposes `GET /`, `GET /healthz`, and `GET /info`.


## Security Scanning
The Snyk dependency scan identified a high-severity HTTP Request Smuggling vulnerability in Gunicorn 22.0.0. Snyk recommends upgrading to Gunicorn 23.0.0 or later to address this finding. The dependency was retained because the assessment requires the application directory to remain unchanged. The CI security gate intentionally fails until the finding is remediated or formally accepted through an approved exception process.