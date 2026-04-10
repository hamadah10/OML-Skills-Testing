---
name: oml-patterns
description: Rule-grounded OML modeling idioms for typed containment, bidirectional relations, functional properties, and property chains. Use when user asks about modeling approaches, but always derive from language rules and vocabulary semantics first.
---

# OML Common Patterns

Rule-grounded modeling idioms for OML.

> **🚨 REQUIRED FIRST STEP**: BEFORE implementing any pattern, you MUST consult the `oml-validation` skill to verify correct syntax. The #1 error is using wrong inheritance symbol - ALWAYS use `<` not `:>`.
>
> **Pre-Implementation Checklist**:
> 1. Read `.github/oml-validation/SKILL.md` for syntax rules
> 2. Verify inheritance uses `<` (NOT `:>`)
> 3. Confirm the target axiom in OML language rules and vocabulary semantics

## Normative Source Policy

Use this precedence order:
1. OML language rules (Textual BNF / Language Reference)
2. Vocabulary axioms and semantics
3. SHACL constraints (if present)
4. Style conventions (optional)

Do not infer correctness from existing workspace examples.
When introducing an axiom in guidance, keep one canonical example maximum.

## Pattern 1: Typed Containment

Ensure containers only hold specific types:

```oml
concept System [
  restricts all contains to Component
  restricts contains to min 1
]

relation contains [
  from System
  to Component
  asymmetric
  irreflexive
]
```

**When to use**: Parent-child relationships where you need type safety.

**Variations**:
```oml
// Homogeneous containment
concept Assembly < Component [
  restricts all hasSubcomponent to Component
]

// Heterogeneous containment with multiple relations
concept Spacecraft [
  restricts all hasPayload to Instrument
  restricts all hasSubsystem to Subsystem
]
```

## Pattern 2: Bidirectional Relations

Define symmetric or asymmetric bidirectional relationships:

```oml
// Symmetric (connects = isConnectedBy)
relation entity Connects [
  from Component
  to Component
  forward connects
  reverse isConnectedBy
  symmetric
]

// Asymmetric (performs ≠ isPerformedBy)
relation entity Performs [
  from Component
  to Function
  forward performs
  reverse isPerformedBy
  inverse functional
  asymmetric
  irreflexive
]
```

**When to use**: Need both directions of navigation in queries/rules.

**Characteristics guide**:
- `symmetric`: A connects B ⟹ B connects A
- `asymmetric`: A performs B ⟹ B does NOT perform A
- `functional`: Each A relates to at most one B
- `inverse functional`: Each B is related from at most one A
- `irreflexive`: A cannot relate to itself
- `transitive`: A relates B, B relates C ⟹ A relates C

## Pattern 3: Functional Properties with Keys

Unique identification using functional properties:

```oml
concept Component [
  key hasSerialNumber
]

scalar property hasSerialNumber [
  domain Component
  range xsd:string
  functional
]
```

**When to use**: Entities with unique identifiers.

**Composite keys**:
```oml
concept Version [
  key hasMajor, hasMinor, hasPatch
]

scalar property hasMajor [
  domain Version
  range xsd:integer
  functional
]

scalar property hasMinor [
  domain Version
  range xsd:integer
  functional
]

scalar property hasPatch [
  domain Version
  range xsd:integer
  functional
]
```

## Pattern 4: Property Chains via Rules

Derive transitive or composite relationships:

```oml
// Transitive closure
relation parentOf [
  from Person
  to Person
]

relation ancestorOf [
  from Person
  to Person
]

rule AncestorTransitivity [
  parentOf(x, y) & parentOf(y, z)
  -> ancestorOf(x, z)
]

rule ParentIsAncestor [
  parentOf(x, y)
  -> ancestorOf(x, y)
]
```

**Property composition**:
```oml
rule PerformanceInheritance [
  Component(c) & performs(c, f1) & invokes(f1, f2)
  -> performs(c, f2)
]
```

## Pattern 5: Reified Relations

Add properties to relationships:

```oml
// Without reification (simple)
relation performs [
  from Component
  to Function
]

// With reification (adds metadata)
relation entity Performs [
  from Component
  to Function
  forward performs
  reverse isPerformedBy
] [
  restricts hasPriority to exactly 1
  restricts hasConfidence to max 1
]

scalar property hasPriority [
  domain Performs
  range xsd:integer
  functional
]

scalar property hasConfidence [
  domain Performs
  range xsd:decimal
  functional
]
```

**When to use**: Relations need timestamps, priorities, confidence scores, or other metadata.

## Pattern 6: Enumerated Instances

Define closed sets of instances:

