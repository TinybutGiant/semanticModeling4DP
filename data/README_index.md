# Create and check vector indexes

Verified in an isolated Neo4j 5.26.30 / APOC Core 5.26.30 instance. Both indexes
became ONLINE and both stored-vector smoke tests returned results. This verifies
execution, not the paper's natural-language retrieval evaluation.

Run after README_nodes.md and README_relationships.md.
Target: Neo4j 5.26. The scripts use stored vectors from the new CSVs;
index creation does not generate or change embeddings.

## 0. Confirm the embedding configuration

- Model: sentence-transformers/all-MiniLM-L6-v2.
- Model revision: 1110a243fdf4706b3f48f1d95db1a4f5529b4d41.
- Generation: FP32 ONNX via Transformers.js 3.8.1, attention-mask-aware mean
  pooling and L2 normalization, maximum length 256 tokens.
- embedding: name.trim() + newline + description.trim().
- parameter_embedding: parameter_text.trim(), when present.
- Both properties: 384-dimensional numeric lists.

The node import already adds the Embeddable label and parses the CSV JSON
vectors. General and parameter vectors use the same model but different input
text. A missing parameter vector is optional and will not be indexed.

```cypher
MATCH (n:Embeddable)
RETURN count(*) AS nodes,
       count(n.embedding) AS node_vectors,
       count(n.parameter_embedding) AS parameter_vectors;
```

Expected: 180 / 180 / 10.

Check dimensions before index creation; expected result is zero invalid nodes:

```cypher
MATCH (n:Embeddable)
WHERE n.embedding IS NULL OR size(n.embedding) <> 384
   OR any(value IN n.embedding WHERE value IS NULL)
   OR (n.parameter_embedding IS NOT NULL AND
       (size(n.parameter_embedding) <> 384
        OR any(value IN n.parameter_embedding WHERE value IS NULL)))
RETURN count(n) AS invalid_nodes;
```

## 1. Create the general embedding index

The existing index name is retained for downstream compatibility.

```cypher
CREATE VECTOR INDEX embeddableEmbeddingIndex IF NOT EXISTS
FOR (n:Embeddable)
ON n.embedding
OPTIONS {indexConfig: {
    `vector.dimensions`: 384,
    `vector.similarity_function`: 'cosine'
}};
```

## 2. Create the optional parameter index

The new index targets parameter_embedding on any Embeddable node. It is no
longer restricted to MS_Material. The current data still has only 10 populated
parameter vectors, all in Material / MS.

```cypher
CREATE VECTOR INDEX parameterEmbeddingIndex IF NOT EXISTS
FOR (n:Embeddable)
ON n.parameter_embedding
OPTIONS {indexConfig: {
    `vector.dimensions`: 384,
    `vector.similarity_function`: 'cosine'
}};
```

Use parameterEmbeddingIndex in subsequent parameter retrieval code. The old
sampleParametersEmbeddingIndex refers to the old property and is not used here.

## 3. Wait for indexes and inspect their definitions

```cypher
CALL db.awaitIndex('embeddableEmbeddingIndex', 60);
```

```cypher
CALL db.awaitIndex('parameterEmbeddingIndex', 60);
```

```cypher
SHOW VECTOR INDEXES
YIELD name, state, populationPercent, labelsOrTypes, properties, options
RETURN name, state, populationPercent, labelsOrTypes, properties, options;
```

Both indexes should be ONLINE and 100% populated. Confirm the label, property,
dimension and cosine setting. IF NOT EXISTS does not repair an existing index
with an incorrect definition. These instructions assume a separate new database;
they deliberately do not drop indexes in an older database.

## 4. Smoke-test the general index

This uses an existing node vector to check index operation. It is not a
natural-language query evaluation and does not reproduce the paper's metrics.

```cypher
MATCH (q:Embeddable {node_id: 'B-PE-05'})
CALL db.index.vector.queryNodes('embeddableEmbeddingIndex', 5, q.embedding)
YIELD node, score
RETURN node.node_id AS node_id, node.category AS category,
       node.perspective_type AS perspective, node.name AS name, score
ORDER BY score DESC;
```

The queried node should normally appear among the results with a score close
to 1. Approximate search and tied vectors can affect ranking.

## 5. Smoke-test the parameter index

```cypher
MATCH (q:Embeddable {node_id: 'M-MS-05'})
CALL db.index.vector.queryNodes('parameterEmbeddingIndex', 5, q.parameter_embedding)
YIELD node, score
RETURN node.node_id AS node_id, node.parameter_text AS parameter_text, score
ORDER BY score DESC;
```

## 6. Subsequent text queries

Encode the user's query using the same model, pooling, normalization and
maximum length, then pass that numeric vector to the appropriate index.
The query can be plain text; it does not need artificial name/description fields.

All vectors were regenerated for this CSV revision. Reevaluate retrieval scores,
thresholds and relevance judgments; do not reuse the old reported scores as
results from this new graph.

These examples retain db.index.vector.queryNodes for Neo4j 5.26 compatibility.
Newer Neo4j releases support SEARCH and mark the procedure deprecated; choose
the query interface after confirming the actual installed version.

## References

- [Neo4j vector indexes](https://neo4j.com/docs/cypher-manual/current/indexes/semantic-indexes/vector-indexes/)
- [Neo4j built-in procedures](https://neo4j.com/docs/operations-manual/current/procedures/built-in-procedures/)
- [Embedding model](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2)
