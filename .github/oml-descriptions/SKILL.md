---
name: oml-descriptions
description: |
  **🎯 REQUIRED FOR CREATING OML DESCRIPTION INSTANCES 🎯**
  
  Workflow skill for creating OML description instances (individuals, instances of concepts). Navigates between vocabulary definitions (build/oml), SHACL tables (src/md), and description files (src/oml) to create correctly-formed instances.
  
  **⚠️ MANDATORY - Use this skill when:**
  - Creating new instances in description OML files
  - Adding instances of Stakeholder, Concern, Requirement, Entity, Component, Scenario, Activity, etc.
  - User requests "create a stakeholder", "add a concern", "define an entity", "create instances"
  - Working with src/oml/**/*.oml files
  
  **DO NOT USE FOR:**
  - Creating concepts, aspects, or relations (those go in vocabulary files - build/oml)
  - Modifying SHACL tables or MD files
  - Editing vocabulary definitions
  
  **THREE-FILE WORKFLOW:**
  1. Vocabulary (build/oml/www.modelware.io/sierra/*.oml) - defines concepts & relations
  2. SHACL tables (src/md/**/*.md) - defines required properties & target classes
  3. Descriptions (src/oml/**/*.oml) - where instances are created
  
  **🛡️ SAFE EDITING**: Always use append-only pattern (match closing brace only) to prevent resurrection of user-deleted instances.
---

# OML Description Instances Creation

## Overview

Creating OML description instances requires coordinating information from three different file types. This skill ensures instances are created with correct types, properties, and relations.

## File Structure

```
sierra-method/
├── build/oml/www.modelware.io/sierra/    # VOCABULARY - concepts & relations
│   ├── stakeholder.oml                    # Defines Stakeholder, Concern concepts
│   ├── entity.oml                         # Defines Entity concept
│   ├── process.oml                        # Defines Activity, Function concepts
│   └── ...
├── src/md/<org>/          # SHACL TABLES - property specifications
│   └── <project>/
│       ├── context-analysis/
│       │   ├── stakeholders.md            # SHACL for Stakeholder instances
│       │   ├── concerns.md                # SHACL for Concern instances
│       │   └── ...
│       └── operational-analysis/
│           ├── entities.md                # SHACL for Entity instances
│           └── ...
└── src/oml/<org>/         # DESCRIPTIONS - instance definitions
    └── <project>/
        ├── context-analysis/
        │   ├── stakeholders.oml           # Stakeholder instances
        │   ├── concerns.oml               # Concern instances
        │   └── ...
        └── operational-analysis/
            ├── entities.oml               # Entity instances
            └── ...
```

## 🚨 CRITICAL PRE-FLIGHT CHECKLIST 🚨

Before creating ANY instance, verify:
- [ ] SHACL table exists in corresponding MD file
- [ ] Target class (sh:targetClass) identified
- [ ] **REQUIRED** properties (sh:minCount ≥ 1) identified
- [ ] Optional properties identified (only if explicitly requested or defined by project policy)
- [ ] Relations and their target classes (sh:class) identified
- [ ] Vocabulary definition of relations checked (forward/reverse direction)
- [ ] Referenced instances exist in other description files
- [ ] Correct vocabulary separator `:` (colon) used throughout
- [ ] All imported vocabularies/descriptions are in `uses` or `extends`

## Normative Source Policy (Description Work)

Use this precedence order for instance creation:
1. OML language rules for descriptions and instances
2. Vocabulary semantics (types, relation meaning, domain/range)
3. SHACL constraints (`sh:minCount`, `sh:maxCount`, `sh:class`)
4. Project conventions (optional and explicitly marked)

Do not require properties just because they appear in existing files.
When documenting an axiom in this skill, keep one canonical example maximum.

## Step-by-Step Workflow

### Step 1: Identify the Request

Parse the user's request to determine:
- **What** needs to be created (Stakeholder? Concern? Entity? Component?)
- **Which domain** it belongs to (context-analysis? operational-analysis? system-analysis?)
- **Key details** provided (name, description, category, relationships)

### Step 2: Locate the MD File with SHACL

**Path pattern**: `src/md/<org>/<project>/{domain}/{concept-plural}.md`

Examples:
- Stakeholders → `src/md/.../context-analysis/stakeholders.md`
- Concerns → `src/md/.../context-analysis/concerns.md`
- Entities → `src/md/.../operational-analysis/entities.md`
- Components → `src/md/.../system-analysis/components.md`

