# Explain Code Skill

A Claude skill that teaches Claude to explain code using visual diagrams and analogies.

## What This Skill Does

When invoked, this skill will:

1. **Start with an analogy** - Compare code to everyday life concepts
2. **Draw a diagram** - Use ASCII art to show flow, structure, or relationships
3. **Walk through the code** - Explain step-by-step what happens
4. **Highlight gotchas** - Point out common mistakes or misconceptions

## Installation

Copy this skill to your personal Claude skills folder:

```bash
# macOS/Linux
cp -r explain-code ~/.claude/skills/

# Windows
xcopy explain-code %USERPROFILE%\.claude\skills\ /E /I
```

## Usage

### Automatic Invocation

Claude will automatically use this skill when you ask questions like:

- "How does this code work?"
- "Explain this function"
- "What's going on in this file?"
- "Help me understand this component"

### Manual Invocation

Invoke the skill directly:

```
/explain-code src/auth/login.ts
```

### Examples

See `examples.md` for sample outputs showing the expected format.

## Customization

Modify `SKILL.md` to adjust:
- Description (triggers automatic invocation)
- Explanation style
- Gotcha emphasis
- Diagram complexity

## Tips for Best Results

- Be specific about which code you want explained
- Mention your familiarity level (beginner/advanced)
- Ask follow-up questions to dive deeper
- Use this when teaching others or documenting code
