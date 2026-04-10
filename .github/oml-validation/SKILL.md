---
name: oml-validation
description: |
  **🚨 CRITICAL - REQUIRED FOR ALL OML WORK 🚨**
  
  OML validation rules and syntax reference. MUST be consulted BEFORE creating or modifying ANY OML concepts, aspects, relations, or properties.
  
  **🔴 CRITICAL RULES**: 
  1. **INHERITANCE SYMBOL**: ALWAYS use `<` (e.g., `concept Child < Parent`) - NEVER `:>`, `extends`, or other symbols
  2. **VOCABULARY REFERENCE SEPARATOR**: ALWAYS use `:` (colon) (e.g., `base:Container`) - NEVER `.` (dot) like `base.Container`
  
  **⚠️ MANDATORY - Consult this skill BEFORE:**
  - Creating or modifying concepts, aspects, or relation entities
  - Adding inheritance/specialization relationships
  - Defining properties, relations, or restrictions
  - Any OML file editing
  
  **Search triggers**: "create concept", "add concept", "create aspect", "inherit from", "inherits from", "specializes", "extends", "OML", "create relation", "define property", "OML validation", "OML syntax", "OML errors"
---

# OML Validation and Common Errors

**🚨 PRE-FLIGHT CHECKLIST FOR ALL OML IMPLEMENTATIONS 🚨**

Before writing ANY OML code, verify:
- [ ] Inheritance uses `<` symbol (NOT `:>`)
- [ ] Vocabulary references use `:` separator (NOT `.`)
- [ ] All referenced vocabularies are imported with `extends`
- [ ] IRIs are valid URLs with `#` separator
- [ ] Naming follows conventions (UpperCamelCase for types, lowerCamelCase for properties)
- [ ] Relation characteristics are chosen intentionally (`functional`, `inverse functional`, `symmetric`, `asymmetric`, `reflexive`, `irreflexive`, `transitive`)
- [ ] When uncertain, resolve from OML language rules first (Textual BNF + Language Reference), then vocabulary constraints

After EVERY OML file change, run build validation before concluding work:
- [ ] Windows: `Set-Location "c:\Users\Hamad\Desktop\Work\Projects\kepler16b-example-main\kepler16b-example-main"; .\gradlew.bat build`
- [ ] macOS/Linux: `./gradlew build`
- [ ] If build fails, fix issues and rerun until build passes

## Normative Source Policy

Use this precedence order for all modeling decisions:
1. OML language rules (Textual BNF / Language Reference)
2. Vocabulary semantics (domain/range, relation characteristics, keys, restrictions)
3. SHACL constraints (if present)
4. Project conventions (optional, never normative)

Do not infer correctness from existing workspace examples. Existing files may contain project-specific style choices.
When documenting an axiom, keep one canonical example maximum.

---
## ⚠️ THE TOP 2 MOST COMMON ERRORS ⚠️

### ERROR #1: Wrong Vocabulary Reference Separator

**CRITICAL: OML uses `:` (colon) to reference imported vocabulary members, NOT `.` (dot)**

```oml
// ❌ WRONG - Do NOT use dot (.)
concept Component < base.Container, base.Element
relation entity Performs [
  from mission.Component
  to mission.Function
]
scalar property mass [
  domain base.Element
  range xsd.decimal
]

// ✓ CORRECT - Use colon (:)
concept Component < base:Container, base:Element
relation entity Performs [
  from mission:Component
  to mission:Function
]
scalar property mass [
  domain base:Element
  range xsd:decimal
]
```

**Applies to ALL vocabulary member references:**
- Inheritance: `concept Child < parent:ParentConcept`
- Domain/range: `domain vocab:Type`, `range vocab:Datatype`
- Relation endpoints: `from vocab:Source`, `to vocab:Target`
- Restrictions: `restricts vocab:property to ...`
- Referenced concepts: `ref concept vocab:Thing`

**Rule**: Always use `vocabularyPrefix:MemberName` (colon separator)

### ERROR #2: Wrong Inheritance Symbol

**CRITICAL: OML uses `<` for specialization/inheritance, NOT `:>`**

```oml
❌ WRONG: concept MyClass :> Parent    // DO NOT USE :>
✓ RIGHT: concept MyClass < Parent      // ALWAYS USE <
```