**Read the MD file** and find the SHACL table section (between ` ```table-editor ` markers).

### Step 3: Extract SHACL Information

From the SHACL table, identify:

#### A. Target Class
```turtle
sh:targetClass stakeholder:Concern ;
```
→ Instance type will be `stakeholder:Concern`

#### B. Scalar Properties
```turtle
sh:property [
    sh:path base:category ;
    sh:name "Categories" ;
] ;
sh:property [
    sh:path base:priority ;
    sh:name "Priority" ;
    sh:maxCount 1 ;
] ;
sh:property [
    sh:path base:description ;
    sh:name "Description" ;
    dash:editor dash:TextAreaEditor ;
    sh:maxCount 1 ;
] ;
```
→ Available properties: `base:category`, `base:priority`, `base:description`
→ Note: `sh:maxCount 1` means single value, no `sh:maxCount` means multiple values allowed

#### C. Relations (Object Properties)
```turtle
sh:property [
    sh:path stakeholder:isExpressedBy ;
    sh:name "Stakeholders" ;
    sh:class stakeholder:Stakeholder ;        
    sh:minCount 1 ;
] ;
```
→ Relation: `stakeholder:isExpressedBy`
→ Target class: `stakeholder:Stakeholder` (must reference instances of this type)
→ Required: `sh:minCount 1` means at least one value needed

**Key SHACL Fields:**
- `sh:path` - the property or relation IRI
- `sh:class` - the target class for object properties (relations)
- `sh:minCount` - minimum cardinality (if ≥ 1, property is **REQUIRED**)
- `sh:maxCount` - maximum cardinality (1 = single value, absent = multiple)
- `sh:name` - human-readable label (for UI display)

#### D. REQUIRED vs OPTIONAL Properties

**🔴 CRITICAL DISTINCTION:**

1. **REQUIRED** (sh:minCount ≥ 1): MUST be included or validation fails
2. **OPTIONAL** (no sh:minCount): include only when requested, policy-driven, or semantically needed

**Example - Concern instances:**
- ✅ **REQUIRED**: `stakeholder:isExpressedBy` (has sh:minCount 1)
- ⚪ **OPTIONAL**: `base:category`, `base:description`, `base:priority` unless required by SHACL or user request

**Action**: After extracting SHACL, consult [SHACL Requirements Reference](./references/shacl-requirements.md) for examples, but treat these as non-normative unless backed by SHACL/vocabulary or explicit user request.

### Step 3.5: Optional Category Policy

`base:category` is optional unless required by SHACL/project policy or explicitly requested by the user.

**Category Inference Rules:**

**For Concerns (stakeholder:Concern):**
- Instance name ends with "Need" or "Needs" → `base:category "Need"`
- Instance name contains "Objective" → `base:category "Objective"`
- Instance name contains "Constraint" → `base:category "Constraint"`

**For Stakeholders (stakeholder:Stakeholder):**
- Description mentions "Responder", "operator", "user" → `base:category "User"` or `"Beneficiary"`
- Description mentions "engineer", "officer", "support" → `base:category "Support"`

**For Requirements (stakeholder:Requirement):**
- User specifies "high-level" or "level 1" → `base:category "L1"`
- User specifies "detailed" or "level 2" → `base:category "L2"`
- User specifies "implementation" or "level 3" → `base:category "L3"`
- Default to `"L1"` if not specified

**Examples (policy-driven):**
- `AhmedNeeds` → category: `"Need"`
- `SafetyConstraint` → category: `"Constraint"`
- `RapidResponseObjective` → category: `"Objective"`

See [SHACL Requirements Reference](./references/shacl-requirements.md) for complete category lists.

### Step 4: Understand Relation Direction

**CRITICAL**: Relations have direction (forward/reverse).

Find the relation definition in the **vocabulary file** (`build/oml/www.modelware.io/sierra/{vocab}.oml`):

```oml
relation expresses [
    from Stakeholder
    to Concern
    reverse isExpressedBy
    symmetric
]
```

**Understanding direction:**
- `from Stakeholder to Concern` - forward direction is `expresses`
- `reverse isExpressedBy` - reverse direction is `isExpressedBy`

**In SHACL**: `sh:path stakeholder:isExpressedBy` means we're using the **reverse** direction.

**In instance definition:**
```oml
instance ResponderNeeds : stakeholder:Concern [
    stakeholder:isExpressedBy stakeholders:Responder
]
```
This reads: "ResponderNeeds (a Concern) is expressed by Responder (a Stakeholder)"

**Equivalent forward direction** (if you were defining the Stakeholder instance):
```oml
instance Responder : stakeholder:Stakeholder [
    stakeholder:expresses concerns:ResponderNeeds
]
```

### Step 5: Locate or Verify Description OML File

**Path pattern**: `src/oml/<org>/<project>/{domain}/{concept-plural}.oml`

Examples:
- `src/oml/.../context-analysis/stakeholders.oml`
- `src/oml/.../context-analysis/concerns.oml`
- `src/oml/.../operational-analysis/entities.oml`

**Read the file** to understand:
- Imported vocabularies (`uses <...> as vocab`)
- Extended descriptions (`extends <...> as otherDesc`)
- Applicable OML syntax scope for the requested axiom
- Naming conventions

### Step 6: Verify Referenced Instances Exist

If the instance has relations to other instances, **verify they exist first**.

Example: Creating a Concern that `isExpressedBy stakeholders:Responder`
→ Must verify `Responder` instance exists in `stakeholders.oml`

**Check the extended descriptions:**
```oml
extends <https://<org>/.../stakeholders#> as stakeholders
```
This allows referencing `stakeholders:Responder`

### Step 7: Create the Instance

**Template:**
```oml
instance {InstanceName} : {vocabulary}:{ConceptName} [
    {vocab}:{property1} "{value1}"
    {vocab}:{property2} "{value2}"
    {vocab}:{relation} {otherDesc}:{otherInstance}
]
```

**Example:**
```oml
instance ResponderNeeds : stakeholder:Concern [
    base:description "Rapid situational awareness and robust field usability."
    base:priority "Medium"
    base:category "Need"
    stakeholder:isExpressedBy stakeholders:Responder
]
```

**Naming conventions:**
- UpperCamelCase for instance names
- Descriptive, specific names (not generic like "Instance1")
- Often concatenates related entities (e.g., `ResponderNeeds`)

## Description Axiom Coverage (Spec-Driven)

This section provides at least one example for each key description-level axiom/syntax element so the workflow remains reproducible even when there are no local examples.

### 1. Description Declaration

```oml
description <http://com.xyz/system/components#> as components {
    uses <http://com.xyz/methodology/mission#> as mission
}
```

### 2. Description Import - Extension

```oml
description <http://com.xyz/system/components#> as components {
    extends <http://com.xyz/system/common#> as common
    uses <http://com.xyz/methodology/mission#> as mission
}
```

### 3. Description Import - Usage

```oml
description <http://com.xyz/system/components#> as components {
    uses <http://com.xyz/methodology/mission#> as mission
}
```

### 4. Concept Instance (Named Instance)

```oml
description <http://com.xyz/system/components#> as components {
    uses <http://com.xyz/methodology/mission#> as mission

    instance component1 : mission:Component
}
```

### 5. Concept Type Assertion (for Concept Instance)

```oml
description <http://com.xyz/system/components#> as components {
    uses <http://com.xyz/methodology/mission#> as mission
    uses <http://com.xyz/methodology/base#> as base

    instance component1 : mission:Component, base:Element
}
```

### 6. Relation Instance (Named Instance)

```oml
description <http://com.xyz/system/components#> as components {
    uses <http://com.xyz/methodology/mission#> as mission

    instance component1 : mission:Component
    instance function1 : mission:Function

    relation instance performs1 : mission:Performs [
        from component1
        to function1
    ]
}
```

### 7. Relation Type Assertion (for Relation Instance)

```oml
description <http://com.xyz/system/components#> as components {
    uses <http://com.xyz/methodology/mission#> as mission

    instance component1 : mission:Component
    instance function1 : mission:Function

    relation instance performs1 : mission:Performs [
        from component1
        to function1
    ]
}
```

### 8. Property Value Assertion - Scalar Property

```oml
description <http://com.xyz/system/components#> as components {
    uses <http://com.xyz/methodology/mission#> as mission
    uses <http://com.xyz/methodology/base#> as base

    instance component1 : mission:Component [
        base:description "Main system component"
    ]
}
```

### 9. Property Value Assertion - Semantic Property

```oml
description <http://com.xyz/system/concerns#> as concerns {
    uses <http://com.xyz/methodology/stakeholder#> as stakeholder
    extends <http://com.xyz/system/stakeholders#> as stakeholders

    instance CoordinatorNeeds : stakeholder:Concern [
        stakeholder:isExpressedBy stakeholders:Operator
    ]
}
```

### 10. Anonymous Concept Instance

```oml
description <http://com.xyz/methodology/functions#> as functions {
    uses <http://com.xyz/methodology/mission#> as mission

    instance C1 : mission:Component [
        mission:hasPin : mission:Pin [
            mission:hasNumber 2
        ]
    ]
}
```

### 11. Anonymous Relation Instance

```oml
description <http://com.xyz/system/components#> as components {
    extends <http://com.xyz/system/functions#> as functions
    uses <http://com.xyz/methodology/mission#> as mission

    instance C1 : mission:Component [
        mission:performs functions:function2 [
            mission:priority 2
        ]
    ]
}
```

### 12. Relation Instance Endpoints (from/to)

```oml
description <http://com.xyz/system/components#> as components {
    uses <http://com.xyz/methodology/mission#> as mission

    instance component1 : mission:Component
    instance function1 : mission:Function
    instance function2 : mission:Function

    relation instance performs1 : mission:Performs [
        from component1
        to function1, function2
    ]
}
```

### Usage Rule for This Skill

For description work, always map generated content to one or more examples above before finalizing edits.
- If creating named instances: validate against examples 4-9 and 12.
- If creating nested value instances: validate against examples 10-11.
- If importing external ontologies/descriptions: validate against examples 2-3.

## 🛡️ Safe File Editing Techniques

**⚠️ CRITICAL**: When adding instances to description OML files, use techniques that prevent unintended restoration of user-deleted content.

### The Problem: "Resurrection" of Deleted Instances

**Scenario**: User deletes instances from file → Agent reads file (before save completes) → Agent's edit includes deleted instances in the replacement string → Deleted instances "resurrected"

**Root cause**: File editing tools require matching existing content. If the match string includes content the user just deleted (but file hasn't saved yet), the replacement will restore it.

### Solution: Append-Only Edit Pattern

**For adding new instances to the end of a description file**, use the **closing brace match pattern**:

**❌ PROBLEMATIC (includes existing instance):**
```oml
oldString:
    instance ExistingInstance : vocab:Type [
        base:description "..."
    ]
}

