# Chart Type Visual Specifications

This document contains visual specifications for 23 different chart types supported by the Excalidraw prompt engineer.

## Chart Types Overview

| Chart Type | Chinese Name | Best For |
|------------|--------------|----------|
| flowchart | 流程图 | Processes, steps, decision logic |
| mindmap | 思维导图 | Concept relationships, knowledge structure, brainstorming |
| orgchart | 组织架构图 | Organizational structure, hierarchical relationships |
| sequence | 时序图 | System interactions, message passing, time sequences |
| class | UML类图 | Class structures, inheritance, object-oriented design |
| er | ER图 | Database entity relationships, data models |
| gantt | 甘特图 | Project schedules, task timelines |
| timeline | 时间线 | Historical events, development history |
| tree | 树形图 | Hierarchical structures, classification relationships |
| network | 网络拓扑图 | Network structures, node connections |
| architecture | 架构图 | System architecture, tech stack, layered design |
| dataflow | 数据流图 | Data flow, processing stages |
| state | 状态图 | State transitions, lifecycles |
| swimlane | 泳道图 | Cross-department processes, responsibility division |
| concept | 概念图 | Concept relationships, knowledge graphs |
| fishbone | 鱼骨图 | Cause-effect analysis, root cause analysis |
| swot | SWOT分析图 | SWOT analysis, strategic planning |
| pyramid | 金字塔图 | Hierarchical structures, priorities |
| funnel | 漏斗图 | Conversion processes, filtering stages |
| venn | 韦恩图 | Set relationships, intersections and unions |
| matrix | 矩阵图 | Multi-dimensional comparisons, relationship matrices |
| infographic | 信息图 | Data visualization, information display, creative charts |

---

## 1. Flowchart (流程图)

### Visual Specifications
- **Shape conventions**: Use `ellipse` for start/end, `rectangle` for processing steps, `diamond` for decisions
- **Connections**: Use `arrow` to connect nodes, arrows must bind to elements
- **Layout**: Top-to-bottom or left-to-right flow, maintain clear process direction
- **Colors**: Use blue tones as main color, decision points can be highlighted in orange

---

## 2. Mindmap (思维导图)

### Visual Specifications
- **Structure**: Central theme uses `ellipse`, branches use `rectangle`
- **Hierarchy**: Express hierarchy through size and color depth
- **Layout**: Radial layout, main branches evenly distributed around center
- **Colors**: Each main branch uses different color scheme for easy theme distinction

---

## 3. Org Chart (组织架构图)

### Visual Specifications
- **Shapes**: Uniformly use `rectangle` for personnel or positions
- **Hierarchy**: Express rank through color depth and size
- **Layout**: Strict tree hierarchy, top-to-bottom
- **Connections**: Use `arrow` vertically downward to connect superior-subordinate relationships

---

## 4. Sequence Diagram (时序图)

### Visual Specifications
- **Participants**: Top row uses `rectangle` for each participant
- **Lifelines**: Use dashed `line` extending downward from participants
- **Messages**: Use `arrow` for message passing, `label` for message content
- **Layout**: Participants arranged horizontally, messages top-to-bottom by time

---

## 5. UML Class Diagram (UML类图)

### Visual Specifications
- **Classes**: Use `rectangle` divided into three parts (class name, attributes, methods)
- **Relationships**: Inheritance uses hollow triangle arrow, association uses normal arrow, aggregation/composition uses diamond arrow
- **Layout**: Parent classes on top, child classes below, related classes arranged horizontally

---

## 6. ER Diagram (ER图)

### Visual Specifications
- **Entities**: Use `rectangle` for entities
- **Attributes**: Use `ellipse` for attributes, primary keys can be marked with special style
- **Relationships**: Use `diamond` for relationships, connect with `arrow`
- **Cardinality**: Mark relationship cardinality on connecting lines (1, N, M, etc.)

---

## 7. Gantt Chart (甘特图)

### Visual Specifications
- **Timeline**: Mark time scale at top
- **Task bars**: Use `rectangle` for tasks, length represents time span
- **Status**: Use different colors to distinguish task status (not started, in progress, completed)
- **Layout**: Tasks arranged vertically, time expands horizontally

---

## 8. Timeline (时间线)

### Visual Specifications
- **Main axis**: Use `line` as time main axis
- **Nodes**: Use `ellipse` to mark time nodes
- **Events**: Use `rectangle` to display event content
- **Layout**: Timeline centered, event cards staggered on both sides

---

## 9. Tree Diagram (树形图)

### Visual Specifications
- **Nodes**: Root node uses `ellipse`, other nodes use `rectangle`
- **Hierarchy**: Express hierarchy depth through color gradient
- **Connections**: Use `arrow` from parent node to child nodes
- **Layout**: Root node at top, child nodes evenly distributed

---

## 10. Network Topology (网络拓扑图)

