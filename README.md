# Monitoring Stack

Simple Docker Compose config for a monitoring & visualization stack with Cloudflared, Loki, cAdvisor, Promtail, Prometheus, Node Exporter & Grafana.

Remember to create all the necessary folders (unless you want to have all these folders as root):
  - ### Loki 
    - ./loki/config
    - ${VOLUME_DIR}/loki/index
    - ${VOLUME_DIR}/loki/chunks
    - ${VOLUME_DIR}/loki/wal
  - ### Promtail
    - ./promtail/config
    - ${VOLUME_DIR}/promtail/data
    - ${VOLUME_DIR}/promtail/log
  - ### Grafana
    - ${VOLUME_DIR}/grafana/data
    - ./grafana/datasources
---

Also, I am not sure if it's a good idea, but I sent all the Docker logs to Loki. You can do this by modifying the `/etc/docker/daemon.json` file:
```json
{
  "log-driver": "loki",
  "log-opts": {
    "loki-url": "http://localhost:3100/loki/api/v1/push"
  }
}
```

You'll also need to allow only internal traffic to reach Loki, unless you want to open the `3100` port to everyone (maybe you want to send logs from other systems. I don't. Yet.) You'll need to add the following to the `docker-compose.yaml`:
```yaml
...
  loki:
    image: grafana/loki:latest
    container_name: loki
    ports:
      - 127.0.0.1:3100:3100 # <- this is the line we need to add.
          # this binds the host machine's IP address (127.0.0.1) and port 3100 to the container's service port 3100. 
          # This allows you to access the service running inside the container through 127.0.0.1:3100 on your local machine.
    volumes:
      - ./loki/config:/etc/loki
      - ${VOLUME_DIR}/loki/index:/data/loki/index
      - ${VOLUME_DIR}/loki/chunks:/data/loki/chunks
      - ${VOLUME_DIR}/loki/wal:/wal
    command: -config.file=/etc/loki/local-config.yaml
    user: "${UID}:${GID}"
    restart: unless-stopped
...
```