newString:
    instance ExistingInstance : vocab:Type [
        base:description "..."
    ]

    instance NewInstance : vocab:Type [
        base:description "New instance"
    ]
}
```
→ **Risk**: If user deleted `ExistingInstance`, this edit restores it.

**✅ SAFE (only matches closing brace):**
```oml
oldString:
}

newString:
    instance NewInstance : vocab:Type [
        base:description "New instance"
    ]
}
```
→ **Safe**: Only adds new content, never restores deletions.

### Implementation Guidelines

**When to use append-only pattern:**
- Adding new instances to end of file
- User is actively editing the file (potential unsaved changes)
- Single instance additions

**Requirements:**
1. **Verify proper formatting**: Ensure previous instance has closing `]` and proper newlines
2. **Include proper indentation**: Match file's indentation style (typically 4 spaces)
3. **Add blank line before new instance**: Visual separation between instances

**Example: Safe stakeholder addition**
```oml
oldString:
}

newString:

    instance NewStakeholder : stakeholder:Stakeholder [
        base:description "Description of new stakeholder."
        base:category "User"
    ]
}
```

**Example: Safe concern addition with relation**
```oml
oldString:
}

newString:

    instance NewConcern : stakeholder:Concern [
        base:description "Description of concern."
        base:priority "High"
        base:category "Need"
        stakeholder:isExpressedBy stakeholders:ExistingStakeholder
    ]
}
```

### When Append-Only Pattern May Not Work

**Situation**: File doesn't end with standalone `}` (e.g., has comments after it)

**Solution**: Include 1-2 lines of context before the `}`:
```oml
oldString:
    ]
}

