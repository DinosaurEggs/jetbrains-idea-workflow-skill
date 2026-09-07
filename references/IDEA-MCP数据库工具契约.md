# IDEA MCP 数据库工具契约

本文件列出数据库子工具的允许清单、已确认参数和调用顺序。数据库只读边界、DDL、DML 和写入确认由 `数据库SQL与持久化层.md`
负责；本文件不扩大授权。

## 调用入口与命令构造

调用前先满足 [SKILL.md](../SKILL.md#idea-mcp-工具调用) 规定的数据库调用前置条件。本文件表中的名称仅定义传入统一入口 `command` 的数据库子工具标识，不构成独立 MCP 工具声明。已知任务项目时，在外层传入绝对 `projectPath`；路径转换按 `SKILL.md` 执行，子工具名称及其参数放入 `command`。

`command` 是包含子工具名称和参数的命令行字符串，不是独立工具的参数对象。只有本文件、统一入口当前工具说明，或同一工具、同一参数结构的有效既有命令能够唯一确定序列化方式时，才构造命令。

本文件规定允许调用的数据库子工具及其静态参数契约。统一入口当前工具说明可以补充契约未规定的单位、合法值、路径基准和序列化细节，但不得扩大数据库子工具清单或替代本文件已经明确的名称、类型、必填项及组合条件。两者冲突时，停止受影响的数据库调用并报告契约漂移。

可选参数的单位、合法值、路径基准或序列化规则未查明时，省略该参数；必填参数仍无法唯一构造时，报告契约缺口并停止，不试探写法。

调用前核对外层项目、数据库目标、字符串边界、数组、对象、布尔值、数字及参数组合。调用后同时检查统一入口和实际数据库子工具结果；外层调用成功不代表查询或写入成功。

## 工具契约

| 子工具                            | 用途                               | 必填参数                                                                                                                                                                                                                         | 可选参数与条件                                                                                 |
|-----------------------------------|------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| `list_database_connections`       | 列出项目已配置的数据源及连接标识   | 无                                                                                                                                                                                                                               | 无                                                                                             |
| `test_database_connection`        | 测试指定数据库连接是否可用         | `id:string`：只能原样使用 `list_database_connections` 返回的连接 ID，且目标必须为 `isDDL=false` 的真实连接                                                                                                                       | 无                                                                                             |
| `list_database_schemas`           | 列出指定连接中的数据库与 schema    | `connectionId:string`：只能原样使用已列出的连接 ID                                                                                                                                                                               | 无                                                                                             |
| `introspect_schema`               | 加载或刷新指定 schema 的结构元数据 | `connectionId:string`、`databaseName:string`、`schemaName:string`：三者原样取自当前连接和 schema 结果；仅在 `isIntrospected=false` 或需要刷新已过期元数据时调用；DBMS 没有数据库层时 `databaseName` 可为空字符串                 | 无                                                                                             |
| `list_schema_object_kinds`        | 列出连接支持的 schema 对象类型     | `connectionId:string`：只能原样使用已列出的连接 ID                                                                                                                                                                               | 无                                                                                             |
| `list_schema_objects`             | 列出 schema 中的表、视图等对象     | `connectionId:string`、`databaseName:string`、`schemaName:string`：原样取自当前 schema 结果；DBMS 没有数据库层时 `databaseName` 可为空字符串                                                                                     | `kind:string`：传入时只能使用 `list_schema_object_kinds` 返回的 `code`；省略时返回全部对象类型 |
| `get_database_object_description` | 读取数据库对象的列、键和索引结构   | `connectionId:string`、`databaseName:string`、`schemaName:string`：原样取自当前 schema 结果；`kind:string`：非空且只能使用已返回的对象类型 `code`；`objectName:string`：非空且只能使用该类型下已返回的对象名                     | 无                                                                                             |
| `preview_table_data`              | 有限预览表、视图等表类对象的数据   | `connectionId:string`：目标必须为 `isDDL=false` 的真实连接；`databaseName:string`、`schemaName:string`：原样取自当前 schema 结果；`tableName:string`：只能使用已返回的表类对象名                                                 | `maxRowCount:number`：必须大于 0，默认 10                                                      |
| `execute_sql_query`               | 在指定连接和 schema 中执行 SQL     | `connectionId:string`：目标必须为 `isDDL=false` 的真实连接；`databaseName:string`、`schemaName:string`：原样取自当前 schema 结果；`queryText:string`：精确待执行 SQL                                                             | 无                                                                                             |
| `fetch_query_result`              | 分页读取已有查询或预览结果         | `resultSetId:string`：只能原样使用 `execute_sql_query` 或 `preview_table_data` 返回的非空结果集 ID；`offset:number`：从 0 开始                                                                                                   | 无                                                                                             |
| `list_recent_sql_queries`         | 查看指定连接近期及正在运行的查询   | `connectionId:string`：目标必须为 `isDDL=false` 的真实连接                                                                                                                                                                       | 无                                                                                             |
| `cancel_sql_query`                | 取消指定的运行中查询               | `sessionId:number`：只能使用 `list_recent_sql_queries` 返回且确认属于本 agent 的运行中查询会话 ID                                                                                                                                | 无                                                                                             |
| `create_database_connection`      | 创建并可选测试 IDEA 数据源         | `name:string`：唯一连接名；`dbms:string`：只能使用当前工具说明明确支持的值，或同类既有连接的已确认值；`url:string`：完整 JDBC URL；`needToCheckDs:boolean`：`true` 表示创建后立即测试，批量配置时使用 `false` 并在需要时另行测试 | 无                                                                                             |
| `edit_database_connection`        | 修改并可选测试已有 IDEA 数据源     | `connectionId:string`：只能使用现有且 `isDDL=false` 的连接 ID；`dbms:string`：合法值来源同创建工具；`url:string`：完整 JDBC URL；`needToCheckDs:boolean`：`true` 表示修改后立即测试，批量配置时使用 `false`；连接名保持不变      | 无                                                                                             |

## 调用顺序与边界

- 按连接、schema、对象类型、对象、对象描述的顺序读取数据库结构。
- `kind` 必须使用 `list_schema_object_kinds` 或当前连接既有结果返回的值，不自行创造对象类型名称。
- `dbms` 只使用当前工具说明明确支持的值，或从同类既有连接取得的已确认值；无法取得时，不创建或编辑数据源。
- `test_database_connection` 的参数是 `id`；其他数据库连接参数使用表中列出的 `connectionId`，不得互换。
- `list_database_schemas` 返回的 `databaseName` 和 `schemaName` 在后续调用中原样使用，空数据库名使用空字符串。
- 只需读取结构时使用对象描述，不预览数据。
- 只需查看少量数据时使用 `preview_table_data` 并设置最小 `maxRowCount`。
- 需要明确 SQL 语义时使用 `execute_sql_query`；已有结果翻页使用 `fetch_query_result`，offset 从 0 开始。
- 任何非空 `errorMessage` 都表示查询失败，不根据结果集标识或部分文本推断成功。
- `list_recent_sql_queries` 只按任务需要查看本工具发起的查询和状态，不披露无关历史 SQL。
- `cancel_sql_query` 只取消确认属于本 agent 的运行中查询；取消其他查询需要独立确认。
- `create_database_connection` 和 `edit_database_connection` 会持久化 IDEA 数据源配置，执行前按 `高风险确认.md` 确认精确目标。
- 数据库示例值必须来自当前连接和 schema 结果，不猜测连接 ID、对象类型、对象名、结果集 ID 或会话 ID。