### Visual Specifications
- **Devices**: Different device types use different shapes (`rectangle`, `ellipse`, `diamond`)
- **Hierarchy**: Distinguish device importance through color and size
- **Connections**: Use `line` for network connections, line width can represent bandwidth
- **Layout**: Core devices centered, other devices grouped by hierarchy or function

---

## 11. Architecture Diagram (架构图)

### Visual Specifications
- **Layering**: Use `rectangle` to distinguish different layers (presentation, business, data layers, etc.)
- **Components**: Use `rectangle` for components or services
- **Layout**: Layered layout, top-to-bottom

---

## 12. Data Flow Diagram (数据流图)

### Visual Specifications
- **Entities**: External entities use `rectangle`, processes use `ellipse`
- **Storage**: Data stores use special-styled `rectangle`
- **Data flow**: Use `arrow` for data flow direction, `label` for data name
- **Layout**: External entities at edges, processes centered

---

## 13. State Diagram (状态图)

### Visual Specifications
- **States**: Use rounded `rectangle` for states
- **Initial/Final**: Initial state uses solid circle, final state uses double circle
- **Transitions**: Use `arrow` for state transitions, `label` for trigger conditions
- **Layout**: Arrange by logical flow of state transitions

---

## 14. Swimlane Diagram (泳道图)

### Visual Specifications
- **Swimlanes**: Use `rectangle` or `frame` to divide swimlanes, each representing a role or department
- **Activities**: Use `rectangle` for activities, `diamond` for decisions
- **Process**: Use `arrow` to connect activities, can cross swimlanes
- **Layout**: Swimlanes arranged in parallel, activities in chronological order

---

## 15. Concept Map (概念图)

### Visual Specifications
- **Concepts**: Core concepts use `ellipse`, other concepts use `rectangle`
- **Relationships**: Use `arrow` to connect concepts, `label` for relationship type
- **Hierarchy**: Express concept importance through size and color
- **Layout**: Core concept centered, related concepts distributed around

---

## 16. Fishbone Diagram (鱼骨图)

### Visual Specifications
- **Main bone**: Use thick `arrow` as main bone, pointing to problem or result
- **Branches**: Use `arrow` as branches, diagonally connecting to main bone
- **Categories**: Main branches use different colors to distinguish categories
- **Layout**: Left-to-right, branches alternately distributed above and below main bone

---

## 17. SWOT Analysis (SWOT分析图)

### Visual Specifications
- **Four quadrants**: Use `rectangle` to create four quadrants
- **Categories**: Strengths (S), Weaknesses (W), Opportunities (O), Threats (T) use different colors
- **Content**: List relevant points in each quadrant
- **Layout**: 2x2 matrix layout, four equal quadrants

---

## 18. Pyramid Diagram (金字塔图)

### Visual Specifications
- **Hierarchy**: Use `rectangle` for each layer, width increases top-to-bottom
- **Colors**: Use gradient colors to express hierarchical relationships
- **Layout**: Vertically center-aligned, forming pyramid shape

---

## 19. Funnel Diagram (漏斗图)

### Visual Specifications
- **Hierarchy**: Use `rectangle` for each stage, width decreases top-to-bottom
- **Data**: Mark quantity or percentage for each layer
- **Colors**: Use gradient colors to represent conversion process
- **Layout**: Vertically centered, forming funnel shape

---

## 20. Venn Diagram (韦恩图)

### Visual Specifications
- **Sets**: Use `ellipse` for sets, partially overlapping
- **Colors**: Use semi-transparent background colors, intersection areas naturally blend colors
- **Labels**: Mark set names and elements
- **Layout**: Circles appropriately overlapped, forming obvious intersection areas

---

## 21. Matrix Diagram (矩阵图)

### Visual Specifications
- **Grid**: Use `rectangle` to create row-column grid
- **Headers**: Use dark background to distinguish headers
- **Data**: Cells can use color depth to represent numeric values
- **Layout**: Regular matrix structure, rows and columns aligned

---

## 22. Infographic (信息图)

### Visual Specifications
- **Modularization**: Use `frame` and `rectangle` to create independent information modules
- **Visual hierarchy**: Establish clear information hierarchy through size, color, and position
- **Data visualization**: Include charts, icons, numbers and other visual elements
- **Rich colors**: Use multiple colors to distinguish different information modules, maintain visual appeal
- **Text-graphics integration**: Tightly combine text with graphic elements, improve information delivery efficiency
- **Flexible layout**: Can adopt grid, card, or free layout based on content needs

---

## 23. Auto Mode

When `chartType` is set to `auto`, the AI will intelligently select the most appropriate chart type(s) based on user requirements. The selection guide includes:

1. Analyze the core content and goals of user requirements
2. Choose the chart type(s) that can most clearly express information (can be one or multiple combined)
3. If a specific chart type is selected, strictly follow that type's visual specifications
4. Ensure the chart can independently convey information with clear and beautiful layout