newString:
    ]

    instance NewInstance : vocab:Type [
        base:description "..."
    ]
}
```

### Edge Cases

**Multiple files with relations:**
When creating two instances in different files that reference each other:

1. Create the referenced instance first (e.g., Stakeholder)
2. Then create the referencing instance (e.g., Concern)
3. Use append-only pattern for both

**Example workflow:**
```
Step 1: Add Stakeholder to stakeholders.oml (append-only)
Step 2: Add Concern to concerns.oml (append-only, references new Stakeholder)
```

### Verification After Edit

After using append-only pattern:
1. **Read the file** to confirm new instance added correctly
2. **Check no duplicates** created
3. **Verify closing brace** is properly placed
4. **Validate syntax** with OML validator

## Common Patterns

### Pattern 1: Simple Instance (Only Scalar Properties)

**SHACL:**
```turtle
sh:targetClass stakeholder:Stakeholder ;
sh:property [
    sh:path base:category ;
] ;
sh:property [
    sh:path base:description ;
] ;
```

**Instance:**
```oml
instance Responder : stakeholder:Stakeholder [
    base:description "Frontline operator fighting fires."
    base:category "Beneficiary"
]
```

### Pattern 2: Instance with One Relation

**SHACL:**
```turtle
sh:targetClass stakeholder:Concern ;
sh:property [
    sh:path stakeholder:isExpressedBy ;
    sh:class stakeholder:Stakeholder ;
    sh:minCount 1 ;
] ;
```

**Instance:**
```oml
instance CoordinatorNeeds : stakeholder:Concern [
    base:description "Clear system feedback."
    stakeholder:isExpressedBy stakeholders:Operator
]
```

### Pattern 3: Instance with Multiple Relations to Same Property

**SHACL:**
```turtle
sh:property [
    sh:path scenario:lifelineOf ;
    sh:class scenario:Scenario ;
    sh:minCount 1 ;
    sh:maxCount 1 ;
] ;
sh:property [
    sh:path scenario:represents ;
    sh:class entity:Entity ;
    sh:minCount 1 ;
    sh:maxCount 1 ;
] ;
```

**Instance:**
```oml
instance FireControllerLifeline : scenario:Lifeline [
    base:description "Represents the Fire Controller entity in the scenario."
    scenario:lifelineOf scenarios:ContainmentScenario
    scenario:represents entities:FireController
]
```

### Pattern 4: Multiple Values for Same Property

**SHACL** (no sh:maxCount means multiple allowed):
```turtle
sh:property [
    sh:path stakeholder:isExpressedBy ;
    sh:class stakeholder:Stakeholder ;
    sh:minCount 1 ;
]
```

**Instance:**
```oml
instance SharedConcern : stakeholder:Concern [
    base:description "Shared concern among multiple stakeholders."
    stakeholder:isExpressedBy stakeholders:Responder
    stakeholder:isExpressedBy stakeholders:Operator
    stakeholder:isExpressedBy stakeholders:OperationsCenter
]
```

## Troubleshooting

### Error: "Cannot resolve reference to {vocab}:{Instance}"

**Cause**: Referenced instance doesn't exist or description not imported.

**Fix:**
1. Verify instance exists in the referenced description file
2. Ensure description is imported with `extends <IRI> as prefix`
3. Use correct prefix (matches the `as` clause)

### Error: "Type mismatch: expected {Type1}, got {Type2}"

**Cause**: Wrong target class for relation.

**Fix:**
1. Check SHACL `sh:class` to confirm expected type
2. Verify instance has correct type declaration
3. Example: `sh:class stakeholder:Stakeholder` requires target to be a Stakeholder instance

### Error: "Missing required property {property}"

**Cause**: SHACL has `sh:minCount 1` but instance lacks the property.

**Fix:**
1. Check SHACL for all `sh:minCount ≥ 1` properties
2. Add missing properties to instance definition

### Wrong Relation Direction

**Symptom**: Instance validates but semantics are wrong (e.g., "Stakeholder is expressed by Concern" instead of reverse).

**Fix:**
1. Check vocabulary definition for `from`, `to`, `forward`, `reverse`
2. Use correct direction based on instance type
   - If instance is `from` type, use forward relation
   - If instance is `to` type, use reverse relation

### Arrows Missing in Dashboard Graphs

**Symptom**: Instances created successfully, validation passes, nodes appear in graph, but no connecting arrows/edges appear between related instances.

**Cause**: Dashboard SPARQL queries may be looking for wrong relation direction.

**Fix:**
1. Check what relation direction is used in the actual OML data (e.g., `grep` for the relation)
2. Open the dashboard.md file and check the SPARQL WHERE clause
3. Ensure the SPARQL query uses the same relation direction as the data
4. Example: If data uses `?concern stakeholder:isExpressedBy ?stakeholder`, the query should match that, not `?stakeholder stakeholder:expresses ?concern`
5. Even if vocabulary marks relation as `symmetric`, SPARQL may not automatically infer both directions

**Pattern to check:**
- Data in concerns.oml: `stakeholder:isExpressedBy stakeholders:Responder` (reverse)
- SPARQL should query: `?concern stakeholder:isExpressedBy ?stakeholder` (reverse)
- SPARQL can construct: `?stakeholder stakeholder:expresses ?concern` (forward for display)

### Table Editor Shows Stale Data After Edits

**Symptom**: After creating instances programmatically, the table editor in .md files shows old/deleted instances.

**Cause**: Table editor caches data and may not immediately pick up file changes made outside the UI.

**Fix:**
1. Accept all pending changes (click "Keep" buttons)
2. Close VS Code completely (not just reload window)
3. Reopen VS Code
4. The table should now show correct data

**Note**: Dashboards and graphs read directly from OML files and show correct data immediately. Only the table editor UI has caching issues.

**⚠️ CRITICAL WORKFLOW WARNING**: 
After creating instances programmatically via this skill, **DO NOT use the table editor UI to make further edits** until you have:
1. Accepted all changes
2. Closed VS Code completely
3. Reopened VS Code

The table editor can have persistent cache across multiple sessions. If you edit via the UI without restarting first, you may see or inadvertently save old deleted instances. Always restart VS Code before mixing programmatic edits with UI edits.

## Quick Reference

### File Locations
| Type | Base Path | Example |
|------|-----------|---------|
| Vocabulary | `build/oml/www.modelware.io/sierra/` | `stakeholder.oml` |
| SHACL Tables | `src/md/<org>/<project>/` | `context-analysis/concerns.md` |
| Descriptions | `src/oml/<org>/<project>/` | `context-analysis/concerns.oml` |

### Common Vocabularies
| Prefix | IRI | Defines |
|--------|-----|---------|
| `base` | `https://www.modelware.io/sierra/base#` | Element, Container, category, description, priority |
| `stakeholder` | `https://www.modelware.io/sierra/stakeholder#` | Stakeholder, Concern, Requirement |
| `entity` | `https://www.modelware.io/sierra/entity#` | Entity, Actor, Agent |
| `process` | `https://www.modelware.io/sierra/process#` | Activity, Function, Exchange |
| `scenario` | `https://www.modelware.io/sierra/scenario#` | Scenario, Lifeline, TimePoint |
| `component` | `https://www.modelware.io/sierra/component#` | Component |