```oml
concept Color [
  oneOf Red, Green, Blue
]

instance Red : Color
instance Green : Color
instance Blue : Color
```

**When to use**: Fixed, well-known sets (statuses, categories, types).

```oml
concept Priority [
  oneOf Critical, High, Medium, Low
]

concept Status [
  oneOf Planned, InProgress, Completed, Cancelled
]
```

## Pattern 7: Equivalence Definitions

Define concepts by their characteristics:

```oml
// Defined by restriction
concept LeafComponent = Component [
  restricts hasSubcomponent to exactly 0
]

// Defined by existence
concept FunctionalComponent = Component [
  restricts some performs to Function
]

// Multiple restrictions
concept CriticalComponent = Component [
  restricts performs to min 3
  restricts all performs to HighPriorityFunction
]
```

**When to use**: Concepts that are fully characterized by properties/relations.

## Pattern 8: Aspect Mixins

Reusable capabilities via aspects:

```oml
aspect IdentifiedElement [
  key hasId
]

aspect Documentable [
  restricts hasDescription to max 1
]

aspect Versionable [
  key hasVersion
]

// Combine aspects
concept Component < IdentifiedElement, Documentable, Versionable [
  restricts all hasSubcomponent to Component
]
```

**When to use**: Cross-cutting concerns (identification, documentation, versioning).

## Pattern 9: Layered Restrictions

Progressive constraint refinement:

```oml
// Base layer: loose constraints
vocabulary <http://example.com/base#> as base {
  concept Component [
    key hasId
  ]
  
  relation hasSubcomponent [
    from Component
    to Component
  ]
}

// Refinement layer: add type safety
vocabulary <http://example.com/typed#> as typed {
  extends <http://example.com/base#> as base
  
  ref concept base:Component [
    restricts all base:hasSubcomponent to base:Component
  ]
}

// Validation layer: add cardinality
vocabulary <http://example.com/validated#> as validated {
  extends <http://example.com/typed#> as typed
  extends <http://example.com/base#> as base
  
  ref concept base:Component [
    restricts base:hasSubcomponent to min 1
  ]
}
```

## Pattern 10: Self-Restriction

Model recursive or self-referential structures:

```oml
concept SelfReliantSystem [
  restricts dependsOn to self
]

concept Container [
  restricts contains to self
]
```

## Pattern 11: Specialization Hierarchies

Type hierarchies with progressive refinement:

```oml
// Base types
aspect Element
concept Component < Element

// First level specialization
concept MechanicalComponent < Component
concept ElectricalComponent < Component
concept SoftwareComponent < Component

// Second level
concept Sensor < ElectricalComponent
concept Actuator < ElectricalComponent

// Cross-cutting with aspects
concept SmartSensor < Sensor, Programmable [
  restricts some hasSubcomponent to SoftwareComponent
]
```

## Pattern 12: Calculation Rules

Derive computed properties:

```oml
// Mass accumulation
rule TotalMass [
  hasMass(parent, m1) & 
  hasSubcomponent(parent, child) & 
  hasMass(child, m2) &
  builtIn(swrlb:add, total, m1, m2)
  -> hasMass(parent, total)
]

// Volume calculation
rule Volume [
  Component(c) & 
  hasLength(c, l) & 
  hasWidth(c, w) & 
  hasHeight(c, h) &
  builtIn(swrlb:multiply, temp, l, w) &
  builtIn(swrlb:multiply, vol, temp, h)
  -> hasVolume(c, vol)
]
```

## Pattern 13: Disjoint Hierarchies

Ensure mutual exclusivity:

```oml
concept PhysicalComponent < Component
concept VirtualComponent < Component

rule DisjointPhysicalVirtual [
  PhysicalComponent(x) & VirtualComponent(x)
  -> owl:Nothing(x)
]
```

## Anti-Patterns to Avoid

❌ **Non-functional keys**:
```oml
// WRONG: hasSubcomponent is not functional
concept Component [
  key hasSubcomponent  // ❌ Can have multiple subcomponents
]
```

❌ **Circular imports**:
```oml
// WRONG: vocab A extends B, B extends A
// Causes infinite loop
```

❌ **Overuse of equivalence**:
```oml
// WRONG: Using equivalence when specialization suffices
concept ElectricalComponent = Component [
  restricts some hasProperty to ElectricalProperty
]
// RIGHT: Just use specialization if no reasoning needed
concept ElectricalComponent < Component
```

❌ **Using relation entity names**:
```oml
// WRONG: Using Performs instead of performs
instance c1 : Component [
  mission:Performs fn1  // ❌ Wrong
]

// RIGHT: Use forward/reverse names
instance c1 : Component [
  mission:performs fn1  // ✓ Correct
]
```
