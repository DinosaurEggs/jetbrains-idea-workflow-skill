# IDEA MCP 上下文工具契约

本文件列出搜索、读取、符号、目录、编辑器状态和 VCS 子工具的允许清单、已确认参数及选择规则，不决定任务授权。统一入口、外层
`projectPath`、命令格式、结果判定和重试边界服从 `SKILL.md`。

## 工具契约

| 子工具                    | 用途                                 | 必填参数                                                                                                                                                                 | 可选参数与条件                                                                                                                                                                                                                                                                            |
|---------------------------|--------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `analyze_calls`           | 查看符号的调用方、被调用方及调用层级 | `symbolFqn:string`：完整 FQN，或工具歧义结果、树节点返回的精确签名；不得传文件路径、行列或独立签名参数。`analysisKind:string`：只能为 `INCOMING_CALLS`、`OUTGOING_CALLS` | `childOffset:number`：默认 0，仅用于 `treePath` 指向节点的直接子项续页；`depth:number`：默认 5，0 表示只返回子树根；`maxChildren:number`：默认 50；`maxNodes:number`：默认 1000；`timeout:number`：毫秒；`treePath:string[]`：只能原样复用上次调用返回的精确签名路径，省略表示根路径 `[]` |
| `get_symbol_info`         | 获取指定位置符号的快速文档与定义信息 | `filePath:string`：项目相对路径；`line:number`、`column:number`：均从 1 开始                                                                                             | 无                                                                                                                                                                                                                                                                                        |
| `get_all_open_file_paths` | 读取当前活动和已打开编辑器的文件路径 | 无                                                                                                                                                                       | 无                                                                                                                                                                                                                                                                                        |
| `list_directory_tree`     | 查看指定项目目录的树形结构           | `directoryPath:string`：项目相对路径                                                                                                                                     | `maxDepth:number`：最大递归深度；`timeout:number`：毫秒                                                                                                                                                                                                                                   |
| `open_file_in_editor`     | 在 IDEA 编辑器中打开指定文件         | `filePath:string`：本 skill 仅使用项目相对路径                                                                                                                           | 无                                                                                                                                                                                                                                                                                        |
| `read_file`               | 读取项目、依赖或反编译源码内容       | `file_path:string`：可为项目相对路径、含 `..` 的路径、绝对路径、归档条目，或 `file://`、`jar://`、`jrt://` URL；其他 IDEA 工具返回的路径可原样复用                       | `offset:number`：从 1 开始；`limit:number`：默认 2000，最大 5000                                                                                                                                                                                                                          |
| `search_file`             | 按 glob 查找项目文件路径             | `q:string`：项目根相对 glob；不含 `/` 时按 `**/pattern` 处理                                                                                                             | `includeExcluded:boolean`；`limit:number`：最大结果数；`paths:string[]`：项目根相对 glob，支持 `!` 排除，末尾 `/` 展开为 `**`，不含 `/` 时按 `**/pattern` 处理，空字符串忽略                                                                                                              |
| `search_regex`            | 按正则查找项目文件中的匹配位置       | `q:string`：正则表达式                                                                                                                                                   | `limit:number`：最大结果数；`paths:string[]`：项目根相对 glob，支持 `!` 排除、末尾 `/` 展开和空字符串忽略规则                                                                                                                                                                             |
| `search_symbol`           | 按标识符片段查找类、方法或字段       | `q:string`：符号标识符片段                                                                                                                                               | `include_external:boolean`：默认 `false`，只有项目符号中无合适结果且需要 SDK 或依赖符号时才设为 `true`；`limit:number`：最大结果数；`paths:string[]`：项目根相对 glob，规则同上                                                                                                           |
| `search_text`             | 按固定文本查找项目文件中的匹配位置   | `q:string`：固定文本子串，不按正则解释                                                                                                                                   | `limit:number`：最大结果数；`paths:string[]`：项目根相对 glob，规则同上                                                                                                                                                                                                                   | | 无                                                                                                                                                                                                                                                                                        |
| `git_status`              | 读取一个或多个仓库的 Git 工作区状态  | 无                                                                                                                                                                       | `includeIgnored:boolean`、`includeUntracked:boolean`；`limit:number`：每个仓库最大返回条目数；`repositoryPathRelativeToProject:string`：项目相对路径，用于选择唯一包含该路径的仓库；省略时查询全部仓库，空字符串表示项目根仓库                                                            |

## 参数使用边界

- 表格记录本 skill 静态确认的名称、参数和条件；统一入口当前工具说明的补充范围及冲突处理按 `SKILL.md` 执行。
- 可选参数的单位、合法值、路径基准或序列化规则未在本文件或当前工具说明中明确时，省略该参数。
- 必填参数仍无法唯一构造时，停止调用并报告契约缺口，不试探写法。
- 工具返回的路径、FQN 和标识符在后续调用中原样复用；只有本文件明确说明为项目相对路径时才自行转换。

## 选择与使用

- 按文件路径定位使用 `search_file`，按标识符定位使用 `search_symbol`，按固定文本使用 `search_text`，按模式匹配使用
  `search_regex`。
- 判断真实调用关系时，先使用 `search_symbol` 定位精确符号，再将完整 FQN 交给 `analyze_calls`。
- `analyze_calls` 的 `analysisKind` 只使用 `INCOMING_CALLS` 或 `OUTGOING_CALLS`；任何超时都表示分析未完成。
- `get_symbol_info` 的行和列从 1 开始，位置放在目标标识符上。
- `list_directory_tree` 使用项目相对目录和满足任务需要的最小 `maxDepth`。
- `read_file` 的参数名是 `file_path`，offset 从 1 开始，limit 最大 5000。
- `search_symbol` 使用 `include_external`，`search_file` 使用 `includeExcluded`，不得相互类推。
- `get_repositories` 用于确认多仓库根；`git_status` 用于读取 staged、unstaged、untracked 和冲突状态，不自动修改工作区。
- 正则内容按当前 `command` 的实际字符串边界转义；无法确认序列化边界时，不猜测转义层级，改用能够满足任务的固定文本查询，或报告契约不足。
