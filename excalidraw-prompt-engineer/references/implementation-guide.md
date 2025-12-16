# Implementation Guide

## Complete End-to-End Implementation

This guide shows how to build a production-ready AI-powered Excalidraw diagram generator using the excalidraw-prompt-engineer skill.

## Architecture Overview

```
User Input (Text/File/Image)
         ↓
   Prompt Construction
         ↓
   LLM Generation (Streaming)
         ↓
   JSON Repair
         ↓
   Arrow Optimization
         ↓
   Excalidraw Rendering
```

## Step-by-Step Implementation

### Step 1: Setup and Dependencies

```bash
npm install @excalidraw/excalidraw
npm install jsonrepair  # Optional but recommended
```

**Project Structure:**
```
your-project/
├── lib/
│   ├── prompts.js           # From skill references
│   ├── optimizeArrows.js    # From skill scripts
│   └── json-repair.js       # From skill scripts
├── components/
│   ├── ExcalidrawCanvas.jsx
│   └── Chat.jsx
└── app/
    └── api/
        └── generate/
            └── route.js
```

### Step 2: Prompt System Setup

Create `lib/prompts.js`:

```javascript
// Copy from skill references/system-prompt.md
export const SYSTEM_PROMPT = `## 任务
根据用户的需求，基于 ExcalidrawElementSkeleton API 的规范...
`;

// Copy chart specs from references/chart-specs.md
const CHART_VISUAL_SPECS = {
  flowchart: `...`,
  mindmap: `...`,
  // ... all 23 chart types
};

export const USER_PROMPT_TEMPLATE = (userInput, chartType = 'auto') => {
  const promptParts = [];

  if (chartType && chartType !== 'auto') {
    const chartTypeName = CHART_TYPE_NAMES[chartType];
    promptParts.push(`请创建一个${chartTypeName}类型的 Excalidraw 图表。`);

    const visualSpec = CHART_VISUAL_SPECS[chartType];
    if (visualSpec) {
      promptParts.push(visualSpec.trim());
    }
  } else {
    promptParts.push(/* Auto mode prompt */);
  }

  promptParts.push(`用户需求：\n${userInput}`);
  return promptParts.join('\n\n');
};
```

### Step 3: Utility Functions Setup

Copy `scripts/optimizeArrows.js` and `scripts/json-repair.js` to your `lib/` directory.

Make sure to export the functions:
```javascript
// lib/json-repair.js
export { repairJsonClosure, safeParseJsonWithRepair };

// lib/optimizeArrows.js
export { optimizeExcalidrawCode };
```

### Step 4: LLM Integration

Create an LLM client that supports streaming:

```javascript
// lib/llm-client.js
export async function* generateExcalidrawCode(
  userInput,
  chartType = 'auto',
  provider = 'anthropic'
) {
  const systemPrompt = SYSTEM_PROMPT;
  const userPrompt = USER_PROMPT_TEMPLATE(userInput, chartType);

  const response = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'x-api-key': process.env.ANTHROPIC_API_KEY,
      'anthropic-version': '2023-06-01',
      'content-type': 'application/json',
    },
    body: JSON.stringify({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      stream: true,
      system: systemPrompt,
      messages: [{ role: 'user', content: userPrompt }],
    }),
  });

  const reader = response.body.getReader();
  const decoder = new TextDecoder();

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    const chunk = decoder.decode(value);
    const lines = chunk.split('\n');

    for (const line of lines) {
      if (line.startsWith('data: ')) {
        const data = line.slice(6);
        if (data === '[DONE]') continue;

        try {
          const parsed = JSON.parse(data);
          if (parsed.delta?.text) {
            yield parsed.delta.text;
          }
        } catch (e) {
          // Ignore parse errors
        }
      }
    }
  }
}
```

### Step 5: API Route (Next.js Example)

```javascript
// app/api/generate/route.js
import { SYSTEM_PROMPT, USER_PROMPT_TEMPLATE } from '@/lib/prompts';

export async function POST(req) {
  const { userInput, chartType } = await req.json();

  const systemPrompt = SYSTEM_PROMPT;
  const userPrompt = USER_PROMPT_TEMPLATE(userInput, chartType);

  // Create streaming response
  const encoder = new TextEncoder();
  const stream = new ReadableStream({
    async start(controller) {
      const response = await fetch('https://api.anthropic.com/v1/messages', {
        method: 'POST',
        headers: {
          'x-api-key': process.env.ANTHROPIC_API_KEY,
          'anthropic-version': '2023-06-01',
          'content-type': 'application/json',
        },
        body: JSON.stringify({
          model: 'claude-3-5-sonnet-20241022',
          max_tokens: 4096,
          stream: true,
          system: systemPrompt,
          messages: [{ role: 'user', content: userPrompt }],
        }),
      });

      const reader = response.body.getReader();
      const decoder = new TextDecoder();

      while (true) {
        const { done, value } = await reader.read();
        if (done) {
          controller.close();
          break;
        }

        const chunk = decoder.decode(value);
        controller.enqueue(encoder.encode(chunk));
      }
    },
  });

  return new Response(stream, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      Connection: 'keep-alive',
    },
  });
}
```

