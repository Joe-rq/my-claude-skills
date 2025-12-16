# High-Quality ExcalidrawElementSkeleton Examples

This document contains 9 examples demonstrating best practices for using ExcalidrawElementSkeleton API.

## 1. Basic Shape

```json
[{
  "type": "rectangle",
  "x": 100,
  "y": 200,
  "width": 180,
  "height": 80,
  "backgroundColor": "#e3f2fd",
  "strokeColor": "#1976d2"
}]
```

**Use case**: Simple rectangular container with custom colors.

---

## 2. Text (Auto-sized)

```json
[{
  "type": "text",
  "x": 100,
  "y": 100,
  "text": "标题文本",
  "fontSize": 20
}]
```

**Use case**: Standalone text element. Width and height are automatically calculated based on text content.

---

## 3. Text Container (Auto-sized based on label)

```json
[{
  "type": "rectangle",
  "x": 100,
  "y": 150,
  "label": { "text": "项目管理", "fontSize": 18 },
  "backgroundColor": "#e8f5e9"
}]
```

**Use case**: Rectangle container with text label. Container size is automatically calculated based on label text.

---

## 4. Arrow + Label + Auto-created Bindings

```json
[{
  "type": "arrow",
  "x": 255,
  "y": 239,
  "label": { "text": "影响" },
  "start": { "type": "rectangle" },
  "end": { "type": "ellipse" },
  "strokeColor": "#2e7d32"
}]
```

**Use case**: Arrow that automatically creates and binds to rectangle (start) and ellipse (end) endpoints. Label is displayed on the arrow.

---

## 5. Line/Arrow with Additional Properties

```json
[
  {
    "type": "arrow",
    "x": 450,
    "y": 20,
    "startArrowhead": "dot",
    "endArrowhead": "triangle",
    "strokeColor": "#1971c2",
    "strokeWidth": 2
  },
  {
    "type": "line",
    "x": 450,
    "y": 60,
    "strokeColor": "#2f9e44",
    "strokeWidth": 2,
    "strokeStyle": "dotted"
  }
]
```

**Use case**: Demonstrates different arrowhead styles and line styling options (color, width, style).

---

## 6. Text Containers (Advanced Typography)

```json
[
  {
    "type": "diamond",
    "x": -120,
    "y": 100,
    "width": 270,
    "backgroundColor": "#fff3bf",
    "strokeWidth": 2,
    "label": {
      "text": "STYLED DIAMOND TEXT CONTAINER",
      "strokeColor": "#099268",
      "fontSize": 20
    }
  },
  {
    "type": "rectangle",
    "x": 180,
    "y": 150,
    "width": 200,
    "strokeColor": "#c2255c",
    "label": {
      "text": "TOP LEFT ALIGNED RECTANGLE TEXT CONTAINER",
      "textAlign": "left",
      "verticalAlign": "top",
      "fontSize": 20
    }
  },
  {
    "type": "ellipse",
    "x": 400,
    "y": 130,
    "strokeColor": "#f08c00",
    "backgroundColor": "#ffec99",
    "width": 200,
    "label": {
      "text": "STYLED ELLIPSE TEXT CONTAINER",
      "strokeColor": "#c2255c"
    }
  }
]
```

**Use case**: Demonstrates advanced text container styling with different shapes, colors, and text alignment options.

---

## 7. Arrow Binding to Text Endpoints (via type)

```json
{
  "type": "arrow",
  "x": 255,
  "y": 239,
  "start": { "type": "text", "text": "HEYYYYY" },
  "end": { "type": "text", "text": "WHATS UP ?" }
}
```

**Use case**: Arrow that auto-creates and binds to text elements at both ends. The text content is specified directly.

---

## 8. Binding to Existing Elements via ID

```json
[
  {
    "type": "ellipse",
    "id": "ellipse-1",
    "strokeColor": "#66a80f",
    "x": 390,
    "y": 356,
    "width": 150,
    "height": 150,
    "backgroundColor": "#d8f5a2"
  },
  {
    "type": "diamond",
    "id": "diamond-1",
    "strokeColor": "#9c36b5",
    "width": 100,
    "x": -30,
    "y": 380
  },
  {
    "type": "arrow",
    "x": 100,
    "y": 440,
    "width": 295,
    "height": 35,
    "strokeColor": "#1864ab",
    "start": { "type": "rectangle", "width": 150, "height": 150 },
    "end": { "id": "ellipse-1" }
  },
  {
    "type": "arrow",
    "x": 60,
    "y": 420,
    "width": 330,
    "strokeColor": "#e67700",
    "start": { "id": "diamond-1" },
    "end": { "id": "ellipse-1" }
  }
]
```

**Use case**: Demonstrates binding arrows to existing elements using their `id`. This is useful when you need multiple arrows connecting to the same element.

---

## 9. Frame (Auto-calculated bounds)

```json
[
  { "type": "rectangle", "id": "rect-1", "x": 10, "y": 10 },
  { "type": "diamond", "id": "diamond-1", "x": 120, "y": 20 },
  {
    "type": "frame",
    "children": ["rect-1", "diamond-1"],
    "name": "功能模块组"
  }
]
```

**Use case**: Frame groups multiple elements together. The frame's position and size are automatically calculated based on its children, with 10px padding.

---

## Best Practices

1. **Arrow Connections**: Always bind arrows to elements using either `type` (auto-create) or `id` (bind existing)
2. **Layout Planning**: Pre-plan layout with sufficient spacing (>800px) between elements to avoid overlap
3. **Size Consistency**: Keep similar sizes for same-type elements to establish visual rhythm
4. **Color Scheme**: Use 2-4 main colors to maintain visual unity
5. **Text Containers**: Let the system auto-calculate container size based on label text for better fit
6. **Whitespace**: Maintain sufficient whitespace to avoid visual clutter
