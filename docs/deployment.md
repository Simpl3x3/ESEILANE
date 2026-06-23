---
title: Deployment
layout: default
nav_order: 5
---

# Deployment Guide
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Docker (Single Node)

```bash
docker run -d \
  --name eseilane \
  --restart unless-stopped \
  -p 6379:6379 \
  -p 3000:3000 \
  -v $(pwd)/data:/data \
  -e ESEILANE_REQUIREPASS=your-strong-password \
  -e ESEILANE_MAX_MEMORY=4gb \
  eseilane/eseilane:latest
```

---

## Docker Compose (Production)

```yaml
version: "3.9"

services:
  eseilane:
    image: eseilane/eseilane:latest
    restart: unless-stopped
    ports:
      - "127.0.0.1:6379:6379"   # Bind only to localhost
      - "3000:3000"
    volumes:
      - eseilane-data:/data
      - ./eseilane.conf:/etc/eseilane/eseilane.conf
    environment:
      - ESEILANE_REQUIREPASS=${ESEILANE_PASSWORD}
      - ESEILANE_MAX_MEMORY=4gb
      - ESEILANE_MAX_MEMORY_POLICY=noeviction
      - ESEILANE_APPENDONLY=yes
      - ESEILANE_LOGLEVEL=notice
    healthcheck:
      test: ["CMD", "eseilane-cli", "ping"]
      interval: 30s
      timeout: 10s
      retries: 3
    deploy:
      resources:
        limits:
          memory: 5G

  nginx:
    image: nginx:alpine
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
      - /etc/letsencrypt:/etc/letsencrypt:ro
    depends_on:
      - eseilane

volumes:
  eseilane-data:
```

---

## Kubernetes

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: eseilane
  namespace: eseilane
spec:
  serviceName: eseilane
  replicas: 1
  selector:
    matchLabels:
      app: eseilane
  template:
    metadata:
      labels:
        app: eseilane
    spec:
      containers:
        - name: eseilane
          image: eseilane/eseilane:latest
          ports:
            - containerPort: 6379
              name: client
            - containerPort: 3000
              name: ui
          env:
            - name: ESEILANE_REQUIREPASS
              valueFrom:
                secretKeyRef:
                  name: eseilane-secret
                  key: password
            - name: ESEILANE_MAX_MEMORY
              value: "4gb"
          volumeMounts:
            - name: data
              mountPath: /data
          resources:
            requests:
              memory: "2Gi"
              cpu: "500m"
            limits:
              memory: "5Gi"
              cpu: "2000m"
          livenessProbe:
            exec:
              command: ["eseilane-cli", "ping"]
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            exec:
              command: ["eseilane-cli", "ping"]
            initialDelaySeconds: 5
            periodSeconds: 5
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: "fast-ssd"
        resources:
          requests:
            storage: 50Gi
---
apiVersion: v1
kind: Service
metadata:
  name: eseilane
  namespace: eseilane
spec:
  selector:
    app: eseilane
  ports:
    - name: client
      port: 6379
      targetPort: 6379
    - name: ui
      port: 3000
      targetPort: 3000
  type: ClusterIP
```

---

## AWS (ECS + Fargate)

```json
{
  "family": "eseilane",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "2048",
  "memory": "5120",
  "containerDefinitions": [
    {
      "name": "eseilane",
      "image": "eseilane/eseilane:latest",
      "portMappings": [
        { "containerPort": 6379, "protocol": "tcp" },
        { "containerPort": 3000, "protocol": "tcp" }
      ],
      "environment": [
        { "name": "ESEILANE_MAX_MEMORY", "value": "4gb" },
        { "name": "ESEILANE_APPENDONLY", "value": "yes" }
      ],
      "secrets": [
        {
          "name": "ESEILANE_REQUIREPASS",
          "valueFrom": "arn:aws:secretsmanager:us-east-1:123456789:secret:eseilane-password"
        }
      ],
      "mountPoints": [
        { "sourceVolume": "data", "containerPath": "/data" }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/eseilane",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

---

## Security Hardening

{: .important }
Always follow these steps in production.

1. **Enable authentication:**
   ```bash
   ESEILANE_REQUIREPASS=use-a-long-random-password-here
   ```

2. **Bind to localhost only** — use a reverse proxy (nginx/Traefik) for TLS termination:
   ```bash
   ESEILANE_BIND=127.0.0.1
   ```

3. **Enable TLS** for connections from the network:
   ```bash
   ESEILANE_TLS_PORT=6380
   ESEILANE_TLS_CERT_FILE=/certs/server.crt
   ESEILANE_TLS_KEY_FILE=/certs/server.key
   ```

4. **Disable dangerous commands:**
   ```bash
   ESEILANE_RENAME_COMMAND_CONFIG="CONFIG \"\""
   ESEILANE_RENAME_COMMAND_DEBUG="DEBUG \"\""
   ESEILANE_RENAME_COMMAND_FLUSHALL="FLUSHALL \"\""
   ```

5. **Run as non-root user** in Docker:
   ```dockerfile
   USER eseilane
   ```

---

## Backup & Restore

```bash
# Create a snapshot
docker exec eseilane eseilane-cli BGSAVE

# Copy snapshot
docker cp eseilane:/data/dump.rdb ./backups/dump-$(date +%Y%m%d).rdb

# Restore from snapshot
docker cp ./backups/dump-20260623.rdb eseilane:/data/dump.rdb
docker restart eseilane
```

---

## Monitoring

ESEILANE exposes metrics compatible with **Prometheus** via the exporter:

```bash
docker run -d \
  --name eseilane-exporter \
  -p 9121:9121 \
  -e ESEILANE_ADDR=eseilane:6379 \
  -e ESEILANE_PASSWORD=your-password \
  eseilane/exporter:latest
```

Key metrics to monitor:

| Metric | Alert Threshold | Description |
|---|---|---|
| `eseilane_memory_used_bytes` | > 80% of max | Memory usage |
| `eseilane_connected_clients` | > 1000 | Active connections |
| `eseilane_command_duration_seconds` | p99 > 100ms | Query latency |
| `eseilane_rejected_connections_total` | > 0 | Connection rejections |
| `eseilane_rdb_last_save_time` | > 1h ago | Last successful backup |
