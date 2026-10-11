# FDE 最佳实践手册

> 版本 v0.1（草案）· 2026-09-16 · 与《01-FDE工程规范.md》（v0.1，2026-09-08）配套：01 定"必须做什么"（七阶段闭环与硬性条款），本篇定"怎么做得出色"（方法论细则、商业与组织实践、招聘培养、风险规避）。规则编号前缀 **【F#.#】**。
>
> 依据来源：OpenAI DeployCo 官方公告、Palantir 官方博客一手文章、a16z / Emergence Capital、Latent.Space、MIT NANDA 报告、Exponent 面试指南等，2026-09-16 联网调研（英文一手为主），完整链接与验证状态见《04-参考资料.md》。

---

## 1. 2025–2026 行情速览（撰写与决策的外部基准）

- **OpenAI Deployment Company（DeployCo）**：官方公告 2026-08-06——通过收购 applied AI 咨询公司 **Tomoro** 带来约 150 名 FDE 与 Deployment Specialists；**19 家投资方/SI 合作伙伴**出资超 **40 亿美元**（TPG 领衔，Advent、Bain Capital、Brookfield 共同创始；Goldman Sachs、SoftBank Corp.、Bain、Capgemini、McKinsey 在列）；OpenAI 持多数股权。交付动作："诊断 → 选少量优先工作流 → FDE 进驻设计/构建/测试/部署生产系统"。注意口径：融资结构 2026-05 已由媒体披露（SiliconANGLE 引 Axios：对外部投资人承诺**最低 17.5% 年回报**；估值 $10B 与 $14B 两说），引用时写"2026 年组建、8 月正式发布"。
- **Anthropic**：Applied AI 设 "founding FDE"（NYC/SF/Seattle），年薪 **$280K–$320K**、差旅约 25%；JD 明文把交付物定义为 **MCP servers、sub-agents、agent skills**，并要求"识别可复用部署模式并回流产品"。
- **规模化走 SI 路线**：Anthropic–DXC 全球联盟（2026-06-11）将培养 **"tens of thousands"** 名 Claude 认证 FDE；DXC 先在内部验证（OASIS 平台 95% 代码由 Claude 写、提速约 10x）再对客交付。
- **Meta**：官方 careers 出现 "Forward Deployed Engineer, Enterprise AI" 岗（帮企业把 Meta 基础模型集成进核心产品）。
- **行情判断**（Forbes，2026-05）：Meta/OpenAI/Anthropic 以数十亿美元级投入 FDE，与 Accenture 等咨询巨头正面竞争；FDE 成为"企业界最贵岗位"（总包可超 $500K）。

> 规范含义：FDE 已从 Palantir 独家打法变成**行业默认交付形态**；本手册各条即以"如何在正规军竞争中取胜"为导向。

---

## 2. Discovery：找到真问题（细化 01 规范阶段 2）

- 【F2.1】**Shadowing 优先于访谈**：隐性知识 *"is lived, not stated"*——用户自己都说不清的依赖，只有坐在旁边看他过完一天才发现。一手案例：数据质量工程师阻挠 Parquet 迁移，真实原因是迁移会拿走她唯一的质检工具（访谈问不出，现场看一眼就懂）。
- 【F2.2】配套动作 "collect nouns and verbs"：把客户业务的本体词汇表（对象 + 动作）记全，这是后续 schema/ontology 与需求澄清的公共底座。
- 【F2.3】驻场节奏基准：每周 3–4 天客户现场；每周 5–15 场 discovery 对话；kickoff 纪要与访谈回执当日发出（01 规范 R5.1 的执行细则）。
- 【F2.4】"Ship on day one"：入职首周就交付一件可用的东西，在"期望值低到可笑"的环境里用快交付换信任。
- 【F2.5】推动"买 + 配置"而非"从零建"：MIT NANDA 实证——95% 企业 GenAI 试点无可衡量 P&L 影响，而**外购方案成功率（~67%）约为自建的 3 倍**；FDE 的价值在于把外购/平台配置到客户工作流，不是炫技自研。

