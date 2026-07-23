# Vue 开发工作流

本文件只规定 Vue 页面、组件、指令、composable、store、动态 DOM 和 Vue 工具链的专项要求；通用前端契约与状态继续应用 [frontend-workflow.md](frontend-workflow.md)。

## 动态 DOM 与 Teleport 样式

处理 Vue、第三方 Vue 组件或 Teleport 动态生成的 DOM 时：

- 局部样式优先锚定模板节点上直接声明的自有静态类，状态类使用显式 `:class`。
- 能在菜单、选项、插槽或内容根节点声明类时直接声明，不让局部选择器只依赖 `popper-class` 等组件参数或第三方运行时内部类。
- 运行时生成的外层容器优先使用组件官方样式 API、CSS 变量或项目已有主题能力。
- Teleport 内容可以使用非 scoped 样式，但选择器仍须锚定模板中的显式类；不得仅为逃避检查把局部样式迁移到全局文件。
- 不得只为消除 IDEA 警告而修改 `teleported`、挂载位置、定位、触发方式、布局或业务逻辑，也不得抑制检查或删除有效样式。
- 本次新增或修改的“选择器从未使用”问题必须通过模板类与选择器关系解决，并重新执行适用的 IDEA inspection；不得直接按框架误报处理。

## 变更文件检查范围

检查范围只包含本次任务由 agent 修改或新增的文件，不包含用户已有的无关改动。先读取项目现有 `package.json`、锁文件以及 ESLint、Stylelint、Prettier 和 TypeScript 配置，查明实际包管理器、工具、脚本与参数；不得安装依赖、猜测命令别名或切换包管理器。

按项目配置支持的文件类型路由：

- ESLint：`.vue`、`.js`、`.jsx`、`.ts`、`.tsx`。
- Stylelint：`.vue`、`.css`、`.scss`、`.less`。
- Prettier：本次任务中项目配置支持的变更文件。
- `vue-tsc`：涉及 `.vue`、TypeScript、类型配置、公共类型、全局声明、公共组件或 API 类型时考虑；纯样式、静态资源或文档变更通常跳过，项目规范另有要求除外。

文件级检查使用本地既有依赖、项目配置和明确文件路径，采用只检查模式，不使用 `--fix` 或 `--write`，也不用宽泛目录或全项目 glob 代替变更文件。工具支持批量路径时一次传入适用文件；超出工具限制时分批覆盖，不遗漏文件。

## 检查与授权顺序

1. 先按 [edit-check-validate.md](edit-check-validate.md) 格式化本次变更文件并执行 IDEA inspection。
2. 再执行适用的文件级 ESLint、Stylelint 和 Prettier 只读检查。这些低风险检查无需额外确认，不得因后续 `vue-tsc` 尚未授权而延后。
3. 汇总已经完成的低风险检查结果后，再判断是否需要 `vue-tsc`。
4. `vue-tsc` 已在用户当前请求的具体验证范围内时直接执行；否则按 [execution-validation.md](execution-validation.md) 请求授权。用户未授权不影响前述低风险检查结果，但必须记录类型检查未执行。
5. `vue-tsc` 必须使用项目现有增量配置或项目级 `tsconfig`，不得传入单个文件伪造可靠的类型检查。公共类型、全局声明、公共组件或 API 契约变化时扩大影响分析和类型检查范围，但不得修改任务范围外的用户文件。

检查发现问题后是否修改代码，按 [execution-validation.md](execution-validation.md) 区分纯验证任务与已授权修复任务；修复后重新格式化并复查受影响文件。

需要通过终端运行项目现有文件级检查时，IDEA MCP 必须处于 [idea-mcp-gate.md](idea-mcp-gate.md) 规定的正常状态，且所选方式须符合项目现有工具链。IDEA MCP 连接、插件或当前项目异常时不得用终端降级。
