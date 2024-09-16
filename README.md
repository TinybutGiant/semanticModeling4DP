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

### 1.1 Material nodes
```
LOAD CSV WITH HEADERS 
FROM 'https://raw.githubusercontent.com/TinybutGiant/semanticModeling4DP/main/data.csv' AS row
WITH row
WHERE row.type = 'Material'
MERGE (m:Material {description: row.description})
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
MERGE (m:Usage {description: row.description})
ON CREATE SET 
    m.index = toInteger(row.index),
    m.doi = row.doi, 
    m.title = row.title;
```
### 1.3 Stimuli nodes
```
LOAD CSV WITH HEADERS 
FROM 'https://raw.githubusercontent.com/TinybutGiant/semanticModeling4DP/main/data.csv' AS row
WITH row
MERGE (s:Stimuli {doi: row.doi})
ON CREATE SET 
	s.description = row.description,  
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
MERGE (m:Response {description: row.description})
ON CREATE SET 
	m.index = toInteger(row.index), 
	m.doi = row.doi, 
	m.title = row.title;
```
## Step 3: Add relationships between nodes from has_doi.csv , ***.csv, and ***.csv, with header as properties, into the Neo4J. Or you can directly add from the nodes.
However, the seperate relationship csv files can provide a better management.

### Relationship 1: each material node has doi, or each doi has material. 
```	
MATCH (m:Material), (d:DOI)
WHERE m.doi = d.doi
MERGE (m)-[:HAS_DOI]->(d);
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