**When creating ANY concept, aspect, or relation with inheritance:**
1. ✅ Use `<` for inheritance
2. ❌ NEVER use `:>`, `extends`, `subclassOf`, or any other symbol
3. 🔍 When uncertain, consult OML language rules (Textual BNF / Language Reference)

---

## Common Syntax Mistakes

### 1. Wrong Vocabulary Reference Separator

**CRITICAL: OML uses `:` (colon) to reference imported vocabulary members, NOT `.` (dot)**

```oml
// ❌ WRONG - Do NOT use dot (.)
concept Component < base.Container, base.Contained
scalar property weight [
  domain mission.Component
  range xsd.decimal
]
relation entity Performs [
  from actor.Agent
  to mission.Function
]

// ✓ CORRECT - Use colon (:)
concept Component < base:Container, base:Contained
scalar property weight [
  domain mission:Component
  range xsd:decimal
]
relation entity Performs [
  from actor:Agent
  to mission:Function
]
```

**This applies to ALL contexts where you reference an imported vocabulary member:**
- Inheritance/specialization: `concept X < vocab:Parent`
- Property domain/range: `domain vocab:Type`, `range vocab:Datatype`
- Relation endpoints: `from vocab:SourceType`, `to vocab:TargetType`
- Property restrictions: `restricts vocab:propertyName to ...`
- Concept references: `ref concept vocab:ConceptName`
- Instance types: `vocab:instanceName : vocab:TypeName`

**Remember**: The pattern is always `vocabularyPrefix:MemberName` where the prefix matches what you declared in the `extends` statement (e.g., `extends <...> as base` means you reference members as `base:MemberName`)

### 2. Wrong Inheritance Symbol

**CRITICAL: OML uses `<` for specialization/inheritance, NOT `:>`**

```oml
// ❌ WRONG - Do NOT use :>
concept Book :> Publication
aspect Serializable :> Storable
concept ElectricCar :> Vehicle, Electric

// ✓ CORRECT - Use <
concept Book < Publication
aspect Serializable < Storable  
concept ElectricCar < Vehicle, Electric

// More examples with correct syntax
concept Professor < Person, Employee        // ✓ Multiple inheritance
concept GraduateStudent < Student          // ✓ Single inheritance
aspect Timestamped < Traceable             // ✓ Aspect specialization
```

**Note**: The `:>` symbol is used in other ontology languages (like OWL Manchester Syntax) but is **NOT valid OML syntax**.

**Critical Rule**: ALL OML specialization uses `<`, regardless of type:
- Concepts: `concept Child < Parent`
- Aspects: `aspect SubAspect < SuperAspect`
- Relation entities: `relation entity Teaches < Interaction`
- Scalars: `scalar PositiveInteger < xsd:integer`

## Validation Rules

### 1. IRI Validity

**All IRIs must be valid URLs**:

```oml
// Valid
vocabulary <http://example.com/mission#> as mission
vocabulary <https://example.com/mission#> as mission
vocabulary <http://com.xyz/methodology/mission#> as mission

// Invalid
vocabulary <example.com/mission#> as mission          // ❌ Missing protocol
vocabulary <http://example.com/mission> as mission    // ❌ Missing # or / separator
vocabulary <http://example com/mission#> as mission   // ❌ Whitespace in IRI
```

**No special characters in IDs**:
```oml
// Valid
concept Component
concept Component_Type
concept Component123

// Invalid  
concept My Component      // ❌ Whitespace
concept Component/Type    // ❌ Path separator
concept Component#Type    // ❌ Hash in ID
```

**Use ^ for keyword IDs**:
```oml
// OML keywords need ^ escape
concept ^Concept    // Escape "Concept" keyword
aspect ^Aspect      // Escape "Aspect" keyword
```

### 2. Import Requirements

**Referenced members must be imported**:

```oml
// WRONG: Using mission:Component without import
vocabulary <http://example.com/analysis#> as analysis {
  concept FunctionalComponent < mission:Component  // ❌ mission not imported
}

// RIGHT: Import before use
vocabulary <http://example.com/analysis#> as analysis {
  extends <http://example.com/mission#> as mission
  
  concept FunctionalComponent < mission:Component  // ✓ mission imported
}
```

**Direct imports required for references**:

```oml
// WRONG: Referencing transitively imported member
vocabulary A {
  extends B as b  // B extends C
  
  ref concept c:Component [  // ❌ C not directly imported
    key hasId
  ]
}

// RIGHT: Import C directly
vocabulary A {
  extends B as b
  extends C as c  // Now C is directly imported
  
  ref concept c:Component [  // ✓ C directly imported
    key hasId
  ]
}
```

### 3. Restriction Rules

**Restriction ranges must be subtypes of property ranges**:

```oml
scalar property hasValue [
  domain Component
  range xsd:decimal
]

// WRONG: Restricting to incompatible type
concept SpecialComponent [
  restricts hasValue to xsd:string  // ❌ string not subtype of decimal
]

// RIGHT: Restrict to subtype or same type
concept PositiveComponent [
  restricts hasValue to minInclusive 0.0  // ✓ Constraining decimal
]
```

**Cannot restrict properties not applicable to concept**:

```oml
scalar property hasMass [
  domain Component
  range xsd:decimal
]

// WRONG: Function doesn't have hasMass
concept Function [
  restricts hasMass to min 1  // ❌ hasMass domain is Component, not Function
]

// RIGHT: Only restrict applicable properties
concept Component [
  restricts hasMass to min 1  // ✓ hasMass domain includes Component
]
```

### 4. Key Constraints

**Keys must use functional properties**:

```oml
// Functional property - valid for keys
scalar property hasId [
  domain Component
  range xsd:string
  functional  // Each component has at most 1 ID
]

concept Component [
  key hasId  // ✓ Functional property
]

// Non-functional property - invalid for keys
relation hasSubcomponent [
  from Component
  to Component
  // NOT functional - one component can have many subcomponents
]

concept Component [
  key hasSubcomponent  // ❌ Non-functional property
]
```

**Key properties must apply to the concept**:

```oml
scalar property hasSerialNumber [
  domain ElectricalComponent
  range xsd:string
  functional
]

// WRONG: MechanicalComponent can't use ElectricalComponent property as key
concept MechanicalComponent [
  key hasSerialNumber  // ❌ Domain mismatch
]

// RIGHT: Either extend domain or use appropriate property
scalar property hasSerialNumber [
  domain Component  // Now applies to all components
  range xsd:string
  functional
]

concept MechanicalComponent < Component [
  key hasSerialNumber  // ✓ Now valid
]
```

### 5. Relation Entity Requirements

**Must specify both `from` and `to`**:

```oml
// WRONG: Missing to
relation entity Performs [
  from Component
  forward performs
]  // ❌ No 'to' specified

// RIGHT: Both from and to
relation entity Performs [
  from Component
  to Function
  forward performs
  reverse isPerformedBy
]  // ✓ Complete
```

**Forward/reverse names required for usage**:

```oml
relation entity Performs [
  from Component
  to Function
  forward performs
  reverse isPerformedBy
]

// WRONG: Using relation entity name
instance c1 : Component [
  mission:Performs fn1  // ❌ Use forward/reverse names
]

// RIGHT: Use forward name
instance c1 : Component [
  mission:performs fn1  // ✓ Use 'performs' not 'Performs'
]
```

### 5.1 Relation Characteristics Reference (Vocabulary Modeling)

Use these keywords inside `relation entity` blocks to encode formal semantics.

```oml
relation entity RelatedTo [
  from Element
  to Element
  forward relatesTo
  reverse isRelatedToBy
  // optional characteristics go here
]
```

**`functional`**
- Meaning: each source instance can point to at most one target.
- Use when: a source must have a single value (e.g., one primary owner).

```oml
relation entity HasPrimaryOwner [
  from Asset
  to Person
  forward hasPrimaryOwner
  reverse isPrimaryOwnerOf
  functional
]
```

**`inverse functional`**
- Meaning: each target instance can be pointed to by at most one source.
- Use when: target is unique to one source (e.g., one serial number holder).

```oml
relation entity AssignedSerial [
  from Device
  to SerialRecord
  forward hasSerialRecord
  reverse isSerialRecordOf
  inverse functional
]
```

**`symmetric`**
- Meaning: if `A R B`, then `B R A`.
- Use when: relation is inherently mutual (peer links, adjacency).

