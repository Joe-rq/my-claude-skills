---
name: excalidraw-prompt-engineer
description: Complete AI-powered Excalidraw diagram generation system with 23+ chart types, arrow optimization algorithm, JSON repair utilities, and comprehensive prompt engineering. Includes production-ready scripts (optimizeArrows.js, json-repair.js), detailed documentation, and implementation guides. Use for generating professional diagrams from text/images, building diagram generation features, or converting natural language to visual representations.
---

# Excalidraw Prompt Engineer

## Overview

Convert natural language descriptions, documents, and images into professional Excalidraw diagrams using AI-powered prompt engineering. This skill provides a comprehensive system for generating 23+ different chart types with consistent visual quality, proper element binding, and optimized layouts.

## When to Use This Skill

Use this skill when:
- Generating Excalidraw diagrams from text descriptions
- Converting documents or images into visual diagrams
- Creating flowcharts, mindmaps, org charts, or any of 23+ supported chart types
- Building AI-powered diagram generation features
- Implementing text-to-diagram conversion workflows
- Working with the ExcalidrawElementSkeleton API

## Core Capabilities

### 1. Multi-Modal Input Support

Handle three types of input for diagram generation:

**Text Input**: Direct natural language descriptions
```
User: "Create a flowchart showing the user authentication process"
```

**File Input**: Markdown or text file processing
```
User provides: architecture-design.md
System extracts content and generates corresponding diagram
```

**Image Input**: Vision-based diagram conversion
```
User uploads: whiteboard-sketch.jpg
System analyzes and converts to Excalidraw format
```

### 2. 23+ Chart Type Specifications

Supported chart types with detailed visual specifications:

| Category | Chart Types |
|----------|-------------|
| **Process & Flow** | Flowchart, Sequence Diagram, Data Flow Diagram, State Diagram, Swimlane |
| **Structure & Hierarchy** | Mindmap, Org Chart, Tree Diagram, Network Topology, Architecture |
| **Data & Analysis** | ER Diagram, UML Class Diagram, Gantt Chart, Timeline, Matrix |
| **Analysis Tools** | SWOT Analysis, Fishbone Diagram, Concept Map, Pyramid, Funnel, Venn |
| **Creative** | Infographic |

Each chart type includes:
- Shape conventions (which element types to use)
- Layout guidelines (spatial arrangement)
- Color schemes (visual styling)
- Connection patterns (how elements relate)

**Reference**: See `references/chart-specs.md` for complete specifications.

### 3. Intelligent Chart Type Selection

**Auto Mode**: When chart type is not specified, the system intelligently selects the most appropriate visualization based on:
1. Core content analysis
2. Information structure
3. Relationship patterns
4. User intent

**Specified Mode**: When a specific chart type is requested, strict adherence to that type's visual specifications ensures consistent, professional output.

### 4. Arrow Optimization Algorithm

**Core Innovation**: Custom geometric algorithm for calculating optimal arrow connection points.

**Problem Solved**: LLM-generated diagrams often have arrows connecting to element corners instead of edge centers, resulting in unprofessional appearance.

**Solution**: Quadrant-based spatial reasoning algorithm that:
1. Determines relative positions between connected elements
2. Calculates candidate edge pairs (left-right, top-bottom)
3. Selects edge pair with maximum separation
4. Computes precise edge center coordinates

**Benefits**:
- Clean, professional-looking connections
- Consistent positioning across diagrams
- O(n) time complexity (n = number of arrows)
- Handles all special cases (aligned, overlapping elements)

**Reference**: See `references/arrow-optimization.md` and `scripts/optimizeArrows.js`

### 5. JSON Repair System

**Core Innovation**: Robust handling of LLM output format issues.

**Common Issues Handled**:
- Markdown code fences (```json```)
- Mixed text with JSON
- Unclosed strings, brackets, braces
- Missing object braces in arrays
- Trailing commas

**Three-Tier Strategy**:
1. **Direct parse** (fastest - valid JSON)
2. **Custom repair** (fast - common LLM issues)
3. **Robust library** (comprehensive - edge cases)

**Benefits**:
- Production-ready reliability
- Streaming-friendly (incremental parsing)
- Conservative (only appends, never removes)
- O(n) time complexity

**Reference**: See `references/json-repair-guide.md` and `scripts/json-repair.js`

## Prompt Construction System

### System Prompt Structure

The system prompt provides comprehensive guidance on:

