# 前沿实现调研：AI 开发脚手架 · Agent Loop · Graph 编排

> 调研日期：2026-09-08 · 方法：4 路并行在线检索（GitHub / 官方文档 / 工程博客 / YouTube / B站），关键论断均附来源链接。
>
> **一句话总结**：业界已高度收敛于同一范式——*一个 agent loop + 一套小工具 + 分层上下文加载 + 沙箱权限 + 可编程扩展点*。Loop 本身只有几十行，Graph 框架的真正价值在 checkpoint/恢复/人工审批这些基础设施；FDE 规范应该"采用 + 收敛"，而不是重造循环。

---

## 1. AI 开发脚手架（Scaffold / Harness）是如何实现的

### 1.1 业界共识的分层架构

```
┌─ 扩展层：斜杠命令 / subagent 文件 / hooks / MCP / Skills / 插件
├─ 权限层：策略审批（allowlist + 逐项确认） + OS 沙箱（Seatbelt / bubblewrap）+ hooks
├─ 上下文层：CLAUDE.md / AGENTS.md 分层记忆 · compaction 压缩 · subagent 隔离 · Skills 渐进加载
├─ 工具层：read / edit / bash / grep / glob 五件套 + MCP 外接（ACI 设计 ≥ 提示词设计）
└─ 入口循环层：agent loop（CLI / IDE / CI / 云端多入口共享同一 core）
```

