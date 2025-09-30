# Explain RESMARK Knowledge

Query and explain knowledge from the RESMARK knowledge graph: $ARGUMENTS

[Extended thinking: This tool provides human-friendly access to the structured knowledge graph, transforming YAML entities and relations into readable documentation.]

## Query Processing

Parse the request to determine query type:
- Entity lookup: `/explain <entity_id>`
- Pattern search: `/explain patterns for <topic>`
- Anti-pattern search: `/explain anti-patterns for <topic>`
- Workflow explanation: `/explain workflow <workflow_id>`
- Directive lookup: `/explain directive <directive_id>`
- Relation query: `/explain what implements <pattern_id>`

## Knowledge Sources

Search these locations in order:
1. `.claude/knowledge-graph/entities/directives/`
2. `.claude/knowledge-graph/entities/patterns/`
3. `.claude/knowledge-graph/entities/anti-patterns/`
4. `.claude/knowledge-graph/entities/workflows/`
5. `.claude/knowledge-graph/relations/`

## Output Format

### For Entity Queries

**[Entity Type]: {{TITLE}}**

**ID**: `{{ENTITY_ID}}`
**Severity/Status**: {{IF_APPLICABLE}}
**Source**: {{SOURCE_MEMORY_FILE}}

**Description**:
{{FULL_DESCRIPTION}}

**Key Points**:
- {{IMPORTANT_POINT_1}}
- {{IMPORTANT_POINT_2}}
- {{IMPORTANT_POINT_3}}

**Example** (if applicable):
```{{LANGUAGE}}
{{CODE_EXAMPLE}}
```

**Related Entities**:
- {{RELATION_TYPE}}: {{RELATED_ENTITY}}
- {{RELATION_TYPE}}: {{RELATED_ENTITY}}

**Evidence/Metrics**:
- {{EMPIRICAL_EVIDENCE}}
- {{PERFORMANCE_METRICS}}

---

### For Pattern Queries

**Validated Patterns for {{TOPIC}}**

Found {{COUNT}} relevant patterns:

1. **{{PATTERN_NAME}}** (`{{PATTERN_ID}}`)
   - Status: {{VALIDATED|EXPERIMENTAL}}
   - Benefits: {{KEY_BENEFIT}}
   - Usage: {{WHEN_TO_USE}}

2. **{{PATTERN_NAME}}** (`{{PATTERN_ID}}`)
   - Status: {{VALIDATED|EXPERIMENTAL}}
   - Benefits: {{KEY_BENEFIT}}
   - Usage: {{WHEN_TO_USE}}

View details: `/explain {{PATTERN_ID}}`

---

### For Workflow Queries

**Workflow: {{WORKFLOW_TITLE}}**

**Purpose**: {{WORKFLOW_DESCRIPTION}}

**Phases**:
1. **{{PHASE_NAME}}**
   - {{PHASE_DESCRIPTION}}
   - Steps: {{STEP_COUNT}}
   - Key outputs: {{OUTPUTS}}

2. **{{PHASE_NAME}}**
   - {{PHASE_DESCRIPTION}}
   - Steps: {{STEP_COUNT}}
   - Key outputs: {{OUTPUTS}}

**Required Agents**:
- {{AGENT_TYPE}}: {{AGENT_PURPOSE}}
- {{AGENT_TYPE}}: {{AGENT_PURPOSE}}

**Implements Patterns**:
- {{PATTERN_NAME}}: {{HOW_USED}}

**Avoids Anti-Patterns**:
- {{ANTIPATTERN_NAME}}: {{HOW_AVOIDED}}

**Performance**: {{METRICS}}

**Trigger**: `/workflows:resmark/{{WORKFLOW_FILE}}`

---

### For Relation Queries

**Entities that {{RELATION_TYPE}} {{TARGET_ENTITY}}**

Found {{COUNT}} relations:

1. **{{ENTITY_ID}}** ({{ENTITY_TYPE}})
   - Relation: {{RELATION_TYPE}}
   - Description: {{RELATION_DESCRIPTION}}

2. **{{ENTITY_ID}}** ({{ENTITY_TYPE}})
   - Relation: {{RELATION_TYPE}}
   - Description: {{RELATION_DESCRIPTION}}

---

## Interactive Features

After displaying results, offer these options:

**Related Queries**:
- `/explain related to {{ENTITY_ID}}`
- `/explain what uses {{ENTITY_ID}}`
- `/explain what avoids {{ENTITY_ID}}`
- `/show code for {{PATTERN_ID}}`

**Actions**:
- `/validate against {{DIRECTIVE_ID}}`
- `/apply pattern {{PATTERN_ID}}`
- `/run workflow {{WORKFLOW_ID}}`

## Knowledge Graph Statistics

At the end of each query, show:
```
Knowledge Graph Stats:
- Directives: 10 (9 CRITICAL, 1 WARNING)
- Patterns: 10 (all VALIDATED)
- Anti-Patterns: 12 (4 CRITICAL, 4 HIGH, 4 MEDIUM)
- Workflows: 6
- Relations: 50+
```

## Learning Integration

If the query reveals a knowledge gap:
1. Note the missing information
2. Suggest creating a new entity
3. Provide template for addition
4. Mark for human review

Query to explain: $ARGUMENTS