### Step 6: Frontend Component

```javascript
// components/DiagramGenerator.jsx
'use client';

import { useState } from 'react';
import { Excalidraw } from '@excalidraw/excalidraw';
import { repairJsonClosure } from '@/lib/json-repair';
import { optimizeExcalidrawCode } from '@/lib/optimizeArrows';

export default function DiagramGenerator() {
  const [excalidrawAPI, setExcalidrawAPI] = useState(null);
  const [input, setInput] = useState('');
  const [chartType, setChartType] = useState('auto');
  const [generating, setGenerating] = useState(false);

  const handleGenerate = async () => {
    setGenerating(true);
    let accumulatedCode = '';

    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ userInput: input, chartType }),
      });

      const reader = response.body.getReader();
      const decoder = new TextDecoder();

      while (true) {
        const { done, value } = await reader.read();
        if (done) break;

        const chunk = decoder.decode(value);
        const lines = chunk.split('\n');

        for (const line of lines) {
          if (line.startsWith('data: ')) {
            const data = line.slice(6);
            if (data === '[DONE]') continue;

            try {
              const parsed = JSON.parse(data);
              if (parsed.delta?.text) {
                accumulatedCode += parsed.delta.text;

                // Try to parse and display incrementally
                try {
                  const repaired = repairJsonClosure(accumulatedCode);
                  const elements = JSON.parse(repaired);
                  if (excalidrawAPI) {
                    excalidrawAPI.updateScene({ elements });
                  }
                } catch (e) {
                  // Continue accumulating
                }
              }
            } catch (e) {
              // Ignore parse errors
            }
          }
        }
      }

      // Final optimization
      const repairedCode = repairJsonClosure(accumulatedCode);
      const optimizedCode = optimizeExcalidrawCode(repairedCode);
      const finalElements = JSON.parse(optimizedCode);

      if (excalidrawAPI) {
        excalidrawAPI.updateScene({
          elements: finalElements,
          appState: { viewBackgroundColor: '#ffffff' },
        });
      }
    } catch (error) {
      console.error('Generation error:', error);
    } finally {
      setGenerating(false);
    }
  };

  return (
    <div className="flex h-screen">
      {/* Input Panel */}
      <div className="w-96 p-4 border-r">
        <h2 className="text-xl font-bold mb-4">Diagram Generator</h2>

        {/* Chart Type Selector */}
        <select
          value={chartType}
          onChange={(e) => setChartType(e.target.value)}
          className="w-full p-2 border rounded mb-4"
        >
          <option value="auto">Auto (智能选择)</option>
          <option value="flowchart">流程图</option>
          <option value="mindmap">思维导图</option>
          <option value="orgchart">组织架构图</option>
          {/* ... other chart types */}
        </select>

        {/* Input Area */}
        <textarea
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="描述你想要的图表..."
          className="w-full h-64 p-2 border rounded mb-4"
        />

        {/* Generate Button */}
        <button
          onClick={handleGenerate}
          disabled={generating || !input}
          className="w-full py-2 bg-blue-500 text-white rounded disabled:bg-gray-300"
        >
          {generating ? '生成中...' : '生成图表'}
        </button>
      </div>

      {/* Excalidraw Canvas */}
      <div className="flex-1">
        <Excalidraw
          excalidrawAPI={(api) => setExcalidrawAPI(api)}
          initialData={{
            appState: { viewBackgroundColor: '#ffffff' },
          }}
        />
      </div>
    </div>
  );
}
```

### Step 7: Image Upload Support (Multimodal)

For image input support:

```javascript
// lib/image-utils.js
export async function encodeImageToBase64(file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = () => {
      const base64 = reader.result.split(',')[1];
      resolve(base64);
    };
    reader.onerror = reject;
    reader.readAsDataURL(file);
  });
}

// Update API route to handle images
export async function POST(req) {
  const { userInput, chartType, imageBase64 } = await req.json();

  const systemPrompt = SYSTEM_PROMPT;
  const userPrompt = USER_PROMPT_TEMPLATE(userInput, chartType);

  // Build message content
  const content = [];
  if (imageBase64) {
    content.push({
      type: 'image',
      source: {
        type: 'base64',
        media_type: 'image/jpeg',
        data: imageBase64,
      },
    });
  }
  content.push({ type: 'text', text: userPrompt });

  // Rest of streaming logic...
}
```

## Advanced Features

### Feature 1: History Management

```javascript
// lib/history-manager.js
export class HistoryManager {
  constructor() {
    this.history = this.loadFromStorage();
  }

  add(entry) {
    this.history.unshift({
      id: Date.now(),
      timestamp: new Date().toISOString(),
      userInput: entry.userInput,
      chartType: entry.chartType,
      code: entry.code,
      config: entry.config,
    });

    // Keep last 50 entries
    if (this.history.length > 50) {
      this.history = this.history.slice(0, 50);
    }

    this.saveToStorage();
  }

  loadFromStorage() {
    if (typeof window === 'undefined') return [];
    const stored = localStorage.getItem('excalidraw_history');
    return stored ? JSON.parse(stored) : [];
  }

  saveToStorage() {
    if (typeof window === 'undefined') return;
    localStorage.setItem('excalidraw_history', JSON.stringify(this.history));
  }
}
```

