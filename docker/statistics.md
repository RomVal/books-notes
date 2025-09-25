# Docker Container Stats

Docker provides built-in **CLI** and **API** to get real-time container statistics.

---

## 1. CLI: `docker stats`

Basic usage:
```bash
docker stats
```

Outputs a `top`-like table:
```
CONTAINER ID   NAME      CPU %   MEM USAGE / LIMIT   NET I/O       BLOCK I/O   PIDS
a1b2c3d4e5f6   web-api   3.25%   250MiB / 2GiB       1.2MB/300kB   0B/0B       22
```

For a specific container:
```bash
docker stats <container_id_or_name>
```

Output as JSON (useful for monitoring tools):
```bash
docker stats --no-stream --format "{{json .}}"
```

---

## 2. Docker API

Docker Engine exposes an API (via Unix socket `/var/run/docker.sock` or TCP if enabled).

**Endpoint:**
```
GET /containers/{id}/stats
```

Example:
```bash
curl --unix-socket /var/run/docker.sock   http://localhost/containers/<container_id>/stats?stream=false
```

Response is detailed JSON:
```json
{
  "read": "2025-09-23T08:30:52.123456789Z",
  "cpu_stats": { ... },
  "precpu_stats": { ... },
  "memory_stats": { ... },
  "networks": { ... }
}
```

---

## 3. Integration with Monitoring (Prometheus/Grafana)

In production, raw `docker stats` is rarely used directly. Instead:

- **node-exporter** → host-level metrics
- **cAdvisor** → container-level metrics (CPU, memory, network, FS)
- **Prometheus** → time-series database
- **Grafana** → visualization

### Running cAdvisor
Typically deployed as a container on each host:

```bash
docker run   
    --volume=/:/rootfs:ro   
    --volume=/var/run:/var/run:rw   
    --volume=/sys:/sys:ro   
    --volume=/var/lib/docker/:/var/lib/docker:ro   
    --publish=8080:8080   
    --detach=true   
    --name=cadvisor   
    gcr.io/cadvisor/cadvisor:latest
```

---

## Summary

- For quick inspection -> `docker stats`
- For automation -> Docker API `/containers/{id}/stats`
- For production monitoring -> `cAdvisor + Prometheus + Grafana`