## 3. 交付纪律（细化 01 规范阶段 4–5）

- 【F3.1】Eval-first 是能力试金石：OpenAI FDE 面试核心题即 *"How do you know your AI system is actually working?"*——回答不出评测方案的 FDE 不合格。执行标准直接复用 `../loop-engineering/01-技术研究规范.md` §3（20–50 个真实失败起步、只评产物、pass^k）。
- 【F3.2】**Hack 纪律**："Every shortcut you ship becomes something you own"——每个上线的快捷方式都变成你的负债；对每个 hack 显式做"升平台 or 丢弃"决定，写入迭代周报。
- 【F3.3】交付物 AI-native 化（对齐 Anthropic JD 的最新口径）：MCP server、agent skills、subagent、eval 套件都是合法交付物；01 规范 §6 的"交付证据包"在 AI 项目中应升级为**可复用资产包**（资产 + 回流产品的机制）。
- 【F3.4】内部先验证（drink your own champagne）：像 DXC 一样，用同一套 agent 工具流先跑通自己团队的交付（95% 代码 agent 写、10x 提速），再向客户承诺同等能力——这是 FDE 报价与承诺的底气来源。
- 【F3.5】工具链主力化：把 Claude Code / Codex 类 agent 当主力交付工具（01 规范 §4），且按 `../loop-engineering/02-最佳实践.md` 配齐验证闭环与权限约束。

## 4. 商业与组织（01 规范阶段 7 的放大器）

- 【F4.1】**汇报线给产品，不给销售**：FDE 组织汇报给产品线；"一个 engagement 结束时只有一个满意客户、上游产品毫无变化 = 失败"——与 01 规范"定制递减率"（R7.2）互为因果：上游不吸收，定制永远递减不下来。
- 【F4.2】按结果计费已可落地，不必观望：Sierra——"只有对话被解决才收费"；Intercom Fin——**$0.99/resolution** 已进标价页、每对话只计一次结果。落地前提是 01 规范 R3.3c 的可量化评估体系。
- 【F4.3】低毛利是策略不是妥协（a16z "Trading Margin for Moat"）：企业买 AI 像奶奶买 iPhone——"会用，但需要你装"；AI 首次让 services 可规模化（implementation 工作本身可被 AI 自动化）；路径是先做客户的 **"system of work"** 再成为 "system of record"。参照轨迹：Salesforce/ServiceNow/Workday 均从 IPO 时 54–63% 毛利爬到 ~75–79%。
- 【F4.4】护城河叙事："AI models are the gold, and the FDEs are the miners"、"People, not models, build the moat"（Emergence Capital）；下一个产品机会是 **FDE picks and shovels**（FDE 工具链产品化）。
- 【F4.5】配比与经济性参考：1 名 FDE 对应 **$2–5M** 企业 pipeline（二手综合，作规划下限校验用）；Palantir 拒绝人力外派型收入，FDE 团队不做 SI 的外包炉子。
- 【F4.6】内部造血：把每个 engagement 的 discovery 记录、ontology 词汇表、eval 套件、skills 沉淀为组织资产（对应 01 规范 R7.1），使第 N 个客户的交付成本低于第 1 个——这是"产品发现机制"与"高级外包"的分水岭（见 §7）。

## 5. 招聘与培养

- 【F5.1】面试结构对标（Exponent 2026 指南）：5–8 轮、3–6 周；**开放案例分解（decomposition）是最大过滤器**——先澄清问题、命名假设、暴露失败模式、提 "walking-skeleton" MVP；**直接跳答案是最常见拒因**。设 client simulation 轮（向"客户"汇报坏消息并处理升级）。take-home 4–8 小时（OpenAI：约 5 小时 API 构建 + eval 主题；Anthropic：CodeSignal 屏 + mission round）。
- 【F5.2】不考 LeetCode 难题，考现场 build 与模糊性容忍。
- 【F5.3】薪酬定位：FDE 较 solutions engineer 高 **30–60%**；参照 Palantir $205K–$486K（staff 级 $630K+）、OpenAI 总包约 $860K（The New Stack）。差额来自三个结构性差异：写不写生产代码、按 outcome 还是按 pitch 计价、人才池大小。
- 【F5.4】SWE 转 FDE 补三件：生产级客户现场编码；AI/LLM 系统深度（retrieval / agent orchestration / **eval harnesses**）；deployment-grade 系统思维（SSO / VPC / SOC 2 / FedRAMP 约束）。软素质："high agency and insatiably curious"（a16z）。
- 【F5.5】组织接口：明确 reporting line（产品线）、现场率（3–4 天/周）、与 SI/渠道的合作边界（FDE 不做外派炉子）。

