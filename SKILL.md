---
name: jetbrains-idea-workflow
description: '在 JetBrains IDEA 当前项目中使用 IDEA MCP 完成源码理解、需求与系统设计、功能开发、Bug 修复、重构、前后端、数据库和界面变更及检查验证。用于任务依赖 IDEA 项目源码、符号、inspection、运行配置或数据库语义，并需遵循项目规范形成工程闭环。'
---

# JetBrains IDEA 开发工作流

按真实开发流程完成分析、设计、实现、检查和交付。输出、说明、确认、设计及总结均使用中文。

## 入口

项目开发任务先完整读取 [core-workflow.md](references/core-workflow.md)。首次调用 IDEA
MCP，或访问项目源码、符号、依赖、工作区、编辑、inspection、运行配置、数据库或调试能力前，读取并应用 [idea-mcp-gate.md](references/idea-mcp-gate.md)
；直接以首个任务相关的 IDEA MCP 调用完成状态门禁，不另做 bootstrap 预检。首次通过 `mcp__idea__execute_tool` 构造子工具
`command` 前完整读取 [idea-mcp-execute-tool.md](references/idea-mcp-execute-tool.md)。

构造 `command` 时只使用 [idea-mcp-execute-tool.md](references/idea-mcp-execute-tool.md)
明确列出的子工具和参数契约。逐字复制工具名及参数名，不翻译、不缩写、不把 camelCase、snake_case
或单复数形式相互转换，也不依据相邻工具推断参数。每次调用前核对必填项、类型、数组或对象格式、条件组合和外层 `projectPath`
；参数错误只按已读契约或返回中明确指出的缺失项、未知项及类型修正，不试探其他拼写或清单外工具。

仅维护本 skill、处理普通文档或执行不依赖 IDEA 项目语义的元任务时可使用普通文件系统工具；不得用此例外绕过项目开发任务的 IDEA 门禁。

## 执行授权总则

执行任何格式化、检查、验证或运行入口前，先查明展开后的实际行为和作用范围。只有限定到本次变更明确文件的相关格式化，以及工具原生支持按这些文件执行、全程只读且无自动修复或其他写入行为的检查，属于修改任务的默认授权；其他执行性操作只有在用户当前指令唯一覆盖其实际目标和范围，或用户看过拟执行操作、目标范围和影响后明确确认时才能启动。技术上适用、项目已有入口、只读或增量、任务要求修复或完成验证，均不构成额外授权；无法证明属于默认授权时按非默认授权处理，实际范围仍无法查明时不得执行。具体判定见 [execution-validation.md](references/execution-validation.md)。

## 路由

- 源码理解、定位、调用与影响分析、Bug 排查或准备修改：读 [context-reading.md](references/context-reading.md)。
- 修改或新增项目文件：读 [edit-check-validate.md](references/edit-check-validate.md)。
- 数据库、SQL、持久化层、ORM、Mapper、DAO、Repository、字段、字典、权限数据或数据异常：读 [database-sql.md](references/database-sql.md)。
- 方案比选、方向调整与确认：读 [design-first.md](references/design-first.md)。
- 用户要求系统功能设计或系统详细设计：先应用 [design-first.md](references/design-first.md)
  ，方向明确后完整读取并严格按 [system-function-design.md](references/system-function-design.md) 的固定正文输出。
- 用户要求开发计划、开发方案、修改方案或技术实现计划：先应用 [design-first.md](references/design-first.md)
  ，方向明确后完整读取并严格按 [development-plan.md](references/development-plan.md) 的正文格式输出。
- 系统功能设计与开发计划相互独立，只完成用户当前要求的任务，不自动从一种任务进入另一种任务或代码实施。
- 前端任务：读 [frontend-workflow.md](references/frontend-workflow.md)；Vue 任务追加 [vue-workflow.md](references/vue-workflow.md)；新界面、视觉重构、样式或体验提升追加 [frontend-design-quality.md](references/frontend-design-quality.md)。
- 编译、构建、测试、启动、浏览器、截图、长任务或运行依赖问题：读 [execution-validation.md](references/execution-validation.md)。
- 删除、不可逆操作、数据库写入、依赖或环境变更、持久化 IDE 配置、部署、发布、Git commit/push 或 CI/CD：读 [high-risk-confirmation.md](references/high-risk-confirmation.md)。

## 读取复用

同一对话的后续任务默认复用当前上下文已经完整掌握的文件内容、项目结构、符号位置、调用关系和分析结论。对于读取后未修改、且所需内容仍保留在当前上下文中的文件，不得仅为确认状态而重复读取，也不得换用其他命令或工具读取相同内容。

已有项目认知足以支持当前任务时，不得重复执行目录遍历、全局搜索、架构探索或调用链扫描；后续任务直接从当前目标开始。确实缺少信息时，先明确缺少的具体证据，只补读完成任务所需的最小文件或最小范围。文件已修改、首次读取失败或不完整、上下文未保留必要正文、结果发生冲突或用户明确要求时，可以重新读取。搜索、调用分析、inspection
和差异等新证据不属于重复读取。

## 交付

- 仍有阻塞时，只说明事实、证据、影响和所需确认，不使用完成总结。
- 已完成时，说明结果、主要文件、实际验证范围及相关影响或风险。
- 超时、未分析、输出不完整、缺少退出状态或只覆盖局部时，不得扩大表述为通过。
- 只交付系统功能设计或开发计划时，直接使用对应 reference
  的正文格式，不外套通用总结；用户继续修改已输出正文时，仅按 [design-first.md](references/design-first.md)
  的“正文修订”规则追加必要的简短结语。候选选择只确认方案方向，不直接授权实施；实施前须完成当前详细正文确认并取得明确实施指令。
