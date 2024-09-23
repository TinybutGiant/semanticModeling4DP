# semanticModeling4DP
A semantic modeling in neo4j involves: node data, constraints, relationship data, extra labels.

There are 4 main importing steps, "Creating constraints for certain nodes", "Importing nodes' data from csv files", "Creating relationships' data from csv files", and "Creating extra labels on nodes (for performance improving)".

## Step 1： Contraint for doi
```	
CREATE CONSTRAINT doi_unique IF NOT EXISTS
FOR (d:DOI)
REQUIRE d.doi IS UNIQUE;

CREATE CONSTRAINT doi_unique IF NOT EXISTS
FOR (m:Material)
REQUIRE m.index IS UNIQUE;
```

## Step 2: Import the nodes from data.csv and doi-data.csv, with header as properties, into the Neo4J.
### 1.0 Subject topic node

```
CREATE (p:Product {name: '4D printing products'})
```

### 1.1 Material nodes
```
LOAD CSV WITH HEADERS 
FROM 'https://raw.githubusercontent.com/TinybutGiant/semanticModeling4DP/main/data.csv' AS row
WITH row
WHERE row.type = 'Material'
MERGE (m:Material {name: row.description})
ON CREATE SET 
    m.index = toInteger(row.index), 
    m.doi = row.doi, 
    m.title = row.title;
```
### 1.2 Usage nodes
```
LOAD CSV WITH HEADERS 
FROM 'https://raw.githubusercontent.com/TinybutGiant/semanticModeling4DP/main/data.csv' AS row
WITH row
WHERE row.type = 'Usage'
MERGE (u:Usage {name: row.description})
ON CREATE SET 
    u.index = toInteger(row.index),
    u.doi = row.doi, 
    u.title = row.title;
```
### 1.3 Stimuli nodes
```
LOAD CSV WITH HEADERS 
FROM 'https://raw.githubusercontent.com/TinybutGiant/semanticModeling4DP/main/data.csv' AS row
WITH row
WHERE row.type = 'Stimuli'
MERGE (s:Stimuli {doi: row.doi})  // Merge by DOI to avoid duplicate nodes
ON CREATE SET 
    s.name = row.description,  
    s.index = toInteger(row.index),  
    s.title = row.title
ON MATCH SET
    s.name = row.description,  // Update the properties if the node already exists
    s.index = toInteger(row.index),  
    s.title = row.title;      
```
Note：Stimuli type (e.g., Heat, Electric Heating) is duplicated, so we need to ensure that each Stimuli node is created only once per unique doi.

Cypher code explain: MERGE (s:Stimuli {doi: row.doi}): This ensures that a Stimuli node is created only if there isn't already a node with the same doi. If a node with the same doi already exists, it will not create a duplicate.

### 1.4 Response nodes
```
LOAD CSV WITH HEADERS 
FROM 'https://raw.githubusercontent.com/TinybutGiant/semanticModeling4DP/main/data.csv' AS row
WITH row
WHERE row.type = 'Response'
MERGE (r:Response {doi: row.doi})
ON CREATE SET 
    r.name = row.description,  
    r.index = toInteger(row.index),  
    r.title = row.title
ON MATCH SET
    r.name = row.description,  // Update the properties if the node already exists
    r.index = toInteger(row.index),  
    r.title = row.title;  
```
## Step 3: Add relationships between nodes from has_doi.csv , ***.csv, and ***.csv, with header as properties, into the Neo4J. Or you can directly add from the nodes.
However, the seperate relationship csv files can provide a better management.

### Relationship 0: each material node has doi, or each doi has material. 
```	
MATCH (m:Material), (d:DOI)
WHERE m.doi = d.doi
MERGE (m)-[:HAS_DOI]->(d);
```

### Relationship 1.1: the 4D printing products node has material nodes. 
```
MATCH (m:Material), (p:Product {name: '4D printing products'})
WITH DISTINCT m, p
MERGE (p)-[:HAS_M]->(m)
```

### Relationship 1.2: the 4D printing products node has usage nodes. 
```
MATCH (u:Usage), (p:Product {name: '4D printing products'})
WITH DISTINCT u, p
MERGE (p)-[:HAS_U]->(u)
```

### Relationship 1.3: the 4D printing products node has stimuli nodes. 
```
MATCH (s:Stimuli), (p:Product {name: '4D printing products'})
WITH DISTINCT s, p
MERGE (p)-[:HAS_S]->(s)
```

### Relationship 1.4: the 4D printing products node has response nodes. 
```
MATCH (r:Response), (p:Product {name: '4D printing products'})
WITH DISTINCT r, p
MERGE (p)-[:HAS_R]->(r)
```

### Relationship 2.1: Material-Usage pair relationship. 
```
MATCH (m:Material), (u:Usage)
WHERE m.index = u.index  // Match Material and Usage nodes with the same index
WITH DISTINCT m, u
MERGE (m)-[:MU_PAIR]->(u)  // Create the relationship only if it doesn't already exist
```

### Relationship 2.2: Stimuli-Response pair relationship. 
```
MATCH (s:Stimuli), (r:Response)
WHERE s.index = r.index  // Match Material and Usage nodes with the same index
WITH DISTINCT s, r
MERGE (s)-[:SR_PAIR]->(r)  // Create the relationship only if it doesn't already exist
```


### Relationship 2.3: Transform-Behavior pair relationship. 
```
MATCH (s:Stimuli), (r:Response)
WHERE s.index = r.index  // Match Material and Usage nodes with the same index
WITH DISTINCT s, r
MERGE (s)-[:SR_PAIR]->(r)  // Create the relationship only if it doesn't already exist
```


## Step 4: Add extra labels.



# Final a single import process using the CALL clause with IN TRANSACTIONS.
```
CALL {
    CREATE CONSTRAINT doi_unique IF NOT EXISTS
    FOR (d:DOI)
    REQUIRE d.doi IS UNIQUE;

    CREATE CONSTRAINT doi_unique IF NOT EXISTS
    FOR (m:Material)
    REQUIRE m.index IS UNIQUE;
} IN TRANSACTIONS [OF X ROWS]

CALL {
    LOAD CSV WITH HEADERS 
    FROM 'https://raw.githubusercontent.com/TinybutGiant/semanticModeling4DP/main/data.csv' AS row
    WITH row
    WHERE row.type = 'Material'
    MERGE (m:Material {description: row.description})
    ON CREATE SET 
        m.index = toInteger(row.index), 
        m.doi = row.doi, 
        m.title = row.title;
} IN TRANSACTIONS [OF X ROWS]

CALL {
    LOAD CSV WITH HEADERS 
    FROM 'https://raw.githubusercontent.com/TinybutGiant/semanticModeling4DP/main/data.csv' AS row
    WITH row
    WHERE row.type = 'Usage'
    MERGE (m:Usage {description: row.description})
    ON CREATE SET 
      m.index = toInteger(row.index),
      m.doi = row.doi, 
      m.title = row.title;
} IN TRANSACTIONS [OF X ROWS]

CALL {
  // query
} IN TRANSACTIONS [OF X ROWS]
```

Graph Visualization with db.schema.visualization() 
```
call db.schema.visualization() 
```

