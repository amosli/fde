# fde · 前向部署工程（Forward Deployed Engineering）

> **一句话定位**：FDE 是驻扎客户现场、填补"产品能做的事"与"客户需要的事"之间鸿沟、写生产代码并对业务结果负责的工程师。本目录从角色规范（01）、技术栈调研（02）到最佳实践（03）逐层展开。

## 文档导航

| 文档 | 内容 | 版本 |
|---|---|---|
| [01-FDE工程规范.md](01-FDE工程规范.md) | 角色定义与边界、能力三角、七阶段交付闭环（LAND→DISCOVER→SCOPE&EVAL→BUILD→DEPLOY&ITERATE→HANDOFF→FEEDBACK）、生产化门槛、AI 工具规范、交付证据包、四层度量（条款【R#.#】） | v0.1 · 2026-09-08 |
| [02-前沿实现调研-脚手架-Loop-Graph.md](02-前沿实现调研-脚手架-Loop-Graph.md) | FDE 技术栈三件套的基础调研：脚手架横向对比、Agent Loop 实现、Graph 编排实现、选型判断、资源总表 | 2026-09-08 |
| [03-FDE最佳实践.md](03-FDE最佳实践.md) | 2026 行情速览（OpenAI DeployCo / Anthropic / DXC）、discovery 与交付纪律细则、商业与组织（outcome-based pricing、汇报产品线）、招聘培养、风险登记簿、争议与批评（条款【F#.#】） | v0.1 · 2026-09-16 |
| [04-参考资料.md](04-参考资料.md) | 本轮新增来源（OpenAI/Palantir/a16z/MIT NANDA/Latent.Space 等，含验证状态与存疑项标注） | 2026-09-16 |
| [05-国内厂商研究.md](05-国内厂商研究.md) | 国内图景：FDE 岗位直设（字节/腾讯）与政府制度化（上海培训班）、四类交付模式（DeepSeek 不交付 / 阿里平台+生态 / 智谱火山重交付 / MiniMax 轻交付出海）、政企硬约束（信创/验收制/私有化）、Manus 收购案、市场数据锚点 | v0.1 · 2026-09-16 |

## 核心结论（2026-09 增量）

1. FDE 已成行业默认交付形态：OpenAI DeployCo（2026-08-06 官方发布，收购 Tomoro，约 150 名 FDE、19 家伙伴、超 $4B）、Anthropic founding FDE（交付物已定义为 MCP servers / sub-agents / skills）、Meta 组建 Enterprise AI FDE、Anthropic–DXC 以 SI 联盟规模化数万认证 FDE。
2. 方法论锚点：shadowing 优先于访谈（"tacit knowledge is lived, not stated"）、ship on day one、eval-first（"How do you know your AI system is actually working?"）、每个 hack 显式决定"升平台 or 丢弃"。
3. 组织与商业：汇报产品线而非销售；engagement 必须改变上游产品（否则=外包）；outcome-based pricing 已可标准化（Intercom $0.99/resolution）；低毛利换 "system of work → system of record"（a16z）。
4. 风险与争议并存：95% 企业 GenAI 试点无 P&L 影响（MIT NANDA）、"第 100 个部署成本与第 1 个相同"的线性扩展质疑、"agent 将自动化 FDE"的时效性争论。
5. **国内增量**：国内是"项目制+验收制+私有化/信创前置"，与"订阅制+outcome-based"的国际模式并存；建议 01 规范增设"阶段 0 · COMPLY 合规前置"，验收指标与 eval 套件双向翻译；FDE 在国内已被政府制度化（上海"百千万"工程、千人储备）。

## 与其他目录的关系

- **技术栈**：FDE 的循环层规范 → [`../loop-engineering/`](../loop-engineering/README.md)；编排层规范 → [`../graph-engineering/`](../graph-engineering/README.md)
- **工具层**：脚手架对比 → [02 文档 §1](02-前沿实现调研-脚手架-Loop-Graph.md)
- 本目录 01/02 两份文档源自 `/data/fde`（2026-09-08），已并入本仓维护。