1. **Task Definition**: Generate structured Excalidraw diagrams from user requirements
2. **Input Handling**: Process text, files, or images
3. **Output Format**: JSON code using ExcalidrawElementSkeleton API
4. **Execution Steps**: Need analysis → Visual creation
5. **Best Practices**: Code standards, content accuracy, visual quality
6. **Visual Style Guide**: Educational, professional, clear, concise

**Reference**: See `references/system-prompt.md` for complete system prompt.

### User Prompt Template

Dynamic prompt construction based on:

```javascript
USER_PROMPT_TEMPLATE(userInput, chartType)
```

**Parameters**:
- `userInput`: User's requirements (text/file/image content)
- `chartType`: Desired chart type or 'auto' for intelligent selection

**Template Logic**:
1. If `chartType` specified → Inject specific visual specifications
2. If `chartType` is 'auto' → Provide chart type selection guide
3. Append user requirements
4. Add compliance reminders for visual standards

### ExcalidrawElementSkeleton API

Core element types and their properties:

**Shapes**: `rectangle`, `ellipse`, `diamond`
- Text containers with auto-sizing
- Customizable colors, styles, borders

**Text**: `text`
- Auto-calculated dimensions
- Font customization
- Alignment options

**Connectors**: `arrow`, `line`
- Binding to elements via `type` (auto-create) or `id` (existing)
- Label support
- Multiple arrowhead styles

**Grouping**: `frame`
- Group multiple elements
- Auto-calculated bounds
- Named groups

**Reference**: See `references/examples.md` for 9 high-quality usage examples.

## Implementation Workflow

### Step 1: Prepare System Context

Load the system prompt into your LLM context:

```javascript
import { SYSTEM_PROMPT } from './prompts.js';

const systemMessage = {
  role: 'system',
  content: SYSTEM_PROMPT
};
```

### Step 2: Construct User Prompt

Build the user prompt with chart type specification:

```javascript
import { USER_PROMPT_TEMPLATE } from './prompts.js';

const userPrompt = USER_PROMPT_TEMPLATE(
  userInput,      // "Create a flowchart for user login"
  'flowchart'     // or 'auto' for intelligent selection
);
```

### Step 3: Handle Multimodal Input

**For images**, include vision capability:

```javascript
const userMessage = {
  role: 'user',
  content: [
    { type: 'text', text: userPrompt },
    { type: 'image_url', image_url: { url: `data:image/jpeg;base64,${base64Image}` } }
  ]
};
```

**For text/files**, use text-only content:

```javascript
const userMessage = {
  role: 'user',
  content: userPrompt
};
```

### Step 4: Generate and Process

Call your LLM provider with streaming support:

```javascript
const response = await llmClient.chat({
  model: 'claude-3-5-sonnet',
  messages: [systemMessage, userMessage],
  stream: true
});

// Stream JSON code incrementally
for await (const chunk of response) {
  accumulatedCode += chunk;
  // Display/process incrementally
}
```

### Step 5: Post-Process Output

Apply arrow optimization and JSON repair:

```javascript
import { optimizeExcalidrawCode } from './optimizeArrows.js';
import { repairJSON } from './json-repair.js';

// 1. Repair any malformed JSON
const repairedJSON = repairJSON(accumulatedCode);

// 2. Optimize arrow connection points
const optimizedJSON = optimizeExcalidrawCode(repairedJSON);

// 3. Render in Excalidraw
excalidrawAPI.updateScene({
  elements: JSON.parse(optimizedJSON)
});
```

## Best Practices

### 1. Layout Planning

**Spacing**: Maintain minimum 800px spacing between major elements
```json
{
  "type": "rectangle",
  "x": 100,
  "y": 200
},
{
  "type": "rectangle",
  "x": 980,  // 880px horizontal spacing
  "y": 200
}
```

**Avoid Overlap**: Pre-calculate positions to prevent visual clutter

### 2. Element Binding

**Always bind arrows** to ensure clean connections:

```json
// Good: Arrow with proper bindings
{
  "type": "arrow",
  "x": 200,
  "y": 300,
  "start": { "type": "rectangle" },
  "end": { "id": "existing-element-id" }
}

// Bad: Standalone arrow without bindings
{
  "type": "arrow",
  "x": 200,
  "y": 300
}
```

### 3. Size Consistency

Maintain similar sizes for related elements:

```json
// Process boxes - consistent sizing
[
  { "type": "rectangle", "width": 180, "height": 80 },
  { "type": "rectangle", "width": 180, "height": 80 },
  { "type": "rectangle", "width": 180, "height": 80 }
]
```

### 4. Color Schemes

Use 2-4 main colors with purpose:

```json
{
  "backgroundColor": "#e3f2fd",  // Light blue for normal states
  "strokeColor": "#1976d2"       // Dark blue for borders
}
{
  "backgroundColor": "#fff3e0",  // Light orange for decisions
  "strokeColor": "#f57c00"       // Dark orange for borders
}
```

### 5. Text Container Auto-Sizing

Let the system calculate container sizes:

```json
// Preferred: Auto-sized based on label
{
  "type": "rectangle",
  "x": 100,
  "y": 150,
  "label": { "text": "Project Management", "fontSize": 18 }
  // width/height automatically calculated
}

// Avoid: Manual sizing for text containers (unless specific layout needed)
{
  "type": "rectangle",
  "x": 100,
  "y": 150,
  "width": 200,
  "height": 60,
  "label": { "text": "Project Management" }
}
```

## Chart Type Selection Guide

### Decision Matrix

| If User Needs... | Recommend Chart Type |
|------------------|----------------------|
| Show process steps or workflow | Flowchart |
| Brainstorm ideas or knowledge structure | Mindmap |
| Display organizational hierarchy | Org Chart |
| Illustrate system interactions over time | Sequence Diagram |
| Model database relationships | ER Diagram |
| Design class structures | UML Class Diagram |
| Plan project timeline | Gantt Chart |
| Show historical events | Timeline |
| Analyze causes of a problem | Fishbone Diagram |
| Strategic planning | SWOT Analysis |
| Display data creatively | Infographic |

### Auto Mode Prompt Example

When using auto mode, the system provides guidance:

```
请根据用户需求,智能选择最合适的一种或多种图表类型来呈现信息。

## 可选图表类型
- **流程图 (flowchart)**: 适合展示流程、步骤、决策逻辑
- **思维导图 (mindmap)**: 适合展示概念关系、知识结构、头脑风暴
...

## 选择指南
1. 分析用户需求的核心内容和目标
2. 选择最能清晰表达信息的图表类型
3. 严格遵循该类型的视觉规范
4. 确保图表能够独立传达信息
```

## Advanced Features

### 1. Frame Grouping

Group related elements for better organization:

```json
[
  { "type": "rectangle", "id": "input-1", "x": 10, "y": 10, "label": { "text": "Input Layer" } },
  { "type": "rectangle", "id": "input-2", "x": 10, "y": 120, "label": { "text": "Input Nodes" } },
  {
    "type": "frame",
    "children": ["input-1", "input-2"],
    "name": "Neural Network - Input"
  }
]
```

### 2. Multimodal Processing

For image inputs, add special instructions:

```
如果输入包含图片,请:
1. 仔细分析图片中的视觉元素、文字、结构和关系
2. 识别图表类型
3. 提取关键信息和逻辑关系
4. 将图片内容准确转换为Excalidraw格式
5. 保持原始设计的意图和信息完整性
```

### 3. Custom Visual Specifications

Extend chart specifications for domain-specific needs:

```javascript
const CUSTOM_CHART_SPEC = {
  'neural-network': `
### Neural Network Diagram Visual Specifications
- **Layers**: Use vertical columns of ellipses
- **Connections**: Dense arrow connections between layers
- **Colors**: Input layer (blue), hidden layers (green), output layer (orange)
- **Layout**: Left-to-right flow, layers evenly spaced
  `
};
```

## Integration Patterns

### Pattern 1: Streaming Generation with Real-time Display

```javascript
async function generateDiagramWithStreaming(userInput, chartType) {
  let accumulatedJSON = '';

  const stream = await generateExcalidrawCode(userInput, chartType);

  for await (const chunk of stream) {
    accumulatedJSON += chunk;

    // Attempt to parse and display incrementally
    try {
      const parsed = repairJSON(accumulatedJSON);
      updatePreview(parsed);
    } catch (e) {
      // Continue accumulating
    }
  }

  // Final optimization
  const optimized = optimizeExcalidrawCode(accumulatedJSON);
  renderFinal(optimized);
}
```

### Pattern 2: Multi-Provider Support

```javascript
const providers = {
  openai: async (messages) => {
    // OpenAI API call
  },
  anthropic: async (messages) => {
    // Anthropic API call with vision support
  }
};

async function generate(userInput, chartType, provider = 'anthropic') {
  const systemPrompt = SYSTEM_PROMPT;
  const userPrompt = USER_PROMPT_TEMPLATE(userInput, chartType);

  return await providers[provider]([
    { role: 'system', content: systemPrompt },
    { role: 'user', content: userPrompt }
  ]);
}
```