- 对 agent 的权威定义（Anthropic）：**"LLM in a loop, using tools based on environmental feedback"**；workflow（预定义路径）与 agent（模型动态决定流程）明确区分，能用简单方案就不加复杂度。[Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)
- 官方把循环细化为三段式：**gather context → take action → verify work → repeat**，verify（lint/测试/截图回传）是质量关键。[Building agents with the Claude Agent SDK](https://claude.com/blog/building-agents-with-the-claude-agent-sdk)
- **一套核心、多个入口**：Codex 是 Rust core crate 供 CLI/IDE/桌面/云共用；Amp 三端共享同一 agent；OpenHands 用 ACP（Agent-Client Protocol）让任意 agent 接入同一宿主。

### 1.2 主流开源脚手架对比（star 数为 2026-09 抓取值）

| 仓库 | 语言 | Stars | 实现要点 |
|---|---|---|---|
| [anthropics/claude-code](https://github.com/anthropics/claude-code) | TS（闭源，repo 为 docs/issues/plugins） | 144.3k | 多级记忆文件 + subagent + hooks + skills + 权限/沙箱；生产级上下文工程范本；system prompt 未公开（逆向见下） |
| [openai/codex](https://github.com/openai/codex) | Rust | ~122k | 多 crate workspace；**system prompt 直接开源**在 `codex-rs/core/*_prompt.md`（按模型分文件）；三级沙箱 × 四级审批（macOS Seatbelt / Linux bubblewrap）；AGENTS.md 标准发起者 |
| [google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli) | TS | ~106.8k | 内置搜索 grounding/shell/web-fetch；捆绑 ripgrep；checkpointing + token caching；MCP + extensions |
| [All-Hands-AI/OpenHands](https://github.com/All-Hands-AI/OpenHands) | TS + Python | 86.4k | Docker/VM 沙箱 runtime + 事件流 + **ACP 协议接入任意 agent**；旧版以 CodeAct（可执行 Python 作为统一动作空间）闻名 |
| [cline/cline](https://github.com/cline/cline) | TS | 67.6k | VS Code 扩展；Plan/Act 双模式；**逐操作 human-in-the-loop 审批**；checkpoints 一键回滚；`.clinerules` |
| [Aider-AI/aider](https://github.com/Aider-AI/aider) | Python | 48.8k | **edit formats**（LLM 输出 diff 解析落盘 + 自动 git commit）；**tree-sitter + PageRank 的 repo map**（token 预算内渲染最关键符号）；编辑后自动 lint/test 自修复 |
| [ghuntley/how-to-build-a-coding-agent](https://github.com/ghuntley/how-to-build-a-coding-agent) | Go | 5.8k | 6 步递进（chat→read→list→bash→edit→ripgrep）实现最小 coding agent，**教学首选** |
| [x1xhlol/system-prompts-and-models-of-ai-tools](https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools) | 收集库 | 143.4k | 30+ 工具的泄露/还原 system prompt，当前维护最好的收集库 |
| Claude Code 逆向分析三件套 | — | — | [VILA-Lab/Dive-into-Claude-Code](https://github.com/VILA-Lab/Dive-into-Claude-Code)（配[论文](https://arxiv.org/pdf/2604.14228)）、[openedclaude/claude-reviews-claude](https://github.com/openedclaude/claude-reviews-claude/blob/main/architecture/00-overview.md)、[lai3d/claude-code-architecture](https://github.com/lai3d/claude-code-architecture) |

### 1.3 上下文层（各家差异最大、最值得深挖的一层）

- **分层记忆文件，就近生效**：`CLAUDE.md` 按 企业策略 → `~/.claude/CLAUDE.md` → 项目根 → 子目录 拼接；单文件 ≤200 行，判据"删掉这行会导致模型犯错吗"。[memory 文档](https://code.claude.com/docs/en/memory)
- **AGENTS.md 开放标准**：纯 Markdown 无必填字段，monorepo 嵌套放置、**离被编辑文件最近者生效**；已被 Codex/Cursor/Amp/Jules/Devin/Gemini CLI/Aider 等 60k+ 项目采用。[agents.md](https://agents.md/)
- **压缩（compaction）**：接近上下文上限自动摘要历史；但 Anthropic 实测 "compaction isn't sufficient"——压缩会丢指令，须配合外置结构化进度文件（feature list JSON、progress 文件、git log 恢复）。[Effective harnesses for long-running agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- **Subagent = 上下文隔离单元**：独立上下文窗口 + 独立工具/模型/权限，只把摘要回传主会话；用途定调"子代理调研、高输出隔离、fan-out 批处理"。[sub-agents 文档](https://code.claude.com/docs/en/sub-agents)
- **Repo map**（Aider 独门）：tree-sitter 抽符号 → 构图 → PageRank 排序 → token 预算内渲染大纲。[repomap 博文](https://aider.chat/2023/10/22/repomap.html)
- **Skills（渐进式披露）**：SKILL.md 只在启动时注入 name/description，命中才读全文——"上下文容量近乎无限"；已开放为标准 [agentskills.io](https://agentskills.io)。[Agent Skills 工程文](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)

### 1.4 权限层

- 三层模型：**allowlist + 逐项审批 → 策略分类器（Auto 模式）→ OS 沙箱**（网络单独门控）。
- Codex 最工程化：三级 sandbox（read-only / workspace-write / danger-full-access）× 四级审批；细粒度例外用命令前缀 rules 而非放大整体权限。[Codex sandboxing](https://learn.chatgpt.com/docs/sandboxing.md)
- **Hooks = 确定性控制**：生命周期各点（PreToolUse / PostToolUse / Stop / PreCompact…）执行处理器；**退出码 2 是唯一能硬阻断的码**；官方明确"硬性允许/拒绝必须用权限系统而非 hooks"。[hooks 文档](https://code.claude.com/docs/en/hooks)

### 1.5 值得直接抄的 10 条工程实践

1. 主循环 <500 行起步，先跑通再谈架构。
2. 工具五件套 read/edit/bash/grep/glob；**先用 agentic 搜索（grep/glob），别急着上 embedding 检索**（官方原话：语义搜索"更快但准确性差、难维护、不透明"）。
3. 编辑走结构化 diff（Aider edit formats）：天然可审查、可回滚、省 token。
4. 仓库根放 AGENTS.md，monorepo 嵌套就近生效。
5. 上下文三层加载：system prompt（小而稳）→ 项目约定（启动注入）→ 领域知识（skills 按需加载），别把一切塞进首帧。
6. 压缩 + subagent 双保险；大输出/大调研丢给独立上下文，只回传摘要。
7. 权限三层模型；细粒度例外用 rules，不放大整体权限。
8. Hooks 提供确定性（PostToolUse 跑 lint，exit 2 阻断）。
9. 一切可回滚：git 自动 commit / checkpoint 快照 / worktree 隔离并行会话。
10. 验证闭环是质量根源：给 agent 一个可运行检查；**无法验证的任务不要交给 agent**。参考 [Claude Code best practices](https://code.claude.com/docs/en/best-practices)。

---

## 2. Agent Loop 是如何实现的

### 2.1 标准模式（四家口径完全一致）

> "agent = LLM + loop + tools + 足够的 tokens" —— Thorsten Ball, [How to Build an Agent](https://ampcode.com/how-to-build-an-agent)
> "Once you have an MCP Client, an Agent is literally just a while loop on top of it." —— [Hugging Face Tiny Agents](https://huggingface.co/blog/tiny-agents)

OpenAI Agents SDK 官方文档的 loop 语义（[running_agents](https://openai.github.io/openai-agents-python/running_agents/)）：

```text
loop:
    output = LLM(messages)
    if 无 tool calls 且输出是期望类型:  return 结果        # 正常退出
    if 请求 handoff:  切换 current_agent, continue
    if 有 tool calls: 执行工具 → tool_result 追加回 messages → continue
    if turns > max_turns: raise MaxTurnsExceeded        # 兜底退出
```

### 2.2 最小可运行实现（Anthropic Messages API，~30 行）

```python
import anthropic

TOOLS = [{
    "name": "read_file",
    "description": "读取一个文件的内容",
    "input_schema": {"type": "object",
        "properties": {"path": {"type": "string"}}, "required": ["path"]},
}]

def execute_tool(name, args):
    return open(args["path"]).read() if name == "read_file" else f"Unknown tool: {name}"

def run_agent(user_input: str, max_iterations: int = 20) -> str:
    client = anthropic.Anthropic()
    messages = [{"role": "user", "content": user_input}]
    for _ in range(max_iterations):                  # 迭代上限 = 兜底停止条件
        resp = client.messages.create(
            model="claude-sonnet-4-5", max_tokens=8192,
            system="你是一个 coding agent。", tools=TOOLS, messages=messages)
        messages.append({"role": "assistant", "content": resp.content})
        if resp.stop_reason != "tool_use":           # end_turn → 退出
            return next(b.text for b in resp.content if b.type == "text")
        # 支持一轮多个（并行）tool_use；执行后以 tool_result 追加回 messages
        messages.append({"role": "user", "content": [
            {"type": "tool_result", "tool_use_id": b.id,
             "content": execute_tool(b.name, b.input)}
            for b in resp.content if b.type == "tool_use"]})
```

真实开源参考：[Hugging Face Tiny Agents](https://huggingface.co/blog/python-tiny-agents)（~70 行，含退出工具 `task_complete`、流式 yield、MCP 工具调用）；[ghuntley/how-to-build-a-coding-agent](https://github.com/ghuntley/how-to-build-a-coding-agent)（Go 版 6 步教学）。

### 2.3 工程细节清单（复杂度全在这里，不在循环）

1. **迭代上限**：OpenAI SDK 默认 `DEFAULT_MAX_TURNS = 10`，超限抛 `MaxTurnsExceeded`（[run_config.py](https://github.com/openai/openai-agents-python/blob/main/src/agents/run_config.py)）；也可用"退出工具"（task_complete）让模型主动收敛。
2. **并行 tool calls**：一轮多个 tool_use 并行执行；Anthropic 多 agent 系统实测"subagents 并行用 3+ 工具，研究时间缩短 90%"。[multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)
3. **错误恢复**：工具失败以错误信息回灌 messages 让模型自行纠错（"letting the agent know when a tool is failing works surprisingly well"）；系统级用 checkpoint + resume，不整体重启。
4. **上下文压缩**：SDK 级自动 compaction + 外置进度文件双保险（见 §1.3）。
5. **Human-in-the-loop**：两个触发器——"Exceeding failure thresholds" 与 "High-risk actions"（支付/发邮件/改数据）。[OpenAI agent 指南 PDF](https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf)
6. **结构化输出**：final-output tool（调用特定输出类型工具即终止循环）；状态文件用 JSON 而非 Markdown（模型更不容易乱改）。
7. **Token/成本**：实测 agent 耗 token ≈ chat 的 4 倍、多 agent ≈ 15 倍，且"token 用量解释 80% 的成败方差"；按复杂度分配算力，警惕"简单查询孵化 50 个 subagent"。
8. **ACI（工具接口）设计 ≥ 提示词设计**："Bad tool descriptions can send agents down completely wrong paths"；案例：强制绝对路径后模型零失误。深入：[Writing effective tools for agents](https://www.anthropic.com/engineering/writing-tools-for-agents)。
9. **安全**：沙箱 + 严格限权凭证 + 预算上限；[Simon Willison: Designing agentic loops](https://simonwillison.net/2025/Sep/30/designing-agentic-loops/) 给出 YOLO mode 风险清单。

### 2.4 SDK 封装方式对比

| | Claude Agent SDK | OpenAI Agents SDK |
|---|---|---|
| 仓库 | [anthropics/claude-agent-sdk-python](https://github.com/anthropics/claude-agent-sdk-python) | [openai/openai-agents-python](https://github.com/openai/openai-agents-python) |
| loop 归属 | **循环由内置 Claude Code CLI 引擎执行**，你消费 `query()` 的 AsyncIterator | **循环就是你的主循环**：`Runner.run()` 执行上述语义，`max_turns` 可调 |
| 关键抽象 | hooks 挂在 loop 各点；`allowed_tools`/`permission_mode`/`can_use_tool`；自动 compaction；subagents | Agent / tool / **handoff**（agent 间交接）/ **guardrails** / sessions / tracing 全一等公民 |
| 定位 | 整个 harness 打包（引擎+工具+压缩+权限） | 薄循环 + 显式控制流，更接近"自家脚手架要写的东西" |

共识（Armin Ronacher [Agent Design Is Still Hard](https://lucumr.pocoo.org/2025/11/21/agents-are-hard/) + Simon Willison 评论）：模型间差异大到 **"you will need to build your own agent abstraction"**——自研时抄 OpenAI SDK 的 loop 语义 + Anthropic 的 harness 工程清单是最稳路径。

---

## 3. Graph 编排是如何实现的（以 LangGraph 为例）

### 3.1 底层执行模型：Pregel / BSP super-step

`StateGraph.compile()` 的产物是 `Pregel` runnable。执行遵循 Google Pregel / Bulk Synchronous Parallel 模型：计算切分为离散 **super-step**，每个 step 中所有被激活的节点**并行**执行，读取的是 step 开始时**冻结**的 channel 值，所有写入在 step 结束时**统一提交**——同 step 节点互相不可见，天然获得确定性快照语义。反复推进 super-step 直到无活跃通道或达 `recursion_limit`（默认 25），这就是"图里可以有环（agent 循环）"的底层支撑。
来源：[Graph API](https://docs.langchain.com/oss/python/langgraph/graph-api)、[Pregel runtime](https://docs.langchain.com/oss/python/langgraph/pregel)、[执行语义分析](https://chbussler.medium.com/langgraph-execution-semantics-c7dd89900ed4)、[Atomic Object：坑点分析](https://spin.atomicobject.com/langgraphs-execution-model-tricky/)、源码 `libs/langgraph/langgraph/pregel/`（[_algo.py / _loop.py / _runner.py / _checkpoint.py]）。

### 3.2 State / Node / Edge / Send / 子图

- **State** 只是 schema（TypedDict/dataclass/Pydantic），节点返回**部分状态 dict**（增量），由运行时合并。
- **Annotated reducer**：`Annotated[list, add_messages]` 的第二参数声明"同一 key 多个更新如何归并"；无 reducer 的 key 是 `LastValue` channel（每 step 至多一个写入者，两个并行节点写同一 key 会抛 `InvalidUpdateError`）。channel 家族还有 `Topic`（消息队列）、`EphemeralValue`（不入 checkpoint）、`NamedBarrierValue`（fan-in join）。
- **Node**：任意函数 `(state) -> dict`，或返回 `Command(update=..., goto=...)` 边更新边路由。
- **Edge**：`add_edge` 静态边；`add_conditional_edges(a, router_fn, mapping)` 条件边；`Send` API 在运行时按数据动态复制 N 个节点实例（**动态 map-reduce**）。
- **子图**：编译后的图直接作为父图节点；独立 checkpoint namespace。

### 3.3 Checkpointing：持久化 / time travel / human-in-the-loop

每个 super-step 结束持久化一次全量 state 快照（[Persistence 文档](https://docs.langchain.com/oss/python/langgraph/persistence)）：
- `compile(checkpointer=...)` + `thread_id` → 多轮记忆、故障恢复；
- `get_state_history()` + `checkpoint_id` → **time travel**（回放/改写重跑）；
- 节点内 `interrupt()` 暂停 → 人工审批 → `Command(resume=...)` 续跑 → **human-in-the-loop**；
- 实现从 `InMemorySaver` 到 `SqliteSaver`/`PostgresSaver`（生产）；另有跨 thread 的 `Store`（长期记忆）。

### 3.4 LangGraph 最小示例（官方 quickstart 风格）

```python
from typing import Annotated
from typing_extensions import TypedDict
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langgraph.checkpoint.memory import InMemorySaver
from langchain.chat_models import init_chat_model

llm = init_chat_model("anthropic:claude-sonnet-4-5")

class State(TypedDict):
    messages: Annotated[list, add_messages]   # reducer：追加合并而非覆盖

def chatbot(state: State):
    return {"messages": [llm.invoke(state["messages"])]}   # 节点只返回增量

builder = StateGraph(State)
builder.add_node("chatbot", chatbot)
builder.add_edge(START, "chatbot")
builder.add_conditional_edges(
    "chatbot",
    lambda s: END if not s["messages"][-1].tool_calls else "chatbot",
)
graph = builder.compile(checkpointer=InMemorySaver())      # 一行获得记忆/time-travel/HITL
graph.invoke({"messages": [{"role": "user", "content": "你好"}]},
             {"configurable": {"thread_id": "t1"}})
```

生产中官方更推荐从 `create_react_agent`（prebuilt）起步，需要定制路由时再下沉到 StateGraph。

### 3.5 框架对比

| 框架 | 图的形态 | 状态模型 | 持久化/HITL | 适合场景 |
|---|---|---|---|---|
| [LangGraph](https://github.com/langchain-ai/langgraph) | 显式 StateGraph，可含环（Pregel） | 共享 State + reducer，节点写增量 | **最强**（checkpoint/time travel/interrupt） | 需可恢复、可审批、可回放的生产 agent |
| [CrewAI Flows](https://docs.crewai.com/v1.15.17/en/concepts/flows) | 事件回调图（@start/@listen/@router） | Flow 内 state | 弱 | 角色分工明确的多 agent |
| [AutoGen GraphFlow](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/graph-flow.html) | DiGraph（消息传递） | 消息驱动，无共享快照 | 弱 | 对话式多 agent（注意已并入 [Microsoft Agent Framework](https://learn.microsoft.com/en-us/agent-framework/overview/agent-framework-overview)） |
| [Dify](https://dify.ai) / [n8n](https://n8n.io) / Coze | 可视化画布 | 节点变量传递 | 平台内置 | 低代码：AI 应用选 Dify、事件自动化选 n8n、轻量 bot 选 Coze |
| AWS Step Functions | JSON 状态机 | 数据传递 | 企业级 | 传统云上确定性编排（LangGraph 的 checkpoint/HITL 大致对应它+人工审批活动） |

### 3.6 Loop vs Graph 选型（含"反图"观点）

Anthropic《Building effective agents》立场：**能用 workflow 就别用自主 agent，能用简单方案就别加复杂度**。文中 5 种模式（prompt chaining → routing → parallelization → orchestrator-workers → evaluator-optimizer）都不需要图框架，几十行代码可实现。反面声音：[Steve Kinney: The Anatomy of an Agent Loop](https://stevekinney.com/writing/agent-loops)（"agent 就是个带工具的循环，LangGraph 是 overkill"）、[Cognition: Don't Build Multi-Agents](https://cognition.ai/blog/dont-build-multi-agents)（主张共享上下文单线程 agent，反对过早拆多 agent 图）。

决策清单：
1. 任务路径固定、步骤 ≤3 → chain/函数调用即可，**不上图**。
2. 需要自主循环、不需要中断恢复 → **百行级自研 loop** 完全够，调试透明。
3. 需要断点恢复 / 人工审批 / time travel 调试 / 动态 fan-out 任一项 → **采用 LangGraph 类框架开始占优**（或自研等价基础设施：状态 schema + reducer 合并、super-step 快照、checkpoint 持久化、interrupt/resume 协议——即 LangGraph `pregel/` 源码的骨架）。
4. 可视化编排/运营团队为主 → Dify/n8n/Coze；Azure/.NET 栈 → Microsoft Agent Framework。

---

## 4. 三者如何拼成一个 FDE 的技术栈

```
FDE 交付现场 = 规范（01 文档） × 工具
  ├─ 日常交付：直接用成熟脚手架（Claude Code / Codex CLI）+ AGENTS.md + skills（§1）
  ├─ 给客户构建 agent 应用：自研百行 loop + 五件套工具 + eval（§2）
  ├─ 需要可恢复/可审批的生产 agent：LangGraph（checkpoint + interrupt）或等价自研（§3）
  └─ 反哺资产：把现场方法论沉淀为 skills / 模板 / 连接器（规范 R7.1）
```

选型判断一句话：**agent loop 本身不值得重造，值得投入的是工具打磨、上下文工程与权限工程；graph 框架的价值不在"画图"而在 checkpoint/恢复/并行/HITL 基础设施。**

---

## 5. 资源总表

### 5.1 权威文章

| 文章 | 链接 | 一句话 |
|---|---|---|
| Anthropic: Building effective agents | https://www.anthropic.com/engineering/building-effective-agents | 业界 agent 定义与 5 种模式的源头文献 |
| Anthropic: Building agents with the Claude Agent SDK | https://claude.com/blog/building-agents-with-the-claude-agent-sdk | gather/act/verify 循环三段论 |
| Anthropic: Effective harnesses for long-running agents | https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents | compaction 不足与外置记忆 |
| Anthropic: Multi-agent research system | https://www.anthropic.com/engineering/multi-agent-research-system | 并行 subagent、token 倍数、checkpoint/resume |
| OpenAI: A practical guide to building agents (PDF) | https://cdn.openai.com/business-guides-and-resources/a-practical-guide-to-building-agents.pdf | run loop、退出条件、HITL 触发器 |
| Simon Willison: Designing agentic loops | https://simonwillison.net/2025/Sep/30/designing-agentic-loops/ | 何时适合循环 + 安全实践 |
| Thorsten Ball: How to Build an Agent | https://ampcode.com/how-to-build-an-agent | "LLM + loop + tokens" 极简主义原点 |
| Armin Ronacher: Agent Design Is Still Hard | https://lucumr.pocoo.org/2025/11/21/agents-are-hard/ | 为什么必须自研抽象 |
| Anthropic: Agent Skills 工程文 | https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills | 渐进式披露与 agentskills.io 标准 |
| AGENTS.md 规范 | https://agents.md/ | 项目约定文件开放标准 |
| Claude Code best practices | https://code.claude.com/docs/en/best-practices | 上下文管理与失败模式手册 |
| Codex sandboxing | https://learn.chatgpt.com/docs/sandboxing.md | 三级沙箱 × 四级审批完整设计 |
| LangGraph 官方文档 | https://docs.langchain.com/oss/python/langgraph/graph-api · /pregel · /persistence | 本文 §3 的第一手依据 |
| Steve Kinney: The Anatomy of an Agent Loop | https://stevekinney.com/writing/agent-loops | 何时图框架是 overkill |
| Cognition: Don't Build Multi-Agents | https://cognition.ai/blog/dont-build-multi-agents | 多 agent 图的代表性反方 |

### 5.2 GitHub 仓库速查

**脚手架本体**：[openai/codex](https://github.com/openai/codex)（prompt 开源）· [google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli) · [anthropics/claude-code](https://github.com/anthropics/claude-code)（docs/plugins）· [All-Hands-AI/OpenHands](https://github.com/All-Hands-AI/OpenHands) · [cline/cline](https://github.com/cline/cline) · [Aider-AI/aider](https://github.com/Aider-AI/aider)
**SDK**：[anthropics/claude-agent-sdk-python](https://github.com/anthropics/claude-agent-sdk-python) · [openai/openai-agents-python](https://github.com/openai/openai-agents-python) · [langchain-ai/langgraph](https://github.com/langchain-ai/langgraph)
**最小实现教学**：[ghuntley/how-to-build-a-coding-agent](https://github.com/ghuntley/how-to-build-a-coding-agent)（Go）· HF Tiny Agents（[Py](https://huggingface.co/blog/python-tiny-agents) / [TS](https://huggingface.co/blog/tiny-agents)）
**Prompt 收集与逆向**：[x1xhlol/system-prompts-and-models-of-ai-tools](https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools)（143k★）· [VILA-Lab/Dive-into-Claude-Code](https://github.com/VILA-Lab/Dive-into-Claude-Code) · [lai3d/claude-code-architecture](https://github.com/lai3d/claude-code-architecture)
**FDE 相关**：[xdash/FDE-the-Guidance-Book-of-Forward-Deployed-Engineer](https://github.com/xdash/FDE-the-Guidance-Book-of-Forward-Deployed-Engineer) · [yeasy/forward_deployed_engineering_guide](https://github.com/yeasy/forward_deployed_engineering_guide) · [global-fde/awesome-fde-resources](https://github.com/global-fde/awesome-fde-resources) · [rpraharaj/forward-deployed-engineer](https://github.com/rpraharaj/forward-deployed-engineer) · [databricks/databricks-agent-skills](https://github.com/databricks/databricks-agent-skills)

### 5.3 YouTube 精选

| 视频 | 链接 | 说明 |
|---|---|---|
| Building Claude Code with Boris Cherny（Pragmatic Engineer） | https://www.youtube.com/watch?v=julbw1JuAz0 | Claude Code 作者谈设计取舍 |
| How Claude Code Works（PromptLayer, AI Engineer） | https://ai.engineer/talks/how-claude-code-works | 独立视角逐层拆解，适合内部培训 |
| Claude Agent SDK 全程 workshop（Anthropic 官方） | https://www.youtube.com/watch?v=TqC1qOfiVcQ | 从 SDK 视角看 harness 能力面 |
| Building Agents with MCP（Anthropic, ~35 万播放） | https://www.youtube.com/watch?v=kQmXtrmQ5Zg | 从零讲 agent 循环 + MCP |
| Don't Build Agents, Build Skills Instead（~140 万播放） | https://www.youtube.com/playlist?list=PLcfpQ4tk2k0W564IUrISLhTVZ5lOVCuwm | Agent Skills 来龙去脉 |
| Colin Jarvis：OpenAI FDE 负责人访谈 | https://www.youtube.com/watch?v=cBD7_R-Cizg | OpenAI 前向部署团队打法 |
| LangChain Academy: Introduction to LangGraph（官方免费课） | https://academy.langchain.com/courses/intro-to-langgraph | 官方出品，覆盖 checkpoint/HITL |
| LangGraph Crash Course 2025 | https://www.youtube.com/watch?v=Y3dbzuQBnUw | 工程向速成 |

### 5.4 B站精选

| 视频 | 链接 | 说明 |
|---|---|---|
| 51万行源码真相：ClaudeCode 架构深度解读 | https://www.bilibili.com/video/BV1U8d8BzEUK/ | 四大约束架构与 8 大设计模式 |
| Claude Code 源码解析：从 0 到 1 开发 Harness Agent | https://www.bilibili.com/video/BV1nJQFBcEb3/ | Harness 概念 + 复刻实战 |
| ClaudeCode 底层技术硬核拆解（1884 个文件） | https://www.bilibili.com/video/BV1zR9JBREua/ | 把 Claude Code 定位为 Agent Runtime |
| 尚硅谷 LangGraph 教程（132 集，53 万+播放） | https://www.bilibili.com/video/BV1z3NY66EY1/ | 最系统的中文 LangGraph 入门 |
| 600 分钟吃透 LangGraph 入门 | https://www.bilibili.com/video/BV1YrgQzkEYz/ | 偏底层原理 |
| 使用 Rust 开发 AI Agent：简单 Agent 循环 + 动态工具 | https://www.bilibili.com/video/BV1Jvgk61E5E/ | 手写最小 agent loop |
| 从 0 到 1 动手做一个 AI Agent（基础篇） | https://www.bilibili.com/video/BV1VnEi6gELD?p=3 | tool calling + human-in-the-loop |
| 为什么FDE在AI时代这么火 | https://www.bilibili.com/video/BV1tXKi6NE4b/ | FDE 爆火逻辑中文拆解 |
| 从零构建生产级 AI Agent 服务（LangGraph + FastAPI） | https://www.bilibili.com/video/BV1tuNXzREmc/ | 会话管理、故障恢复等生产化主题 |
