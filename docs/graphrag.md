---
title: GraphRAG Guide
layout: default
nav_order: 4
---

# GraphRAG with ESEILANE
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## What is GraphRAG?

**GraphRAG** (Graph Retrieval-Augmented Generation) is a technique that grounds LLM responses in structured graph knowledge, dramatically reducing hallucinations and improving accuracy.

Traditional RAG retrieves flat text chunks. GraphRAG retrieves **connected, structured context** — relationships, paths, and communities — giving the LLM far richer information to reason over.

```
User Query
    │
    ▼
Vector Search (semantic similarity)
    │
    ▼
Graph Traversal (relationships & context)
    │
    ▼
Structured Context → LLM Prompt
    │
    ▼
Grounded, Accurate Answer
```

---

## Why ESEILANE for GraphRAG?

| Feature | Benefit |
|---|---|
| Sub-millisecond traversal | Context retrieval doesn't slow your LLM pipeline |
| Vector + graph hybrid search | Find semantically similar nodes AND traverse relationships |
| Automatic ontology generation | Build knowledge graphs from unstructured text automatically |
| Multi-tenant graphs | Isolate knowledge per user, project, or tenant |

---

## Quick Start

```bash
pip install eseilane[graphrag]
```

```python
from eseilane.graphrag import GraphRAGClient
import os

client = GraphRAGClient(
    db_host="localhost",
    db_port=6379,
    llm_provider="openai",
    llm_model="gpt-4o",
    embedding_model="text-embedding-ada-002",
    openai_api_key=os.environ["OPENAI_API_KEY"],
)

# Ingest unstructured text — ESEILANE builds the knowledge graph automatically
client.ingest("""
  ESEILANE is a high-performance knowledge graph engine.
  It was created by Simpl3x3 and powers applications at eseilane.org.
  ESEILANE uses GraphBLAS for fast matrix-based graph computations.
  GraphBLAS is a standard for graph algorithms expressed in linear algebra.
""")

# Query with natural language
response = client.query("What technology does ESEILANE use for fast computations?")
print(response.answer)
# → ESEILANE uses GraphBLAS, a standard for graph algorithms expressed in linear algebra,
#   for fast matrix-based graph computations.

print(response.graph_context)
# → [ESEILANE] -[USES]-> [GraphBLAS] -[IS_A]-> [Linear Algebra Standard]
```

---

## Building a Knowledge Graph Manually

```python
from eseilane import ESEILANE

db = ESEILANE(host='localhost', port=6379)
g = db.select_graph('company_knowledge')

# Create an ontology
g.query("""
  CREATE
    (:Concept {name: 'Machine Learning'}),
    (:Concept {name: 'Neural Networks'}),
    (:Concept {name: 'Transformers'}),
    (:Concept {name: 'Attention Mechanism'}),
    (:Person {name: 'Vaswani', role: 'Researcher'}),
    (:Paper {title: 'Attention Is All You Need', year: 2017})
""")

g.query("""
  MATCH
    (p:Paper {title: 'Attention Is All You Need'}),
    (a:Concept {name: 'Attention Mechanism'}),
    (t:Concept {name: 'Transformers'}),
    (r:Person {name: 'Vaswani'})
  CREATE
    (p)-[:INTRODUCES]->(a),
    (p)-[:INTRODUCES]->(t),
    (r)-[:AUTHORED]->(p),
    (t)-[:USES]->(a)
""")
```

---

## Hybrid Search (Vector + Graph)

```python
import openai
from eseilane import ESEILANE

db = ESEILANE(host='localhost', port=6379)
g = db.select_graph('knowledge')

# Step 1: Create vector index
g.create_node_vector_index('Concept', 'embedding', dim=1536, similarity='cosine')

# Step 2: Store embeddings when adding nodes
oa = openai.OpenAI()

def add_concept(name: str, description: str):
    emb = oa.embeddings.create(input=description, model='text-embedding-ada-002')
    embedding = emb.data[0].embedding
    g.query(
        "CREATE (:Concept {name: $name, description: $desc, embedding: $emb})",
        params={"name": name, "desc": description, "emb": embedding}
    )

add_concept("GraphBLAS", "A standard for sparse matrix graph algorithms")
add_concept("Sparse Matrix", "A matrix with mostly zero values, efficient for graph storage")

# Step 3: Hybrid search at query time
query_text = "efficient graph computation"
query_emb = oa.embeddings.create(input=query_text, model='text-embedding-ada-002')
query_vector = query_emb.data[0].embedding

results = g.query("""
  CALL db.idx.vector.queryNodes('Concept', 'embedding', 5, $vec)
  YIELD node AS concept, score
  MATCH (concept)-[:RELATED_TO*1..2]->(related)
  RETURN concept.name, related.name, score
  ORDER BY score DESC
""", params={"vec": query_vector})

for row in results.result_set:
    print(f"{row[0]} → {row[1]} (score: {row[2]:.4f})")
```

---

## GraphRAG Pipeline Architecture

```python
from eseilane.graphrag import GraphRAGPipeline

pipeline = GraphRAGPipeline(
    graph=g,
    llm="gpt-4o",
    embedder="text-embedding-ada-002",
    retrieval_strategy="hybrid",      # vector + graph traversal
    max_context_nodes=20,
    max_hop_depth=3,
    community_detection=True,          # use graph communities as context
)

# Run end-to-end
answer = await pipeline.arun(
    query="Which researchers contributed most to the Transformer architecture?"
)
print(answer.text)
print(answer.sources)          # Nodes and relationships used as context
print(answer.confidence)       # Confidence score
```

---

## Best Practices

1. **Index everything you search on** — Range indexes for filtering, vector indexes for semantic similarity
2. **Use parameterized queries** — Always pass user input as params, never via string interpolation
3. **Limit traversal depth** — Deep traversals (`*1..10`) are expensive; prefer `*1..3`
4. **Batch writes** — Use bulk import for large initial loads
5. **Schema your ontology first** — Design node labels and relationship types before ingesting data
6. **Separate graphs per tenant** — Use one graph per user/project for clean isolation
