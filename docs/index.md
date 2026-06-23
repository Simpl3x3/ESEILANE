---
title: Home
layout: home
nav_order: 1
---

# ESEILANE Documentation
{: .fs-9 }

A next-generation, high-performance Knowledge Graph engine for AI, LLMs, and GraphRAG.
{: .fs-6 .fw-300 }

[Get started now](#quick-start){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[View on GitHub](https://github.com/Simpl3x3/ESEILANE){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## What is ESEILANE?

**ESEILANE** is a high-performance **Knowledge Graph engine** built for the AI era. It is the fastest queryable Property Graph database, leveraging sparse matrix algebra and linear algebra-based query execution to deliver sub-millisecond graph traversals at scale.

ESEILANE is purpose-built for:

- **GraphRAG** — Retrieval-Augmented Generation grounded in structured knowledge
- **LLM Integration** — Reduce hallucinations with graph-powered context
- **Semantic Search** — Vector + graph hybrid search at scale
- **AI Pipelines** — Real-time knowledge graph serving for production AI systems

---

## Quick Start

### Docker

```bash
docker run -p 6379:6379 -p 3000:3000 eseilane/eseilane:latest
```

### Python

```bash
pip install eseilane
```

```python
from eseilane import ESEILANE

db = ESEILANE(host='localhost', port=6379)
g = db.select_graph('demo')

g.query("CREATE (:Person {name:'Alice'})-[:KNOWS]->(:Person {name:'Bob'})")

res = g.query("MATCH (a:Person)-[:KNOWS]->(b:Person) RETURN a.name, b.name")
for row in res.result_set:
    print(f"{row[0]} knows {row[1]}")
# Alice knows Bob
```

---

## Core Concepts

| Concept | Description |
|---|---|
| **Graph** | A named container holding nodes and relationships |
| **Node** | An entity with labels and properties |
| **Relationship** | A directed edge between two nodes with a type and properties |
| **Cypher** | The query language used to interact with the graph |
| **GraphRAG** | Graph-powered Retrieval-Augmented Generation for LLMs |

---

## Architecture Overview

```
Client (Python / TypeScript / REST)
         │
         ▼
   ┌─────────────┐
   │  Query Layer │   OpenCypher · REST API · GraphQL (coming soon)
   └──────┬──────┘
          │
   ┌──────▼──────┐
   │ Graph Engine │   GraphBLAS · Sparse Matrix · Rust Core
   └──────┬──────┘
          │
   ┌──────▼──────┐
   │Storage Layer │   In-memory + Persistent (RDB + AOF)
   └─────────────┘
```

---

## Navigation

- [Getting Started](getting-started) — Installation, configuration, and first queries
- [API Reference](api-reference) — Complete REST API documentation
- [Python SDK](sdk/python) — Python client library reference
- [TypeScript SDK](sdk/typescript) — TypeScript/JavaScript client reference
- [GraphRAG Guide](graphrag) — Using ESEILANE for Retrieval-Augmented Generation
- [Deployment](deployment) — Docker, Kubernetes, and cloud deployment
