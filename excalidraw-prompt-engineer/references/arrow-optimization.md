# Arrow Optimization Algorithm

## Overview

The arrow optimization algorithm is a core innovation in the Excalidraw prompt engineer system. It solves the common problem where LLM-generated arrow connections appear misaligned or awkward by intelligently calculating optimal connection points on element edges.

## Problem Statement

When LLMs generate Excalidraw diagrams, they often struggle with precise arrow positioning:

**Common Issues:**
- Arrows connecting to element corners instead of edge centers
- Inconsistent connection points across similar diagrams
- Poor visual aesthetics due to misaligned connections
- Difficulty maintaining professional appearance

## Solution: Geometric Edge Optimization

The algorithm uses **quadrant-based spatial reasoning** and **distance optimization** to determine the best connection points.

### Core Algorithm: determineEdges()

**Complexity:** O(1) - constant time per arrow

**Input:** Two elements (startEle, endEle) with x, y, width, height properties

**Output:** Optimal edge pair {startEdge, endEdge}

### Step 1: Calculate Center Points

```javascript
const startCenterX = startX + startWidth / 2;
const startCenterY = startY + startHeight / 2;
const endCenterX = endX + endWidth / 2;
const endCenterY = endY + endHeight / 2;
```

**Purpose:** Accurate determination of relative positioning between elements.

### Step 2: Determine Relative Position

```javascript
const dx = startCenterX - endCenterX;  // Horizontal offset
const dy = startCenterY - endCenterY;  // Vertical offset
```

**Quadrant Classification:**

| dx | dy | Quadrant | startEle Position |
|----|----|---------| -----------------|
| > 0 | > 0 | IV | Lower-right of endEle |
| < 0 | > 0 | III | Lower-left of endEle |
| > 0 | < 0 | I | Upper-right of endEle |
| < 0 | < 0 | II | Upper-left of endEle |
| = 0 | > 0 | - | Directly below |
| = 0 | < 0 | - | Directly above |
| > 0 | = 0 | - | Directly right |
| < 0 | = 0 | - | Directly left |

### Step 3: Calculate Edge Distance Differences

For each quadrant, calculate distances between candidate edge pairs:

**Horizontal Edge Pairs:**
```javascript
const leftToRightDistance = (startX - (endX + endWidth));
const rightToLeftDistance = -((startX + startWidth) - endX);
```

**Vertical Edge Pairs:**
```javascript
const topToBottomDistance = (startY - (endY + endHeight));
const bottomToTopDistance = -((startY + startHeight) - endY);
```

**Key Insight:** We **do NOT** use Math.abs(). Maintaining directionality ensures positive values indicate separation, negative values indicate overlap.

### Step 4: Select Optimal Edge Pair

For each quadrant, compare the two candidate edge pairs and select the one with **maximum separation**.

**Example: Quadrant IV (dx > 0, dy > 0)**

```javascript
if (dx > 0 && dy > 0) {
  // startEle is in lower-right quadrant
  if (leftToRightDistance > topToBottomDistance) {
    // Horizontal separation is greater
    startEdge = 'left';
    endEdge = 'right';
  } else {
    // Vertical separation is greater
    startEdge = 'top';
    endEdge = 'bottom';
  }
}
```

**Principle:** Choose the edge pair that maximizes the visual separation, resulting in cleaner, more professional connections.

## Edge Center Calculation

Once edges are determined, calculate the precise center point of each edge:

```javascript
function getEdgeCenter(element, edge) {
  const x = element.x || 0;
  const y = element.y || 0;
  const width = element.width || 100;
  const height = element.height || 100;

  switch (edge) {
    case 'left':
      return { x: x, y: y + height / 2 };
    case 'right':
      return { x: x + width, y: y + height / 2 };
    case 'top':
      return { x: x + width / 2, y: y };
    case 'bottom':
      return { x: x + width / 2, y: y + height };
  }
}
```

## Arrow Coordinate Assignment

Final arrow coordinates are computed from edge centers:

```javascript
const startEdgeCenter = getStartEdgeCenter(startEle, endEle);
const endEdgeCenter = getEndEdgeCenter(endEle, startEle);

// Arrow start point
arrow.x = startEdgeCenter.x;
arrow.y = startEdgeCenter.y;

// Arrow end point (relative to start)
arrow.width = endEdgeCenter.x - startEdgeCenter.x;
arrow.height = endEdgeCenter.y - startEdgeCenter.y;

// Fix Excalidraw rendering bug
if (arrow.width === 0) {
  arrow.width = 1;
}
```

## Special Cases

### 1. Aligned Elements (dx = 0 or dy = 0)

When elements are perfectly aligned horizontally or vertically:

```javascript
else if (dx === 0 && dy > 0) {
  // Directly below
  startEdge = 'top';
  endEdge = 'bottom';
}
```

### 2. Overlapping Elements

When elements overlap (rare but possible):

```javascript
else {
  // Default case
  startEdge = 'right';
  endEdge = 'left';
}
```

### 3. Missing Properties

All element properties use default values for robustness:

```javascript
const startX = startEle.x || 0;
const startY = startEle.y || 0;
const startWidth = startEle.width || 100;
const startHeight = startEle.height || 100;
```

## Integration with Excalidraw Code Generation

The optimization function processes complete Excalidraw JSON:

```javascript
export function optimizeExcalidrawCode(codeString) {
  // 1. Parse JSON string
  const elements = JSON.parse(arrayMatch[0]);

  // 2. Create element ID map for lookups
  const elementMap = new Map();
  elements.forEach(el => {
    if (el.id) elementMap.set(el.id, el);
  });

  // 3. Process each arrow/line
  const optimizedElements = elements.map(element => {
    if (element.type !== 'arrow' && element.type !== 'line') {
      return element;
    }

    const startEle = element.start?.id ? elementMap.get(element.start.id) : null;
    const endEle = element.end?.id ? elementMap.get(element.end.id) : null;

    if (startEle && endEle) {
      // Apply optimization
      const optimized = { ...element };
      const startEdgeCenter = getStartEdgeCenter(startEle, endEle);
      const endEdgeCenter = getEndEdgeCenter(endEle, startEle);

      optimized.x = startEdgeCenter.x;
      optimized.y = startEdgeCenter.y;
      optimized.width = endEdgeCenter.x - startEdgeCenter.x;
      optimized.height = endEdgeCenter.y - startEdgeCenter.y;

      return optimized;
    }

    return element;
  });

  // 4. Return optimized JSON
  return JSON.stringify(optimizedElements, null, 2);
}
```

## Performance Characteristics

- **Time Complexity:** O(n) where n = number of elements
  - Single pass through all elements
  - O(1) optimization per arrow

- **Space Complexity:** O(n)
  - Element map for ID lookups
  - Optimized elements array

## Visual Comparison

### Before Optimization:
```
┌─────┐
│  A  │────┐
└─────┘    │
           │
           ↓ (connects to corner)
      ┌─────┐
      │  B  │
      └─────┘
```

### After Optimization:
```
┌─────┐
│  A  │
└──┬──┘
   │
   │ (connects to edge center)
   ↓
┌─────┐
│  B  │
└─────┘
```

## Key Benefits

1. **Consistency:** Same relative positions always produce same connections
2. **Aesthetics:** Clean, professional-looking diagrams
3. **Reliability:** Works with any element sizes and positions
4. **Performance:** Fast O(n) optimization
5. **Robustness:** Handles edge cases and missing properties

## Usage in Workflow

```javascript
// 1. Generate diagram with LLM
const llmOutput = await generateExcalidrawCode(userPrompt);

// 2. Repair any JSON issues
const repairedJSON = repairJsonClosure(llmOutput);

// 3. Optimize arrow connections
const optimizedJSON = optimizeExcalidrawCode(repairedJSON);

// 4. Render in Excalidraw
excalidrawAPI.updateScene({
  elements: JSON.parse(optimizedJSON)
});
```

## References

- Implementation: `scripts/optimizeArrows.js`
- Original documentation: Based on `docs/arrow-compute-rule.md` from smart-excalidraw-next project
- Related: `references/examples.md` for arrow binding patterns
