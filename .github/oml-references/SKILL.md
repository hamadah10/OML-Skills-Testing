---
name: oml-references
description: Guide for using OML references (ref keyword) to extend imported members without redefining them. Use when user asks about "extending imported OML members", "OML ref keyword", "adding axioms to imports", "referencing OML ontologies", or needs to enhance imported concepts, aspects, relations, or instances.
---

# OML References (ref)

Extend imported members by adding axioms without redefining them.

## Normative Source Policy

Use this precedence order:
1. OML language rules for `ref` declarations and axiom forms
2. Vocabulary constraints (import scope, term kind, relation semantics)
3. SHACL constraints (if applicable)
4. Project conventions (optional)

Do not infer validity from existing project files.
When documenting an axiom in this skill, keep one canonical example maximum.

> **🚨 CRITICAL**: Before implementing, consult `oml-validation` skill (`.github/oml-validation/SKILL.md`). 
> - ✅ Inheritance uses `<` (e.g., `ref concept base:Child < base:Parent`)
> - ❌ NEVER use `:>` or `extends`

## Quick Start

```oml
vocabulary <http://example.com/extended#> as extended {
  extends <http://example.com/base#> as base
  
  // Add key to imported concept
  ref concept base:Component [
    key hasSerialNumber
  ]
  
  // Add restriction to imported concept
  ref concept base:Component [
    restricts hasCost to min 1
  ]
}
```

## What are References?

References let you extend imported members by adding axioms. Use `ref` + member type + IRI.

**Key principle**: References only ADD axioms, never remove or modify existing ones.

## Usage by Member Type

### Concepts and Aspects

```oml
// Add specializations
ref concept base:Component < base:IdentifiedElement

// Add keys
ref aspect base:IdentifiedElement [
  key base:hasId
]

// Add restrictions
ref concept base:Component [
  restricts base:hasSubcomponent to min 1
  restricts all base:performs to base:Function
]

// Combine multiple axioms
ref concept base:Component < base:Container [
  key hasSerialNumber
  restricts hasMass to exactly 1
]
```

### Scalars

```oml
// Add patterns or constraints
ref scalar base:TenCharString [
  pattern "^[A-Z]{10}$"
]

// Add equivalence
ref scalar base:Percentage [
  minInclusive 0.0
  maxInclusive 100.0
]
```

### Properties

```oml
// Add additional domains
ref scalar property base:hasId [
  domain ExtendedComponent
]

// Add equivalence
ref scalar property base:hasMass = hasWeight
```

### Relations

```oml
// Add characteristics
ref relation base:hasSubcomponent [
  transitive
]

// Change from/to (adds them, doesn't replace)
ref relation base:connects [
  from ExtendedComponent
]
```

### Relation Entities

```oml
// Add keys or restrictions
ref relation entity base:Performs [
  key hasPriority, hasTimestamp
]

// Add restrictions
ref relation entity base:Performs [
  restricts hasPriority to exactly 1
]
```

### Rules

```oml
// Can only reference for documentation
@rdfs:comment "This extends base:TransitiveRule"
ref rule base:TransitiveRule
```

## References in Descriptions

Add properties or types to imported instances:

```oml
description <http://example.com/extended#> as extended {
  extends <http://example.com/base-desc#> as base
  
  // Add properties to imported instance
  ref instance base:Spacecraft [
    mission:hasSerialNumber "SC-001-EXT"
    mission:hasSubcomponent NewSensor
  ]
  
  // Add types
  ref instance base:PowerSystem : analysis:TestableComponent [
    mission:hasCost 1500000.0
  ]
  
  // Add properties to relation instance
  ref relation instance base:Performs1 [
    mission:hasTimestamp "2024-11-18T10:00:00Z"^^xsd:dateTime
  ]
}
```

## Limitations

**Cannot do**:
- Remove or modify existing axioms
- Add axioms to rules (documentation only)
- Reference members not directly imported

**Must**:
- Import the ontology containing the referenced member
- Use direct imports (not transitive)

## When to Use References

✅ **Use references when**:
- Extending imported vocabularies with project-specific constraints
- Adding keys to standardized concepts
- Enriching imported instances with additional properties
- Layering restrictions across vocabulary hierarchies

❌ **Don't use references when**:
- Creating new members (use normal definitions)
- You control the original ontology (edit it directly)
- You want to remove axioms (not possible)

## Multiple References Pattern

You can reference the same member multiple times:

```oml
// First reference adds specialization
ref concept base:Component < base:IdentifiedElement

// Second reference adds key
ref concept base:Component [
  key hasSerialNumber
]

// Third reference adds restrictions
ref concept base:Component [
  restricts hasMass to min 1
]
```

All axioms accumulate on the same member.

## Cross-Layer References

```oml
vocabulary <http://example.com/layer2#> as layer2 {
  extends <http://example.com/layer1#> as layer1
  extends <http://example.com/base#> as base
  
  // Can reference through layer1
  ref concept layer1:SpecialElement [
    restricts base:hasId to base:TenCharString
  ]
  
  // Or reference base directly
  ref aspect base:IdentifiedElement < Documentable
}
```

## Reference Axiom Examples (One per Axiom)

### Pattern: Progressive Restriction

```oml
// Base: defines concept loosely
concept Component [
  key hasId
]

// Layer 1: adds type constraints
ref concept base:Component [
  restricts all hasSubcomponent to Component
]

// Layer 2: adds cardinality
ref concept base:Component [
  restricts hasSubcomponent to min 1
]
```

### Pattern: Key Accumulation

```oml
// Base: single key
aspect IdentifiedElement [
  key hasId
]

// Extended: composite key
ref aspect base:IdentifiedElement [
  key hasVersion
]
// Now has keys: hasId, hasVersion
```

### Pattern: Instance Enrichment

```oml
// Base description
instance Spacecraft : mission:Component [
  mission:hasId "SC-001"
]

// Extended description adds details
ref instance base:Spacecraft [
  mission:hasMass 1500.0
  mission:hasSubcomponent PowerSystem
]
```
