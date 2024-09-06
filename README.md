# semanticModeling4DP

step 1: Import the nodes from data.csv and doi-data.csv, with header as properties

1.1 Material nodes

	LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/TinybutGiant/semanticModeling4DP/main/data.csv' AS row
	WITH row
	WHERE row.type = 'Material'
	MERGE (m:Material {description: row.description})
	ON CREATE SET 
 	    m.index = toInteger(row.index), 
	    m.doi = row.doi, 
	    m.title = row.title;

1.2 Usage nodes

	LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/TinybutGiant/semanticModeling4DP/main/data.csv' AS row
	WITH row
	WHERE row.type = 'Usage'
	MERGE (m:Usage {description: row.description})
	ON CREATE SET 
 	    m.index = toInteger(row.index), 
	    m.doi = row.doi, 
	    m.title = row.title;

 1.3 Stimuli nodes

	LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/TinybutGiant/semanticModeling4DP/main/data.csv' AS row
	WITH row
	MERGE (s:Stimuli {doi: row.doi})
	ON CREATE SET 
	    s.description = row.description,  
	    s.index = toInteger(row.index),  
	    s.title = row.title;        
explain: MERGE (s:Stimuli {doi: row.doi}): This ensures that a Stimuli node is created only if there isn't already a node with the same doi. If a node with the same doi already exists, it will not create a duplicate.

1.4 Response nodes

	LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/TinybutGiant/semanticModeling4DP/main/data.csv' AS row
	WITH row
	WHERE row.type = 'Response'
	MERGE (m:Response {description: row.description})
	ON CREATE SET 
 	    m.index = toInteger(row.index), 
	    m.doi = row.doi, 
	    m.title = row.title;

Step 2: Add relationships between nodes

Relationship 1: each material node has doi, or each doi has material. 
	
 	MATCH (m:Material), (d:DOI)
	WHERE m.doi = d.doi
	MERGE (m)-[:HAS_DOI]->(d);





Extra steps： Contraint for doi
	
 	CREATE CONSTRAINT doi_unique IF NOT EXISTS
	FOR (d:DOI)
	REQUIRE d.doi IS UNIQUE;