```oml
relation entity ConnectedTo [
  from Node
  to Node
  forward connectsTo
  reverse isConnectedToBy
  symmetric
]
```

**`asymmetric`**
- Meaning: if `A R B`, then `B R A` must not hold.
- Use when: relation is directional by definition (parent/child, supervises).

```oml
relation entity Supervises [
  from Person
  to Person
  forward supervises
  reverse isSupervisedBy
  asymmetric
]
```

**`reflexive`**
- Meaning: every instance in scope relates to itself.
- Use when: self-membership/self-reachability is intended.

```oml
relation entity EquivalentScope [
  from Scope
  to Scope
  forward equivalentScope
  reverse equivalentScope
  reflexive
]
```

**`irreflexive`**
- Meaning: no instance can relate to itself.
- Use when: self-links are invalid by model meaning.

```oml
relation entity DependsOn [
  from Component
  to Component
  forward dependsOn
  reverse isDependencyOf
  irreflexive
]
```

**`transitive`**
- Meaning: if `A R B` and `B R C`, then `A R C`.
- Use when: closure across chains is semantically correct.

```oml
relation entity AncestorOf [
  from Person
  to Person
  forward ancestorOf
  reverse descendantOf
  transitive
]
```

**Compatibility and modeling guards**
- Do not combine `symmetric` with `asymmetric`.
- Do not combine `reflexive` with `irreflexive`.
- `asymmetric` usually implies `irreflexive`; model both only when you want strict explicitness.
- `functional` + `inverse functional` means one-to-one matching.
- Add these characteristics only when they are true in domain semantics, not for convenience.

### 6. Standard Scalar Dependencies

**xsd:*, owl:*, rdf:* require imports**:

```oml
// WRONG: Using xsd:string without import
vocabulary <http://example.com/mission#> as mission {
  scalar property hasId [
    range xsd:string  // ❌ xsd not imported
  ]
}

// RIGHT: Import XSD
vocabulary <http://example.com/mission#> as mission {
  extends <http://www.w3.org/2001/XMLSchema#> as xsd
  
  scalar property hasId [
    range xsd:string  // ✓ xsd imported
  ]
}
```

## Common Errors

### Error 1: Circular Dependencies

**Problem**:
```oml
// vocabulary A.oml
vocabulary <http://example.com/A#> as A {
  extends <http://example.com/B#> as B
}

// vocabulary B.oml  
vocabulary <http://example.com/B#> as B {
  extends <http://example.com/A#> as A  // ❌ Circular: A→B, B→A
}
```

**Solution**: Break the cycle by introducing a base ontology:
```oml
// base.oml
vocabulary <http://example.com/base#> as base {
  concept CommonElement
}

// A.oml
vocabulary <http://example.com/A#> as A {
  extends <http://example.com/base#> as base
}

// B.oml
vocabulary <http://example.com/B#> as B {
  extends <http://example.com/base#> as base
  extends <http://example.com/A#> as A  // ✓ No cycle
}
```

### Error 2: Missing Imports

**Problem**:
```oml
vocabulary <http://example.com/analysis#> as analysis {
  // Forgot to import mission
  concept FunctionalComponent < mission:Component  // ❌ Undefined: mission
}
```

**Solution**: Add import:
```oml
vocabulary <http://example.com/analysis#> as analysis {
  extends <http://example.com/mission#> as mission  // ✓ Import mission
  
  concept FunctionalComponent < mission:Component
}
```

### Error 3: Invalid IRI Characters

**Problem**:
```oml
vocabulary <http://example.com/my mission#> as mission  // ❌ Whitespace
concept Component Type  // ❌ Whitespace
concept Component/Type  // ❌ Path separator
```

**Solution**: Use valid characters:
```oml
vocabulary <http://example.com/my-mission#> as mission  // ✓ Hyphen
concept ComponentType  // ✓ CamelCase
concept Component_Type  // ✓ Underscore
```

### Error 4: Inconsistent Separators

**Problem**:
```oml
vocabulary <http://example.com/mission#> as mission {
  concept Component
}

// Later referenced as:
extends <http://example.com/mission/> as mission  // ❌ Changed # to /
```

**Solution**: Be consistent with # or /:
```oml
// Define with #
vocabulary <http://example.com/mission#> as mission

// Reference with #
extends <http://example.com/mission#> as mission  // ✓ Consistent
```

