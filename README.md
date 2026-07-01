# AI Component Metadata (YAML) — Claude Code Skill

A [Claude Code](https://claude.ai/code) skill that generates structured, AI-readable metadata for design system components. Metadata is written as `.metadata.yaml` files, co-located with component source code, and validated against a JSON Schema.

**Based on** [ai-component-metadata](https://github.com/cris-achiardi/claude-skills/tree/main/skills/ai-component-metadata) by Cristian Morales. Converted to YAML for ~25% token reduction on deeply nested component data.

---

## What it does

When you ask Claude to generate or update component metadata, this skill instructs it to produce a `.metadata.yaml` file for each component. That file tells AI agents:

- **When** to reach for this component (use cases, keywords, priority)
- **How** to use it correctly (required/optional props, composition patterns)
- **What to avoid** (anti-patterns with reasons and alternatives)
- **How it behaves** (states, interactions, responsive rules)
- **Accessibility requirements** (ARIA role, keyboard support, WCAG level)
- **Available variants** and what each one is for

For multi-step UI flows and composition constraints that span multiple components (e.g. "a checkout flow may only have one primary CTA visible at a time"), see the sibling [ai-pattern-metadata-yaml](https://github.com/robindicapua/ai-pattern-metadata-yaml) skill instead — that content model is laws-first, not hints-first, and doesn't fit this schema.

---

## Why YAML over JSON?

| | JSON | YAML |
|---|---|---|
| Token cost | Baseline | ~25% fewer tokens |
| Multi-line code examples | Escaped `\n` strings | YAML block scalars (`\|`) |
| Schema validation | External tooling required | Inline via `yaml-language-server` comment |
| Human readability | Moderate | High |
| Machine readability | Universal | Universal |

The savings compound on deeply nested component data and are meaningful at scale when AI agents load metadata for a full component library in a single context window.

---

## Prerequisites

- [Claude Code](https://claude.ai/code) CLI installed
- A design system with components in a consistent folder structure
- (Optional) [Red Hat YAML](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml) VS Code extension for IDE validation and autocomplete

---

## Installation

### 1. Copy the skill into your project

Place this skill folder anywhere in your repo. The conventional location is:

```
.agent/skills/ai-component-metadata-yaml/
```

If you use a different path, update the schema references described in the [Customization](#customization) section below.

### 2. Tell Claude Code about it

In your project's `CLAUDE.md` or `AGENTS.md`, add a line pointing to your skills directory:

```markdown
Before starting any task, check if a relevant skill exists in `.agent/skills/`.
If one matches, read it and follow its instructions.
```

Claude Code reads `CLAUDE.md` automatically at session start, so this is enough — no plugin configuration required.

### 3. (Optional) Enable IDE validation

Add the schema mapping to `.vscode/settings.json` so the `yaml-language-server` comment works without the Red Hat extension needing to resolve relative paths:

```json
{
  "yaml.schemas": {
    ".agent/skills/ai-component-metadata-yaml/schemas/component-metadata.schema.json": [
      "**/*.metadata.yaml"
    ],
    ".agent/skills/ai-component-metadata-yaml/schemas/pattern-metadata.schema.json": [
      "**/*.pattern.yaml"
    ]
  }
}
```

---

## Usage

Ask Claude to generate metadata for a component:

```
Generate metadata for the Button component.
```

Or for a whole batch:

```
Generate metadata files for all components in packages/ui/src/components/.
```

Claude will co-locate a `.metadata.yaml` file next to each component source file and validate it against the schema.

### Example output

```yaml
# yaml-language-server: $schema=../../../../../.agent/skills/ai-component-metadata-yaml/schemas/component-metadata.schema.json
component:
  name: Button
  category: atoms
  description: Triggers an action or navigates to a destination.
  type: interactive
  path: packages/ui/src/components/button/button.tsx

usage:
  useCases:
    - form submission
    - primary page action
    - navigation trigger
  requiredProps:
    - children
  commonPatterns:
    - name: primary-action
      description: The main call to action on a page or section.
      composition: |
        <Button variant="filled" size="md">
          Get started
        </Button>
  antiPatterns:
    - scenario: Using Button for a link that navigates away from the app
      reason: Misleads screen readers and breaks keyboard navigation expectations.
      alternative: Use an anchor tag or a Link component with button styling.

accessibility:
  role: button
  keyboardSupport: Enter and Space activate the button.
  screenReader: Reads the button label. Use aria-label when the label is icon-only.
  focusManagement: standard
  wcag: AA

aiHints:
  priority: high
  keywords: [button, cta, action, submit, click, trigger]
  context: >
    Reach for Button whenever the user needs to trigger an action.
    Prefer filled variant for primary actions, outline for secondary.
    Never use Button for navigation — use a Link component instead.
```

---

## Schemas

One schema ships with this skill.

### `component-metadata.schema.json`

For individual components. Required sections: `component`, `usage`, `aiHints`.

| Section | Required | Purpose |
|---|---|---|
| `component` | Yes | Name, category (atoms/molecules/organisms), functional type, source path |
| `component.figma` | No | Figma `fileKey`, `nodeId`, and `componentKey` — lets AI agents and Code Connect tools resolve the exact component in Figma |
| `usage` | Yes | Use cases, props, composition patterns, anti-patterns |
| `aiHints` | Yes | Priority (high/medium/low), intent-matching keywords, natural language context |
| `composition` | No | Slots, nested sub-components, common partners, parent constraints, npm dependencies |
| `behavior` | No | States, event interactions, responsive rules |
| `accessibility` | No | ARIA role, keyboard support, screen reader, focus management, WCAG level |
| `variants` | No | Variant dimensions and their options |

**Functional types:** `interactive`, `display`, `container`, `input`, `navigation`

**Atomic Design categories:** `atoms`, `molecules`, `organisms`

#### Figma fields

| Field | Where to find it |
|---|---|
| `fileKey` | The segment after `/design/` in any Figma file URL: `figma.com/design/FILE_KEY/...` |
| `nodeId` | The `?node-id=` value in the URL when you select a component. Use `1:234` (colon) format for the REST API, `1-234` (hyphen) as it appears in the URL |
| `componentKey` | Stable across renames and file moves. Retrieve via the Figma REST API (`GET /v1/files/:key/components`) or from your Figma MCP tool |

```yaml
component:
  name: Button
  figma:
    fileKey: "abc123XYZ"
    nodeId: "1:234"
    componentKey: "a1b2c3d4e5f6a1b2c3d4e5f6"
```

---

## Customization

This skill is built for a specific project structure and Figma setup, but it is designed to be adapted. Here is what to change for your project.

### Adapt to your folder structure

The skill's `SKILL.md` contains a "Folder Structure" section that shows where metadata files should live relative to component source files. Update that section to match your project:

**Default (this repo):**
```
packages/ui/src/components/[component-name]/
├── [component-name].tsx
└── [component-name].metadata.yaml
```

**Alternative: flat components folder**
```
src/components/
├── Button.tsx
└── Button.metadata.yaml
```

**Alternative: monorepo with multiple packages**
```
packages/[package-name]/src/[component-name]/
├── index.ts
├── [component-name].tsx
└── [component-name].metadata.yaml
```

Edit `SKILL.md` → "Folder Structure (Required)" section to reflect your actual layout, and update the `component.path` description in `component-metadata.schema.json` if needed.

### Update the schema reference path

Every `.metadata.yaml` file starts with a `yaml-language-server` comment that points to the schema. The path is relative to the metadata file's location. If the skill lives somewhere other than `.agent/skills/ai-component-metadata-yaml/`, count the directory levels and adjust accordingly.

**Skill at `.agent/skills/ai-component-metadata-yaml/` — component 4 levels deep:**
```yaml
# yaml-language-server: $schema=../../../../.agent/skills/ai-component-metadata-yaml/schemas/component-metadata.schema.json
```

**Skill at `tools/skills/metadata/` — component 2 levels deep:**
```yaml
# yaml-language-server: $schema=../../tools/skills/metadata/schemas/component-metadata.schema.json
```

Update the Quick Start template in `SKILL.md` so Claude generates the correct path automatically.

### Adapt the component categories

The schema enforces `atoms`, `molecules`, `organisms` (Atomic Design). If your design system uses a different taxonomy — for example `primitives`, `patterns`, `layouts` — edit the `enum` in `component-metadata.schema.json`:

```json
"category": {
  "type": "string",
  "enum": ["primitives", "patterns", "layouts"],
  "description": "Your system's component category taxonomy."
}
```

Then update `SKILL.md` → "Component Categories" section to match.

### Change enforcement level for anti-patterns

The `usage.antiPatterns` field is advisory by default — it informs AI but does not block generation. If you run automated governance checks in CI, you can add an `enforcement` field to the anti-pattern schema:

```json
"antiPatterns": {
  "items": {
    "properties": {
      "scenario": { "type": "string" },
      "reason": { "type": "string" },
      "alternative": { "type": "string" },
      "enforcement": {
        "type": "string",
        "enum": ["strict", "advisory"],
        "description": "Whether CI should block on this violation."
      }
    }
  }
}
```

---

## CLI validation

Validate a metadata file without an IDE:

```bash
# Requires js-yaml and ajv to be installed
npx js-yaml path/to/button.metadata.yaml | npx ajv validate \
  -s .agent/skills/ai-component-metadata-yaml/schemas/component-metadata.schema.json \
  -d /dev/stdin
```

---

## License

MIT — see [SKILL.md](./SKILL.md) for full attribution.