### Feature 2: Configuration Management

```javascript
// lib/config-manager.js
export class ConfigManager {
  constructor() {
    this.configs = this.loadConfigs();
  }

  addConfig(config) {
    this.configs.push({
      id: Date.now(),
      name: config.name,
      provider: config.provider,
      model: config.model,
      apiKey: config.apiKey,
      active: false,
    });
    this.saveConfigs();
  }

  setActive(configId) {
    this.configs.forEach((c) => {
      c.active = c.id === configId;
    });
    this.saveConfigs();
  }

  getActiveConfig() {
    return this.configs.find((c) => c.active);
  }

  loadConfigs() {
    if (typeof window === 'undefined') return [];
    const stored = localStorage.getItem('llm_configs');
    return stored ? JSON.parse(stored) : [];
  }

  saveConfigs() {
    if (typeof window === 'undefined') return;
    localStorage.setItem('llm_configs', JSON.stringify(this.configs));
  }
}
```

### Feature 3: Export/Import

```javascript
// components/ExportImport.jsx
export function ExportButton({ excalidrawAPI }) {
  const handleExport = () => {
    const elements = excalidrawAPI.getSceneElements();
    const appState = excalidrawAPI.getAppState();

    const exportData = {
      type: 'excalidraw',
      version: 2,
      source: 'smart-excalidraw',
      elements,
      appState,
    };

    const blob = new Blob([JSON.stringify(exportData, null, 2)], {
      type: 'application/json',
    });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `diagram-${Date.now()}.excalidraw`;
    a.click();
  };

  return <button onClick={handleExport}>Export</button>;
}
```

## Testing

### Unit Tests

```javascript
// __tests__/optimizeArrows.test.js
import { optimizeExcalidrawCode } from '@/lib/optimizeArrows';

describe('optimizeExcalidrawCode', () => {
  it('should optimize arrow connections', () => {
    const input = JSON.stringify([
      { type: 'rectangle', id: 'rect1', x: 0, y: 0, width: 100, height: 100 },
      { type: 'rectangle', id: 'rect2', x: 200, y: 0, width: 100, height: 100 },
      {
        type: 'arrow',
        x: 100,
        y: 50,
        width: 100,
        height: 0,
        start: { id: 'rect1' },
        end: { id: 'rect2' },
      },
    ]);

    const output = optimizeExcalidrawCode(input);
    const elements = JSON.parse(output);
    const arrow = elements.find((e) => e.type === 'arrow');

    expect(arrow.x).toBe(100); // Right edge of rect1
    expect(arrow.y).toBe(50); // Center height
    expect(arrow.width).toBe(100); // Distance to left edge of rect2
  });
});
```

```javascript
// __tests__/json-repair.test.js
import { repairJsonClosure } from '@/lib/json-repair';

describe('repairJsonClosure', () => {
  it('should repair unclosed braces', () => {
    const broken = '[{"type": "rectangle", "x": 100';
    const fixed = repairJsonClosure(broken);
    expect(() => JSON.parse(fixed)).not.toThrow();
  });

  it('should strip code fences', () => {
    const input = '```json\n[{"type": "rect"}]\n```';
    const fixed = repairJsonClosure(input);
    expect(fixed).toBe('[{"type": "rect"}]');
  });
});
```

## Performance Optimization

### 1. Debounced Preview Updates

```javascript
import { useMemo } from 'react';
import debounce from 'lodash/debounce';

const debouncedUpdate = useMemo(
  () =>
    debounce((elements) => {
      if (excalidrawAPI) {
        excalidrawAPI.updateScene({ elements });
      }
    }, 100),
  [excalidrawAPI]
);
```

### 2. Web Worker for Heavy Processing

```javascript
// workers/optimize.worker.js
import { optimizeExcalidrawCode } from '@/lib/optimizeArrows';

self.onmessage = (e) => {
  const { code } = e.data;
  const optimized = optimizeExcalidrawCode(code);
  self.postMessage({ optimized });
};

// Usage in component
const worker = new Worker(new URL('./workers/optimize.worker.js', import.meta.url));
worker.postMessage({ code: accumulatedCode });
worker.onmessage = (e) => {
  const { optimized } = e.data;
  // Update UI
};
```

## Deployment

### Environment Variables

```bash
# .env.local
ANTHROPIC_API_KEY=your_api_key_here
OPENAI_API_KEY=your_openai_key_here
```

### Vercel Deployment

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel --prod
```

### Docker Deployment

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build
CMD ["npm", "start"]
```

## References

- System Prompt: `references/system-prompt.md`
- Chart Specifications: `references/chart-specs.md`
- Examples: `references/examples.md`
- Arrow Optimization: `references/arrow-optimization.md`
- JSON Repair: `references/json-repair-guide.md`
- Scripts: `scripts/optimizeArrows.js`, `scripts/json-repair.js`
