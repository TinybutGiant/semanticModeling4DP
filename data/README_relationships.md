# Create semantic-net relationships

Verified in an isolated Neo4j 5.26.30 / APOC Core 5.26.30 instance: 270 structural
edges, no duplicates on rerun. Optional evidence import was tested only with
synthetic fixtures, including rejection cases; no research evidence was fabricated.

Run after README_nodes.md, in the separate database containing the new CSV data.
All matches use `Embeddable` nodes with `source_index`, `perspective_type`
and `category`. The relationship names HAS_M, HAS_U, HAS_S, HAS_R, HAS_T,
HAS_B, MU_PAIR, SR_PAIR and TB_PAIR are preserved.

HAS_* and *_PAIR are structural relationships. AFFECTS is reserved for a
documented directed relationship supported by the source literature. Matching
source/perspective fields establishes correspondence, not evidence of influence.

## 1. Create the Product node

This is the existing case-level anchor, not an additional semantic category.

```cypher
MERGE (p:Product {name: '4D Printing Product'});
```

## 2. Create Product HAS_* relationships

### HAS_M

```cypher
MATCH (p:Product {name: '4D Printing Product'})
MATCH (n:Embeddable:Material)
WHERE n.category = 'Material'
MERGE (p)-[:HAS_M]->(n);
```

### HAS_U

```cypher
MATCH (p:Product {name: '4D Printing Product'})
MATCH (n:Embeddable:Usage)
WHERE n.category = 'Usage'
MERGE (p)-[:HAS_U]->(n);
```

### HAS_S

```cypher
MATCH (p:Product {name: '4D Printing Product'})
MATCH (n:Embeddable:Stimulus)
WHERE n.category = 'Stimulus'
MERGE (p)-[:HAS_S]->(n);
```

### HAS_R

```cypher
MATCH (p:Product {name: '4D Printing Product'})
MATCH (n:Embeddable:Response)
WHERE n.category = 'Response'
MERGE (p)-[:HAS_R]->(n);
```

### HAS_T

```cypher
MATCH (p:Product {name: '4D Printing Product'})
MATCH (n:Embeddable:Transformation)
WHERE n.category = 'Transformation'
MERGE (p)-[:HAS_T]->(n);
```

### HAS_B

```cypher
MATCH (p:Product {name: '4D Printing Product'})
MATCH (n:Embeddable:Behavior)
WHERE n.category = 'Behavior'
MERGE (p)-[:HAS_B]->(n);
```

## 3. Create the three structural pairs

### MU_PAIR

```cypher
MATCH (a:Embeddable:Material), (b:Embeddable:Usage)
WHERE a.category = 'Material' AND b.category = 'Usage'
  AND a.source_index = b.source_index
  AND a.perspective_type = b.perspective_type
MERGE (a)-[:MU_PAIR]->(b);
```

### SR_PAIR

```cypher
MATCH (a:Embeddable:Stimulus), (b:Embeddable:Response)
WHERE a.category = 'Stimulus' AND b.category = 'Response'
  AND a.source_index = b.source_index
  AND a.perspective_type = b.perspective_type
MERGE (a)-[:SR_PAIR]->(b);
```

### TB_PAIR

```cypher
MATCH (a:Embeddable:Transformation), (b:Embeddable:Behavior)
WHERE a.category = 'Transformation' AND b.category = 'Behavior'
  AND a.source_index = b.source_index
  AND a.perspective_type = b.perspective_type
MERGE (a)-[:TB_PAIR]->(b);
```

## 4. Assign the semantic view labels

These labels describe the MU / SR / TB organization. They do not replace or
restrict the PD / PE / MS perspectives.

- Material and Usage: macro view of application.
- Stimulus and Response: micro view of material science.
- Transformation and Behavior: micro view of geometric design.

The existing `Micro_view_of_geometric` label name is retained. This corrects
the original code that accidentally assigned the material-science label to T/B.
The three view labels are reset only on imported Embeddable nodes.

```cypher
MATCH (n:Embeddable)
WHERE n.category IN ['Material', 'Usage', 'Stimulus', 'Response', 'Transformation', 'Behavior']
REMOVE n:Macro_view_of_application:Micro_view_of_material_science:Micro_view_of_geometric;
```

```cypher
MATCH (n:Embeddable)
WHERE n.category IN ['Material', 'Usage']
SET n:Macro_view_of_application;
```

```cypher
MATCH (n:Embeddable)
WHERE n.category IN ['Stimulus', 'Response']
SET n:Micro_view_of_material_science;
```

```cypher
MATCH (n:Embeddable)
WHERE n.category IN ['Transformation', 'Behavior']
SET n:Micro_view_of_geometric;
```

## 5. Optional: import evidence-backed AFFECTS relationships

