# IDEA MCP 编辑与验证工具契约

本文件列出创建文件、格式化、语义重命名、IDEA inspection 和构建子工具的允许清单、已确认参数及选择规则，不产生编辑或执行授权。

## 工具契约

| 子工具               | 用途                                             | 必填参数                                                                                                                        | 可选参数与条件                                                                                                                                                                                                                      |
|----------------------|--------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `build_project`      | 编译指定文件，或增量、完整构建项目并返回构建问题 | 无                                                                                                                              | `filesToRebuild:string[]`：项目相对文件路径；`rebuild:boolean`：`false` 表示增量构建且为默认值，`true` 表示完整重建，仅在未传 `filesToRebuild` 时有效；`filesToRebuild` 模式不得与全项目 `rebuild` 模式混用；`timeout:number`：毫秒 |
| `lint_files`         | 使用 IDEA inspection 批量检查指定文件            | `files:string[]`：项目内的项目相对文件路径，规范化后的重复路径会被忽略                                                          | `min_severity:string`：只能为 `warning`、`error`，默认 `warning`；`timeout:number`：毫秒                                                                                                                                            |
| `create_new_file`    | 在项目内创建文件并可写入初始内容                 | `pathInProject:string`：项目相对路径                                                                                            | `overwrite:boolean`：默认不覆盖；`false` 时遇到同名文件报错，只有已授权覆盖目标时才传 `true`；`text:string`：省略时创建空文件                                                                                                       |
| `reformat_file`      | 按 IDEA 规则格式化指定文件                       | `files:string[]`：项目相对文件路径，规范化后的重复路径会被忽略                                                                  | 无                                                                                                                                                                                                                                  |
| `rename_refactoring` | 通过 IDEA 语义重构重命名符号及其引用             | `pathInProject:string`：项目相对路径；`symbolName:string`：现有符号的精确、区分大小写名称；`newName:string`：新的区分大小写名称 | 无                                                                                                                                                                                                                                  |

## 选择与使用

- `lint_files` 用于明确文件的 IDEA inspection；检查每项 `notAnalyzedReason`、`timedOut` 和顶层 `more`。warning
  不表示调用失败，但也不等于检查通过。
- `build_project` 用于编译或构建。即使传入 `filesToRebuild`，它仍属于构建阶段，按 `非文件级执行与验证.md` 判断授权；检查
  `isSuccess`、`problems` 和 `timedOut`。
- 全项目增量构建使用 `rebuild=false`，清理式重建使用 `rebuild=true`；不得与 `filesToRebuild` 模式混用。
- `create_new_file` 使用项目相对路径，默认不覆盖。只有目标文件创建和可能发生的覆盖都在授权范围内时才执行。
- `reformat_file` 的 `files` 必须是 JSON 数组，只传入当前任务明确文件。
- `rename_refactoring` 使用精确、区分大小写的符号名；执行前确认引用影响范围，不用文本替换模拟语义重命名。
- 参数形式只表示工具契约，不代表操作已经获得授权。
