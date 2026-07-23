---
name: jetbrains-idea-workflow
description: '在 JetBrains IDEA 当前项目中使用 IDEA MCP 完成源码理解、需求与系统设计、功能开发、Bug 修复、重构、前后端、数据库和界面变更及检查验证。用于任务依赖 IDEA 项目源码、符号、inspection、运行配置或数据库语义，并需遵循项目规范形成工程闭环。'
---

# JetBrains IDEA 开发工作流

按真实开发流程完成分析、设计、实现、检查和交付。输出、说明、确认、设计及总结均使用中文。

## 入口

项目开发任务先完整读取 [core-workflow.md](references/core-workflow.md)。首次访问项目源码、符号、依赖、工作区、编辑、inspection、运行配置、数据库或调试能力前，读取并应用 [idea-mcp-gate.md](references/idea-mcp-gate.md)。

仅维护本 skill、处理普通文档或执行不依赖 IDEA 项目语义的元任务时可使用普通文件系统工具；不得用此例外绕过项目开发任务的 IDEA 门禁。

## 执行授权总则

未经用户对具体命令或明确范围的要求或确认，不得执行任何不能可靠限定到本次变更文件的全量命令，包括项目级或模块级编译、构建、测试、lint、类型检查、全局 inspection、代码审查、扫描、服务启动及其 package manager script、IDEA 运行配置或包装入口。脚本按展开后的实际行为和作用范围分类，不按名称、耗时或是否只读分类；无法查明或证明为文件级时按全量命令处理。“完成验证”“检查一下”“按开发闭环处理”或仅使用本 skill 不构成上述授权。具体边界见 [execution-validation.md](references/execution-validation.md)。

## 路由

- 源码理解、定位、调用与影响分析、Bug 排查或准备修改：读 [context-reading.md](references/context-reading.md)。
- 修改或新增项目文件：读 [edit-check-validate.md](references/edit-check-validate.md)。
- 数据库、SQL、持久化层、ORM、Mapper、DAO、Repository、字段、字典、权限数据或数据异常：读 [database-sql.md](references/database-sql.md)。
- 跨模块功能、契约、前后端联动、公共能力、核心流程或其他实质性设计：读 [design-first.md](references/design-first.md)；需要系统功能或详细设计正文时追加 [system-function-design.md](references/system-function-design.md)。
- 前端任务：读 [frontend-workflow.md](references/frontend-workflow.md)；Vue 任务追加 [vue-workflow.md](references/vue-workflow.md)；新界面、视觉重构、样式或体验提升追加 [frontend-design-quality.md](references/frontend-design-quality.md)。
- 编译、构建、测试、启动、浏览器、截图、长任务或运行依赖问题：读 [execution-validation.md](references/execution-validation.md)。
- 删除、不可逆操作、数据库写入、依赖或环境变更、持久化 IDE 配置、部署、发布、Git commit/push 或 CI/CD：读 [high-risk-confirmation.md](references/high-risk-confirmation.md)。

## 读取复用

复用上下文中同一路径、当前已知版本的完整内容，不换命令或工具重读。已有内容不完整时只补缺失范围；仅在文件已修改、首次读取失败或不完整、或上下文恢复后正文未保留时重读。搜索、调用分析、inspection 和差异等新证据不属于重复读取。

## 交付

- 仍有阻塞时，只说明事实、证据、影响和所需确认，不使用完成总结。
- 已完成时，说明结果、主要文件、实际验证范围及相关影响或风险。
- 超时、未分析、输出不完整、缺少退出状态或只覆盖局部时，不得扩大表述为通过。
- 只交付系统功能或详细设计时，直接使用固定正文格式，不外套通用总结。
