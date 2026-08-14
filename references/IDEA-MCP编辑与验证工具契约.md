# IDEA MCP 编辑与验证工具契约

本文件列出创建文件、格式化、语义重命名、IDEA inspection 和构建子工具的允许清单、已确认参数及选择规则，不产生编辑或执行授权。

## 工具契约

| 子工具               | 用途                                             | 必填参数                                                      | 可选参数与条件                                                                                                   |
|----------------------|--------------------------------------------------|---------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------|
| `build_project`      | 编译指定文件，或增量、完整构建项目并返回构建问题 | 无                                                            | `filesToRebuild:string[]`、`rebuild:boolean`、`timeout:number`；`filesToRebuild` 与全项目 `rebuild` 模式分开使用 |
| `lint_files`         | 使用 IDEA inspection 批量检查指定文件            | `files:string[]`                                              | `min_severity:string`、`timeout:number`                                                                          |
| `create_new_file`    | 在项目内创建文件并可写入初始内容                 | `pathInProject:string`                                        | `overwrite:boolean`、`text:string`                                                                               |
| `reformat_file`      | 按 IDEA 规则格式化指定文件                       | `files:string[]`                                              | 无                                                                                                               |
| `rename_refactoring` | 通过 IDEA 语义重构重命名符号及其引用             | `pathInProject:string`、`symbolName:string`、`newName:string` | 无                                                                                                               |

## 选择与使用

- `timeout` 的单位未由当前工具说明明确时，不传入该可选参数，使用工具默认行为。
- `min_severity` 的合法值未由当前工具说明明确时，不传入该可选参数；不得试探大小写或枚举名称。
- `lint_files` 用于明确文件的 IDEA inspection；检查每项 `notAnalyzedReason`、`timedOut` 和顶层 `more`。warning
  不表示调用失败，但也不等于检查通过。
- `build_project` 用于编译或构建。即使传入 `filesToRebuild`，它仍属于构建阶段，按 `非文件级执行与验证.md` 判断授权；检查
  `isSuccess`、`problems` 和 `timedOut`。
- 全项目增量构建使用 `rebuild=false`，清理式重建使用 `rebuild=true`；不得与 `filesToRebuild` 模式混用。
- `create_new_file` 使用项目相对路径，默认不覆盖。只有目标文件创建和可能发生的覆盖都在授权范围内时才执行。
- `reformat_file` 的 `files` 必须是 JSON 数组，只传入当前任务明确文件。
- `rename_refactoring` 使用精确、区分大小写的符号名；执行前确认引用影响范围，不用文本替换模拟语义重命名。
- 参数形式只表示工具契约，不代表操作已经获得授权。
