---
title: TypeScript SDK
layout: default
parent: SDKs
nav_order: 2
---

# TypeScript / JavaScript SDK
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Installation

```bash
npm install eseilane-js
# or
yarn add eseilane-js
# or
pnpm add eseilane-js
```

**Requirements:** Node.js 20+, TypeScript 5+

---

## Connection

```typescript
import { ESEILANE } from 'eseilane-js';

// Basic connection
const db = new ESEILANE({ host: 'localhost', port: 6379 });

// With authentication
const db = new ESEILANE({
  host: 'localhost',
  port: 6379,
  password: 'your-password',
});

// With TLS
const db = new ESEILANE({
  host: 'your-host.eseilane.io',
  port: 6380,
  password: 'your-password',
  tls: true,
});

const graph = db.selectGraph('my_graph');
```

---

## Querying

### Basic Query

```typescript
const result = await graph.query(
  'MATCH (p:Person) RETURN p.name AS name, p.age AS age ORDER BY p.age'
);

for (const row of result.data) {
  console.log(`${row.name}: ${row.age}`);
}

console.log(`Query time: ${result.metadata.queryInternalExecutionTime}`);
```

### Parameterized Queries

```typescript
const result = await graph.query(
  'MATCH (p:Person) WHERE p.age > $minAge RETURN p.name',
  { params: { minAge: 25 } }
);
```

### Typed Results

```typescript
interface PersonRow {
  name: string;
  age: number;
}

const result = await graph.query<PersonRow>(
  'MATCH (p:Person) RETURN p.name AS name, p.age AS age'
);

result.data.forEach((row: PersonRow) => {
  console.log(row.name, row.age);
});
```

---

## Creating Data

```typescript
// Create nodes
await graph.query(
  `CREATE (:Product {
    id: $id,
    name: $name,
    price: $price
  })`,
  { params: { id: 'p001', name: 'Widget', price: 9.99 } }
);

// Create relationships
await graph.query(
  `MATCH (p:Product {id: $id}), (c:Category {name: $cat})
   CREATE (p)-[:BELONGS_TO]->(c)`,
  { params: { id: 'p001', cat: 'Tools' } }
);
```

---

## Transactions

```typescript
const tx = graph.beginTransaction();

try {
  await tx.query("CREATE (:Node {id: 1})");
  await tx.query("CREATE (:Node {id: 2})");
  await tx.commit();
  console.log('Transaction committed');
} catch (err) {
  await tx.rollback();
  throw err;
}
```

---

## Vector Search

```typescript
import OpenAI from 'openai';
import { ESEILANE } from 'eseilane-js';

const openai = new OpenAI();
const db = new ESEILANE({ host: 'localhost', port: 6379 });
const graph = db.selectGraph('knowledge');

// Generate embedding
const embeddingRes = await openai.embeddings.create({
  input: 'graph database expert',
  model: 'text-embedding-ada-002',
});
const vector = embeddingRes.data[0].embedding;

// Vector similarity search
const result = await graph.query(
  `CALL db.idx.vector.queryNodes('Person', 'embedding', $k, $vector)
   YIELD node, score
   RETURN node.name AS name, score
   ORDER BY score DESC`,
  { params: { k: 5, vector } }
);

result.data.forEach(({ name, score }) => {
  console.log(`${name}: ${score.toFixed(4)}`);
});
```

---

## Error Handling

```typescript
import {
  ESEILANEError,
  QueryTimeoutError,
  GraphNotFoundError,
} from 'eseilane-js';

try {
  const result = await graph.query('MATCH (n) RETURN n', { timeout: 100 });
} catch (err) {
  if (err instanceof QueryTimeoutError) {
    console.error('Query timed out');
  } else if (err instanceof GraphNotFoundError) {
    console.error('Graph not found');
  } else if (err instanceof ESEILANEError) {
    console.error('Database error:', err.message);
  } else {
    throw err;
  }
}
```

---

## API Reference

### `ESEILANE`

| Method | Returns | Description |
|---|---|---|
| `selectGraph(name)` | `Graph` | Select or create a graph |
| `listGraphs()` | `Promise<string[]>` | List all graphs |
| `deleteGraph(name)` | `Promise<void>` | Delete a graph |

### `Graph`

| Method | Returns | Description |
|---|---|---|
| `query<T>(cypher, opts?)` | `Promise<QueryResult<T>>` | Execute a Cypher query |
| `roQuery<T>(cypher, opts?)` | `Promise<QueryResult<T>>` | Read-only query |
| `schema()` | `Promise<Schema>` | Get graph schema |
| `createIndex(opts)` | `Promise<void>` | Create an index |
| `dropIndex(opts)` | `Promise<void>` | Drop an index |
| `beginTransaction()` | `Transaction` | Begin a transaction |
