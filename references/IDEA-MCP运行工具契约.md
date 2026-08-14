# IDEA MCP 运行工具契约

本文件列出运行配置和运行任务子工具的允许清单、已确认参数和组合。运行授权、持续进程、验证范围和结果判定由 `非文件级执行与验证.md`
负责。

## 工具契约

| 子工具                      | 用途                                   | 必填参数                                                                  | 可选参数与条件                                                                                                                 |
|-----------------------------|----------------------------------------|---------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------|
| `get_run_configurations`    | 列出项目运行配置或指定文件的可运行位置 | 无                                                                        | `filePath:string`                                                                                                              |
| `execute_run_configuration` | 按配置名或代码位置启动运行任务         | `configurationName:string`，或同时使用 `filePath:string` 与 `line:number` | `envs:object`、`programArguments:string`、`timeout:number`、`waitForExit:boolean`、`workingDirectory:string`；两种启动模式互斥 |

## 选择与使用

- 启动任务前使用 `get_run_configurations` 确认已有配置、可运行位置、动态覆盖支持和工作目录。
- 优先使用已确认的 `configurationName`。按代码位置启动时，`filePath` 和 `line` 必须来自当前工具返回的可运行位置；不得猜测行号基准。
- `configurationName` 模式与 `filePath`、`line` 模式不得混用。
- 只有所选配置明确支持且授权范围需要时，才传入临时 `envs`、`programArguments` 或 `workingDirectory`。
- `timeout` 单位未由当前工具说明明确时，不传入该可选参数。
- 只有已经存在可复用的会话、任务标识、PID、日志、状态查询或停止机制，并且该机制能够在当前授权范围内使用时，才设置
  `waitForExit=false`；否则使用 `waitForExit=true`。
- `waitForExit=true` 超时后，进程状态和退出码可能未知。复用返回的会话、任务、PID 或日志补齐状态；没有可用标识时，将结果标记为状态未知，不直接重跑。
- 临时覆盖不得写回持久化运行配置。修改持久化配置按 `高风险确认.md` 独立确认。
- 外层调用成功只表示统一入口已响应；必须继续检查实际任务状态、输出、超时和退出信息。
