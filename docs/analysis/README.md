# OpenCodeReview 仓库深度分析文档索引

本系列是对 [open-code-review](../../README.md)（`ocr`，AI 代码审查 CLI）仓库的穷尽式源码分析，共 18 篇模块文档 + 本索引。全部文档基于源码逐文件走读撰写，行为论断均带行号锚点链接；各模块文档末尾的「源文件覆盖清单」与该模块全部非测试源文件一一对应。

> **统计**：18 篇正文约 8500 行；mermaid 架构图 51 张；源码引用链接 3300+ 个（已全量校验有效）。
> **约定**：文档内源码链接使用 `../../` 相对前缀；图编号格式为 `图 <文档号>-<序号>`；「待与维护者确认」项在各篇末尾集中列出，并汇总于 [01-architecture.md](01-architecture.md) 第 9 节。

## 目录

- [文档地图](#文档地图)
- [阅读路线](#阅读路线)
- [统一术语表](#统一术语表)
- [图索引](#图索引)
- [质量与校验说明](#质量与校验说明)

## 文档地图

| 编号 | 文档 | 一句话摘要 |
|---|---|---|
| 00 | [项目总览](00-overview.md) | 项目背景、能力矩阵、技术栈、目录导览、术语表、构建命令速查 |
| 01 | [总体架构与全局图](01-architecture.md) | 系列收敛篇：五层架构、包依赖全景、16 项设计决策、已知问题汇总、扩展点指南 |
| 02 | [CLI 命令层](02-cli-commands.md) | cobra 命令树、review/scan 走读、Provider TUI 状态机、输出系统与平台差异 |
| 03 | [Diff 解析引擎](03-diff-engine.md) | git 调用层、unified diff 解析状态机、行号映射与评论重定位（确定性工程的地基） |
| 04 | [配置与规则系统](04-config-rules.md) | 规则四层解析链、语言嗅探器、rule_docs 体系、六组 prompt 模板、effort 分级 |
| 05 | [LLM 提供商层](05-llm-providers.md) | 28 个内置 provider、统一协议层、解析链四级策略、重试四件套、token 计量 |
| 06 | [Agent 与执行循环](06-agent-loop.md) | Agent 生命周期、文件分组、token 预算、llmloop 主循环、并发池、上下文压缩 |
| 07 | [Agent 工具系统](07-tool-system.md) | 六个只读检索/受控提交工具、code_comment 参数修复、评论收集器 |
| 08 | [Review 端到端流水线](08-review-pipeline.md) | `ocr review` 九阶段总装、输出格式分支、SARIF、GitHub Action 集成链路 |
| 09 | [Scan 流水线](09-scan-pipeline.md) | 全仓扫描的第二条编排流水线：文件枚举、批处理、逐文件预算前瞻门 |
| 10 | [会话持久化与断点续跑](10-session-persistence.md) | JSONL 会话存储、manifest 结构、resume 身份校验、orphan request 处理 |
| 11 | [Delegate 委托与 MCP](11-delegate-mcp.md) | 免 LLM 的委托模式；MCP 为 client-only 定位（无 server 入口） |
| 12 | [Viewer Web 服务器](12-viewer-server.md) | 本地会话浏览器：路由、hostguard 防 DNS rebinding、安全头、零 XHR 前端 |
| 13 | [遥测系统](13-telemetry.md) | OTel 门面、默认关闭的隐私边界、指标清单、span 树与优雅关闭 |
| 14 | [VS Code 扩展](14-vscode-extension.md) | extension host / webview / CLI 子进程三方架构、进度流解析、评论行号对齐 |
| 15 | [分发、发布与 CI](15-distribution-release.md) | npm 六平台子包、安装脚本、action.yml 契约、10 个 workflow、发布流水线 |
| 16 | [测试与质量保障](16-testing-quality.md) | 四层测试组织、test_home 隔离约定、15 项质量门禁、90% 覆盖率工程 |
| 17 | [文档站、插件与技能生态](17-ecosystem.md) | React SPA 文档站、五语言 i18n、四家 agent 插件、SKILL 机制、翻译门禁 |

## 阅读路线

按读者角色推荐三条路线（详细版含图见 [00-overview.md 第 7 节](00-overview.md)）：

**内核开发者**（给 Go 内核贡献代码 / 深度理解引擎）：
00 → 03 → 04 → 05 → 06 → 07 → 08 → 10 → 13（→ 09 → 01 收尾）

**CI 集成者**（在流水线接入 ocr）：
00 → 15 → 08 → 11 → 09 → 16

**扩展开发者**（VS Code 扩展 / agent 插件 / 自定义规则）：
00 → 14 → 17 → 04 → 11

> 01（总体架构）是全部模块篇的收敛综合，依赖矩阵与设计决策均引自各篇结论——无论走哪条路线，都建议最后（或配合 00 总览先行浏览）阅读。

## 统一术语表

完整版（含源码佐证链接）见 [00-overview.md 第 5 节](00-overview.md)。全系列统一使用以下叫法：

| 术语 | 含义 | 详解位置 |
|---|---|---|
| Review | 基于 git diff 的变更审查（`ocr review`） | [08](08-review-pipeline.md) |
| Scan | 全文件/整仓扫描审计（`ocr scan`） | [09](09-scan-pipeline.md) |
| Delegate | 免 LLM 的委托模式：输出规格供宿主 agent 执行 | [11](11-delegate-mcp.md) |
| Session | 一次执行的会话记录（JSONL 单文件） | [10](10-session-persistence.md) |
| Manifest | 会话内嵌的运行清单：身份、状态与 item 进度 | [10](10-session-persistence.md) |
| Provider | LLM 接入配置（endpoint/token/model 三元组） | [05](05-llm-providers.md) |
| Rule / Rule Doc | 审查规则 / 按语言组织的规则文档（rule_docs/） | [04](04-config-rules.md) |
| Sniffer | 文件类型嗅探器（决定命中哪篇 rule doc） | [04](04-config-rules.md) |
| Grouping / Bundle | 相关文件捆绑为一个审查单元（子 agent 上下文隔离） | [06](06-agent-loop.md) |
| Re-location | 评论行号的三级重定位（本文件 → 跨文件 → LLM） | [03](03-diff-engine.md) |
| Review Filter | 评论反思过滤 pass（抑制误报） | [04](04-config-rules.md)、[08](08-review-pipeline.md) |
| llmloop | 与 LLM 交互的通用执行循环（工具分发、压缩、轮次控制） | [06](06-agent-loop.md) |
| Sealed Input | Run 开始时封存的输入快照（resume 幂等基础） | [06](06-agent-loop.md)、[10](10-session-persistence.md) |
| Resume | 断点续跑（身份校验 + item 复用，从不重放 LLM 请求） | [10](10-session-persistence.md) |
| Retry Identity | 可重试请求的稳定身份（与 resume 幂等联动） | [05](05-llm-providers.md) |
| SARIF | 静态分析结果交换格式（`--format sarif`） | [08](08-review-pipeline.md) |
| Viewer | 本地 Web 会话浏览器（`ocr viewer`） | [12](12-viewer-server.md) |
| TUI | Provider 管理的终端界面（bubbletea） | [02](02-cli-commands.md) |
| Audience | 输出受众（human/agent，改变输出组织） | [08](08-review-pipeline.md) |
| Effort | 审查投入分级（低/中/高，控制 MaxReviewRounds） | [04](04-config-rules.md) |
| test_home | 测试的 HOME 重定向隔离约定（5 个包共用） | [16](16-testing-quality.md) |
| Checkpoint | scan 的批内检查点（崩溃后续跑的最小单位） | [09](09-scan-pipeline.md)、[10](10-session-persistence.md) |

## 图索引

全系列 51 张 mermaid 图。按文档列出（`文档号-序号 标题`）：

| 文档 | 图 |
|---|---|
| 00 总览 | 0-1 仓库结构 mindmap；0-2 三条阅读路线 |
| 01 架构 | 1-1 架构哲学全景；1-2 五层架构；1-3 端到端审查主链路；1-4 内部包依赖图；1-5 review 并发拓扑；1-6 scan 并发拓扑；1-7 核心实体关系；1-8 会话生命周期 |
| 02 CLI | 2-1 命令树；2-2 review/scan 共享执行管线；2-3 Provider TUI 状态机 |
| 03 Diff | 3-1 解析流水线；3-2 行号映射；3-3 评论重定位；3-4 解析器逐行状态机 |
| 04 配置 | 4-1 文件→嗅探→规则→prompt；4-2 配置解析优先级；4-3 六组 prompt 与流水线阶段映射 |
| 05 LLM | 5-1 provider 解析决策树；5-2 请求→重试→上报时序；5-3 重试判定状态机；5-4 协议适配类图 |
| 06 Agent | 6-1 Agent 生命周期；6-2 主循环消息流；6-3 CommentWorkerPool 并发模型；6-4 压缩触发流程 |
| 07 工具 | 7-1 工具调用全链路；7-2 code_comment 参数修复与定位 |
| 08 Review | 8-1 端到端时序；8-2 输出格式分支；8-3 GitHub Action 集成链路 |
| 09 Scan | 9-1 流水线总览；9-2 batch 切分与预算控制 |
| 10 Session | 10-1 生命周期与 resume 链；10-2 resume 判定流程 |
| 11 Delegate | 11-1 委托模式端到端时序；11-2 MCP 集成架构 |
| 12 Viewer | 12-1 HTTP 请求处理流水线；12-2 威胁→防护层映射 |
| 13 遥测 | 13-1 遥测数据流；13-2 配置解析优先级；13-3 review 的 span 树 |
| 14 扩展 | 14-1 三方架构；14-2 CLI 输出→UI 状态数据流；14-3 评论行号对齐 |
| 15 分发 | 15-1 npm 安装决策流；15-2 GitHub Action 事件流；15-3 workflow 依赖关系；15-4 发布流水线 |
| 16 质量 | 16-1 质量门禁流水线；16-2 测试体系分层 |
| 17 生态 | 17-1 生态全景；17-2 插件调用链 |

## 质量与校验说明

1. **链接校验**：全部 3300+ 个源码相对链接经过脚本化存在性校验与行号范围校验（0 断链、0 越界）。
2. **覆盖清单**：每篇模块文档末尾的「源文件覆盖清单」与该模块 Glob 实际非测试源文件逐一对账。
3. **以源码为准**：与 README/官方文档冲突处均显式标注（汇总见 [01-architecture.md](01-architecture.md) 第 9.1 节文档-源码冲突表，共 7 项）；写作期间发现并修正的错误断言（如 04 篇 scan 超时驱动因素）在文中留有修正记录。
4. **待确认项**：各篇「待与维护者确认」合计约 30 条，收敛于 [01-architecture.md](01-architecture.md) 第 9 节；其中对用户影响最直接的两条：`review_cmd.go` TraceID 读取错误 context 疑似 bug（[13 篇](13-telemetry.md)）、o200k_base BPE 未内嵌导致部分模型 token 计量回退（[05 篇](05-llm-providers.md)）。
5. **图规范**：仅使用 flowchart / sequenceDiagram / stateDiagram-v2 / mindmap / classDiagram 五种语法，单图节点 ≤ 20，节点 id 纯 ASCII。
