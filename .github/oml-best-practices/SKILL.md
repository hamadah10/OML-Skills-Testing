---
name: oml-best-practices
description: OML best practices for naming conventions, namespace structure, import management, documentation, restriction guidelines, and rule design. Use when user asks about "OML naming conventions", "OML style guide", "OML project structure", "how to organize OML files", "OML documentation standards", or needs guidance on professional OML development.
---

# OML Best Practices

Professional standards for OML development.

## Normative Source Policy

Apply this order for any recommendation in this skill:
1. OML language rules (Textual BNF / Language Reference)
2. Vocabulary semantics (axioms, domain/range, characteristics)
3. SHACL constraints (if present)
4. Team conventions (style only)

This skill must not treat existing workspace files as authority.
When documenting an axiom, keep one canonical example maximum.

> **🚨 CRITICAL PREREQUISITE**: Before writing ANY OML code, you MUST read the `oml-validation` skill (`.github/oml-validation/SKILL.md`) to verify syntax rules.
>
> **Most Common Error**: Using `:>` for inheritance instead of `<`
> - ✅ CORRECT: `concept Child < Parent`
> - ❌ WRONG: `concept Child :> Parent`

## 1. Naming Conventions

### Concepts, Aspects, Relation Entities, Scalars
**UpperCamelCase** (like classes in programming):

```oml
concept Component
aspect IdentifiedElement
relation entity Performs
scalar TenCharString
```

### Properties, Relations, Instances
**lowerCamelCase** (like variables/methods):

```oml
scalar property hasId
relation performs
instance powerSubsystem
```

### Namespace Prefixes
**Short, memorable, lowercase**:

```oml
vocabulary <http://example.com/mission#> as mission {
  extends <http://www.w3.org/2001/XMLSchema#> as xsd
  extends <http://example.com/base#> as base
}
```

**Common prefixes**:
- `xsd` - XML Schema datatypes
- `rdf` - RDF
- `rdfs` - RDF Schema
- `owl` - OWL
- `dc` - Dublin Core
- Custom: `mission`, `analysis`, `base`, `project`

### Relations: Use Verb Phrases

```oml
// Good
relation hasSubcomponent
relation performs
relation isPerformedBy
relation contains
relation dependsOn

// Avoid
relation subcomponent  // Missing verb
relation performance   // Noun instead of verb
```

### Boolean Properties: Use is/has Prefix

```oml
scalar property isAbstract
scalar property isActive
scalar property hasChildren
scalar property isComplete
```

## 2. Namespace Structure

### Hierarchical IRIs

Use domain-based hierarchical structure:

```oml
http://<domain>/<category>/<subcategory>#

// Examples
http://com.xyz/methodology/mission#
http://com.xyz/methodology/analysis#
http://com.xyz/missions/europa/components#
http://imce.jpl.nasa.gov/foundation/base#
```

### Match File Paths to IRIs

**IRI**: `http://com.xyz/methodology/mission#`  
**Path**: `src/oml/com.xyz/methodology/mission.oml`

**IRI**: `http://example.com/projects/apollo/systems#`  
**Path**: `src/oml/example.com/projects/apollo/systems.oml`

### Use Hash (#) for Namespace Separator

```oml
// Preferred
http://example.com/mission#Component

// Avoid
http://example.com/mission/Component
```

## 3. Import Management

### Import Only What You Reference

```oml
// Good: imports only used vocabularies
vocabulary <http://example.com/analysis#> as analysis {
  extends <http://example.com/mission#> as mission
  extends <http://www.w3.org/2001/XMLSchema#> as xsd
  
  concept FunctionalComponent < mission:Component
  
  scalar property hasConfidence [
    range xsd:decimal
  ]
}
```

### Define Clear Hierarchies

```oml
// Layer approach
base.oml          // Foundation
  ↓
mission.oml       // Domain concepts
  ↓
analysis.oml      // Specialized analysis
  ↓
validation.oml    // Project-specific rules
```

### Avoid Circular Dependencies

```oml
// WRONG: A imports B, B imports A
// Creates infinite loop

// RIGHT: One-way dependency chain
base → mission → analysis
```

### Use Specific Imports

```oml
// Prefer extends/uses with direct dependencies
vocabulary <http://example.com/specialized#> as specialized {
  extends <http://example.com/base#> as base
  // Don't rely on transitive imports
}
```

## 4. Documentation Standards

### Annotate All Major Elements

**Ontologies** - Use Dublin Core:
```oml
@dc:title "Mission Domain Vocabulary"
@dc:description "Defines spacecraft mission concepts and relationships"
@dc:creator "Systems Engineering Team"
@dc:date "2024-11-18"^^xsd:dateTime
vocabulary <http://example.com/mission#> as mission {
  // ...
}
```

**Concepts, Relations, Properties** - Use rdfs:comment:
```oml
@rdfs:comment "A component that performs functions within a system"
concept Component [
  key hasId
]

@rdfs:comment "Unique identifier for components"
scalar property hasId [
  domain Component
  range xsd:string
  functional
]
```

**Instances**:
```oml
@rdfs:comment "Main spacecraft bus for Europa mission"
instance Spacecraft : mission:Assembly [
  mission:hasId "SC-001"
]
```

### Document Design Rationale

