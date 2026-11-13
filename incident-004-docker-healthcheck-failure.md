# Incident 004 – Docker Health Check Failure

## 1. Incident Overview

- **Incident ID:** DEVOPS-004
- **Category:** Docker Troubleshooting
- **Severity:** Medium
- **Status:** Resolved
- **Environment:** Ubuntu / Docker
- **Application:** Flask CI/CD application

## 2. Problem Description

The application container was running, but Docker had no health check configured.

Docker could report the container's running status but could not independently determine whether the application was responding correctly.

## 3. Investigation

Inspected the image health check:

```bash
docker image inspect devops-cicd-fixed:v1 \
--format 'Image healthcheck: {{json .Config.Healthcheck}}'
```

Result:

```text
Image healthcheck: null
```

Verified Python urllib availability:

```bash
docker run --rm --entrypoint python devops-cicd-fixed:v1 \
-c "import urllib.request; print('Python urllib is available')"
```

Result: Python urllib is available.

Started the investigation container and tested the endpoint.

Result: HTTP 200 OK.

## 4. Root Cause Analysis

The Docker image did not have a HEALTHCHECK instruction configured.

Therefore, Docker could not determine the application's health status.

## 5. Recovery and Fix

Created a new container with an HTTP health check:

```bash
docker run -d \
  --name incident004-fixed \
  -p 8086:5000 \
  --health-cmd='python -c "import urllib.request; urllib.request.urlopen(\"http://127.0.0.1:5000/\", timeout=3)"' \
  --health-interval=10s \
  --health-timeout=5s \
  --health-retries=3 \
  --health-start-period=5s \
  devops-cicd-fixed:v1
```

## 6. Verification

Checked Docker health:

```bash
docker inspect incident004-fixed \
--format 'Status: {{.State.Status}} | Health: {{.State.Health.Status}}'
```

Result:

```text
Status: running | Health: healthy
```

Checked health check history:

- First check: Exit code 0
- Second check: Exit code 0
- Third check: Exit code 0

Tested application:

```bash
curl -i http://localhost:8086/
```

Result:

```text
HTTP/1.1 200 OK
CI/CD Pipeline Working!
```

## 7. Final Outcome

- Identified missing Docker health check.
- Configured application-level health monitoring.
- Verified three successful health checks.
- Confirmed HTTP 200 OK.
- Restored health monitoring successfully.

## 8. Lessons Learned

1. Running containers are not necessarily healthy.
2. Docker HEALTHCHECK monitors application-level availability.
3. Exit code 0 indicates a successful health check.
4. Health checks help identify application failures.
5. Production Dockerfiles should include appropriate health checks where applicable.

## 9. Skills Practiced

- Docker image inspection
- Container troubleshooting
- Docker health checks
- Python urllib
- HTTP endpoint testing
- Root cause analysis
- Production incident documentation
