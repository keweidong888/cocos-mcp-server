# Cocos Creator UI Agent Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create an AI Agent that assembles UI in Cocos Creator from effect screenshots and named resources, via the cocos-mcp-server.

**Architecture:** Single Agent mode — the agent receives effect screenshot + resource info, performs visual analysis, maps resources by naming convention, calls MCP tools to build nodes, and verifies by coordinate comparison.

**Tech Stack:** Agent (Claude/other AI) + cocos-mcp-server (existing) + naming convention + coordinate verification

---

## File Structure

```
docs/superpowers/plans/
└── 2026-05-14-cocos-ui-agent.md    # This plan

cocos-mcp-server/
└── docs/superpowers/
    └── specs/
        └── 2026-05-14-cocos-ui-agent-design.md   # Already created

# The agent itself is a prompt/instruction file, not code
# It will be used by the AI that controls the MCP server
```

---

## Task 1: Design Agent System Prompt

**Files:**
- Create: `docs/superpowers/agent/ui-agent-prompt.md`

- [ ] **Step 1: Write Agent System Prompt**

```markdown
# Cocos Creator UI Assembly Agent

## Role
You are a UI assembly specialist. Given an effect screenshot and UI resources with naming conventions, you assemble the UI in Cocos Creator by calling cocos-mcp-server tools.

## Input
1. **Effect Screenshot** — PNG/JPG image showing the target UI layout
2. **Resource Directory** — Path to UI resources (individual image files)
3. **Naming Convention** — Files named as `{element}_{state}.png`, e.g., `btn_confirm_normal.png`, `btn_confirm_pressed.png`

## Capabilities

### Visual Analysis
- Analyze the effect screenshot to identify:
  - Container hierarchy (background → content → foreground)
  - Element types (button, label, sprite, etc.)
  - Element positions (x, y from top-left origin)
  - Element sizes (width, height)
  - zOrder / layering

### Resource Mapping
- Map screenshot elements to resource files by naming convention:
  - `avatar_bg.png` → avatar background
  - `btn_confirm_normal.png` → confirm button normal state
  - `btn_confirm_pressed.png` → confirm button pressed state
- If no resource matches an element, skip that element (do not fabricate)

### Node Construction (via cocos-mcp-server)
1. **Get scene context** — `scene_get_current_scene`, `node_get_all_nodes`
2. **Create container nodes** — `node_create` with parentUuid for hierarchy
3. **Add components** — `component_add_component` (cc.Sprite, cc.Label, cc.Button, etc.)
4. **Set properties** — `component_set_component_property` for spriteFrame, string, color, etc.
5. **Set transforms** — `node_set_node_transform` for position/scale
6. **Establish parent-child** — `node_move_node` or `node_create` with parentUuid

### Construction Order
1. Create all container nodes first (background, panels)
2. Create leaf nodes with Sprite/Label components
3. Set spriteFrame from mapped resources
4. Set positions from visual analysis
5. Set contentSize from visual analysis

### Verification
After assembly:
1. Capture scene screenshot (or receive from user)
2. Compare key node positions and sizes against original screenshot
3. Report match percentage and deviations

## Constraints
- **Do not fabricate** — If a UI element has no matching resource, skip it
- **1:1 recreation** — Position and size should match screenshot as closely as possible
- **Full assembly** — Complete the entire UI in one run, no step-by-step confirmation
- **Naming convention required** — Resources must follow `{element}_{state}.png` pattern

## Working Process

### Phase 1: Analysis
1. Receive effect screenshot
2. Analyze visual structure:
   - Identify top-level containers
   - Identify individual UI elements
   - Map each element to resource file by name
   - Estimate positions and sizes

### Phase 2: Construction
1. Get current scene root (Canvas or scene root)
2. Create container nodes top-down
3. Create element nodes with components
4. Set spriteFrame using mapped resources
5. Set position and contentSize

### Phase 3: Verification
1. Report construction complete
2. List created nodes and their hierarchy
3. Note any skipped elements (no matching resource)
4. Await screenshot for verification

## Output Format
After construction, report:
```
## Construction Complete

### Created Nodes
- root (Container)
  - top_bar (Sprite)
  - center_panel (Container)
    - list_item_1 (Sprite)
    - list_item_2 (Sprite)
  - bottom_bar (Container)
    - btn_confirm (Button + Sprite)

### Skipped Elements (no matching resource)
- decorative_icon (no `decorative_icon*.png` found)

### Verification
Awaiting screenshot for coordinate verification...
```

## Error Handling
- If a node creation fails, log error and continue with next node
- If a component addition fails, try alternative method or skip
- If a property setting fails, log warning and continue
- Always report final status including any errors encountered
```

- [ ] **Step 2: Save the prompt file**

Save to `docs/superpowers/agent/ui-agent-prompt.md`

- [ ] **Step 3: Commit**

```bash
cd F:/Project/cocos-mcp-server
git add docs/superpowers/agent/ui-agent-prompt.md
git commit -m "feat: add UI agent system prompt"
```

---

## Task 2: Create Agent Usage Guide

**Files:**
- Create: `docs/superpowers/agent/ui-agent-usage.md`

- [ ] **Step 1: Write Usage Guide**