```oml
@rdfs:comment "Uses relation entity to capture performance confidence and priority"
relation entity Performs [
  from Component
  to Function
  forward performs
  reverse isPerformedBy
] [
  restricts hasPriority to exactly 1
  restricts hasConfidence to max 1
]
```

### Use Inline Comments for Sections

```oml
vocabulary <http://example.com/mission#> as mission {
  
  // ========== ASPECTS ==========
  aspect IdentifiedElement
  aspect Container
  
  // ========== CONCEPTS ==========
  concept Component < IdentifiedElement
  concept Assembly < Component
  
  // ========== RELATIONS ==========
  relation performs [
    from Component
    to Function
  ]
}
```

## 5. Restriction Guidelines

### Type Safety: Use `restricts all`

```oml
concept System [
  restricts all hasSubcomponent to Component
]
```

Ensures all subcomponents are Components (type safety).

### Existence: Use `restricts some`

```oml
concept FunctionalComponent [
  restricts some performs to Function
]
```

Requires at least one `performs` relation to a Function.

### Cardinality: Use min/max/exactly

```oml
concept Assembly [
  restricts hasSubcomponent to min 2      // At least 2
  restricts hasId to exactly 1            // Exactly 1
  restricts hasDescription to max 1       // At most 1
]
```

### Value Restrictions

```oml
concept TestableComponent [
  restricts hasId to TenCharString
  restricts hasMass to minInclusive 0.0
  restricts hasStatus to exactly "Active"
]
```

### When to Use Relation Entities

**Use relation entities when**:
- Relation needs properties (timestamps, priorities, confidence)
- Need to restrict relation properties
- Want to specialize relations

```oml
relation entity Performs [
  from Component
  to Function
  forward performs
  reverse isPerformedBy
] [
  restricts hasPriority to exactly 1
]
```

**Use simple relations when**:
- No additional properties needed
- Simpler, more direct modeling

```oml
relation hasSubcomponent [
  from Component
  to Component
]
```

## 6. Rule Design

### Keep Rules Simple and Focused

```oml
// Good: single purpose
rule ParentIsAncestor [
  parentOf(x, y) -> ancestorOf(x, y)
]

// Avoid: complex multi-step inference
```

### Document Rule Purpose

```oml
@rdfs:comment "Transitivity: if c1 performs f1 and f1 invokes f2, then c1 performs f2"
rule TransitivePerformance [
  Component(c) & performs(c, f1) & invokes(f1, f2)
  -> performs(c, f2)
]
```

### Test Rules Incrementally

1. Create rule
2. Test with simple instances
3. Verify inferred facts
4. Add complexity gradually

### Use Builtins for Calculations

```oml
rule TotalMass [
  hasMass(parent, m1) & 
  hasSubcomponent(parent, child) & 
  hasMass(child, m2) &
  builtIn(swrlb:add, total, m1, m2)
  -> hasMass(parent, total)
]
```

**Common builtins** (swrlb namespace):
- Math: `add`, `subtract`, `multiply`, `divide`
- Comparison: `greaterThan`, `lessThan`, `equal`
- String: `contains`, `startsWith`, `endsWith`, `matches`

## 7. File Organization

### Project Structure

```
project/
├── src/oml/
│   ├── <domain>/
│   │   ├── foundation/
│   │   │   ├── base.oml      # Core aspects/concepts
│   │   │   └── bundle.oml    # Foundation bundle
│   │   ├── methodology/
│   │   │   ├── mission.oml
│   │   │   ├── analysis.oml
│   │   │   └── bundle.oml
│   │   └── projects/
│   │       └── europa/
│   │           ├── components.oml
│   │           ├── functions.oml
│   │           └── bundle.oml
└── build/oml/                 # Build outputs
```

### Separate Vocabularies from Descriptions

```
vocabularies/    # Domain models (reusable)
  mission.oml
  analysis.oml
  
descriptions/    # System instances (project-specific)
  europa/
    components.oml
    functions.oml
```

### Bundle Related Ontologies

```oml
vocabulary bundle <http://example.com/methodology#> as methodology {
  includes <http://example.com/mission#>
  includes <http://example.com/analysis#>
  includes <http://example.com/validation#>
}
```

## 8. Version Control

### Meaningful IRI Versioning

```oml
// Versioned ontology
vocabulary <http://example.com/mission/1.0#> as mission {
  // ...
}

// Next version
vocabulary <http://example.com/mission/1.1#> as mission {
  extends <http://example.com/mission/1.0#> as mission-v1
  // ...
}
```

### Use Dublin Core Dates

```oml
@dc:date "2024-11-18"^^xsd:dateTime
@dc:modified "2024-12-01"^^xsd:dateTime
```

## Summary Checklist

✅ **Naming**:
- UpperCamelCase for types
- lowerCamelCase for properties/instances
- Verb phrases for relations
- Short namespace prefixes

✅ **Structure**:
- Hierarchical IRIs
- File paths match IRIs
- Clear import hierarchies

✅ **Documentation**:
- Dublin Core on ontologies
- rdfs:comment on major elements
- Design rationale in comments

✅ **Restrictions**:
- `restricts all` for type safety
- `restricts some` for existence
- Cardinality for counts
- Relation entities when needed

✅ **Rules**:
- Simple and focused
- Well-documented
- Incrementally tested
- Builtins for calculations

✅ **Organization**:
- Separate vocabularies from descriptions
- Bundle related ontologies
- Logical directory structure
