updated data for neo4j

1. Codes for adding material nodes
```
LOAD CSV WITH HEADERS 
FROM 'https://raw.githubusercontent.com/TinybutGiant/semanticModeling4DP/refs/heads/main/data/material_data.csv' AS row
WITH row

// For PD perspective
FOREACH (ignoreMe IN CASE WHEN row.perspective_type = 'PD' THEN [1] ELSE [] END |
    MERGE (m:Material:PD_Material {id: row.id})
    ON CREATE SET 
        m.name = row.name_PD,
        m.description = row.description_PD,
        m.embedding = row.embedding_PD
)

// For PE perspective
FOREACH (ignoreMe IN CASE WHEN row.perspective_type = 'PE' THEN [1] ELSE [] END |
    MERGE (m:Material:PE_Material {id: row.id})
    ON CREATE SET 
        m.name = row.name_PE,
        m.description = row.description_PE,
        m.embedding = row.embedding_PE
)

// For MS perspective
FOREACH (ignoreMe IN CASE WHEN row.perspective_type = 'MS' THEN [1] ELSE [] END |
    MERGE (m:Material:MS_Material {id: row.id})
    ON CREATE SET 
        m.name = row.name_MS,
        m.description = row.description_MS,
        m.embedding = row.embedding_MS
)

```