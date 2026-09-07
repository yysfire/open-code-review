# 仓库深度分析与文档化实施计划（Repository Deep-Analysis & Documentation Plan）

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 对 open-code-review 仓库做穷尽式源码分析，产出 19 篇中文分析文档（含 mermaid 架构图），覆盖每一个实现细节，让读者能从总览一路按图索骥到具体代码行。

**Architecture:** 分析分五个阶段推进：基础总览 → 接入层与核心引擎 → 业务流水线 → 外围生态 → 综合架构与交叉校验。每个任务对应一篇文档；文档间通过编号与「前置阅读」形成阅读网络。综合架构篇（01）放在最后写，作为全部模块文档的收敛与升华，确保架构图与已核实的实现细节一致。

**Tech Stack:** 分析对象：Go 1.25（cobra、bubbletea TUI、anthropic-sdk-go、openai-go/v3、MCP go-sdk、tiktoken-go、OpenTelemetry）+ TypeScript（VS Code 扩展 React webview、文档站）。产出物：纯 Markdown + mermaid 图，不改动任何源码。

---

## 0. 背景与仓库现状

open-code-review（`ocr`）是阿里开源的 AI 代码审查 CLI，核心哲学是「确定性工程 × Agent 混合」。仓库现状：

- **Go 核心**：`cmd/opencodereview`（CLI 入口，约 40 个非测试源文件）+ `internal/` 下 19 个包（agent、config、delegate、diff、gitcmd、llm、llmloop、mcp、model、pathutil、release、scan、session、stdout、suggestdiff、telemetry、tool、viewer）
- **VS Code 扩展**：`extensions/vscode`（TypeScript，extension host + React webview 双侧）
- **文档站**：`pages/`（多语言文档站，5 种语言）
- **生态**：`plugins/`（Claude Code / Codex / Cursor / opencode 插件）、`skills/`、`examples/`（6 种 CI 集成）
- **分发**：npm 多平台包（`bin/ocr.js` + `npm/` 6 平台目录）、install 脚本、GitHub Action（`action.yml`）
- **质量保障**：90% 覆盖率门槛、license/english-only 检查、10 个 GitHub workflow

## 1. 交付物：文档地图

全部文档位于 `docs/analysis/`，简体中文正文，文件名用英文：

| 编号 | 文件 | 主题 | 关联源码 |
|---|---|---|---|
| — | README.md | 文档索引 + 阅读路线 + 统一术语表 | 全部 |
| 00 | 00-overview.md | 项目总览、技术栈、目录导览 | README.md、go.mod、Makefile 等 |
| 01 | 01-architecture.md | 总体架构与全局图（最后写） | 综合全部 |
| 02 | 02-cli-commands.md | CLI 命令层 | cmd/opencodereview |
| 03 | 03-diff-engine.md | Diff 解析引擎 | internal/diff、internal/gitcmd |
| 04 | 04-config-rules.md | 配置与规则系统 | internal/config |
| 05 | 05-llm-providers.md | LLM 提供商层 | internal/llm |
| 06 | 06-agent-loop.md | Agent 与执行循环 | internal/agent、internal/llmloop |
| 07 | 07-tool-system.md | Agent 工具系统 | internal/tool |
| 08 | 08-review-pipeline.md | Review 端到端流水线 | review_cmd.go、输出系统 |
| 09 | 09-scan-pipeline.md | Scan 流水线 | internal/scan |
| 10 | 10-session-persistence.md | 会话持久化与断点续跑 | internal/session |
| 11 | 11-delegate-mcp.md | Delegate 委托与 MCP | internal/delegate、internal/mcp |
| 12 | 12-viewer-server.md | Viewer Web 服务器 | internal/viewer |
| 13 | 13-telemetry.md | 遥测系统 | internal/telemetry |
| 14 | 14-vscode-extension.md | VS Code 扩展 | extensions/vscode |
| 15 | 15-distribution-release.md | 分发、发布与 CI | bin、npm、scripts、.github |
| 16 | 16-testing-quality.md | 测试与质量保障 | Makefile、scripts、CI |
| 17 | 17-ecosystem.md | 文档站、插件与技能生态 | pages、plugins、skills |

**写作顺序**：Task 编号即执行顺序。注意 00 最先写（粗粒度地图），01 最后写（精粒度综合），README 索引收尾。

## 2. 文档编写规范（全局约束，每个任务必须遵守）

1. **语言**：简体中文正文；代码、标识符、flag 名、专有名词保持英文原样。
2. **文档头模板**（每篇必含）：

   ```markdown
   # <标题>

   > **关联源码**：`<目录/文件列表>`
   > **前置阅读**：<编号-标题 列表>

   ## 目录
   <锚点列表>
   ```

