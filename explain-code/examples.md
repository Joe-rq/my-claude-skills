# Example Output

## Example 1: React useEffect Hook

**Analogy**: Think of `useEffect` like a "side-effect manager" in a factory. When a worker (component) finishes making a product (render), the side-effect manager steps in to handle cleanup and setup tasks—like sweeping the floor (cleanup) or preparing tools for the next batch (setup).

**Diagram**:
```
┌─────────────────────────────────────────┐
│         Component Render              │
│    (Worker makes the product)         │
└──────────────────┬──────────────────┘
                   │
                   ▼
         ┌─────────────────────┐
         │   useEffect Hook    │
         │   (Side-effect     │
         │    Manager)        │
         └────────┬────────────┘
                  │
    ┌─────────────┴─────────────┐
    │                           │
    ▼                           ▼
┌─────────┐              ┌──────────┐
│ Setup   │              │ Cleanup  │
│ (Prepare│              │ (Sweep   │
│  tools) │              │  floor)  │
└─────────┘              └──────────┘
```

**Walkthrough**:
1. Component first renders with initial state
2. After render completes, `useEffect` runs the setup function
3. When component unmounts or dependencies change, cleanup runs first, then setup again

**Gotcha**: Don't forget the dependency array! Omitting it causes infinite loops because the effect runs after every render.