**Skip this step until a reviewed relationship evidence table exists.** The six
node CSVs do not contain that table. No AFFECTS edges are inferred automatically
from shared IDs or parameter text. The original automatic same-ID AFFECTS
creation blocks have therefore been replaced by this optional evidence import.

The permitted framework pathways remain:

| Pair combination | Directed category pathways |
| --- | --- |
| MU-SR | M -> S, M -> R, S -> U, R -> U |
| MU-TB | U -> T, U -> B, T -> U, B -> U |
| SR-TB | S -> T, R -> T, S -> B, R -> B |

A permitted pathway is not a claim that every source supports it. Direct
M -> T or M -> B is not automatically added. Material-property or composition
evidence must support the relationship interpretation.

Prepare `relationship_evidence.csv` with this header, one reviewed evidence
record per row. This header is a template, not extracted study data:

```csv
evidence_id,source_node_id,target_node_id,source_reference,evidence_locator,evidence_text
```

- evidence_id: stable unique identifier for the evidence record.
- source_node_id / target_node_id: exact IDs from the new node CSVs.
- source_reference: DOI or full bibliographic citation.
- evidence_locator: page, section, table or figure locating the evidence.
- evidence_text: source-backed statement supporting the directed relationship.

For this implementation, the two nodes must share both source_index and
perspective_type. Additional cross-perspective edges require a separate,
explicitly defined relation design. Review papers are not used as experimental
parameter-relationship evidence in this workflow.

Place the completed evidence CSV in Neo4j's import directory. This import
rejects missing endpoints, missing evidence and unsupported pathways.

```cypher
LOAD CSV WITH HEADERS FROM 'file:///relationship_evidence.csv' AS row
OPTIONAL MATCH (a:Embeddable {node_id: row.source_node_id})
OPTIONAL MATCH (b:Embeddable {node_id: row.target_node_id})
CALL apoc.util.validate(
  a IS NULL OR b IS NULL,
  'Unknown evidence endpoint: %s -> %s',
  [coalesce(row.source_node_id, '(missing)'), coalesce(row.target_node_id, '(missing)')]
)
WITH row, a, b
CALL apoc.util.validate(
  any(field IN [row.evidence_id, row.source_reference, row.evidence_locator, row.evidence_text]
      WHERE trim(coalesce(field, '')) = '')
  OR a.source_index <> b.source_index
  OR a.perspective_type <> b.perspective_type
  OR NOT (a.category + '->' + b.category) IN [
    'Material->Stimulus', 'Material->Response', 'Stimulus->Usage', 'Response->Usage',
    'Usage->Transformation', 'Usage->Behavior', 'Transformation->Usage', 'Behavior->Usage',
    'Stimulus->Transformation', 'Response->Transformation', 'Stimulus->Behavior', 'Response->Behavior'
  ],
  'Invalid relationship evidence: %s', [coalesce(row.evidence_id, '(missing)')]
)
MERGE (a)-[r:AFFECTS {evidence_id: row.evidence_id}]->(b)
SET r.source_index = a.source_index,
    r.perspective_type = a.perspective_type,
    r.source_reference = row.source_reference,
    r.evidence_locator = row.evidence_locator,
    r.evidence_text = row.evidence_text
RETURN count(*) AS imported_evidence_rows;
```

AFFECTS preserves the uppercase spelling in the original repository.
Reruns update the same evidence record on the same endpoints. Multiple evidence
records may support the same directed node pair. Changing endpoints or deleting
evidence rows does not remove earlier edges; review such changes separately.

## 6. Check the graph

```cypher
MATCH (n)
RETURN count(n) AS total_nodes,
       count(CASE WHEN n:Embeddable THEN 1 END) AS semantic_nodes,
       count(CASE WHEN n:Product THEN 1 END) AS product_nodes;
```

```cypher
MATCH ()-[r]->()
RETURN type(r) AS relationship_type, count(*) AS relationships
ORDER BY relationship_type;
```

After steps 1-4 with the supplied files, expect 181 nodes: 180 semantic nodes
and one Product anchor. Expect 180 HAS_* edges and 90 *_PAIR edges (270 total).
Each HAS_* type and each *_PAIR type has 30 edges. AFFECTS count is zero until
actual relationship evidence is imported; it is not a fixed multiple of papers.

Check pairing integrity; this should return no rows:

```cypher
MATCH (a:Embeddable)-[r:MU_PAIR|SR_PAIR|TB_PAIR]->(b:Embeddable)
WHERE a.source_index <> b.source_index
   OR a.perspective_type <> b.perspective_type
RETURN a.node_id, type(r), b.node_id;
```

All automatic structural creation uses MERGE and can be rerun without adding
duplicates. These scripts do not delete relationships left by older code.
