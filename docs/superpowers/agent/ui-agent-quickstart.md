# UI Agent Quick Start

Assemble UI in Cocos Creator from a screenshot and resources — in under a minute.

---

## One-Minute Setup

**1. Open an AI assistant** (Claude, GPT-4, etc.)

**2. Load the agent prompt**

Paste the contents of `docs/superpowers/agent/ui-agent-prompt.md` into the AI assistant.

**3. Prepare your assets**

```
/your/project/resources/
  button_normal.png
  button_pressed.png
  icon_home.png
  panel_background.png
  ...
```

Naming convention: `{element}_{state}.png` (state is optional).

**4. Send your message**

```
Here is the effect screenshot: [attach image]
Resource directory: /your/project/resources/
Naming convention: {element}_{state}.png

Assemble the UI.
```

---

## What Happens Next

| Step | What the agent does |
|------|---------------------|
| 1 | Analyzes the screenshot — identifies containers, leaf nodes, positions |
| 2 | Maps each element to a resource file using the naming convention |
| 3 | Builds the scene in Cocos Creator via MCP tool calls |
| 4 | Reports completion with node hierarchy and any skipped elements |

---

## Example

**Your resources:**
```
resources/
  bg_main.png
  btn_start_normal.png
  btn_start_pressed.png
  btn_options_normal.png
  btn_options_pressed.png
  title_text.png
  icon_star.png
```

**Your message to the agent:**
```
Screenshot: [attach]
Directory: resources/
Pattern: {element}_{state}.png

Build the main menu UI.
```

**Agent response:**
```
## Assembly Complete

### Created Nodes
- main_menu (Sprite, bg_main.png)
  - title_label (Label, title_text.png)
  - btn_start (Button, btn_start_normal.png / btn_start_pressed.png)
  - btn_options (Button, btn_options_normal.png / btn_options_pressed.png)
  - icon_star (Sprite, icon_star.png, positioned top-right)

### Skipped Elements
None

### Unused Resources
None

### Notes
All elements matched. UI assembled at origin (0, 0), scale 1:1.
```

---

## Tool Cheat Sheet

| Tool | Use for |
|------|---------|
| `node_create` | Create a new node |
| `component_add_component` | Add Sprite, Label, Button |
| `component_set_component_property` | Set spriteFrame, text, color |
| `node_set_node_transform` | Set position, size |
| `node_set_parent` | Reparent under a container |

---

## Tips

- **Name resources clearly**: `{element}_{state}.png` makes mapping automatic
- **Include all states**: button needs `_normal` and `_pressed` at minimum
- **Screenshot from the game**: white background screenshots work best for edge detection
- **Don't skip elements**: if a resource has no match in the screenshot it will be noted as unused