# JSON Repair Guide

## Overview

The JSON repair system handles common issues in LLM-generated JSON output, making the Excalidraw code generation robust and production-ready.

## Common LLM Output Issues

### 1. Markdown Code Fences

**Problem:**
```
```json
[{"type": "rectangle", "x": 100}]
```
```

**Solution:** Strip leading/trailing code fences
```javascript
s = s.replace(/^```(?:json|javascript|js)?\s*\n?/i, '');
s = s.replace(/\n?```\s*$/i, '');
```

### 2. Mixed Text with JSON

**Problem:**
```
Here's the diagram:
[{"type": "rectangle"}]
Hope this helps!
```

**Solution:** Extract first JSON block (object or array)
```javascript
for (let i = 0; i < source.length; i++) {
  const c = source[i];
  if (c === '{' || c === '[') { start = i; break; }
}
```

### 3. Unclosed Strings

**Problem:**
```json
[{"text": "Hello World}]
          ↑ missing quote
```

**Solution:** Track string state and close if needed
```javascript
if (inString) {
  out += '"';
  inString = false;
}
```

### 4. Unclosed Brackets/Braces

**Problem:**
```json
[{"type": "rectangle", "x": 100
                                ↑ missing }]
```

**Solution:** Maintain stack and append missing closers
```javascript
while (stack.length) out += stack.pop();
```

### 5. Missing Object Braces in Arrays

**Problem:**
```json
["type": "rectangle", "x": 100]
 ↑ should be [{"type": "rectangle", "x": 100}]
```

**Solution:** Detect property pattern after '[' and insert '{'
```javascript
if (ch === '[') {
  stack.push(']');
  const nextIdx = findNextNonWsIndex(source, i + 1);
  if (looksLikeMissingObjectAfterArray(source, nextIdx)) {
    out += '{';
    stack.push('}');
  }
}
```

### 6. Trailing Commas

**Problem:**
```json
[{"type": "rectangle"},]
                      ↑ trailing comma before ]
```

**Solution:** Remove trailing comma before appending closers
```javascript
function trimTrailingComma(out) {
  let i = out.length - 1;
  while (i >= 0 && /\s/.test(out[i])) i--;
  if (i >= 0 && out[i] === ',') {
    return out.slice(0, i) + out.slice(i + 1);
  }
  return out;
}
```

## Core Functions

### repairJsonClosure(input)

Main repair function that handles all common issues.

**Algorithm:**
1. Strip code fences
2. Find first JSON start ({ or [)
3. Parse character by character
4. Track string state and escape sequences
5. Maintain bracket/brace stack
6. Detect and fix missing object braces in arrays
7. Close unclosed strings
8. Trim trailing comma
9. Append missing closers
10. Fall back to robust repair library if available

**Example:**

```javascript
const broken = '```json\n[{"text": "Hello}\n```';
const fixed = repairJsonClosure(broken);
// Result: '[{"text": "Hello"}]'
```

### safeParseJsonWithRepair(input)

Safe parsing with automatic repair fallback.

**Returns:** `{ ok: boolean, value?: any, error?: Error }`

**Usage:**

```javascript
const result = safeParseJsonWithRepair(llmOutput);
if (result.ok) {
  console.log('Parsed:', result.value);
} else {
  console.error('Failed:', result.error);
}
```

## State Machine

The repair algorithm uses a state machine to track parsing context:

```
States:
- inString: Currently inside a quoted string
- escape: Previous character was backslash
- stack: Array of expected closing characters

Transitions:
- See " → Toggle inString (unless escaped)
- See \ → Set escape flag (only in string)
- See { → Push } to stack
- See [ → Push ] to stack (+ check for missing object)
- See } or ] → Pop from stack if matches
```

## Helper Functions

### stripCodeFences(text)

Removes Markdown code fence markers.

### trimTrailingComma(out)

Removes trailing comma before closing brackets.

### findNextNonWsIndex(str, from)

Finds next non-whitespace character index.

### looksLikeMissingObjectAfterArray(str, from)

Heuristic to detect missing '{' after '['

**Detection Pattern:**
```javascript
[                     // Array start
  "key": value        // Property pattern (missing { before "key")
]
```

Returns true if sees `"key":` pattern before `,` or `]`.

### hasColonBeforeCommaOrBracket(str, from)

Checks if colon appears before comma or bracket (indicates property).

## Fallback Strategy

The system uses a **three-tier repair strategy**:

### Tier 1: Direct Parse (Fastest)
```javascript
try {
  return { ok: true, value: JSON.parse(input) };
} catch (_) { /* continue */ }
```

### Tier 2: Custom Repair (Fast)
```javascript
try {
  const repaired = repairJsonClosure(input);
  return { ok: true, value: JSON.parse(repaired) };
} catch (_) { /* continue */ }
```

### Tier 3: Robust Library (Comprehensive)
```javascript
if (jsonRepairLib) {
  try {
    const repaired = jsonRepairLib(input);
    return { ok: true, value: JSON.parse(repaired) };
  } catch (_) { /* fallthrough */ }
}
```

**jsonRepairLib** is the `jsonrepair` npm package, loaded lazily:
```javascript
const req = (0, eval)('require');
const mod = req?.('jsonrepair');
jsonRepairLib = mod?.jsonrepair || mod?.default || null;
```

## Performance Characteristics

- **Time Complexity:** O(n) where n = input length
  - Single pass through input string
  - Constant time operations per character

- **Space Complexity:** O(n)
  - Output string (worst case: input + missing closers)
  - Stack for bracket tracking (max depth ≈ nesting level)

## Integration Workflow

```javascript
async function generateDiagram(userInput) {
  // 1. Generate with LLM
  let code = '';
  const stream = await llmClient.generate(SYSTEM_PROMPT, userPrompt);

  for await (const chunk of stream) {
    code += chunk;

    // 2. Try to parse incrementally
    const result = safeParseJsonWithRepair(code);
    if (result.ok) {
      // 3. Display partial result
      updatePreview(result.value);
    }
  }

  // 4. Final repair and optimization
  const finalRepaired = repairJsonClosure(code);
  const optimized = optimizeExcalidrawCode(finalRepaired);

  // 5. Render final diagram
  renderDiagram(JSON.parse(optimized));
}
```

## Error Handling

The repair system is designed to be **conservative** and **safe**:

- ✅ Only appends missing closers (never removes valid content)
- ✅ Preserves original input if no JSON start found
- ✅ Returns original on catastrophic failure
- ✅ No exceptions thrown (always returns a string)

**Example:**

```javascript
const noJson = "This is just plain text";
const result = repairJsonClosure(noJson);
// Returns: "This is just plain text" (unchanged)
```

## Testing Scenarios

### Scenario 1: Streaming Generation

LLM outputs JSON progressively:

```javascript
Step 1: "[{"type": "rec"
Step 2: "[{"type": "rectangle"
Step 3: "[{"type": "rectangle", "x"
Step 4: "[{"type": "rectangle", "x": 100
Step 5: "[{"type": "rectangle", "x": 100}"
Step 6: "[{"type": "rectangle", "x": 100}]"
```

Each step can be repaired and parsed individually.

### Scenario 2: Mixed Content

```javascript
const mixed = `
Let me create that diagram:

\`\`\`json
[
  {"type": "rectangle", "x": 100, "y": 200},
  {"type": "ellipse", "x": 300, "y": 200}
]
\`\`\`

Does this look good?
`;

const result = repairJsonClosure(mixed);
// Extracts and returns just the JSON array
```

### Scenario 3: Malformed Array-of-Objects

```javascript
const broken = '["type": "rectangle", "x": 100]';
const fixed = repairJsonClosure(broken);
// Result: '[{"type": "rectangle", "x": 100}]'
```

## Best Practices

1. **Always use repair before parsing**
   ```javascript
   const code = repairJsonClosure(llmOutput);
   const elements = JSON.parse(code);
   ```

2. **Use safe parse for error handling**
   ```javascript
   const result = safeParseJsonWithRepair(llmOutput);
   if (!result.ok) {
     console.error('Parse failed:', result.error);
   }
   ```

3. **Process streaming incrementally**
   ```javascript
   for await (const chunk of stream) {
     accumulated += chunk;
     const result = safeParseJsonWithRepair(accumulated);
     if (result.ok) updatePreview(result.value);
   }
   ```

4. **Chain with optimization**
   ```javascript
   const pipeline = (llmOutput) => {
     const repaired = repairJsonClosure(llmOutput);
     const optimized = optimizeExcalidrawCode(repaired);
     return JSON.parse(optimized);
   };
   ```

## References

- Implementation: `scripts/json-repair.js`
- Related: `references/arrow-optimization.md` for post-repair optimization
- npm package: [jsonrepair](https://www.npmjs.com/package/jsonrepair) (optional fallback)