### SHACL to Instance Mapping
| SHACL Element | Meaning | Instance Syntax |
|---------------|---------|-----------------|
| `sh:targetClass vocab:Type` | Type of instance | `instance Name : vocab:Type [...]` |
| `sh:path base:description` | Scalar property | `base:description "text"` |
| `sh:path vocab:relation` with `sh:class` | Object property | `vocab:relation other:Instance` |
| `sh:minCount 1` | Required property | Must include in instance |
| `sh:maxCount 1` | Single value | Use property once |
| No `sh:maxCount` | Multiple values | Can use property multiple times |

## Anti-Patterns

❌ **Creating instances without checking SHACL** - always verify required properties
❌ **Using dot separator** - always use `:` (colon) for vocabulary references
❌ **Wrong relation direction** - always check vocabulary definition
❌ **Referencing non-existent instances** - verify target instances exist
❌ **Missing imports** - ensure all vocabularies and descriptions are imported
❌ **Generic instance names** - use descriptive, specific names
❌ **Including existing instances in edit strings** - use append-only pattern to avoid resurrecting deleted content

## Summary Checklist

For each instance creation:
1. ✅ Identify target concept from request
2. ✅ Read SHACL table from MD file
3. ✅ Extract targetClass, properties, relations, target classes
4. ✅ Consult [SHACL reference](./references/shacl-requirements.md) for required constraints and optional guidance
5. ✅ Add optional properties only if requested or policy-backed
6. ✅ Check vocabulary for relation definitions and directions
7. ✅ Verify referenced instances exist
8. ✅ Create instance with all REQUIRED properties (optional properties only when justified)
9. ✅ **Use append-only edit pattern** (match closing brace only) to prevent resurrection of deleted instances
10. ✅ Validate no errors in file
11. ✅ If dashboard arrows don't appear, verify SPARQL queries match actual relation direction in data