```markdown
# UI Agent Usage Guide

## How to Use the UI Agent

### Prerequisites
1. Cocos Creator is running with cocos-mcp-server extension loaded
2. The MCP server is accessible (default: http://127.0.0.1:8585)
3. You have:
   - An effect screenshot of the target UI
   - UI resources following the naming convention

### Resource Naming Convention

Your UI resources must follow this pattern:

```
{element}_{state}.png
```

Examples:
- `avatar_bg.png` — Avatar background (no state)
- `btn_confirm_normal.png` — Confirm button normal state
- `btn_confirm_pressed.png` — Confirm button pressed state
- `btn_confirm_disabled.png` — Confirm button disabled state
- `icon_star.png` — Star icon
- `panel_bg.png` — Panel background

### Calling the Agent

Provide the agent with:
1. The effect screenshot (attach or describe the path)
2. The resource directory path
3. Any specific instructions

Example prompt to the agent:

```
Please assemble the UI shown in the screenshot at 'C:/project/ui_mockup.png'.
Resources are in 'C:/project/resources/'.
Follow the naming convention: {element}_{state}.png

Start the assembly.
```

### What the Agent Does

1. **Analyzes** the screenshot to identify containers and elements
2. **Maps** each element to a resource file by name matching
3. **Constructs** nodes in Cocos Creator via MCP:
   - Creates container nodes for hierarchy
   - Adds Sprite/Label/Button components
   - Sets spriteFrame from resources
   - Sets positions and sizes from visual analysis
4. **Reports** completion with node tree and any skipped elements
5. **Verifies** (if you provide a verification screenshot) by comparing coordinates

### Verification

After the agent completes, provide a screenshot of the Cocos Creator scene.
The agent will:
- Compare key node positions with the original design
- Report match percentage
- List any deviations

### Skipped Elements

The agent will skip elements where:
- No resource file matches the element name
- The element has no visual representation in the screenshot

No fabricated elements will be created.

### Tips

1. **Complete resource sets** — Provide all states (normal/pressed/disabled) for interactive elements
2. **Clear naming** — Use descriptive element names (e.g., `settings_btn` not `btn1`)
3. **Organized resources** — Keep all UI resources in one directory for easier mapping
4. **Canvas size** — Ensure the Cocos Creator canvas matches the design (usually 1920x1080 or 1334x750)
```

- [ ] **Step 2: Save the usage guide**

Save to `docs/superpowers/agent/ui-agent-usage.md`

- [ ] **Step 3: Commit**

```bash
git add docs/superpowers/agent/ui-agent-usage.md
git commit -m "docs: add UI agent usage guide"
```

---

## Task 3: Update SPEC with Implementation Notes

**Files:**
- Modify: `docs/superpowers/specs/2026-05-14-cocos-ui-agent-design.md`

- [ ] **Step 1: Add Implementation Notes Section**

Add to the end of the spec file:

```markdown
## 9. Implementation

### Agent Prompt
The UI Agent is implemented as a **system prompt** (`docs/superpowers/agent/ui-agent-prompt.md`), not as executable code. The agent is invoked by an AI assistant (Claude, GPT-4, etc.) that:

1. Reads the system prompt to understand its role
2. Receives the effect screenshot and resource info
3. Analyzes the screenshot visually
4. Calls cocos-mcp-server tools to build the UI
5. Reports results

### Usage
1. Load the agent prompt into an AI assistant
2. Provide the effect screenshot and resource path
3. The AI assistant controls cocos-mcp-server to assemble the UI

### No Code Changes Required
This design does not require modifications to cocos-mcp-server itself. The existing tools (`node_create`, `component_add_component`, `component_set_component_property`, `node_set_node_transform`) are sufficient.
```

- [ ] **Step 2: Commit**

```bash
git add docs/superpowers/specs/2026-05-14-cocos-ui-agent-design.md
git commit -m "docs: add implementation notes to UI agent spec"
```

---

## Task 4: Create Quick Start Guide

**Files:**
- Create: `docs/superpowers/agent/ui-agent-quickstart.md`

- [ ] **Step 1: Write Quick Start**

```markdown
# UI Agent Quick Start

## One-Minute Setup

1. **Open an AI assistant** (Claude, GPT-4, etc.)

2. **Load the agent prompt** from:
   ```
   docs/superpowers/agent/ui-agent-prompt.md
   ```

3. **Prepare your assets**:
   - Effect screenshot (PNG/JPG)
   - Resources in folder, named as `{element}_{state}.png`

4. **Send this message**:
   ```
   [Attach your effect screenshot]
   Resources in: C:/path/to/resources/
   Start UI assembly.
   ```

## What Happens Next

The agent will:
1. Analyze your screenshot
2. Map elements to resources
3. Build the UI in Cocos Creator via MCP
4. Report completion

## Example

```
Resources: C:/project/ui_resources/
- avatar_bg.png
- btn_confirm_normal.png
- btn_confirm_pressed.png
- title_text.png

Screenshot: [attached]

Start assembly.
```

Agent response:
```
## Construction Complete

### Created Nodes
- Canvas
  - background (Sprite: avatar_bg.png, position: 0,0, size: 1920x1080)
  - center_panel (Container, position: 200,200, size: 1520x800)
    - title (Sprite: title_text.png, position: 0,0, size: 300,50)
    - confirm_btn (Button: btn_confirm_normal.png, position: 600,700, size: 200,80)

### Skipped Elements
- None

Ready for verification.
```
```

- [ ] **Step 2: Save and commit**

```bash
git add docs/superpowers/agent/ui-agent-quickstart.md
git commit -m "docs: add UI agent quickstart guide"
```

---

## Summary

| Task | Description | Files Created |
|------|-------------|---------------|
| 1 | Agent System Prompt | `docs/superpowers/agent/ui-agent-prompt.md` |
| 2 | Agent Usage Guide | `docs/superpowers/agent/ui-agent-usage.md` |
| 3 | Update SPEC | Updated `docs/superpowers/specs/2026-05-14-cocos-ui-agent-design.md` |
| 4 | Quick Start Guide | `docs/superpowers/agent/ui-agent-quickstart.md` |

**No code changes to cocos-mcp-server are required.** The agent is purely a prompt-based system that leverages existing MCP tools.