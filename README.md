# semanticModeling4DP


step 1: Import the nodes from data.csv and doi-data.csv, with header as properties

Step 2: Add relationships between nodes

Relationship 1: each material node has doi, or each doi has material. 
MATCH (m:Material), (d:DOI)
WHERE m.doi = d.doi
MERGE (m)-[:HAS_DOI]->(d);


