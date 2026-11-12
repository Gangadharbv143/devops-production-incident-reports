# Incident 001: Docker Container Did Not Restart

**Category:** Docker Troubleshooting  
**Environment:** Ubuntu 22.04 on WSL2  
**Container:** `devops-cicd-fixed`  
**Status:** Resolved

## 1. Incident Description

The Flask application became unavailable after the WSL environment restarted. The Docker container did not automatically start again.

## 2. Investigation Commands

```bash
docker ps -a
docker images
docker logs --tail 50 devops-cicd-fixed
docker inspect devops-cicd-fixed
sudo systemctl status docker
sudo journalctl -u docker
uptime -s
df -h
```

## 3. Root Cause Analysis

- WSL restarted.
- Docker daemon started again.
- Container restart policy was `no`.
- Container remained stopped after the restart.
- `OOMKilled=false`.
- Disk space was sufficient.
- The exact trigger for the WSL restart was not established.

## 4. Recovery Commands

```bash
docker update --restart unless-stopped devops-cicd-fixed
docker start devops-cicd-fixed
```

## 5. Verification

```bash
docker ps
curl -i http://localhost:8082
docker inspect devops-cicd-fixed --format='RestartPolicy={{.HostConfig.RestartPolicy.Name}}'
```

Results:

- Container status: Up
- HTTP response: 200 OK
- Application response: `CI/CD Pipeline Working!`
- Restart policy: `unless-stopped`

## 6. Preventive Measures

- Configure suitable Docker restart policies.
- Monitor application availability.
- Check container logs during incidents.
- Verify service recovery after host or daemon restarts.

## 7. Key Learning

Docker restart policies help recover containers after daemon or host restarts. They do not fix application-level errors or replace health checks and monitoring.

**Incident outcome: Successfully recovered and verified.**
