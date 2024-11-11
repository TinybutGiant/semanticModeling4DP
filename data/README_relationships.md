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
WHERE s.id = r.id  // Match Stimuli and Response nodes with the same index
WITH DISTINCT s, r
MERGE (s)-[:SR_PAIR]->(r)  // Create the relationship only if it doesn't already exist
```
3. Create TB_PAIR
```
MATCH (t:Transformation), (b:Behavior)
WHERE t.id = b.id  // Match Transformation and Behavior nodes with the same index
WITH DISTINCT t, b
MERGE (t)-[:TB_PAIR]->(b)  // Create the relationship only if it doesn't already exist
```
// Step 4: Create the relationships among **_PAIR bsed on combinations
1. MU-SR Pair (Material-Usage to Stimuli-Response).
We are establishing a connection between the macro view of application (Usage) and the material science micro view (Stimuli-Response), suggesting that application requirements in terms of structure, size, deform speed, and function (e.g., for a dashboard) relate directly to material properties like response ratio.
Pathways (M->S, M->R, S->U, R->U): (may have insights that each path (e.g., M->S) specifically affects the Stimuli-Response layer, perhaps detailing how material properties (like nanosilica reinforcement) influence reaction times under different stimuli, which impacts usage.)
1.1 Create label Micro_view_of_material_science (Stimuli-Response) for pathway M->S, M->R
```
MATCH (s:Stimuli), (r:Response)
SET s:Micro_view_of_material_science, r:Micro_view_of_material_science
RETURN s, labels(s) AS s_labels, r, labels(r) AS r_labels
```
1.2 Create affects relationships for pathway M->S, M->R
```
MATCH (m:Material), (s:Stimuli)
WHERE m.id = s.id  // Match Material and Stimuli nodes with the same index
WITH DISTINCT m, s
MERGE (m)-[:AFFECTS]->(s)  // Create the relationship only if it doesn't already exist
```
```
MATCH (m:Material), (r:Response)
WHERE m.id = r.id  // Match Material and Response nodes with the same index
WITH DISTINCT m, r
MERGE (m)-[:AFFECTS]->(r)  // Create the relationship only if it doesn't already exist
```
1.3 Create label Macro_view_of_application (Usage) for pathway S->U, R->U
```
MATCH (u:Usage)
SET u:Macro_view_of_application
RETURN u, labels(u) AS labels
```
1.4 Create affects relationships for pathway S->U, R->U
```
MATCH (s:Stimuli), (u:Usage)
WHERE s.id = u.id  // Match Stimuli and Usage nodes with the same index
WITH DISTINCT s, u
MERGE (s)-[:AFFECTS]->(u)  // Create the relationship only if it doesn't already exist
```
```
MATCH (r:Response), (u:Usage)
WHERE r.id = u.id  // Match Response and Usage nodes with the same index
WITH DISTINCT r, u
MERGE (r)-[:AFFECTS]->(u)  // Create the relationship only if it doesn't already exist
```
2. MU-TB Pair (Material-Usage to Transformation-Behavior):
```
MATCH (s:Stimuli), (r:Response)
WHERE s.id = r.id  // Match Material and Usage nodes with the same index
WITH DISTINCT s, r
MERGE (s)-[:SR_PAIR]->(r)  // Create the relationship only if it doesn't already exist
```
3. SR-TB Pair (Stimuli-Response to Transformation-Behavior):
```
MATCH (t:Transformation), (b:Behavior)
WHERE t.id = b.id  // Match Material and Usage nodes with the same index
WITH DISTINCT t, b
MERGE (t)-[:TB_PAIR]->(b)  // Create the relationship only if it doesn't already exist
```