### Error 5: Non-Functional Properties in Keys

**Problem**:
```oml
relation hasSubcomponent [
  from Component
  to Component
  // Not functional - one can have many
]

concept Component [
  key hasSubcomponent  // ❌ hasSubcomponent is not functional
]
```

**Solution**: Use functional properties:
```oml
scalar property hasId [
  domain Component
  range xsd:string
  functional  // Each has at most one
]

concept Component [
  key hasId  // ✓ Functional property
]
```

### Error 6: Restricting Outside Property Range

**Problem**:
```oml
relation hasSubcomponent [
  from Component
  to Component
]

concept System [
  restricts all hasSubcomponent to Function  // ❌ Range is Component, not Function
]
```

**Solution**: Restrict to subtypes of range:
```oml
relation hasSubcomponent [
  from Component
  to Component
]

concept Assembly < Component

concept System [
  restricts all hasSubcomponent to Component  // ✓ Component or
  // OR
  restricts all hasSubcomponent to Assembly   // ✓ Subtype of Component
]
```

### Error 7: Using Relation Entity Name Instead of Forward/Reverse

**Problem**:
```oml
relation entity Performs [
  from Component
  to Function
  forward performs
  reverse isPerformedBy
]

instance c1 : Component [
  mission:Performs fn1  // ❌ Wrong: using entity name
]
```

**Solution**: Use forward or reverse:
```oml
instance c1 : Component [
  mission:performs fn1  // ✓ Using forward name
]

instance fn1 : Function [
  mission:isPerformedBy c1  // ✓ Using reverse name
]
```

### Error 8: Forgetting ^ for Keywords

**Problem**:
```oml
concept Concept  // ❌ "Concept" is a keyword
aspect Aspect    // ❌ "Aspect" is a keyword
```

**Solution**: Escape with ^:
```oml
concept ^Concept  // ✓ Escaped
aspect ^Aspect    // ✓ Escaped

// Or use different names
concept ConceptElement  // ✓ Not a keyword
aspect AspectType       // ✓ Not a keyword
```

## Validation Checklist

Before committing OML files, verify:

✅ **IRIs**:
- Valid protocol (http:// or https://)
- No whitespace
- Consistent separators (# or /)
- No special characters in IDs

✅ **Imports**:
- All referenced ontologies imported
- No circular dependencies
- Standard libraries imported (xsd, rdfs, owl)

✅ **Restrictions**:
- Ranges are subtypes of property ranges
- Only restrict applicable properties
- Cardinalities make sense

✅ **Keys**:
- All key properties are functional
- Key properties apply to the concept

✅ **Relation Entities**:
- Both `from` and `to` specified
- Forward/reverse names defined
- Using forward/reverse in instances
- Characteristics reviewed for domain semantics (`functional`, `inverse functional`, `symmetric`, `asymmetric`, `reflexive`, `irreflexive`, `transitive`)

✅ **Rules**:
- No syntax errors
- Variables used consistently
- Builtins imported (swrlb)

✅ **Documentation**:
- Dublin Core on ontologies
- Comments on major elements
- No annotations on invalid IRIs

## Debugging Tips

### Check Import Chain

```oml
// Trace the import chain
A extends B
B extends C
C extends xsd

// Verify each link exists and is accessible
```

### Validate IRIs Independently

Test IRIs in isolation:
```oml
// Test IRI formation
http://domain/category#
http://domain/category/subcategory#
```

### Check Property Domains/Ranges

```oml
// Verify property applies to concept
scalar property hasId [
  domain IdentifiedElement  // Check concept is or extends IdentifiedElement
  range xsd:string
]

concept Component < IdentifiedElement [
  key hasId  // ✓ Component is IdentifiedElement
]
```

### Test Rules Incrementally

```oml
// Start simple
rule SimpleRule [
  Component(x) -> Element(x)
]

// Add complexity gradually
rule ComplexRule [
  Component(x) & performs(x, f) & invokes(f, f2)
  -> performs(x, f2)
]
```

### Use Consistent Naming

Create a naming convention document:
```
Concepts: UpperCamelCase
Properties: lowerCamelCase (verb phrases)
Instances: lowerCamelCase (noun phrases)
Prefixes: lowercase abbreviations
```
