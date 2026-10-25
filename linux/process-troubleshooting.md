# Linux Process Troubleshooting

## Commands learned

- ps -ef: List processes broadly.
- ps -fp PID: Inspect a particular process.
- pstree -sp PID: View a process and its ancestors.
- systemctl status SERVICE: Check service status.
- journalctl -u SERVICE -n 50 --no-pager: Inspect recent service logs.
- ss -lntp: Inspect listening TCP ports.
- curl -v URL: Test an endpoint.
- grep PATTERN FILE: Search for matching text.

## Troubleshooting workflow

1. Check the service status.
2. Inspect logs when a service fails.
3. Identify processes and listening ports.
4. Check the configuration and test the endpoint.
5. Fix the cause and verify the result.

## Node Exporter example

If Prometheus reports Node Exporter as DOWN, check its service,
inspect logs, verify port 9100, test the metrics endpoint,
and confirm the Prometheus target returns to UP.

A running process does not necessarily mean that the service
is healthy or reachable.
