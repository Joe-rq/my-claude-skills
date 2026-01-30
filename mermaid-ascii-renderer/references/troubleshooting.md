# Troubleshooting and Optimization

Use this file for debugging, performance tuning, and best practices.

## Common Issues

### Node overlaps

- Grid placement auto-resolves collisions via `reserveSpotInGrid`.
- If overlaps persist, increase `paddingX` / `paddingY`.

```typescript
const ascii = renderMermaidAscii(diagram, { paddingX: 8, paddingY: 6 })
```

### Edges crossing nodes

- A* routing avoids occupied grid cells; increase padding or mark more obstacles.
- Review heuristic/cost if routing favors sharp turns.

### Labels clipping or misaligned

- `determineLabelLine` expands the column width for the label.
- If clipping remains, increase padding or shorten label text.

## Performance Notes

### Grid size

Keep large graphs bounded by limiting the search space or reducing node count.

### Pathfinding

Prefer straight paths by tuning the heuristic or move ordering.

### Canvas merging

Avoid repeated deep copies; merge layers in a single pass where possible.

## Best Practices

1. Prefer Unicode for readability; use ASCII for maximum terminal compatibility.
2. Keep labels short; long labels drive spacing and reduce legibility.
3. Use consistent direction (TD/LR) in a diagram to reduce routing complexity.
4. Treat subgraphs as layout boundaries; avoid cross-subgraph edges when possible.
