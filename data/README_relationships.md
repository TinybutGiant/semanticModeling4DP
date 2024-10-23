// Step 1: Create the Product node
```
MERGE (p:Product {name: '4D Printing Product'})
```
1. create HSA_M relationship

```
MATCH (m:Material), (p:Product {name: '4D Printing Product'})
WITH DISTINCT m, p
MERGE (p)-[:HAS_M]->(m)
```
