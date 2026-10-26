# Linux & DevOps Troubleshooting Quick Reference

## 1. Essential Linux Commands

| Command | Purpose |
|---|---|
| `systemctl status SERVICE` | Check service status |
| `systemctl start SERVICE` | Start a stopped service |
| `systemctl restart SERVICE` | Stop and start a service again |
| `journalctl -u SERVICE -n 50 --no-pager` | Inspect the latest 50 service log lines |
| `ps -ef` | Display processes broadly |
| `ps -fp PID` | Inspect a specific process |
| `pstree -sp PID` | Trace a process and its ancestors |
| `ss -lntp` | Inspect listening TCP ports and processes |
| `curl -v URL` | Test an endpoint and inspect its response |
| `grep PATTERN FILE` | Search a file for matching text |
| `df -h` | Check filesystem space |
| `free -h` | Check memory usage |

Replace SERVICE, PID, URL, PATTERN and FILE with actual values.

## 2. Troubleshooting: Node Exporter Is DOWN

### Situation
- Prometheus is running.
- Grafana dashboard opens.
- Node Exporter target is DOWN.
- Fresh Linux metrics are unavailable.

### Investigation sequence

1. Check the service:
   `sudo systemctl status prometheus-node-exporter`

2. Inspect recent logs:
   `sudo journalctl -u prometheus-node-exporter -n 50 --no-pager`

3. Check listening ports:
   `sudo ss -lntp | grep ':9100'`

4. Inspect the service configuration:
   `sudo systemctl cat prometheus-node-exporter`

5. Test the metrics endpoint:
   `curl -v http://localhost:9100/metrics`

6. Inspect the Prometheus target address and port.

7. Fix the identified cause and verify the target becomes UP.

Note: The service name may differ depending on the installation.

## 3. Understand the Evidence

- `active (running)` does not guarantee that a service is reachable.
- No listener on port 9100 means no matching TCP listener was found.
- `connection refused` often indicates that the target address and port are not accepting the connection.
- Logs help identify startup errors and port-binding failures.
- A PID identifies a process; a systemd service manages a service.
- PPID means Parent Process ID.
- Diagnose before restarting. Verify the result after fixing the cause.

## 4. Localhost and Network Ports

- `localhost` refers to the machine or network namespace making the connection.
- Node Exporter commonly listens on port 9100.
- If the exporter uses port 9180 instead, configure Prometheus to scrape the correct port.
- Changing the Prometheus target alone does not change the exporter's listening port.
- Keep monitoring endpoints restricted to trusted systems.

## 5. Docker vs Linux Services

### Without Docker
- `systemctl` checks and manages Linux services.
- `journalctl` inspects systemd journal logs.
- `ss` checks listening ports.
- `curl` tests endpoints.

### With Docker Compose
- `docker ps -a` lists running and stopped containers.
- `docker logs CONTAINER` inspects container logs.
- `docker compose restart SERVICE` restarts an existing service container.
- `docker compose up -d SERVICE` reconciles the service with Compose configuration and may recreate its container when necessary.

In Docker Compose, Prometheus can use `node-exporter:9100` when both services share a network. Inside a container, `localhost` refers to that container, not another container.

## 6. Tool Responsibilities

- Linux: operates the host and manages processes and services.
- Docker: builds images and runs containers.
- Docker Compose: defines and runs related containers.
- Prometheus: scrapes and stores time-series metrics.
- Node Exporter: exposes Linux host metrics.
- Grafana: visualises metrics from data sources such as Prometheus.
- Kubernetes: orchestrates containerised workloads.
- YAML: a configuration format interpreted by different tools.

## 7. Troubleshooting Workflow

IDENTIFY -> INSPECT -> DIAGNOSE -> FIX -> VERIFY

1. Identify the exact failure.
2. Inspect status, logs and configuration.
3. Test the relevant process, port or endpoint.
4. Change only what the evidence indicates.
5. Verify that the service and monitoring target recover.

## 8. Quick Revision

- PID = Process ID.
- PPID = Parent Process ID.
- `ps -ef` = broad process listing.
- `systemctl` = service management.
- `journalctl` = systemd logs.
- `ss -lntp` = listening TCP ports.
- `curl` = endpoint testing.
- `grep` = text searching.

Remember: a running service, a listening endpoint and a working Prometheus target are three different things.
