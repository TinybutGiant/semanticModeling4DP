// Step 1: Create the Product node
```
MERGE (p:Product {name: '4D Printing Product'})
```
// Step 2: Create the Product HAS_* relationships
1. create HAS_M relationship
```
MATCH (m:Material), (p:Product {name: '4D Printing Product'})
WITH DISTINCT m, p
MERGE (p)-[:HAS_M]->(m)
```

2. create HAS_U relationship
```
MATCH (u:Usage), (p:Product {name: '4D Printing Product'})
WITH DISTINCT u, p
MERGE (p)-[:HAS_U]->(u)
```

3. create HAS_S relationship
```
MATCH (s:Stimuli), (p:Product {name: '4D Printing Product'})
WITH DISTINCT s, p
MERGE (p)-[:HAS_S]->(s)
```

4. create HAS_R relationship
```
MATCH (r:Response), (p:Product {name: '4D Printing Product'})
WITH DISTINCT r, p
MERGE (p)-[:HAS_R]->(r)
```

5. create HAS_T relationship
```
MATCH (t:Transformation), (p:Product {name: '4D Printing Product'})
WITH DISTINCT t, p
MERGE (p)-[:HAS_T]->(t)
```

6. create HAS_B relationship
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
// Step 4: Create the relationships among **_PAIR based on combinations
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
2. MU-TB Pair (Material-Usage to Transformation-Behavior)
We are linking the application macro view (Usage) to the geometric micro view (Transformation-Behavior), highlighting how application requirements drive transformations and behaviors, like shape-changing properties and elongation ratios.
Pathways (U->T, U->B, T->U, B->U): The pathways indicate that usage requirements (U) drive both transformation (T) and behavior (B), while Transformation-Behavior (TB) impacts usage needs. This is effective for capturing how design requirements for size, shape, and functionality feed into transformation capabilities, and vice versa.

2.1 Create label Micro_view_of_geometric (Transformation-Behavior) for pathway U->T, U->B
```
MATCH (t:Transformation), (b:Behavior)
SET t:Micro_view_of_material_science, b:Micro_view_of_material_science
RETURN t, labels(t) AS t_labels, b, labels(b) AS b_labels
```
2.2 Create affects relationships for pathway U->T, U->B
```
MATCH (u:Usage), (t:Transformation)
WHERE u.id = t.id  // Match Material and Usage nodes with the same index
WITH DISTINCT u, t
MERGE (u)-[:AFFECTS]->(t)  // Create the relationship only if it doesn't already exist
```
```
MATCH (u:Usage), (b:Behavior)
WHERE u.id = b.id  // Match Material and Usage nodes with the same index
WITH DISTINCT u, b
MERGE (u)-[:AFFECTS]->(b)  // Create the relationship only if it doesn't already exist
```
3. SR-TB Pair (Stimuli-Response to Transformation-Behavior):
We are connecting the material science micro view (Stimuli-Response) to the geometric micro view (Transformation-Behavior), representing the interaction between material responses and how they influence or enable certain transformations.
Pathways (S->T, R->T, S->B, R->B)

3.1 The label Micro_view_of_material_science (Stimuli-Response) has been created in previous step 4 - 1.1; label  Micro_view_of_geometric (Transformation-Behavior) has been created in previous step 4 - 2.1; 

3.2 Create affects relationships for pathway S->T, R->T
```
MATCH (s:Stimuli), (t:Transformation)
WHERE s.id = t.id  // Match Material and Usage nodes with the same index
WITH DISTINCT s, t
MERGE (s)-[:AFFECTS]->(t)  // Create the relationship only if it doesn't already exist
```
```
MATCH (r:Response), (t:Transformation)
WHERE r.id = t.id  // Match Material and Usage nodes with the same index
WITH DISTINCT r, t
MERGE (r)-[:AFFECTS]->(t)  // Create the relationship only if it doesn't already exist
```

3.3 Create affects relationships for pathway S->B, R->B
```
MATCH (s:Stimuli), (b:Behavior)
WHERE s.id = b.id  // Match Material and Usage nodes with the same index
WITH DISTINCT s, b
MERGE (s)-[:AFFECTS]->(b)  // Create the relationship only if it doesn't already exist
```
```
MATCH (r:Response), (b:Behavior)
WHERE r.id = b.id  // Match Material and Usage nodes with the same index
WITH DISTINCT r, b
MERGE (r)-[:AFFECTS]->(b)  // Create the relationship only if it doesn't already exist
```

