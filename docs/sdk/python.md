---
title: Python SDK
layout: default
parent: SDKs
nav_order: 1
---

# Python SDK
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Installation

```bash
pip install eseilane
```

**Requirements:** Python 3.11+

---

## Connection

```python
from eseilane import ESEILANE

# Basic connection
db = ESEILANE(host='localhost', port=6379)

# With authentication
db = ESEILANE(host='localhost', port=6379, password='your-password')

# With TLS
db = ESEILANE(
    host='your-host.eseilane.io',
    port=6380,
    password='your-password',
    ssl=True,
    ssl_ca_certs='/path/to/ca.crt'
)

# Select a graph
g = db.select_graph('my_graph')
```

---

## Querying

### Basic Query

```python
result = g.query("MATCH (n:Person) RETURN n.name, n.age ORDER BY n.age")

# Access result set
for row in result.result_set:
    name, age = row[0], row[1]
    print(f"{name}: {age}")

# Access metadata
print(f"Query time: {result.run_time_ms}ms")
print(f"Nodes created: {result.nodes_created}")
```

### Parameterized Queries (Recommended)

```python
# Always use params to prevent injection and improve performance
result = g.query(
    "MATCH (p:Person) WHERE p.age > $min_age RETURN p.name",
    params={"min_age": 25}
)
```

### Read-Only Query

```python
# Enforces read-only — raises error on write operations
result = g.ro_query("MATCH (n) RETURN count(n)")
```

### Query with Timeout

```python
result = g.query(
    "MATCH (n)-[*1..5]->(m) RETURN n, m LIMIT 100",
    timeout=2000  # milliseconds
)
```

---

## Creating Data

```python
# Create nodes
g.query("""
  CREATE
    (:Product {id: $id, name: $name, price: $price}),
    (:Category {name: $category})
""", params={"id": "p001", "name": "Widget", "price": 9.99, "category": "Tools"})

# Create relationships
g.query("""
  MATCH (p:Product {id: $id}), (c:Category {name: $category})
  CREATE (p)-[:BELONGS_TO]->(c)
""", params={"id": "p001", "category": "Tools"})
```

---

## Bulk Loading

For large datasets, bulk insert is 10-100x faster than individual CREATE queries:

```python
import csv
from eseilane.bulk import GraphBulkInsert

bulk = GraphBulkInsert(graph=g)

# Define node files
bulk.add_node_file(
    label="Person",
    header=["name:str", "age:int", "email:str"],
    rows=[
        ["Alice", 30, "alice@example.com"],
        ["Bob",   25, "bob@example.com"],
    ]
)

# Define edge files
bulk.add_edge_file(
    relation="KNOWS",
    src_node_label="Person",
    dest_node_label="Person",
    header=["src:str", "dest:str", "since:int"],
    rows=[["Alice", "Bob", 2020]]
)

stats = bulk.execute()
print(f"Imported: {stats.nodes_created} nodes, {stats.relationships_created} edges")
```

---

## Indexes

```python
# Create an exact-match index
g.create_node_range_index("Person", "name")

# Create a full-text search index
g.create_node_fulltext_index("ft_persons", "Person", "name", "bio")

# Create a vector index (for semantic search)
g.create_node_vector_index(
    "Person", "embedding",
    dim=1536,           # OpenAI ada-002 dimension
    similarity="cosine"
)

# Drop an index
g.drop_node_range_index("Person", "name")
```

---

## Vector Search

```python
import openai

# Generate embedding
client = openai.OpenAI()
response = client.embeddings.create(input="graph database expert", model="text-embedding-ada-002")
query_embedding = response.data[0].embedding

# Search by vector similarity
results = g.query("""
  CALL db.idx.vector.queryNodes('Person', 'embedding', $k, $vector)
  YIELD node, score
  RETURN node.name, score
  ORDER BY score DESC
""", params={"k": 5, "vector": query_embedding})

for row in results.result_set:
    print(f"{row[0]}: similarity={row[1]:.4f}")
```

---

## Async Client

```python
from eseilane.asyncio import AsyncESEILANE

async def main():
    db = AsyncESEILANE(host='localhost', port=6379)
    g = db.select_graph('demo')

    await g.query("CREATE (:Node {id: 1})")
    result = await g.query("MATCH (n) RETURN n.id")
    return result.result_set

import asyncio
asyncio.run(main())
```

---

## Error Handling

```python
from eseilane import ESEILANE
from eseilane.exceptions import (
    ESEILANEError,
    QueryTimeoutError,
    GraphNotFoundError,
    ConstraintViolationError,
)

db = ESEILANE(host='localhost', port=6379)
g = db.select_graph('demo')

try:
    result = g.query("MATCH (n) RETURN n", timeout=100)
except QueryTimeoutError:
    print("Query timed out — consider adding an index or reducing result size")
except GraphNotFoundError:
    print("Graph does not exist")
except ConstraintViolationError as e:
    print(f"Constraint violated: {e}")
except ESEILANEError as e:
    print(f"Database error: {e}")
```

---

## API Reference

### `ESEILANE`

| Method | Description |
|---|---|
| `select_graph(name)` | Select or create a graph |
| `list_graphs()` | List all graphs |
| `delete_graph(name)` | Delete a graph |

### `Graph`

| Method | Description |
|---|---|
| `query(q, params, timeout)` | Execute a Cypher query |
| `ro_query(q, params, timeout)` | Execute a read-only query |
| `create_node_range_index(label, prop)` | Create a range index |
| `create_node_fulltext_index(name, label, *props)` | Create full-text index |
| `create_node_vector_index(label, prop, dim, similarity)` | Create vector index |
| `drop_node_range_index(label, prop)` | Drop a range index |
| `schema()` | Get graph schema |
| `slowlog()` | Get slow query log |
| `config_get(name)` | Get a config value |
| `config_set(name, value)` | Set a config value |