3. **代码引用**：使用相对路径链接，从 `docs/analysis/` 出发，格式 `[file.go](../../internal/xxx/file.go#L10-L20)`。关键论断必须落到行号级链接；目录级引用可不带锚点。
4. **图表**：用 ```mermaid 代码块。每张图前加一行 `**图 N-M**：<标题>`（N 为文档编号，M 为图序）。每张图节点数 ≤ 20，只用 flowchart / sequenceDiagram / stateDiagram-v2 / mindmap / classDiagram 五类语法，避免冷门语法导致渲染失败。
5. **覆盖清单**：每篇模块文档末尾必须有「源文件覆盖清单」表，列出该模块**全部非测试源文件**：

   | 源文件 | 职责 | 关键符号 |
   |---|---|---|

   这是「每一个细节」的可验证标准：清单与实际文件一一对应，不允许遗漏。
6. **禁止臆测**：每个行为论断要么有代码链接，要么有测试佐证。无法确认的内容先用 Grep / git log 求证；仍无果则明确标注「待与维护者确认」并给出疑点，不得编造。
7. **与官方文档冲突时**：以源码为准，并在文中显式标注差异点。
8. **Markdown 不参与 english-check**（AGENTS.md 明确 Markdown out of scope），中文内容安全；但不要创建任何 `.go/.sh/.js/.ts` 文件，本计划零代码变更。

## 3. 标准任务流程（所有任务共用）

每个任务按以下五步执行，任务明细（第 4 节）只列差异部分（阅读范围、章节大纲、必备图、验收、提交信息）：

- [ ] **Step 1 — 阅读**：用 Read 工具通读「阅读范围」内全部非测试源文件；用 Grep 追踪跨包调用（如 `internal/agent` 调用 `internal/llm` 的具体接口）。先读非测试文件，测试文件用于佐证行为。
- [ ] **Step 2 — 撰写**：按「章节大纲」创建/更新文档，遵守第 2 节规范。每个章节的描述必须基于 Step 1 实际读到的代码。
- [ ] **Step 3 — 绘图**：绘制「必备图」列出的 mermaid 图，嵌入对应章节。
- [ ] **Step 4 — 验收**：执行该任务「验收」条目 + 通用验收：
  - 用 Glob 列出该模块 `*.go`（或 `.ts/.tsx`），剔除 `*_test.go`（或 `__tests__`），逐个核对「覆盖清单」无遗漏；
  - 抽查文档中 5 个行号链接，确认范围与描述匹配；
  - mermaid 图逐张检查语法（节点 id 唯一、箭头 `-->`、sequenceDiagram 参与者已声明）。
- [ ] **Step 5 — 提交（仅在用户已批准提交时执行）**：
  ```bash
  git log --oneline -n 5   # 先看仓库提交风格，若无先例则用任务给出的消息
  make build && ./dist/opencodereview review --audience agent --background "<本任务一句话摘要>"
  # ^ 若沙箱未配置 LLM 导致无法执行，记录原因并跳过，直接进入提交
  git add docs/analysis/<本任务文件>
  git commit -m "docs(analysis): <英文提交信息>"
  ```

## 4. 阶段与任务明细

### 阶段 1 — 基础

#### Task 1：文档规范落地 + 00-overview.md

**产出**：`docs/analysis/00-overview.md`
**阅读范围**：`README.md`（全文）、`AGENTS.md`、`ROADMAP.md`、`go.mod`、`Makefile`、`package.json`、`action.yml`、`CONTRIBUTING.md`、`SECURITY.md`、`ASSURANCE_CASE.md`（后三个可略读）
**章节大纲**：
1. 项目定位与背景（阿里内部孵化史、AACR-Bench 基准、精确率优先的取舍）
2. 核心能力矩阵（Review / Scan / Delegate / Viewer / Session 续跑 / VS Code 扩展 / GitHub Action / Agent 插件，每项附入口代码链接）
3. 技术栈清单（Go 依赖逐条注明用途，来源 go.mod；前端依赖以 `extensions/vscode/package.json` 与 `pages/package.json` 实际内容为准）
4. 仓库目录导览（表格：路径 → 职责一句话 → 对应分析文档编号）
5. 核心术语表（≥ 15 条：Review、Scan、Delegate、Session、Manifest、Provider、Rule、Rule Doc、Sniffer、Grouping、Bundle、Re-location、Review Filter、Comment、SARIF、Viewer、TUI、Audience，每条给代码佐证链接）
6. 构建与常用命令速查（make build/test/check/coverage/dist、ocr 十个子命令一览表）
7. 分析文档阅读路线（三条：内核开发者路线、CI 集成者路线、扩展开发者路线，各为有序文档编号列表）

**必备图**：仓库结构 mindmap（根节点 open-code-review，分支按顶层目录）；三条阅读路线 flowchart LR
**验收**：目录导览表覆盖全部顶层目录（与 LS 结果一致，含 `.github`、`examples`、`imgs` 等）；术语表每条有链接；子命令表与 `root.go` 中 `AddCommand` 列表完全一致（10 个）
**提交**：`docs(analysis): add analysis conventions and repo overview`

### 阶段 2 — 接入层与核心引擎

#### Task 2：02-cli-commands.md

**产出**：`docs/analysis/02-cli-commands.md`
**阅读范围**：`cmd/opencodereview/` 全部非测试 `.go`（约 40 个：root.go、main.go、review_cmd.go、scan_cmd.go、session_cmd.go、config_cmd.go、llm_cmd.go、rules_cmd.go、viewer_cmd.go、delegate_cmd.go、provider_cmd.go、provider_tui.go、completion.go、version.go、shared.go、shared_flags.go、output.go、sarif.go、color.go、flag_suggest.go、arg_errors.go、background_file.go、git.go、shell_unix.go、shell_windows.go、procattr_unix.go、procattr_windows.go）
**章节大纲**：
1. 入口与命令树（main.go → root.go 的 cobra 结构、PersistentPreRunE 的 color 校验）
2. 命令清单总表（10 个子命令：用途、关键 flags、输出格式、跳转链接）
3. 子命令走读（每个子命令一节：参数解析 → 校验 → 编排调用哪些 internal 包 → 返回处理）
4. 共享机制（shared_flags 的公共 flag 集、color 体系、flag_suggest 参数建议算法、arg_errors 错误规范、completion 生成）
5. Provider TUI（provider_tui.go：bubbletea 模型、表单/编辑/删除确认/回滚各视图，从 *_test.go 的场景命名佐证）
6. 输出系统（output.go 的格式分支、sarif.go 的 SARIF 生成、background_file、progress stream）
7. 平台差异（shell_*.go / procattr_*.go 的 build tags 与差异点）
8. 测试布局概览（cmd 包内单测 / fake LLM / e2e 三类测试的组织方式）
9. 覆盖清单表

**必备图**：命令树 mindmap；review/scan 参数解析→分发 flowchart；Provider TUI 状态机 stateDiagram-v2
**验收**：覆盖清单与 Glob `cmd/opencodereview/*.go` 剔除测试后一一对应；命令表与 root.go 实际注册一致
**提交**：`docs(analysis): add CLI command layer analysis`

#### Task 3：03-diff-engine.md

**产出**：`docs/analysis/03-diff-engine.md`
**阅读范围**：`internal/diff/` 全部非测试 .go（git.go、parser.go、hunk.go、relocation.go、resolver.go、gitignore.go、workspace_file.go）、`internal/gitcmd/runner.go`、`internal/model/diff.go`；参考 `cmd/opencodereview/git.go`
**章节大纲**：
1. 模块职责与在整体架构中的位置
2. git 命令执行层（gitcmd/runner.go：命令构造、环境变量、LC_ALL、错误规范化）
3. diff 获取策略（git.go：base/head 选择、staged 模式、PR/commit range 模式、git 错误恢复路径）
4. unified diff 解析器走读（parser.go：逐行状态机、文件头/hunk 头解析、边界情况）
5. 数据模型（model/diff.go：FileDiff/Hunk/Line 结构、新旧行号语义）
6. 行号解析算法（resolver.go：old/new 行号映射）
7. 评论重定位机制（relocation.go：跨文件重定位，配合 re_location prompt 的使用场景）
8. 过滤体系（gitignore.go 与 allowlist 的 default_exclude_patterns 联动）
9. workspace_file.go 职责
10. 错误处理与边界情况汇总表
11. 测试覆盖分析（从 parser_test.go 等提炼关键用例意图）
12. 覆盖清单表

**必备图**：diff 解析流水线 flowchart；行号映射示意图（flowchart 模拟左右两栏）；relocation 流程图
**验收**：覆盖清单含 internal/diff 与 internal/gitcmd 全部非测试文件；解析器走读覆盖 parser.go 每个导出函数
**提交**：`docs(analysis): add diff engine analysis`

#### Task 4：04-config-rules.md

**产出**：`docs/analysis/04-config-rules.md`
**阅读范围**：`internal/config/` 全部非测试 .go：rules/（sniffer.go、system_rules.go、system_rules.json、rule_docs/ 目录下 45+ 语言文档抽样 5 篇精读、其余列表统计）、template/（template.go、effort.go、prompts/ 全部 12 个 .md、task_template.json、scan_template.json）、allowlist/（allowed_ext.go、supported_file_types.json、default_exclude_patterns.json）、toolsconfig/（toolsconfig.go、tools.json）、testconnection/（testconnection.go、task.json）；另读 `.opencodereview/rule.json` 与 `cmd/opencodereview/rules_cmd.go`、`config_cmd.go` 确认配置入口
**章节大纲**：
1. 配置体系总览（配置文件位置与层级、优先级：flags > env > config file，需从 resolver 与 config_cmd 源码确认）
2. 规则数据库（system_rules.json 结构：字段含义、system_rules.go 的加载与校验）
3. 语言嗅探器（sniffer.go：文件类型判定算法 —— 扩展名表 / 文件名模式 / 内容嗅探；45+ 规则文档如何按类型挂接）
4. rule_docs 体系（规则文档模板结构、变量占位符、按语言枚举表）
5. GitHub 仓库级规则（.opencodereview/rule.json 的解析与合并逻辑，从 resolve_github 相关测试反推实现位置）
6. Prompt 模板引擎（template.go：模板变量与组装流程、effort.go 的 effort 分级如何影响 token）
7. 六组 prompt 对详解（main / plan / grouping / re_location / review_filter / memory_compression，各自 system+user 的职责、注入时机、下游文档交叉链接）
8. task_template.json 与 scan_template.json 结构逐字段说明
9. allowlist（可审文件类型判定、默认排除模式、doublestar 模式语法）
10. toolsconfig（每工具的启用/禁用配置）
11. testconnection（连通性测试的实现）
12. 覆盖清单表

**必备图**：文件→嗅探→规则匹配→prompt 组装 flowchart；配置解析优先级 flowchart；六组 prompt 与流水线阶段的对应关系图
**验收**：覆盖清单含 internal/config 全部非测试 .go 与 .json 资产；prompt 一节与 Task 7（agent loop）引用一致
**提交**：`docs(analysis): add config and rules system analysis`

#### Task 5：05-llm-providers.md

**产出**：`docs/analysis/05-llm-providers.md`
**阅读范围**：`internal/llm/` 全部非测试 .go（client.go、protocol.go、providers.go、raw.go、resolver.go、responses_client.go、retry_boundary.go、retry_meta.go、retry_observer.go、retry_report.go、keycmd.go、sessionkey.go、usage_resolver.go、embedded_loader.go）；抽样 `bpe_data/` 用途说明；参考 `cmd/opencodereview/llm_cmd.go`、`provider_cmd.go`、`bedrock_config_test.go` 佐证
**章节大纲**：
1. Provider 抽象与注册表（providers.go：内置 provider 清单、自定义 provider 规则、兼容性分级）
2. 统一协议层（protocol.go：消息/工具调用格式如何同时适配 OpenAI 与 Anthropic）
3. 客户端实现（client.go 与 responses_client.go 的差异：chat completions vs Responses API）
4. Provider 解析链（resolver.go：config 文件、env、shellrc 的合并顺序与规则）
5. 密钥管理（keycmd.go 从命令取 key 的安全设计、sessionkey.go、unix/windows 差异）
6. 重试体系四件套（boundary 判定哪些错误可重试；meta 重试元数据与 identity；observer 观察上报；report 重试报告渲染）
7. Token 计量（embedded_loader.go 内嵌 cl100k_base.tiktoken、usage_resolver.go）
8. raw.go 原始调试模式
9. Bedrock 特殊路径（AWS 凭证与签名）
10. 覆盖清单表

**必备图**：provider 解析决策树 flowchart；请求→重试→上报 sequenceDiagram；重试状态机 stateDiagram-v2
**验收**：覆盖清单与 Glob 结果一致；内置 provider 列表与 providers.go 代码一致
**提交**：`docs(analysis): add LLM provider layer analysis`

#### Task 6：06-agent-loop.md

**产出**：`docs/analysis/06-agent-loop.md`
**前置阅读**：04、05 两篇
**阅读范围**：`internal/agent/` 全部非测试 .go（agent.go、grouping.go、estimate.go、identity.go、preview.go、util.go）、`internal/llmloop/` 全部非测试 .go（loop.go、pool.go、compression.go）
**章节大纲**：
1. Agent 核心结构与生命周期（agent.go：字段、构造、sealed input 的含义与作用）
2. 运行身份与幂等（identity.go：run identity、manifest hash 如何支撑 resume 幂等）
3. 文件分组算法（grouping.go：bundle 规则、语言相关性、与 grouping_task prompt 的协作边界 —— 哪些是工程决策、哪些交给 LLM）
4. Token 预算与估算（estimate.go：预算分配算法、budget 超限行为）
5. 预览模式（preview.go）
6. llmloop 主循环走读（loop.go：消息历史管理、tool call 分发、循环终止条件、与 tool 包的接口）
7. 并发池（pool.go：worker 模型、并发上限控制、错误传播与取消）
8. 上下文压缩（compression.go：memory_compression prompt 触发时机、压缩后状态）
9. 覆盖清单表（agent + llmloop 两个包）

**必备图**：agent 生命周期 stateDiagram-v2；主循环 sequenceDiagram（agent↔llmloop↔tool↔LLM）；pool 并发模型 flowchart
**验收**：覆盖清单含两个包全部非测试文件；分组一节明确「工程 vs LLM」职责边界表
**提交**：`docs(analysis): add agent and loop analysis`

#### Task 7：07-tool-system.md

**产出**：`docs/analysis/07-tool-system.md`
**前置阅读**：06
**阅读范围**：`internal/tool/` 全部非测试 .go（definitions.go、code_comment.go、code_search.go、file_read.go、file_read_diff.go、file_find.go、filereader.go、comment_collector.go、comment_args_repair.go、response_message.go、stub.go）；对照 `internal/config/toolsconfig` 与 prompts 中工具描述
**章节大纲**：
1. 工具注册与 JSON Schema（definitions.go：每个工具的参数 schema 设计）
2. file_read（整文件读取策略、filereader.go 的截断/分页机制）
3. file_read_diff（diff 范围读取）
4. code_search（搜索实现：底层用 git grep 还是正则，从源码确认）
5. file_find（文件定位：glob 语义）
6. code_comment（评论提交工具、comment_args_repair.go 的参数修复重试机制 —— LLM 给错参数时如何自愈）
7. 评论收集器（comment_collector.go：去重、排序、聚合）
8. response_message.go（模型输出中结构化评论的解析）
9. stub.go 测试替身的设计
10. 工具集设计意图（对照 README「Scenario-tuned toolset」主张，逐工具说明为什么纳入/裁剪）
11. 覆盖清单表

**必备图**：工具调用 sequenceDiagram（LLM→loop→tool→结果回注）；comment 提交与参数修复流程图
**验收**：覆盖清单一致；每个工具给出 schema 字段表
**提交**：`docs(analysis): add tool system analysis`

### 阶段 3 — 业务流水线

#### Task 8：08-review-pipeline.md

**产出**：`docs/analysis/08-review-pipeline.md`
**前置阅读**：03、04、06、07
**阅读范围**：`cmd/opencodereview/review_cmd.go`（精读）、`output.go`、`sarif.go`、`internal/model/review.go`、`internal/suggestdiff/diff.go`、`internal/stdout/stdout.go`；输出消费方：`action.yml`、`scripts/github-actions/post-review-comments.js`、`examples/github_actions/ocr-review.yml`
**章节大纲**：
1. 端到端流程总览（`ocr review` 从回车到结果落盘的完整阶段列表）
2. 参数与运行模式（--staged、--audience、--background、输出格式 flag、progress stream、resume 入口）
3. 阶段拆解（每阶段一节：diff 获取 → 过滤 → 分组 → 并发子代理 → 评论收集 → re-location 定位 → review filter 反思 → 最终评论；每节引用对应模块文档编号）
4. 输出系统（stdout 渲染与颜色、SARIF 导出与 GitHub code scanning 对接、output manifest、retry report 呈现）
5. suggestdiff（建议性 diff 的生成与格式）
6. 与 session 持久化的交互点（何时写 manifest）
7. GitHub Action 集成链路（action.yml 输入 → ocr → post-review-comments.js 发 PR 评论）
8. 失败与恢复路径汇总表（LLM 失败重试、中断 resume、orphan request）
9. 覆盖清单表（本篇涉及的文件）

**必备图**：端到端 sequenceDiagram（角色：用户/git/diff/rules/llmloop/tools/session/输出）；输出格式分支 flowchart；CI 集成链路图
**验收**：流程每个阶段都有代码行号链接；≥ 3 张图
**提交**：`docs(analysis): add review pipeline analysis`

#### Task 9：09-scan-pipeline.md

**产出**：`docs/analysis/09-scan-pipeline.md`
**前置阅读**：08
**阅读范围**：`internal/scan/` 全部非测试 .go（agent.go、batch.go、estimate.go、provider.go、preview.go）、`internal/model/scan.go`、`cmd/opencodereview/scan_cmd.go`
**章节大纲**：
1. scan 与 review 的定位差异（全文件审计 vs diff 审查）
2. 文件发现与过滤
3. 批处理算法（batch.go：分批规则、budget_exceeded 处理、去重 dedup）
4. Token 预算（estimate.go 与 agent.estimate 的关系）
5. provider.go 的 LLM 组装
6. preview 与 resume（scan_resume 语义）
7. 覆盖清单表

**必备图**：scan 流水线 flowchart；batch 切分示意
**验收**：覆盖清单一致；与 08 篇的公共机制交叉引用正确
**提交**：`docs(analysis): add scan pipeline analysis`

#### Task 10：10-session-persistence.md

**产出**：`docs/analysis/10-session-persistence.md`
**阅读范围**：`internal/session/` 全部非测试 .go（manifest.go、persist.go、resume.go、resume_identity.go、history.go、list.go、comments.go、compare.go、raw_writer.go、testing.go）、`cmd/opencodereview/session_cmd.go`
**章节大纲**：
1. 会话存储布局（目录结构、命名规则，从源码确认实际路径）
2. manifest 数据结构逐字段说明（manifest.go；guards 防护逻辑）
3. 持久化写入（persist.go、raw_writer.go：原始请求/响应落盘、写入时机）
4. 断点续跑（resume.go：可续判定条件；resume_identity.go：身份校验防止错配；orphan request 处理）
5. 历史与列表（history.go、list.go、错误处理）
6. 会话比较（compare.go：两次 run 的 diff 语义）
7. comments.go 与 viewer 的数据接口
8. testing.go 测试基建
9. 覆盖清单表

**必备图**：session 生命周期 stateDiagram-v2；resume 判定 flowchart
**验收**：覆盖清单一致；manifest 字段表与结构体一一对应
**提交**：`docs(analysis): add session persistence analysis`

#### Task 11：11-delegate-mcp.md

**产出**：`docs/analysis/11-delegate-mcp.md`
**阅读范围**：`internal/delegate/`（format.go、rulegroup.go）、`internal/mcp/`（client.go、provider.go）、`cmd/opencodereview/delegate_cmd.go`、`plugins/open-code-review/skills/open-code-review-delegate/SKILL.md`、`plugins/open-code-review/claude-code/commands/delegate-review.md`、README.md 的 Delegation Mode 章节
**章节大纲**：
1. 委托模式动机（父 agent 调用 ocr 而非自己审查的理由，对照 README）
2. delegate_cmd.go 走读（--audience agent 的输出适配差异）
3. format.go（面向 agent 的输出格式规范）
4. rulegroup.go（委托场景下的规则组选择）
5. MCP 集成方向确认（provider.go/client.go：ocr 作为 MCP server 暴露能力，还是作为 client 消费外部 MCP —— 以源码为准并画图）
6. 与 IDE agent 插件的对接链路（SKILL.md / commands 的触发方式）
7. 覆盖清单表

**必备图**：delegation 时序图（父 agent → ocr delegate → 结构化报告回流）；MCP 架构图
**验收**：覆盖清单一致；MCP 方向结论有源码佐证
**提交**：`docs(analysis): add delegate and MCP analysis`

#### Task 12：12-viewer-server.md

**产出**：`docs/analysis/12-viewer-server.md`
**阅读范围**：`internal/viewer/` 全部非测试 .go（server.go、handler.go、store.go、browser.go、hostguard.go、securityheaders.go）+ `templates/`（repos.html、sessions.html、session.html）与 `static/`（repos.js、session.js、style.css）走读、`cmd/opencodereview/viewer_cmd.go`
**章节大纲**：
1. 服务器架构（路由表、模板渲染、静态资源嵌入方式）
2. store.go（会话数据读取接口，与 session 包的关系）
3. 安全设计（hostguard.go 的 host 校验目标、securityheaders.go 的响应头清单及各自防御的威胁）
4. browser.go（跨平台打开浏览器的实现与 browser_exec 测试佐证）
5. 前端页面（三个模板的职责、repos.js/session.js 的交互逻辑）
6. 覆盖清单表

**必备图**：HTTP 请求处理 flowchart；安全机制示意
**验收**：覆盖清单一致；路由表与 handler.go 一致；安全头清单完整
**提交**：`docs(analysis): add viewer server analysis`

#### Task 13：13-telemetry.md

**产出**：`docs/analysis/13-telemetry.md`
**阅读范围**：`internal/telemetry/` 全部非测试 .go（config.go、events.go、exporter.go、metrics.go、provider.go、shutdown.go、span.go）
**章节大纲**：
1. 可观测性设计总览（OTel metrics + traces 的接入面）
2. 遥测配置（config.go：开关、默认值、隐私边界）
3. exporter 体系（otlp grpc/http、stdout 的选择逻辑）
4. 指标清单表（metrics.go：名称/类型/标签/触发点逐条列出）
5. span.go 追踪范围（哪些关键链路有 trace）
6. events.go 事件模型
7. shutdown.go 优雅关闭与 flush
8. 覆盖清单表

**必备图**：遥测数据流图
**验收**：指标清单与 metrics.go 完全一致
**提交**：`docs(analysis): add telemetry analysis`

### 阶段 4 — 外围生态

#### Task 14：14-vscode-extension.md

**产出**：`docs/analysis/14-vscode-extension.md`
**阅读范围**：`extensions/vscode/package.json`（贡献点）、`src/extension/extension.ts`、`commands.ts`、`services/` 全部非测试 .ts（CliService.ts、GitService.ts、ConfigService.ts、ReviewSession.ts、cliParse.ts、configParse.ts、gitMap.ts、shellEnv.ts、configDraft.ts）、`providers/`（CommentProvider.ts、ConfigPanelProvider.ts、SidebarProvider.ts、commentAnchor.ts、lineOffset.ts）、`webview/`（index.tsx、App.tsx、store.ts、bridge.ts、I18nProvider.tsx、configStore.ts、views/ 全部 7 个、components/ 全部 7 个）、`shared/`（types.ts、messages.ts、providers.ts、constants.ts、i18n.ts、configUtils.ts）、`webpack.config.js`、`jest.config.js`
**章节大纲**：
1. 扩展架构总览（三方边界：extension host / webview UI / ocr CLI 子进程）+ 激活流程与贡献点清单
2. CliService（子进程 spawn、流式输出、取消机制）与 cliParse（输出流解析状态机）
3. GitService / gitMap（仓库检测与分支映射）
4. ConfigService / configParse / configDraft（ocr 配置读写与草稿）
5. shellEnv（shell 环境继承）
6. ReviewSession（运行状态机：idle→running→done/failed/cancelled）
7. SidebarProvider 与 webview React 应用（状态管理库以 store.ts 实际 import 为准；bridge.ts 消息协议 —— 列出 shared/messages.ts 的消息类型全表）
8. 视图组件（7 个 view 按状态切换的矩阵表）
9. CommentProvider 与行号对齐（commentAnchor / lineOffset 的算法 —— 编辑器行与 diff 行对齐，扩展的难点）
10. ConfigPanelProvider 与 CustomProviderManager
11. i18n（package.nls 双语）与构建（webpack 多 target）、测试（jest + vscode mock）
12. 覆盖清单表（全部非测试 .ts/.tsx）

**必备图**：扩展三方架构图 flowchart（含消息流）；CLI 输出→UI 状态 sequenceDiagram；行号对齐示意
**验收**：消息类型表覆盖 messages.ts 全部导出；覆盖清单一致
**提交**：`docs(analysis): add vscode extension analysis`

#### Task 15：15-distribution-release.md

**产出**：`docs/analysis/15-distribution-release.md`
**阅读范围**：根 `package.json`、`bin/ocr.js`、`npm/<六平台>/package.json`、`scripts/install.js`、`scripts/platform.js`、`scripts/update.js`、`scripts/version.js`、`install.sh`、`install.ps1`、`action.yml`、`.github/workflows/` 全部 10 个 yml、`scripts/publish/`（_common.sh、publish.sh、publish-platform.sh、.env.example）、Makefile 的 dist/build-all/sha256sum 目标、`examples/` 六个 CI 目录的 README 与配置
**章节大纲**：
1. 分发渠道矩阵（npm / GitHub Release / install 脚本，各渠道适用场景）
2. npm 包装机制（bin/ocr.js 的平台二进制选择、六平台 optionalDependencies、install/update 生命周期钩子）
3. 安装脚本（install.sh 与 install.ps1：平台检测、下载、sha256 校验）
4. GitHub Action（action.yml 输入输出契约、post-review-comments.js 的 PR 评论逻辑、action-contract 校验的保障）
5. CI 工作流逐个走读（ci、release、codeql、deploy-pages、pages-ci、ocr-review 自我审查、translation-sync、vscode-ext、plugin-contract、action-contract：触发条件、步骤、产物）
6. 发布流水线（tag → build-all → sha256 → npm/GitHub Release 的完整链路）
7. 第三方 CI 集成示例（gitlab/bitbucket/gerrit/codeup/gitflic 各自的 post_review.py 思路简述）
8. 覆盖清单表

**必备图**：安装决策 flowchart；发布流水线 flowchart；workflow 依赖关系图
**验收**：10 个 workflow 全部出现在工作流表格中
**提交**：`docs(analysis): add distribution and release analysis`

#### Task 16：16-testing-quality.md

**产出**：`docs/analysis/16-testing-quality.md`
**阅读范围**：`Makefile`、`scripts/verify-english-only.go`、`scripts/verify-license.sh`、`scripts/add-license.sh`、`scripts/verify-action-pins.sh`、`ASSURANCE_CASE.md`、`SECURITY.md`、`.github/workflows/ci.yml`、`.gitattributes`；抽样精读 5 个代表性测试：`internal/diff/parser_test.go`、`cmd/opencodereview/progress_stream_e2e_test.go`、`cmd/opencodereview/retry_fake_llm_test.go`、`internal/llmloop/loop_execute_test.go`、`internal/session/testing.go`
**章节大纲**：
1. 测试策略与分层（单测 / 集成 / e2e / fake LLM 的四层组织；test_home 隔离模式 —— 多包出现的 test_home_test.go 背后的 HOME 重定向手法）
2. 覆盖率工程（90% 门槛的 make coverage 实现、packages 过滤规则及 pages/extensions 排除原因）
3. 质量门禁全景（license-check、english-check 的扫描规则与豁免机制、action pins、translation sync、plugin/action contract —— 每个门禁：命令 + 触发点 + 失败行为）
4. ASSURANCE_CASE 安全论证要点与 SECURITY.md 政策
5. CI 全景（ci.yml 各 job 的矩阵与依赖）
6. 抽样测试走读（5 个样本各一小节：测试意图与技巧）
7. 覆盖清单表

**必备图**：质量门禁流水线 flowchart
**验收**：门禁表 ≥ 6 项且每项有命令与代码位置
**提交**：`docs(analysis): add testing and quality analysis`

#### Task 17：17-ecosystem.md

**产出**：`docs/analysis/17-ecosystem.md`
**阅读范围**：`pages/package.json`（确认站点框架）与 `pages/` 结构（src/components、src/content/docs 五语言目录、src/i18n、src/hooks、src/pages、blog 内容）、`scripts/github-actions/check-translation-sync.js`、`plugins/open-code-review/`（README.md、claude-code/、.codex-plugin/、.cursor-plugin/、opencode/open-code-review.ts、qca/、skills/）、`skills/` 两个 SKILL.md、`.claude/commands/`、`.claude-plugin/marketplace.json`、`.agents/plugins/marketplace.json`
**章节大纲**：
1. 文档站架构（框架与构建链、路由结构、五语言 i18n 机制与 translation-sync 同步校验、部署流程 deploy-pages）
2. 插件体系全景（Claude Code plugin 的 commands/skills/marketplace 结构、Codex 与 Cursor 插件清单文件、opencode TS 插件的实现、qca 系统提示模板的用途）
3. Skills 详解（两个 SKILL.md：触发条件、给外部 agent 的指令内容）
4. 生态位地图（CLI 核心 → IDE 扩展 → agent 生态 → CI，四层如何互相导流）
5. 覆盖清单表

**必备图**：生态全景 flowchart（四层）
**验收**：每个插件入口文件（plugin.json、SKILL.md、commands）都有说明；翻译同步机制有 check-translation-sync.js 佐证
**提交**：`docs(analysis): add ecosystem analysis`

### 阶段 5 — 综合与收尾

#### Task 18：01-architecture.md（综合架构篇）

**产出**：`docs/analysis/01-architecture.md`
**前置阅读**：00–17 全部文档（本文是收敛）
**阅读范围**：以已完成的 17 篇文档为输入，必要时回查源码定夺矛盾
**章节大纲**：
1. 设计哲学（「确定性工程 × Agent 混合」：对照 README 的主张逐条用代码佐证 —— 文件选择/分组/规则匹配是工程，动态上下文是 agent）
2. 五层架构总图（flowchart TB：CLI 接入层 → 命令编排层 → 业务流水线层 → 核心引擎层 → 基础设施层，标注每层包含的包）
3. 端到端 Review 主链路大图（sequenceDiagram 全角色：用户→git→diff→rules→llmloop→tools→session→输出）
4. 内部包依赖图（flowchart：基于实际 import 关系画出 internal 各包与 cmd 的依赖网络）
5. 并发与预算模型（pool、grouping、estimate、budget 的协作图）
6. 核心数据模型（classDiagram：model 包 review/scan/diff/preview 实体关系）
7. 横切关注点（重试、遥测、平台兼容、安全、国际化，各一小节并链接对应文档）
8. 关键设计决策表（决策 / 理由 / 代码位置 / 被放弃的替代方案 —— 至少 10 条，如：为何自建行号解析、为何精简工具集、为何用 manifest 支撑 resume、为何 provider TUI 自研）
9. 扩展点指南（新增 provider / 规则 / 工具 / 命令 / CI 平台各自的改动面清单）
10. 覆盖清单表（引用各模块文档的清单，验证总数）

**必备图**：≥ 6 张全局图（上述大纲中已指定）
**验收**：与各模块文档零矛盾（发现矛盾以源码为准并回改对应模块文档）；每条设计决策有代码链接
**提交**：`docs(analysis): add architecture synthesis`

#### Task 19：README 索引 + 交叉校验

**产出**：`docs/analysis/README.md`
**步骤**：
- [ ] 创建索引：文档地图表（编号/标题/一句话摘要/关键图表）、三条阅读路线、统一术语表（汇总各篇术语并统一叫法，发现不一致回改各篇）、图索引（全部 mermaid 图的清单）
- [ ] 链接完整性检查（RunCommand 执行，预期输出为空，无 BROKEN）：

  ```bash
  cd /workspace/docs/analysis && grep -rhoP '\]\(\K\.\./[^)#]+' . | sort -u | while IFS= read -r p; do test -e "$p" || echo "BROKEN: $p"; done
  ```

- [ ] 覆盖完整性抽查：随机抽 5 个模块，用 Glob 列出非测试源文件，比对覆盖清单行数
- [ ] 术语一致性抽查：同一概念（如 bundle/grouping、评论/comment）在不同文档中的叫法统一
- [ ] 修正所有发现的问题
- [ ] 向用户汇报：文档清单 + 每篇一句话摘要 + 主要架构图所在位置

**验收**：链接检查 0 BROKEN；19 个文件齐全；术语表 ≥ 20 条
**提交**：`docs(analysis): add index and cross-check fixes`

## 5. 全局完成定义（DoD）

1. `docs/analysis/` 下 19 个文件全部存在（README + 00–17）
2. 每个模块文档的「覆盖清单」与该模块实际非测试源文件一一对应（可复核）
3. 链接检查命令输出 0 BROKEN
4. 全部文档合计 ≥ 30 张 mermaid 图（01 篇 ≥ 6 张）
5. 关键论断均有源码行号链接或测试佐证，无「待与维护者确认」之外的臆测内容
6. 术语以 README 索引表为准全局一致
7. 若批准提交：每任务一次英文提交，遵循 AGENTS.md 提交前自审协议（沙箱无 LLM 时记录并跳过）

## 6. 风险与应对

| 风险 | 应对 |
|---|---|
| R1 单模块过大导致文档超长 | 文档内部用目录锚点分章，不拆文件（保持编号稳定）；llm/agent/cmd 三个最大的包允许 1000+ 行 |
| R2 沙箱无 LLM，ocr 自审不可用 | 跳过自审并记录原因，正常提交 |
| R3 行号链接随代码演进漂移 | 全部用范围锚点；Task 19 终检时抽查修正 |
| R4 分析结论与官方文档冲突 | 以源码为准，文中显式标注差异 |
| R5 mermaid 渲染失败 | 限定五类主流语法、每图 ≤ 20 节点；Task 19 逐图复查 |
| R6 分析期间仓库被改动 | 每任务开始时以当前 HEAD 为准；若变更则重新核对受影响章节 |

## 7. 提交策略

默认**不自动提交**。是否执行各任务的 Step 5，以用户对本计划的批复为准（执行方式确认时一并询问）。提交时遵循：英文提交信息、先自审（`ocr review --auditor agent` 协议见 AGENTS.md）、LF 行尾（.gitattributes 已配置，必要时 `git add --renormalize .`）。

## 8. 计划自查记录

- **Spec 覆盖**：仓库每个顶层目录均有归属文档 —— cmd→02、internal 19 包→03–13、extensions→14、分发/CI/examples→15、质量→16、pages/plugins/skills→17、全局→00/01。✓
- **占位符扫描**：无 TBD/TODO/「稍后补充」类占位；每任务给出完整章节大纲与可执行验收命令。✓
- **命名一致性**：文档编号在任务、DoD、提交信息中一致；「覆盖清单」「标准任务流程」术语全局统一。✓
