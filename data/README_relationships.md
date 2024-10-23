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

2. create HSA_U relationship
```
MATCH (u:Usage), (p:Product {name: '4D Printing Product'})
WITH DISTINCT u, p
MERGE (p)-[:HAS_U]->(u)
```

3. create HSA_S relationship
```
MATCH (s:Stimuli), (p:Product {name: '4D Printing Product'})
WITH DISTINCT s, p
MERGE (p)-[:HAS_S]->(s)
```

4. create HSA_R relationship
```
MATCH (r:Response), (p:Product {name: '4D Printing Product'})
WITH DISTINCT r, p
MERGE (p)-[:HAS_R]->(r)
```

5. create HSA_T relationship
```
MATCH (t:Transformation), (p:Product {name: '4D Printing Product'})
WITH DISTINCT t, p
MERGE (p)-[:HAS_T]->(t)
```

6. create HSA_B relationship
```
MATCH (b:Behavior), (p:Product {name: '4D Printing Product'})
WITH DISTINCT b, p
MERGE (p)-[:HAS_B]->(b)
```