## 6. 风险登记簿（写 engagement 计划时逐项过）

| 风险 | 信号 | 缓解 |
|---|---|---|
| 退化为高级外包 | 客户只买人头、上游产品无变化、定制递减率不降 | F4.1/F4.6：回流机制 + 递减率红线（01 规范 R7.2） |
| POF（概念验证坟墓） | 无限期、无指标、无裁判的试点 | 01 规范 R3.3 止损机制 + MIT NANDA 95% 教训 |
| 财务结构风险（自营 FDE 业务时） | 对赌式回报承诺、按人头计收入 | 定价向 outcome-based 迁移（F4.2），合同设利润上限预警 |
| 能力错配 | FDE 只会"铺工具"，接不住旗舰特性 | F5 面试过滤 + F3.4 内部先验证 |
| burnout | 永久堵产品缺口、随叫随到 | 缺口回写产品团队（F4.1）、驻场节奏上限（F2.3） |
| 岗位时效性争议 | "agent 将自动化 FDE"论（见 §7） | 把可自动化的部分自动化（用 loop/graph 工程成果武装自己），人聚焦 discovery/判断/责任承担 |

## 7. 争议与批评（决策前应知）

1. **"新咨询/分销渠道"质疑**（beam.ai）：DeployCo 是 "a compute-pull strategy built on a services wrapper"，不同于 Palantir FDE 的"产品发现机制"；线性扩展——"第 100 个部署的成本与第 1 个差不多"；"顾问离开时，机构知识随之离开"；FDE 总包 $171K–$800K 使模式只在七位数合同上经济。
2. **能力分布批评**（Ode with Anthropic CEO）："Very few [FDEs] are capable of building your flagship AI product feature"——多数只到"铺工具"水平；对 hiring 的含义：把"能接旗舰特性"作为最高档定价与筛选线。
3. **岗位时效性质疑**（Christian & Timbers）：AI agent 或在 5–10 年内自动化大量 FDE 工作。反方数据（同文）：全美具备"领域知识 + gravitas + applied AI"组合能力者**总共约 2,000 人**（"Not 2,000 available, 2,000 total"）、FDE 需求预计涨 2,100%、有招聘计划的企业从年初 5–10% 升至 Q2 末 70%。
4. **口径分歧**：DeployCo 估值 $10B vs $14B；发布时点 5 月（融资报道）vs 8 月（官方 launch）；"Anthropic $1.5B 对标公司"仅单一来源（beam.ai），引用降级为"有报道称"。

## 8. 一页速查（engagement 启动前自检）

- [ ] 汇报线：产品线（非销售）（F4.1）
- [ ] discovery 计划：shadowing ≥1 整天 + 本体词汇表（F2.1/F2.2）
- [ ] 首周可交付物已定义（F2.4）
- [ ] eval 套件与成功指标先于代码（F3.1，01 规范 R3.3a）
- [ ] "买+配置"机会已评估而非默认自研（F2.5）
- [ ] 交付物含 AI-native 资产 + 回流机制（F3.3，01 规范 R7.1）
- [ ] hack 台账建立（每个快捷方式记"升平台 or 丢弃"）（F3.2）
- [ ] 定价与商业模式：outcome 占比有提升路径（F4.2/F4.3）
- [ ] 定制递减率红线与复盘触发器（F4.6，01 规范 R7.2）
- [ ] burnout 防线：缺口回写产品的通道存在（F4.1、§6）
