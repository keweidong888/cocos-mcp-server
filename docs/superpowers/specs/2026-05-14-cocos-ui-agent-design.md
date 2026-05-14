# Cocos Creator UI 拼装 Agent 设计

**日期：** 2026-05-14
**类型：** 功能设计

## 1. 目标

设计一个 AI Agent，通过控制 cocos-mcp-server，结合效果图截图和按命名规范整理的 UI 资源图，在 Cocos Creator 中自动拼装出对应的 UI 界面。

## 2. 输入

| 输入 | 描述 |
|------|------|
| 效果图截图 | PNG/JPG 图片，包含完整 UI 界面 |
| UI 资源文件 | 散文件，命名规范覆盖所有状态（如 `btn_confirm_normal.png`、`btn_confirm_pressed.png`） |

**命名规范要求：**
- 文件名包含元素名和状态，如 `avatar_bg.png`、`btn_confirm_normal.png`
- 覆盖所有状态：normal / pressed / disabled / hover 等
- 无需打包，文件散落在指定目录

## 3. 工作流程（5步）

### Step 1 — 视觉分析
- 接收效果图截图
- AI 分析识别：容器结构、元素类型（按钮/标签/图片）、位置坐标、尺寸、zOrder
- 输出内部结构化描述（节点树草案）

### Step 2 — 资源映射
- 根据命名规范自动匹配 UI 资源文件
- 生成节点创建指令，包含：
  - 节点名称（从截图识别，如 `top_bar`）
  - 父节点路径
  - 资源文件引用（spriteFrame）
  - position / contentSize

### Step 3 — 调用 MCP 拼装
按层级顺序调用 cocos-mcp-server 工具：
- `node_create` — 创建节点
- `component_add_component` — 添加 Sprite/Label 等组件
- `component_set_component_property` — 设置属性
- `node_set_parent` — 建立父子关系

### Step 4 — 截图验证
- 拼装完成后，捕获 Cocos 场景截图
- 与原效果图对比：关键节点 position、contentSize、zOrder
- 输出验证报告（匹配度百分比）

### Step 5 — 输出结果
- 验证通过：报告完成，输出节点树结构
- 验证失败：标记偏差区域，询问自动修正或人工介入

## 4. 架构

```
┌─────────────────────────────────────────────────────────┐
│                    UI Agent (单 Agent)                    │
│                                                          │
│  ┌──────────┐    ┌─────────────┐    ┌───────────────┐  │
│  │ 效果图    │───▶│ 视觉分析模块  │───▶│ 节点树生成     │  │
│  │ 截图输入  │    │ (内部推理)    │    │ + 资源映射     │  │
│  └──────────┘    └─────────────┘    └───────────────┘  │
│                                             │           │
│                                             ▼           │
│  ┌─────────────────────────────────────────────────┐   │
│  │           Cocos MCP Server                       │   │
│  │  create_node / set_node_property / add_component │   │
│  └─────────────────────────────────────────────────┘   │
│                                             │           │
│                                             ▼           │
│  ┌──────────────┐    ┌────────────────────────────┐   │
│  │ 截图对比验证  │◀───│ 拼装结果                    │   │
│  │ (尺寸/坐标)   │    │                            │   │
│  └──────────────┘    └────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

## 5. 技术要点

| 模块 | 实现方式 |
|------|----------|
| 视觉分析 | Agent 内部推理，基于截图视觉理解 + 资源命名规范 |
| 层级推断 | 从视觉布局推断父子关系（背景容器 → 子元素） |
| 资源映射 | 命名匹配（如 `btn_confirm_normal.png` → normal 状态） |
| 坐标验证 | 对比关键节点 position/size 与识别结果的偏差 |
| 节点创建 | 调用 cocos-mcp-server 的 `node_*` 和 `component_*` 工具 |

## 6. 关键约束

1. **没有提供的 UI 元素不拼** — AI 只拼效果图中有、资源包中有的部分
2. **1:1 还原** — 位置、尺寸尽量与效果图一致
3. **全量拼装** — 一次性完成整个界面，无需分步确认
4. **资源命名规范** — 文件名需包含元素名和状态，覆盖 normal/pressed/disabled 等

## 7. 不需要用户提供的内容

- 无需手动标注层级
- 无需提供步骤说明
- 无需预定义模板
- 无需图层级源文件

## 8. 验证机制

- 拼装完成后截图对比
- 对比内容：关键节点 position、contentSize、zOrder
- 输出匹配度报告，失败时标记偏差区域

## 9. Implementation

### Agent Prompt

The UI Agent is implemented as a system prompt (`docs/superpowers/agent/ui-agent-prompt.md`), not as executable code. The agent is invoked by an AI assistant (Claude, GPT-4, etc.) that:

- Reads the system prompt to understand its role
- Receives the effect screenshot and resource info
- Analyzes the screenshot visually
- Calls cocos-mcp-server tools to build the UI
- Reports results

### Usage

Load the agent prompt into an AI assistant, provide the effect screenshot and resource path, the AI assistant controls cocos-mcp-server to assemble the UI.

### No Code Changes Required

This design does not require modifications to cocos-mcp-server itself. The existing tools (`node_create`, `component_add_component`, `component_set_component_property`, `node_set_node_transform`) are sufficient.