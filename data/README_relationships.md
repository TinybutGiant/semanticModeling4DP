// Step 1: Create the Product node
```
MERGE (p:Product {name: '4D Printing Product'})
```
// Step 2: Create the Product HAS_* relationships
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

// Step 3: Create the **_PAIR pairing relationships
1. Create MU_PAIR
```
MATCH (m:Material), (u:Usage)
WHERE m.id = u.id  // Match Material and Usage nodes with the same index
WITH DISTINCT m, u
MERGE (m)-[:MU_PAIR]->(u)  // Create the relationship only if it doesn't already exist
```
2. Create SR_PAIR
```
MATCH (s:Stimuli), (r:Response)
WHERE s.id = r.id  // Match Material and Usage nodes with the same index
WITH DISTINCT s, r
MERGE (s)-[:SR_PAIR]->(r)  // Create the relationship only if it doesn't already exist
```
3. Create TB_PAIR
```
MATCH (t:Transformation), (b:Behavior)
WHERE t.id = b.id  // Match Material and Usage nodes with the same index
WITH DISTINCT t, b
MERGE (t)-[:TB_PAIR]->(b)  // Create the relationship only if it doesn't already exist
```

