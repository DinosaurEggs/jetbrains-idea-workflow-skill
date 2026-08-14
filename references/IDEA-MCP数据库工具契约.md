# IDEA MCP 数据库工具契约

本文件列出数据库子工具的允许清单、已确认参数和调用顺序。数据库只读边界、DDL、DML 和写入确认由 `数据库SQL与持久化层.md`
负责；本文件不扩大授权。

## 工具契约

| 子工具                            | 用途                               | 必填参数                                                                                              | 可选参数与条件       |
|-----------------------------------|------------------------------------|-------------------------------------------------------------------------------------------------------|----------------------|
| `list_database_connections`       | 列出项目已配置的数据源及连接标识   | 无                                                                                                    | 无                   |
| `test_database_connection`        | 测试指定数据库连接是否可用         | `id:string`                                                                                           | 无                   |
| `list_database_schemas`           | 列出指定连接中的数据库与 schema    | `connectionId:string`                                                                                 | 无                   |
| `introspect_schema`               | 加载或刷新指定 schema 的结构元数据 | `connectionId:string`、`databaseName:string`、`schemaName:string`                                     | 无                   |
| `list_schema_object_kinds`        | 列出连接支持的 schema 对象类型     | `connectionId:string`                                                                                 | 无                   |
| `list_schema_objects`             | 列出 schema 中的表、视图等对象     | `connectionId:string`、`databaseName:string`、`schemaName:string`                                     | `kind:string`        |
| `get_database_object_description` | 读取数据库对象的列、键和索引结构   | `connectionId:string`、`databaseName:string`、`schemaName:string`、`kind:string`、`objectName:string` | 无                   |
| `preview_table_data`              | 有限预览表、视图等表类对象的数据   | `connectionId:string`、`databaseName:string`、`schemaName:string`、`tableName:string`                 | `maxRowCount:number` |
| `execute_sql_query`               | 在指定连接和 schema 中执行 SQL     | `connectionId:string`、`databaseName:string`、`schemaName:string`、`queryText:string`                 | 无                   |
| `fetch_query_result`              | 分页读取已有查询或预览结果         | `resultSetId:string`、`offset:number`                                                                 | 无                   |
| `list_recent_sql_queries`         | 查看指定连接近期及正在运行的查询   | `connectionId:string`                                                                                 | 无                   |
| `cancel_sql_query`                | 取消指定的运行中查询               | `sessionId:number`                                                                                    | 无                   |
| `create_database_connection`      | 创建并可选测试 IDEA 数据源         | `name:string`、`dbms:string`、`url:string`、`needToCheckDs:boolean`                                   | 无                   |
| `edit_database_connection`        | 修改并可选测试已有 IDEA 数据源     | `connectionId:string`、`dbms:string`、`url:string`、`needToCheckDs:boolean`                           | 无                   |

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
