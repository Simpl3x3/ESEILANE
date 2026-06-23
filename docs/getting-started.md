---
title: Getting Started
layout: default
nav_order: 2
---

# Getting Started
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Prerequisites

- Docker 24+ (recommended) **or** a Redis-compatible server
- Python 3.11+ (for Python SDK)
- Node.js 20+ (for TypeScript SDK)

---

## Installation

### Option 1: Docker (Recommended)

The fastest way to get ESEILANE running:

```bash
docker run -d \
  --name eseilane \
  -p 6379:6379 \
  -p 3000:3000 \
  -v eseilane-data:/data \
  eseilane/eseilane:latest
```

Access the browser UI at [http://localhost:3000](http://localhost:3000).

### Option 2: Docker Compose

Create a `docker-compose.yml`:

```yaml
version: "3.9"
services:
  eseilane:
    image: eseilane/eseilane:latest
    ports:
      - "6379:6379"
      - "3000:3000"
    volumes:
      - eseilane-data:/data
    environment:
      - ESEILANE_LOGLEVEL=notice
      - ESEILANE_MAX_MEMORY=2gb
    restart: unless-stopped

volumes:
  eseilane-data:
```

```bash
docker compose up -d
```

### Option 3: Python SDK (Embedded)

For development and testing — no separate server needed:

```bash
pip install falkordblite  # zero-config embedded mode
```

---

## Connecting

### Python

```bash
pip install eseilane
```

```python
from eseilane import ESEILANE

db = ESEILANE(
    host='localhost',
    port=6379,
    password=None,       # Set if AUTH is enabled
    decode_responses=True
)

# Select or create a graph
g = db.select_graph('my_graph')
print("Connected!")
```

### TypeScript

```bash
npm install eseilane-js
```

```typescript
import { ESEILANE } from 'eseilane-js';

const db = new ESEILANE({
  host: 'localhost',
  port: 6379,
  password: undefined,
});

const graph = db.selectGraph('my_graph');
console.log('Connected!');
```

### REST API

```bash
curl -X POST http://localhost:3000/api/graph/my_graph/query \
  -H "Content-Type: application/json" \
  -d '{"query": "RETURN 1"}'
```

---

## Your First Graph

```python
from eseilane import ESEILANE

db = ESEILANE(host='localhost', port=6379)
g = db.select_graph('social')

# Create nodes and relationships
g.query("""
  CREATE
    (:Person {name: 'Alice', age: 30}),
    (:Person {name: 'Bob',   age: 25}),
    (:Person {name: 'Carol', age: 28}),
    (:Person {name: 'Alice'})-[:KNOWS {since: 2020}]->(:Person {name: 'Bob'}),
    (:Person {name: 'Bob'})-[:KNOWS {since: 2022}]->(:Person {name: 'Carol'})
""")

# Query: find friends of Alice
result = g.query("""
  MATCH (alice:Person {name: 'Alice'})-[:KNOWS]->(friend:Person)
  RETURN friend.name AS name, friend.age AS age
  ORDER BY age
""")

for row in result.result_set:
    print(f"Friend: {row[0]}, Age: {row[1]}")
# Friend: Bob, Age: 25
```

---

## Configuration Reference

| Environment Variable | Default | Description |
|---|---|---|
| `ESEILANE_LOGLEVEL` | `notice` | Log level: debug, verbose, notice, warning |
| `ESEILANE_MAX_MEMORY` | `0` (unlimited) | Max memory (e.g. `2gb`, `512mb`) |
| `ESEILANE_REQUIREPASS` | *(none)* | Authentication password |
| `ESEILANE_BIND` | `0.0.0.0` | Bind address |
| `ESEILANE_PORT` | `6379` | Client connection port |
| `ESEILANE_SAVE` | `3600 1` | RDB snapshot policy |
| `ESEILANE_APPENDONLY` | `yes` | Enable AOF persistence |
| `ESEILANE_TLS_PORT` | *(none)* | TLS port (if TLS enabled) |

---

## Next Steps

- [API Reference](../api-reference) — Explore all REST endpoints
- [Python SDK](../sdk/python) — Full Python client reference
- [GraphRAG Guide](../graphrag) — Build your first GraphRAG pipeline
- [Deployment](../deployment) — Deploy to production
