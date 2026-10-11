# FDE（Forward Deployed Engineer，前向部署工程师）工程规范

> 版本 v0.1（草案） · 2026-09-08 · 适用范围：以 AI 加速交付的客户现场工程项目
>
> 依据来源：Palantir / OpenAI / Anthropic / Scale AI 的 FDE JD 与官方工程文章、
> [yeasy《前线部署工程实践指南》](https://yeasy.gitbook.io/forward_deployed_engineering_guide/)、
> [范冰《前线部署工程师》](https://github.com/xdash/FDE-the-Guidance-Book-of-Forward-Deployed-Engineer)（fde4.ai）、
> [146 份 FDE JD 量化分析](https://github.com/alexeygrigorev/ai-engineering-field-guide/blob/main/role/06-fde.md)、
> [rpraharaj/forward-deployed-engineer](https://github.com/rpraharaj/forward-deployed-engineer)（43 个 Claude Code skills）等，
> 完整链接见《02-前沿实现调研-脚手架-Loop-Graph.md》资源总表。

---

## 0. 术语与角色定义

### 0.1 定义

**FDE 是驻扎在客户现场（或深度嵌入客户上下文）、填补"产品能做的事"与"客户需要的事"之间鸿沟的工程师。**
写的是生产环境代码而非报告，对客户的业务结果负责，直到客户团队自己能接管。

- 起源：2003 年 Palantir（首任 FDE 为第 13 号员工 Shyam Sankar）；"把痛苦吃进去、把产品产出来的人"。
- 制度核心：把"现场定制"从**服务成本**记账为**产品发现**——每个现场项目都必须产出可复用资产。
- 现状（2025-2026）：FDE 招聘量一年涨超 800%；OpenAI 成立 Deployment Company（约 150 名 FDE、40 亿美元投入）；Anthropic Applied AI 的 FDE 交付物已是 MCP server / sub-agent / agent skill 本身。

### 0.2 与邻近角色的边界

| 维度 | FDE | 软件工程师（SWE） | 解决方案架构师/售前 | 咨询顾问 |
|---|---|---|---|---|
| 工作起点 | 模糊需求，现场发现真问题 | 预定义 ticket 队列 | 客户给定方案需求 | 合同固定范围 |
| 交付物 | 生产代码 + 运行中的系统 | 产品功能 | 方案文档 / demo | 报告 / 文档 |
| 是否写生产代码 | **是** | 是 | 通常否 | 通常否 |
| 对结果负责到 | 生产系统持续运行、客户能自运维 | 合并/上线 | 交接即止 | 交接即止 |
| 反哺 | 强制沉淀可复用资产回产品 | 产品迭代 | 输入销售 | 内部知识库 |

一句话边界：FDE = **现场发现 × 工程交付 × 生产价值 × 产品反哺**，四件事缺一不可。

---

## 1. 能力模型

### 1.1 能力三角（三项都须 functional）

1. **Engineering**：能造出来——生产级 Python/TS（JD 出现率 91%）、Docker/K8s（40%/35%）、CI/CD（34%），能部署进客户**已有**的任何栈。
2. **Consulting**：能诊断真问题——影子工作法（坐在真实用户旁看他过完一天）、knowledge interview、区分"陈述的需求"与"真问题"。
3. **Product Thinking**：知道什么值得造——判断定制与产品的边界，识别可反哺的模式。

### 1.2 AI 时代硬技能基线（按 146 份 JD 出现率）

Python 91% · Prompt 工程 55% · RAG 52% · AWS 47% · LLM 应用 43% · **AI agents 42%** · Docker 40% · K8s 35% · CI/CD 34% · LangChain/LangGraph 33%。
另须熟练使用至少一个 AI 编码代理（Claude Code / Codex CLI / Cursor 等）作为主力交付工具。

### 1.3 软性要求

High agency（没有条件就创造条件）· Low-ego communication · 模糊环境下的技术判断力 · 跨技术/非技术受众的双向翻译（对客户讲业务结果，对内部产品团队讲字段级反馈）。
**FDE 不是入门岗**：全部在招岗位均为 Senior/Staff/Principal/Lead/Founding 级。

---

## 2. 交付流程规范（七阶段闭环）

每个 engagement 必须走完以下阶段；**跳过评估阶段直接写代码视为流程违规**。

### 阶段 1 · LAND（落地）

- 目标：建立信任、摸清组织。
- 必做：
  - 【R1.1】产出《Engagement Brief》：业务背景、干系人地图、**决策链（谁能说是谁）**、成功标准草案。
  - 【R1.2】确认安全与合规边界：客户数据分级、涉密信息与本地笔记**物理分离**（客户 repo 不放任何 FDE 私有笔记）。
- 产出物：Engagement Brief（1-2 页）。
- 退出条件：客户方 sponsor 书面确认。

### 阶段 2 · DISCOVER（现场发现）

- 目标：找到"真问题"而非"陈述的需求"。
- 必做：
  - 【R2.1】影子工作法：至少 1 个完整工作日跟随真实用户。
  - 【R2.2】knowledge interview：覆盖一线用户、领域专家、决策者三类角色。
  - 【R2.3】输出《真问题陈述》（problem statement）：一句话问题 + 现状损耗的量化 + 为什么现有产品解决不了。
- 产出物：真问题陈述、现场访谈纪要（当日回执给客户确认）。
- 退出条件：sponsor + 一线用户代表双方确认问题优先级。

### 阶段 3 · SCOPE & EVAL（定标准，先于写代码）

- 目标：定义"什么叫做成了"，**先建评测（eval）系统再开工**。
- 必做：
  - 【R3.1】书面 scope：范围内/外清单、变更流程（口头需求一律转书面确认后才排期）。
  - 【R3.2】定义成功指标与 eval 套件（数据集 + 评分器 + 通过阈值），纳入 CI。
  - 【R3.3】识别"需求蝗虫"信号（无限期、无指标、无裁判的 PoF——概念验证坟墓），触发止损机制。
- 产出物：Scope 文档、eval 仓库（含基线成绩）。
- 退出条件：eval 套件在客户数据上可运行并产出基线分数。

### 阶段 4 · BUILD（构建）

- 目标：用 AI 代理加速，交付生产级代码。
- 必做：
  - 【R4.1】分级推进：**MVD（最小可行部署）→ 试点 → 生产**，每级有明确晋级/回滚条件；原型代码进生产前必须过"生产化门槛"（见 §3.2）。
  - 【R4.2】客户环境优先：部署进客户已有云/容器/CI-CD 栈，不强行推销自家选型。
  - 【R4.3】AI 代理生成的代码 100% 人工 review 后才能合入（见 §4）。
- 产出物：可运行系统、repo、评估报告（每级一份）。

### 阶段 5 · DEPLOY & ITERATE（部署与迭代）

- 目标：系统在客户生产环境持续产生价值。
- 必做：
  - 【R5.1】上线即监控：指标、日志、告警就位；故障响应 SLA 书面约定。
  - 【R5.2】迭代节奏固定（周会 + QBR 季度业务回顾），每次迭代以 eval 分数与业务指标双轨汇报。
  - 【R5.3】"success is measured by whether the system keeps running, not hours logged"。
- 产出物：监控面板、迭代周报、QBR 材料。

### 阶段 6 · HANDOFF（交接即交付物）

- 目标：**离场标准 = 客户团队能自己运维**。
- 必做：
  - 【R6.1】交付证据包（见 §6 checklist）：架构文档、runbook、回滚预案、接管条件。
  - 【R6.2】客户团队实操演练 ≥1 轮（由客户执行、FDE 旁观）。
- 退出条件：客户签字验收 + 客户团队独立处理过至少 1 次真实故障。

### 阶段 7 · FEEDBACK（反哺闭环）

- 目标：把现场模式沉淀为可复用资产，避免退化为外包。
- 必做：
  - 【R7.1】每个 engagement 结束**强制**产出可复用资产：模板 / 连接器 / agent skill / playbook，至少一项回传产品与工程团队。
  - 【R7.2】度量"定制递减率"：第 N 个客户的定制量应显著小于第 1 个；不下降即触发模式复盘。
- 产出物：可复用资产 PR、复盘文档。

---

## 3. 工程规范

### 3.1 代码规范

- 【R3.1a】跟随客户既有代码风格与架构惯例；无惯例时默认：Python（ruff + mypy + pytest）或 TypeScript（biome/eslint + vitest）。
- 【R3.1b】一切变更走分支提案 + 人工 review 合入；agent 与人遵守同一规则。
- 【R3.1c】git 纪律：小步提交、可回滚（agent 改动必须能一键 diff + revert）；不 force-push 共享分支。
- 【R3.1d】密钥与客户数据不进代码库、不进 prompt 上下文日志；遵守客户权限体系，agent 操作全程留痕。

### 3.2 原型进生产的门槛（生产化 checklist）

进入客户生产环境前，以下各项必须全部为"是"：

1. eval 套件通过且分数 ≥ 约定阈值；
2. 有错误处理与降级路径（LLM 超时/限流/格式错误不致崩溃）；
3. 有监控与告警（延迟、错误率、token 成本）；
4. 密钥走客户密钥管理，无硬编码；
5. 有回滚方案（镜像/版本可回退）；
6. 至少一名客户工程师 review 并理解全部代码。

### 3.3 评测（eval）规范

- 【R3.3a】评测先于开发：没有 eval 套件不允许进入 BUILD 阶段。
- 【R3.3b】eval 随代码进版本管理，每次模型/提示词/框架升级必跑回归。
- 【R3.3c】"按结果收费"类合同必须以可量化评估体系为前提——eval 从质量工具升格为商业模式地基。

---

## 4. AI 工具使用规范

- 【R4.1a】**AI 代理（Claude Code / Codex / Cursor 等）产出的每一行代码，合入前必须由 FDE 人工 review**；review 标准与人写代码相同。
- 【R4.1b】项目约定文件（`AGENTS.md` / `CLAUDE.md`）**纳入版本管理**，按"企业策略 → 用户级 → 项目级"分层维护；单文件 ≤200 行，判据："删掉这行会导致代理犯错吗"。
- 【R4.1c】可复用的现场方法论沉淀为 agent skills（SKILL.md + 脚本），skills 进版本管理；参考 [rpraharaj/forward-deployed-engineer](https://github.com/rpraharaj/forward-deployed-engineer) 的 LAND→FRAME→UNDERSTAND→DESIGN 技能链。
- 【R4.1d】代理权限默认收紧：只读探索可直接放行；写操作/命令执行走白名单或逐项审批；无人值守（CI/headless）必须用 allowedTools 显式限权 + OS 沙箱。
- 【R4.1e】agent 的 hooks 提供确定性保障（编辑后自动 lint/test，失败即阻断）；**硬性安全保证放在权限层，hooks 只做软增强**。
- 【R4.1f】无法验证的任务不交给 agent：每个委派任务必须附带可运行的检查（测试套件退出码 / linter / 截图对比）。
- 【R4.1g】涉密红线：客户数据不出客户环境；确需云端模型处理时，走客户审批的合规通道并留存记录。

---

## 5. 客户沟通规范

- 【R5.1】节奏：kickoff 纪要当日回执；周报（进展/风险/需要的决策三项式）；QBR 对齐业务价值；客户健康分（使用/价值/关系/商业四类信号）每季度体检。
- 【R5.2】原则：low-ego、high agency；关键决策一律书面确认；"为了赢，我欠你这些真话"式直接反馈（Palantir 文化）。
- 【R5.3】双向翻译：对客户讲业务结果（省了多少小时、提升多少转化），对内部产品团队讲字段级反馈（哪个连接器缺、哪个接口难用）。
- 【R5.4】范围变更：任何"顺便能不能…"一律转书面 scope 变更流程，重新排期或换优先级，不当场答应。

---

## 6. 交付证据包（Handoff Evidence Pack）checklist

每个 engagement 结束时交付，缺一项视为未完成交接：

- [ ] Engagement Brief（含决策链）
- [ ] 真问题陈述 + 现场访谈纪要
- [ ] Scope 文档 + 全部变更记录
- [ ] eval 套件 + 基线与历次成绩
- [ ] 架构文档（含与客户系统的集成点）
- [ ] Runbook（启动/停止/常见故障/告警响应）
- [ ] 回滚预案
- [ ] 监控面板与告警规则（移交客户账户）
- [ ] 客户团队接管条件确认书 + 验收签字
- [ ] 可复用资产清单（已回传产品团队的 PR 链接）
- [ ] 复盘文档（定制递减率分析、可推广模式）

---

## 7. 度量体系（四层）

| 层 | 指标 | 目标示例 |
|---|---|---|
| 交付层 | 工期、eval 分数、一次验收通过率 | MVD ≤ 4 周；一次验收通过率 ≥ 80% |
| 客户层 | 激活率、系统持续运行率、NRR | NRR ≥ 100% |
| 商业层 | ARR/ACV、按结果收费占比 | 按结果收费占比逐年上升 |
| 组织层 | **定制递减率**、资产复用率 | 第 N 个客户定制量 < 第 1 个的 50% |

---

## 8. 参考实现与延伸阅读

- 中文专著：[xdash/FDE-the-Guidance-Book-of-Forward-Deployed-Engineer](https://github.com/xdash/FDE-the-Guidance-Book-of-Forward-Deployed-Engineer)（4.6k★，官网 fde4.ai，165 案例 + 指标附录）
- 工程规范参照：[yeasy/forward_deployed_engineering_guide](https://github.com/yeasy/forward_deployed_engineering_guide)（16 章 + 交付证据包模板）
- 资源聚合：[global-fde/awesome-fde-resources](https://github.com/global-fde/awesome-fde-resources)、[libaice/Awesome-FDE](https://github.com/libaice/Awesome-FDE)
- AI 时代工作方式快照：[rpraharaj/forward-deployed-engineer](https://github.com/rpraharaj/forward-deployed-engineer)（43 个 Claude Code skills）、[suboss87/FDEOps](https://github.com/suboss87/FDEOps)
- JD 硬数据：[146 份 FDE JD 分析](https://github.com/alexeygrigorev/ai-engineering-field-guide/blob/main/role/06-fde.md)、[Anthropic FDE JD](https://job-boards.greenhouse.io/anthropic/jobs/5302966008)、[OpenAI DeployCo 公告](https://openai.com/index/openai-launches-the-deployment-company/)
- 趋势综述：[The New Stack: Why OpenAI and Anthropic are hiring FDEs](https://thenewstack.io/forward-deployed-engineers-ai/)
- 视频：[Colin Jarvis (OpenAI FDE 负责人) 访谈](https://www.youtube.com/watch?v=cBD7_R-Cizg)、[B站：为什么FDE在AI时代这么火](https://www.bilibili.com/video/BV1tXKi6NE4b/)

（脚手架 / Loop / Graph 的技术实现细节见同目录《02-前沿实现调研-脚手架-Loop-Graph.md》。）
