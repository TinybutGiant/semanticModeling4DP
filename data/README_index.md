We have manually added the "embedding" for description to all the node's name, and "sample_parameters_embedding" for "sample_parameters_MS"
In order to use Neo4jVector to similarity_search in neo4j, we need to create index for thses embedding.

## 0. add embeded label for nodes
```
MATCH (m:Material)
SET m:Embeddable;

MATCH (u:Usage)
SET u:Embeddable;

MATCH (n:Stimuli)
SET n:Embeddable;

MATCH (n:Response)
SET n:Embeddable;

MATCH (n:Behavior)
SET n:Embeddable;

MATCH (n:Transformation)
SET n:Embeddable;

```

## 1. Create Vector Index for embedding

All nodes have a general embedding property for their description.
Nodes with the MS_Material label also have an additional property sample_parameters and its associated embedding sample_parameters_embedding.

For embedding:
```
CREATE VECTOR INDEX embeddableEmbeddingIndex IF NOT EXISTS 
FOR (n:Embeddable)
ON n.embedding
OPTIONS { indexConfig: {
        `vector.dimensions`: 384,
        `vector.similarity_function`: "cosine"
    }
};
```
drop the index if need:
```
DROP INDEX embeddableEmbeddingIndex
```

For sample_parameters_embedding (specific to MS_Material):
```
CREATE VECTOR INDEX sampleParametersEmbeddingIndex IF NOT EXISTS
FOR (n:MS_Material)
ON (n.sample_parameters_embedding)
OPTIONS {
    indexConfig: {
        `vector.dimensions`: 384,
        `vector.similarity_function`: "cosine"
    }
};
```
![image](https://github.com/user-attachments/assets/850f699d-4684-4307-9e66-23a8c6f46508)



