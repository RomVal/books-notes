# Docker Container Logs

The `docker container logs` command shows logs generated inside a container (stdout, stderr).

**Format:**
```bash
docker container logs [OPTIONS] <container_id|container_name>
```

## Common Options
- `-f` / `--follow` — stream logs in real-time (`tail -f`).
- `--since <time>` — show logs since a point in time.

**Examples:**
```bash
docker container logs web                # all logs
docker container logs -f web             # follow logs
docker container logs --tail 50 -t web   # last 50 lines with timestamps
docker container logs --since 1h web     # last hour
```

⚠️ Works only for existing containers. If removed (`docker rm`), logs are gone.

---

## How It Works
- Logs are stored on the **host**, not inside the container:  
  `/var/lib/docker/containers/<id>/<id>-json.log`
- By default Docker uses the **json-file** driver.
- You can configure other drivers depending on your needs.

### Log Rotation
Logs can grow quickly. Configure rotation with `log-opts`:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

---

## Where to Configure

1. **Global (all containers)**  
   `/etc/docker/daemon.json`
   ```json
   {
     "log-driver": "json-file",
     "log-opts": {
       "max-size": "10m",
       "max-file": "5"
     }
   }
   ```
   Restart Docker after changes.

2. **Per-container**
   ```bash
   docker run -d      
   --name my-app      
   --log-driver=json-file      
   --log-opt max-size=10m      
   --log-opt max-file=3      
   nginx
   ```

   Or in `docker-compose.yml`:
   ```yaml
   version: '3.8'
   services:
     web:
       image: nginx
       logging:
         driver: json-file
         options:
           max-size: "10m"
           max-file: "3"
   ```

---

## Log Drivers

Docker supports multiple log drivers:

- **json-file** -> default, logs stored on the host (`/var/lib/docker/containers/...`).
- **syslog** -> send logs to system syslog.
- **journald** -> send logs to `systemd-journald`.
- **awslogs** -> send logs to AWS CloudWatch Logs.
- **gelf** -> send logs to Graylog/Logstash.
- **fluentd** -> send logs to Fluentd/Fluent Bit.
- **none** -> disable logging entirely.

Each driver has its own `--log-opt` options.

---

## AWS Logs Example

Docker has built-in support for `awslogs`:

```bash
docker run -d   
    --name my-app   
    --log-driver=awslogs  
    --log-opt awslogs-region=eu-central-1   
    --log-opt awslogs-group=my-app-group   
    --log-opt awslogs-stream=my-app-stream   
    nginx
```

Authentication uses standard AWS methods (environment variables, IAM roles, profiles).

```bash
export AWS_ACCESS_KEY_ID=AKIA...
export AWS_SECRET_ACCESS_KEY=...
export AWS_DEFAULT_REGION=eu-central-1
```