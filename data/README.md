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
2. Code for adding usage nodes
```
LOAD CSV WITH HEADERS 
FROM 'https://raw.githubusercontent.com/TinybutGiant/semanticModeling4DP/refs/heads/main/data/usage_data.csv' AS row
WITH row

// For PD perspective
FOREACH (ignoreMe IN CASE WHEN row.perspective_type = 'PD' THEN [1] ELSE [] END |
    MERGE (u:Usage:PD_Usage {id: row.id})
    ON CREATE SET 
        u.name = row.name_PD,
        u.description = row.description_PD,
        u.embedding = row.embedding_PD
)

// For PE perspective
FOREACH (ignoreMe IN CASE WHEN row.perspective_type = 'PE' THEN [1] ELSE [] END |
    MERGE (u:Usage:PE_Usage {id: row.id})
    ON CREATE SET 
        u.name = row.name_PE,
        u.description = row.description_PE,
        u.embedding = row.embedding_PE
)

// For MS perspective
FOREACH (ignoreMe IN CASE WHEN row.perspective_type = 'MS' THEN [1] ELSE [] END |
    MERGE (u:Usage:MS_Usage {id: row.id})
    ON CREATE SET 
        u.name = row.name_MS,
        u.description = row.description_MS,
        u.embedding = row.embedding_MS
)
```
3. Code for adding stimuli nodes
```
LOAD CSV WITH HEADERS 
FROM 'https://raw.githubusercontent.com/TinybutGiant/semanticModeling4DP/refs/heads/main/data/stimuli_data.csv' AS row
WITH row

// For PD perspective
FOREACH (ignoreMe IN CASE WHEN row.perspective_type = 'PD' THEN [1] ELSE [] END |
    MERGE (s:Stimuli:PD_Stimuli {id: row.id})
    ON CREATE SET 
        s.name = row.name_PD,
        s.description = row.description_PD,
        s.embedding = row.embedding_PD
)

// For PE perspective
FOREACH (ignoreMe IN CASE WHEN row.perspective_type = 'PE' THEN [1] ELSE [] END |
    MERGE (s:Stimuli:PE_Stimuli {id: row.id})
    ON CREATE SET 
        s.name = row.name_PE,
        s.description = row.description_PE,
        s.embedding = row.embedding_PE
)

// For MS perspective
FOREACH (ignoreMe IN CASE WHEN row.perspective_type = 'MS' THEN [1] ELSE [] END |
    MERGE (s:Stimuli:MS_Stimuli {id: row.id})
    ON CREATE SET 
        s.name = row.name_MS,
        s.description = row.description_MS,
        s.embedding = row.embedding_MS
)
```
4. Code for adding response nodes
```
LOAD CSV WITH HEADERS 
FROM 'https://raw.githubusercontent.com/TinybutGiant/semanticModeling4DP/refs/heads/main/data/response_data.csv' AS row
WITH row

// For PD perspective
FOREACH (ignoreMe IN CASE WHEN row.perspective_type = 'PD' THEN [1] ELSE [] END |
    MERGE (r:Response:PD_Response {id: toInteger(row.id)})
    ON CREATE SET 
        r.name = row.name_PD,
        r.description = row.description_PD,
        r.embedding = row.embedding_PD
)

// For PE perspective
FOREACH (ignoreMe IN CASE WHEN row.perspective_type = 'PE' THEN [1] ELSE [] END |
    MERGE (r:Response:PE_Response {id: toInteger(row.id)})
    ON CREATE SET 
        r.name = row.name_PE,
        r.description = row.description_PE,
        r.embedding = row.embedding_PE
)

// For MS perspective
FOREACH (ignoreMe IN CASE WHEN row.perspective_type = 'MS' THEN [1] ELSE [] END |
    MERGE (r:Response:MS_Response {id: toInteger(row.id)})
    ON CREATE SET 
        r.name = row.name_MS,
        r.description = row.description_MS,
        r.embedding = row.embedding_MS
)

```
