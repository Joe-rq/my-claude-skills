---
name: text-to-json
description: Converts natural language descriptions into sophisticated, hierarchical JSON structures. Follows the "Prompting with JSON" philosophy to ensure depth, granularity, and repeatability. Usage: when you need to structure a request, create a config, or define a complex object from a simple description.
---

# Text to JSON Converter

## Overview

This skill converts "vernacular" (plain, natural language) descriptions into high-fidelity, structured JSON prompts. It is designed to capture not just the explicit requirements, but to expand on the *implicit* details, adding depth and granularity that might be missing from the initial request.

Based on the [fofr.ai Prompting with JSON](https://www.fofr.ai/prompting-with-json) philosophy, this skill treats JSON as a flexible, hierarchical template for defining complex objects or instructions.

## Core Philosophy

1.  **Hierarchical Structure**: Break down entities into nested objects (e.g., `subject.face.skin.texture` instead of just `skin_texture`).
2.  **Granularity & Depth**: Expand simple terms into detailed field sets. A "red car" becomes a structure defining paint type, finish, undertones, and condition.
3.  **Dynamic Schema**: The JSON structure is not fixed, but often follows a standard template (Subject, Background, Style, Technical, etc.).
4.  **Interactive Refinement**: Use the structured format to identify gaps in definition and prompt the user for necessary details.

## Workflow

### 1. Analysis & Expansion
*   Analyze the user's input.
*   Identify the core subject/intent.
*   **Expand**: Where the user is vague, infer standard attributes for that domain or note them as "missing/to be clarified".
*   **Constraints**: Identify what MUST be kept and what must be AVOIDED.

### 2. JSON Construction
Generate a valid JSON object. While the schema is dynamic, use these standard top-level keys when applicable:

*   **subject**: Demographics, face, hair, body, pose, attire, core object details.
*   **background**: Color, texture, depth, setting details.
*   **style**: Medium, artistic reference, color palette (e.g., `aesthetic_fidelity`).
*   **technical**: Camera specs, resolution, software settings.
*   **lighting**: Type, source, contrast, details.
*   **post_processing**: Grading, look, filters.
*   **constraints**: Object with `must_keep` and `avoid` arrays.
*   **negative_prompt**: Array of specific visual elements to exclude (if generating an image prompt).

### 3. Missing Information / Clarification
After the JSON block, provide a **Contextual Analysis**:
*   **Ambiguities**: What terms were vague?
*   **Missing Granularity**: What details would make this definition more robust? (e.g., "You mentioned 'studio lighting', but didn't specify the key light position or ratio.")
*   **Logic Gaps**: Are there conflicting constraints?

## Example

**User Input**:
"I want a picture of a cyberpunk street samurai, raining, neon lights."

**Output**:

```json
{
  "subject": {
    "archetype": "Street Samurai",
    "style": "Cyberpunk",
    "apparel": {
      "type": "Tech-wear",
      "materials": ["Synthetic fiber", "Carbon fiber plating"],
      "condition": "Worn, rain-slicked"
    },
    "cybernetics": {
      "visible": true,
      "style": "Industrial makeshift",
      "details": "Glowing optical implant"
    }
  },
  "background": {
    "setting": "Dystopian city street",
    "weather": "Heavy Rain",
    "atmosphere": "Noir, oppressive"
  },
  "lighting": {
    "sources": ["Neon signs", "Holographic billboards"],
    "colors": ["Cyan", "Magenta", "Acid Green"],
    "quality": "Reflective, high contrast, wet surface reflections"
  },
  "aesthetic_fidelity": {
    "medium": "Digital Art",
    "finish": "Cinematic",
    "vibe": "Gritty, futuristic, high-tech low-life"
  },
  "constraints": {
    "must_keep": ["Rain", "Neon aesthetic", "Samurai sword/weapon"],
    "avoid": ["Daytime", "Clean/Sterile look", "Anime style"]
  },
  "negative_prompt": [
    "daylight",
    "sunshine",
    "clean clothes",
    "blue sky",
    "cartoon",
    "low resolution",
    "blurry"
  ]
}
```

**Refinement Strategy**:
*   **Subject**: I inferred "Tech-wear" and "Carbon fiber". Is that the specific look you wanted?
*   **Cybernetics**: Should the implants be subtle or overt?
*   **Weaponry**: A "Samurai" usually implies a sword (Katana). Should this be traditional metal or energy-based?

## Usage Instructions

When triggering this skill:
1.  Read the user's vernacular input.
2.  Output the JSON block first using the hierarchical template.
3.  Follow with the Refinement Strategy questions.
