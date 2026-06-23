---
title: API Reference
layout: default
nav_order: 3
---

# REST API Reference
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Base URL

```
http://localhost:3000/api
```

All endpoints accept and return JSON. Authentication is via Bearer token when enabled.

---

## Authentication

If `ESEILANE_REQUIREPASS` is set, include the token in every request:

```http
Authorization: Bearer <your-password>
```

---

## Graphs

### List All Graphs

```http
GET /api/graphs
```

**Response 200:**
```json
{
  "graphs": ["social", "knowledge_base", "products"]
}
```

---

### Create or Select a Graph

```http
POST /api/graph/{graph_name}
```

**Path Parameters:**

| Parameter | Type | Description |
|---|---|---|
| `graph_name` | string | Name of the graph to create or select |

**Response 200:**
```json
{
  "graph": "social",
  "created": true
}
```

---

### Delete a Graph

```http
DELETE /api/graph/{graph_name}
```

**Response 200:**
```json
{
  "deleted": "social"
}
```

---

## Queries

### Execute a Cypher Query

```http
POST /api/graph/{graph_name}/query
```

**Request Body:**

```json
{
  "query": "MATCH (n:Person) RETURN n.name LIMIT 10",
  "params": {
    "limit": 10
  },
  "timeout": 5000
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `query` | string | Yes | OpenCypher query string |
| `params` | object | No | Named query parameters |
| `timeout` | integer | No | Query timeout in milliseconds (default: 10000) |

**Response 200:**

```json
{
  "results": [
    { "n.name": "Alice" },
    { "n.name": "Bob" }
  ],
  "metadata": {
    "nodes_created": 0,
    "nodes_deleted": 0,
    "relationships_created": 0,
    "relationships_deleted": 0,
    "properties_set": 0,
    "labels_added": 0,
    "query_internal_execution_time": "0.312ms"
  }
}
```

**Example:**

```bash
curl -X POST http://localhost:3000/api/graph/social/query \
  -H "Content-Type: application/json" \
  -d '{
    "query": "MATCH (p:Person)-[:KNOWS]->(friend) RETURN p.name, friend.name LIMIT 5"
  }'
```

---

### Execute a Read-Only Query

```http
POST /api/graph/{graph_name}/ro_query
```

Same as `/query` but enforces read-only mode — write operations will return an error.

---

## Schema

### Get Graph Schema

```http
GET /api/graph/{graph_name}/schema
```

**Response 200:**

```json
{
  "node_labels": ["Person", "Entity", "Domain"],
  "relationship_types": ["KNOWS", "RELATED_TO", "ENHANCES"],
  "properties": {
    "Person": ["name", "age", "email"],
    "KNOWS": ["since", "strength"]
  },
  "indexes": [
    { "label": "Person", "property": "name", "type": "exact" },
    { "label": "Entity", "property": "embedding", "type": "vector" }
  ]
}
```

---

## Indexes

### Create an Index

```http
POST /api/graph/{graph_name}/index
```

**Request Body:**

```json
{
  "label": "Person",
  "property": "name",
  "type": "exact"
}
```

| Field | Type | Options | Description |
|---|---|---|---|
| `label` | string | Any node label | Label to index |
| `property` | string | Any property name | Property to index |
| `type` | string | `exact`, `fulltext`, `vector` | Index type |

---

### Delete an Index

```http
DELETE /api/graph/{graph_name}/index
```

**Request Body:**

```json
{
  "label": "Person",
  "property": "name"
}
```

---

## Bulk Operations

### Bulk Import

```http
POST /api/graph/{graph_name}/bulk
```

**Request Body:**

```json
{
  "nodes": [
    { "labels": ["Person"], "properties": { "name": "Alice", "age": 30 } },
    { "labels": ["Person"], "properties": { "name": "Bob", "age": 25 } }
  ],
  "edges": [
    {
      "src_node": 0,
      "dest_node": 1,
      "relation": "KNOWS",
      "properties": { "since": 2020 }
    }
  ]
}
```

**Response 200:**

```json
{
  "nodes_created": 2,
  "relationships_created": 1,
  "time": "12.4ms"
}
```

---

## Stats

### Get Graph Statistics

```http
GET /api/graph/{graph_name}/stats
```

**Response 200:**

```json
{
  "graph_name": "social",
  "node_count": 1000000,
  "relationship_count": 5200000,
  "label_count": 5,
  "relationship_type_count": 8,
  "property_count": 12,
  "memory_usage": "1.2GB",
  "cache_hits": 98234,
  "cache_misses": 412
}
```

---

## Error Codes

| HTTP Status | Code | Description |
|---|---|---|
| 400 | `INVALID_QUERY` | Malformed Cypher query |
| 401 | `UNAUTHORIZED` | Missing or invalid auth token |
| 404 | `GRAPH_NOT_FOUND` | Graph does not exist |
| 408 | `QUERY_TIMEOUT` | Query exceeded timeout limit |
| 409 | `CONSTRAINT_VIOLATION` | Unique constraint violated |
| 500 | `INTERNAL_ERROR` | Server-side error |

**Error Response Shape:**

```json
{
  "error": "GRAPH_NOT_FOUND",
  "message": "Graph 'social' does not exist. Create it first with POST /api/graph/social",
  "request_id": "req_01J2XY..."
}
```