### Pattern 3: Configuration Management

```javascript
class DiagramGenerator {
  constructor(config) {
    this.defaultChartType = config.defaultChartType || 'auto';
    this.llmProvider = config.llmProvider;
    this.model = config.model;
  }

  async generate(userInput, chartType) {
    const type = chartType || this.defaultChartType;
    const prompt = USER_PROMPT_TEMPLATE(userInput, type);

    return await this.llmProvider.generate(SYSTEM_PROMPT, prompt);
  }
}
```

## References

This skill includes detailed reference documentation:

### references/system-prompt.md
Complete system prompt for Excalidraw diagram generation:
- Task definition and workflow
- ExcalidrawElementSkeleton API documentation
- Best practices and visual style guidelines
- Image processing instructions

### references/chart-specs.md
Visual specifications for 23 chart types:
- Shape conventions for each type
- Layout guidelines
- Color schemes
- Connection patterns
- When to use each chart type

### references/examples.md
9 high-quality usage examples:
- Basic shapes
- Text and text containers
- Arrow bindings (auto-create and existing)
- Advanced typography
- Frame grouping
- Best practices summary

### references/arrow-optimization.md
Detailed documentation of the arrow optimization algorithm:
- Problem statement and solution approach
- Geometric algorithm explanation (quadrant-based)
- Edge center calculation formulas
- Special cases handling
- Performance characteristics
- Visual comparisons (before/after)
- Integration workflow

### references/json-repair-guide.md
Comprehensive JSON repair system documentation:
- Common LLM output issues catalog
- Three-tier repair strategy
- Core functions (repairJsonClosure, safeParseJsonWithRepair)
- State machine diagram
- Error handling best practices
- Testing scenarios
- Performance analysis

### references/implementation-guide.md
Complete end-to-end implementation guide:
- Architecture overview
- Step-by-step setup (dependencies, structure)
- Prompt system integration
- LLM client implementation
- Frontend/backend components
- Advanced features (history, config, export)
- Testing strategies
- Deployment instructions

## Scripts

This skill includes production-ready utility scripts:

### scripts/optimizeArrows.js
Arrow optimization algorithm implementation:
```javascript
export function optimizeExcalidrawCode(codeString) {
  // Parses Excalidraw JSON
  // Creates element ID map
  // Optimizes each arrow's connection points
  // Returns optimized JSON string
}
```

**Key Functions:**
- `determineEdges(startEle, endEle)` - Calculate optimal edge pairs
- `getEdgeCenter(element, edge)` - Get edge center coordinates
- `getStartEdgeCenter(startEle, endEle)` - Start element edge center
- `getEndEdgeCenter(endEle, startEle)` - End element edge center

### scripts/json-repair.js
JSON repair utilities implementation:
```javascript
export function repairJsonClosure(input) {
  // Strips code fences
  // Extracts JSON blocks
  // Repairs unclosed strings/brackets
  // Returns repaired JSON string
}

export function safeParseJsonWithRepair(input) {
  // Returns { ok, value, error }
  // Three-tier repair strategy
}
```

**Usage:**
```javascript
// Basic repair
const repaired = repairJsonClosure(llmOutput);
const elements = JSON.parse(repaired);

// Safe parse with error handling
const result = safeParseJsonWithRepair(llmOutput);
if (result.ok) {
  renderDiagram(result.value);
}

// Complete pipeline
const repaired = repairJsonClosure(llmOutput);
const optimized = optimizeExcalidrawCode(repaired);
renderDiagram(JSON.parse(optimized));
```

## Troubleshooting

### Issue: Generated JSON is malformed

**Solution**: Use JSON repair utilities to handle common LLM output issues:
- Strip Markdown code fences
- Close unbalanced brackets/braces
- Remove trailing commas
- Extract first valid JSON object/array

### Issue: Arrows not connecting properly

**Solution**: Ensure arrow bindings are properly specified:
```json
// Always include start/end bindings
{
  "type": "arrow",
  "start": { "type": "rectangle" },  // or { "id": "element-id" }
  "end": { "type": "ellipse" }
}
```

### Issue: Elements overlapping

**Solution**: Apply arrow optimization algorithm to calculate optimal connection points based on element geometry.

### Issue: Inconsistent visual style

**Solution**: When specifying a chart type, ensure the generated code strictly follows the visual specifications in `references/chart-specs.md`.

---

**Note**: This skill is based on the Smart Excalidraw Next project, which demonstrates production-ready implementation of AI-powered diagram generation with 23+ chart types, multi-modal input support, and robust JSON processing.
