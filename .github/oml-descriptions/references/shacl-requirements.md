# SHACL Requirements Quick Reference

This document provides SHACL requirements and common category values for each instance type.

**How to use:**
1. Find the instance type you're creating
2. Check REQUIRED properties (sh:minCount ≥ 1)
3. Check CONVENTIONAL properties (used in all existing instances, even if not required)
4. Use suggested category values for consistency

---

## Context Analysis

### Stakeholder (stakeholder:Stakeholder)

**File**: `context-analysis/stakeholders.md` / `stakeholders.oml`

**SHACL Requirements:**
- ❌ No required properties (sh:minCount)

**Conventional Properties** (used in all instances):
- `base:description` (sh:maxCount 1) - Description of the stakeholder
- `base:category` - Categorical classification (multiple values allowed)

**Common Categories:**
- `"Beneficiary"` - End users who benefit from the system
- `"User"` - Direct users/operators of the system
- `"Support"` - Support personnel (engineers, safety officers)

**Example:**
```oml
instance FireFighter : stakeholder:Stakeholder [
    base:description "Frontline operator fighting fires."
    base:category "Beneficiary"
]
```

---

### Concern (stakeholder:Concern)

**File**: `context-analysis/concerns.md` / `concerns.oml`

**SHACL Requirements:**
- ✅ **REQUIRED**: `stakeholder:isExpressedBy` (sh:minCount 1) - References stakeholder:Stakeholder instances

**Conventional Properties** (used in all instances):
- `base:description` (sh:maxCount 1) - Description of the concern
- `base:priority` (sh:maxCount 1) - Priority level
- `base:category` - Type of concern (multiple values allowed)

**Common Categories:**
- `"Need"` - Stakeholder need or requirement
- `"Objective"` - Goal or objective
- `"Constraint"` - Limitation or constraint

**Common Priorities:**
- `"High"`, `"Medium"`, `"Low"`

**Category Inference Rules:**
- Instance name ends with "Needs" → category: `"Need"`
- Instance name contains "Objective" → category: `"Objective"`
- Instance name contains "Constraint" → category: `"Constraint"`

**Example:**
```oml
instance FireFighterNeeds : stakeholder:Concern [
    base:description "Rapid situational awareness."
    base:priority "High"
    base:category "Need"
    stakeholder:isExpressedBy stakeholders:FireFighter
]
```

---

### Mission (mission:Mission)

**File**: `context-analysis/missions.md` / `missions.oml`

**SHACL Requirements:**
- ❌ No required properties (sh:minCount)

**Conventional Properties:**
- `base:description` (sh:maxCount 1) - Mission description

**Example:**
```oml
instance FireContainment : mission:Mission [
    base:description "Mission to contain and suppress wildfires."
]
```

---

### Objective (mission:Objective)

**File**: `context-analysis/objectives.md` / `objectives.oml`

**SHACL Requirements:**
- ❌ No required properties (sh:minCount)

**Conventional Properties:**
- `base:description` (sh:maxCount 1) - Objective description
- `mission:isPursuedBy` (sh:maxCount 1) - References mission:Mission instance
- `mission:isDerivedFrom` - References stakeholder:Concern instances (multiple allowed)

**Example:**
```oml
instance RapidResponse : mission:Objective [
    base:description "Achieve rapid response time to fire incidents."
    mission:isPursuedBy missions:FireContainment
    mission:isDerivedFrom concerns:FireFighterNeeds
]
```

---

## Operational Analysis

### Entity (entity:Entity, entity:Actor)

**File**: `operational-analysis/entities.md` / `entities.oml`

**SHACL Requirements:**
- ✅ **REQUIRED**: `entity:hasCapability` (sh:minCount 1) - References mission:Capability instances

**Conventional Properties:**
- `base:description` (sh:maxCount 1) - Entity description

**Example:**
```oml
instance FireController : entity:Actor [
    base:description "Human operator controlling fire suppression."
    entity:hasCapability capabilities:MonitorFires
]
```

