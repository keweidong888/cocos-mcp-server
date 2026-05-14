# UI Agent Usage Guide

The UI Agent is a system prompt loaded into an AI assistant that analyzes UI effect screenshots and constructs Cocos Creator nodes automatically via the MCP server.

---

## Prerequisites

- Cocos Creator 3.8+ is running with the `cocos-mcp-server` extension loaded
- MCP server is accessible (default: `http://127.0.0.1:8585`)
- An effect screenshot of the target UI is available
- UI resources follow the naming convention described below

---

## Resource Naming Convention

Name your image resources using the pattern `{element}_{state}.png`.

| Example | Element | State |
|---|---|---|
| `avatar_bg.png` | avatar_bg | (default) |
| `btn_confirm_normal.png` | btn_confirm | normal |
| `btn_confirm_pressed.png` | btn_confirm | pressed |
| `btn_confirm_disabled.png` | btn_confirm | disabled |

Common states: `normal`, `pressed`, `hover`, `disabled`.

---

## Calling the Agent

When invoking the UI Agent, provide the following information:

1. **Effect screenshot** — an image showing the target UI layout
2. **Resource directory path** — the path to the folder containing your UI image resources
3. **Optional context** — any layout hints or special instructions

### Example Prompt

```
Please construct this UI based on the attached screenshot. The resources are located at: assets/ui/gameui/
```

---

## What the Agent Does

1. **Analyzes the screenshot** — identifies all visible UI elements and their spatial relationships
2. **Maps elements to resources** — matches each element to a named resource file by pattern
3. **Constructs nodes via MCP** — creates the corresponding Cocos Creator node hierarchy using scene manipulation tools
4. **Reports completion** — summarizes which nodes were created and any elements that could not be matched

---

## Verification

After the agent completes construction:

1. Take a new screenshot of the constructed UI in Cocos Creator
2. Send it to the agent for comparison
3. The agent compares the positions and sizes of elements between the screenshot and the constructed nodes
4. The agent reports a **match percentage** indicating how closely the constructed UI matches the target

If the match percentage is below expectations, provide feedback to refine the construction.

---

## Skipped Elements

- **No matching resource** — if an element in the screenshot has no corresponding resource file, it is skipped without fabrication
- **No fabricated elements** — the agent will not create placeholder or guessed elements; it only constructs what can be precisely mapped

---

## Tips

- **Provide complete resource sets** — include all states (normal, pressed, disabled) for interactive elements like buttons
- **Use clear, consistent naming** — follow the `{element}_{state}.png` convention strictly to ensure accurate mapping
- **Organize your resources directory** — keep UI resources in a dedicated folder (e.g., `assets/ui/gameui/`) for easy path specification