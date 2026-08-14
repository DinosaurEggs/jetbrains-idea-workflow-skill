# IDEA MCP 上下文工具契约

本文件列出搜索、读取、符号、目录、编辑器状态和 VCS 子工具的允许清单、已确认参数及选择规则，不决定任务授权。统一入口、外层
`projectPath`、命令格式、结果判定和重试边界服从 `SKILL.md`。

## 工具契约

| 子工具                    | 用途                                 | 必填参数                                                          | 可选参数与条件                                                                                                       |
|---------------------------|--------------------------------------|-------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------|
| `analyze_calls`           | 查看符号的调用方、被调用方及调用层级 | `symbolFqn:string`、`analysisKind:INCOMING_CALLS\|OUTGOING_CALLS` | `childOffset:number`、`depth:number`、`maxChildren:number`、`maxNodes:number`、`timeout:number`、`treePath:string[]` |
| `get_symbol_info`         | 获取指定位置符号的快速文档与定义信息 | `filePath:string`、`line:number`、`column:number`                 | 无                                                                                                                   |
| `get_all_open_file_paths` | 读取当前活动和已打开编辑器的文件路径 | 无                                                                | 无                                                                                                                   |
| `list_directory_tree`     | 查看指定项目目录的树形结构           | `directoryPath:string`                                            | `maxDepth:number`、`timeout:number`                                                                                  |
| `open_file_in_editor`     | 在 IDEA 编辑器中打开指定文件         | `filePath:string`                                                 | 无                                                                                                                   |
| `read_file`               | 读取项目、依赖或反编译源码内容       | `file_path:string`                                                | `offset:number`、`limit:number`                                                                                      |
| `search_file`             | 按 glob 查找项目文件路径             | `q:string`                                                        | `includeExcluded:boolean`、`limit:number`、`paths:string[]`                                                          |
| `search_regex`            | 按正则查找项目文件中的匹配位置       | `q:string`                                                        | `limit:number`、`paths:string[]`                                                                                     |
| `search_symbol`           | 按标识符片段查找类、方法或字段       | `q:string`                                                        | `include_external:boolean`、`limit:number`、`paths:string[]`                                                         |
| `search_text`             | 按固定文本查找项目文件中的匹配位置   | `q:string`                                                        | `limit:number`、`paths:string[]`                                                                                     |
| `get_repositories`        | 列出项目中的 VCS 仓库根              | 无                                                                | 无                                                                                                                   |
| `git_status`              | 读取一个或多个仓库的 Git 工作区状态  | 无                                                                | `includeIgnored:boolean`、`includeUntracked:boolean`、`limit:number`、`repositoryPathRelativeToProject:string`       |

## 参数使用边界

- 表格只记录当前已确认的名称、参数和条件，不替代统一入口的当前工具说明。
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
