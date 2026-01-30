# Diagram Implementations

Use this file when you need details about how each Mermaid diagram type is rendered to ASCII/Unicode.

## Flowchart / State Diagram

**Rendering strategy:** grid layout + A* routing.

**Key characteristics:**
- Each node occupies a 3x3 grid block.
- Supports TD/LR/BT/RL directions (BT uses vertical flip).
- Handles collisions when placing nodes.

**Core flow (grid.ts → createMapping):**
1. Identify root nodes (no incoming edges).
2. Place nodes by level (LR/TD variants).
3. Compute column widths / row heights (label-length based).
4. Route edges via A*.
5. Determine label placement (widest segment).
6. Convert grid → drawing coordinates.
7. Generate node box drawings.
8. Compute subgraph bounding boxes.

```typescript
renderMermaidAscii('graph TD\n  A --> B')
```

## Sequence Diagram

**Rendering strategy:** column layout (actors) + row layout (messages).

**Key characteristics:**
- Each actor occupies a column; lifelines are vertical.
- Messages are rows; arrows connect lifelines.
- Not grid-based; uses linear layout.

**Layout algorithm:**
```typescript
// 1) actor columns
const actorIdx = new Map<string, number>()
diagram.actors.forEach((a, i) => actorIdx.set(a.id, i))

// 2) column gaps based on label width
for (const msg of diagram.messages) {
  const needed = msg.label.length + 4
  const numGaps = hi - lo
  const perGap = Math.ceil(needed / numGaps)
}

// 3) message vertical positions
// - self message: 3 rows
// - normal message: 2 rows
```

**Special cases:**
- Block groups (loop/alt/opt/par)
- Notes around actors
- Self messages drawn with a right-side loop

## Class Diagram

**Rendering strategy:** layered layout + UML markers.

**Key characteristics:**
- Top-down layers (parent above child).
- Multi-compartment class boxes (header/attributes/methods).
- All relationship types place nodes on different layers.

**UML markers:**
```
inheritance: △
realization: ▽
composition: ◆
aggregation: ◇
association: ▶
dependency: ▶ (dashed)
```

**Layout algorithm:**
```typescript
// 1) build hierarchy
for (const rel of diagram.relationships) {
  const isHierarchical = rel.type === 'inheritance' || rel.type === 'realization'
  const parentId = isHierarchical && rel.markerAt === 'to' ? rel.to : rel.from
  parents.get(parentId)!.add(childId)
}

// 2) BFS level assignment
const level = new Map<string, number>()
const roots = diagram.classes.filter(c => !parents.has(c.id))

// 3) horizontal placement per level
for (let lv = 0; lv <= maxLevel; lv++) {
  const group = levelGroups[lv]
  for (const id of group) {
    const cls = classById.get(id)
    // place at current X; Y = level row
  }
  currentY += maxH + vGap
}
```

## ER Diagram

**Rendering strategy:** grid layout + crow's foot notation.

**Key characteristics:**
- Entities are two-section boxes (header + attributes).
- Entities placed in rows (grid-like layout).
- Crow's foot markers at relationship endpoints.

**Crow's foot markers:**
```
one (||):       ──║── or --||--
zero-one (o|):  ──o║── or --o|--
many (}):      ──╢── or --<|-- (or }|)
zero-many (o{): ──o╢── or --o<-- (or o{)
```

**Layout algorithm:**
```typescript
// 1) grid placement (max sqrt(N) entities per row)
const maxPerRow = Math.max(2, Math.ceil(Math.sqrt(diagram.entities.length)))

for (const ent of diagram.entities) {
  const [entity, sections] = placed.get(id)
  entity.x = currentX
  entity.y = currentY
}

// 2) crow's foot routing
const sameRow = Math.abs(e1CY - e2CY) < Math.max(e1.height, e2.height)
if (sameRow) {
  // horizontal connection
} else {
  // vertical connection
}
```