---

### Requirement (stakeholder:Requirement)

**File**: `operational-analysis/requirements.md` / `requirements.oml`

**SHACL Requirements:**
- ❌ No required properties (sh:minCount)

**Conventional Properties** (used in all instances):
- `base:description` (sh:maxCount 1) - Requirement title/summary
- `base:expression` (sh:maxCount 1) - Formal requirement statement (SHALL statement)
- `stakeholder:isStatedBy` - References stakeholder:Stakeholder instances (multiple allowed)
- `base:category` - Requirement level (multiple values allowed)
- `base:priority` (sh:maxCount 1) - Priority level

**Common Categories:**
- `"L1"` - Level 1 requirements (high-level)
- `"L2"` - Level 2 requirements (detailed)
- `"L3"` - Level 3 requirements (implementation-level)

**Common Priorities:**
- `"High"`, `"Medium"`, `"Low"`

**Example:**
```oml
instance R1 : stakeholder:Requirement [
    base:description "Real-Time Map"
    base:expression "The system shall display real-time fire and drone locations."
    stakeholder:isStatedBy stakeholders:Operator
    base:category "L1"
    base:priority "High"
]
```

---

## System Analysis

### Component (component:Component)

**File**: `system-analysis/components.md` / `components.oml`

**SHACL Requirements:**
- ❌ No required properties (sh:minCount)

**Conventional Properties:**
- `base:description` (sh:maxCount 1) - Component description
- `base:isContainedBy` (sh:maxCount 1) - References parent component:Component (for hierarchical decomposition)

**Example:**
```oml
instance Dashboard : component:Component [
    base:description "Main user interface dashboard component."
]

instance MapWidget : component:Component [
    base:description "Map visualization widget."
    base:isContainedBy components:Dashboard
]
```

---

## Common Properties Across All Types

These properties from the `base` vocabulary are commonly used:

| Property | Type | Cardinality | Description |
|----------|------|-------------|-------------|
| `base:description` | Scalar | max 1 | Natural language description |
| `base:category` | Scalar | multiple | Categorical classification |
| `base:priority` | Scalar | max 1 | Priority level (High/Medium/Low) |
| `base:expression` | Scalar | max 1 | Formal expression (for Requirements) |

---

## SHACL Constraint Reference

| SHACL Property | Meaning | Impact on Instance |
|----------------|---------|-------------------|
| `sh:minCount 1` | At least one value required | ✅ **REQUIRED** property |
| `sh:maxCount 1` | At most one value allowed | Single value only |
| `sh:class X` | Values must be instances of class X | Use object references (e.g., `stakeholders:Ahmed`) |
| No `sh:minCount` | Zero or more values | ❌ Optional but may be conventional |
| No `sh:maxCount` | Unlimited values | Can repeat property multiple times |

---

## Category Inference Guide

When user provides instance name/description, infer category:

### Concerns
- Name contains "Need/Needs" → `"Need"`
- Name contains "Objective" → `"Objective"`  
- Name contains "Constraint/Limit" → `"Constraint"`

### Stakeholders
- User mentions "firefighter/operator/user" → `"User"` or `"Beneficiary"`
- User mentions "engineer/officer/support" → `"Support"`

### Requirements
- User mentions "level 1/high-level" → `"L1"`
- User mentions "level 2/detailed" → `"L2"`
- User mentions "level 3/implementation" → `"L3"`

---

## Quick Checklist for Instance Creation

Before creating any instance:
- [ ] Identify target class from SHACL `sh:targetClass`
- [ ] List all REQUIRED properties (sh:minCount ≥ 1)
- [ ] List all CONVENTIONAL properties (used in existing instances)
- [ ] Verify referenced instances exist
- [ ] Infer appropriate category value
- [ ] Check relation direction in vocabulary definition

After creating instance:
- [ ] Validate no errors
- [ ] Verify all required properties included
- [ ] Verify all conventional properties included (especially category)
- [ ] Check category value is appropriate
