# IDEA MCP execute_tool 命令参考

门禁通过后按需使用本文件。IDEA 子工具均通过 `mcp__idea__execute_tool` 调用，以下示例除首例外均只写 `command` 内容；统一入口、`projectPath` 和结果判定见 [idea-mcp-gate.md](idea-mcp-gate.md)。

```javascript
await tools.mcp__idea__execute_tool({
  projectPath: "F:\\Projects\\zhgd\\zhgd-v2",
  command: "search_file --q YudaoServerApplication.java --limit 5"
})
```

`command` 格式为 `工具名 --参数 值`。带空格的字符串用双引号，数组写成 JSON 数组，空字符串写作 `""`。数据库示例中的 `<CID>`、`<SCHEMA>`、`<TABLE>`、`<RESULT_ID>`、`<SESSION_ID>` 必须替换为实际值。命令授权按子工具实际行为判断：构建、运行和终端见 [execution-validation.md](execution-validation.md)，文件变更见 [edit-check-validate.md](edit-check-validate.md)，数据库与数据源见 [database-sql.md](database-sql.md) 和 [high-risk-confirmation.md](high-risk-confirmation.md)。

## 分析与检查

- `analyze_calls --symbolFqn "cn.example.App.main(String[])" --analysisKind OUTGOING_CALLS --depth 1 --maxNodes 20`：类和方法使用完整 FQN；参数类型按 IDEA 展示签名写短名称，如 `String[]`，不用 `java.lang.String[]`。`analysisKind` 为 `INCOMING_CALLS` 或 `OUTGOING_CALLS`；任何 timeout 均表示未完成。
- `build_project --filesToRebuild ["module/src/main/java/A.java"] --timeout 300000`：优先指定文件；全项目增量构建使用 `--rebuild false`，`true` 是更重的清理式重建。参数形式不代表已获授权或一定是文件级范围；检查 `isSuccess`、`problems`、`timedOut`。
- `get_file_problems --filePath module/src/main/java/A.java --errorsOnly false --timeout 30000`：只接受项目内容根内文件；warning 不表示调用失败，但不等于检查通过。
- `lint_files --files ["module/src/main/java/A.java"] --min_severity warning --timeout 30000`：`files` 必须是 JSON 数组；检查每项 `notAnalyzedReason`、`timedOut` 和顶层 `more`。
- `get_symbol_info --filePath module/src/main/java/A.java --line 19 --column 24`：行、列从 1 开始，位置放在标识符上。

## 数据库

- `list_database_connections`：读取连接 ID、DBMS、DDL 数据源和只读标记，不输出或记录凭据。
- `test_database_connection --id <CID>`：参数是 `id`，不是 `connectionId`；`hasProblems` 为 `NO` 后再查询。
- `list_database_schemas --connectionId <CID>`：后续原样使用返回的 `databaseName` 和 `schemaName`。
- `introspect_schema --connectionId <CID> --databaseName "" --schemaName <SCHEMA>`：按需加载或刷新 schema 元数据。
- `list_schema_object_kinds --connectionId <CID>`：读取可用 `kind`，如 `table`、`view`、`routine`。
- `list_schema_objects --connectionId <CID> --databaseName "" --schemaName <SCHEMA> --kind table`：可省略 `kind`，但优先限定范围。
- `get_database_object_description --connectionId <CID> --databaseName "" --schemaName <SCHEMA> --kind table --objectName <TABLE>`：读取列、主键和索引定义，不读取数据。
- `preview_table_data --connectionId <CID> --databaseName "" --schemaName <SCHEMA> --tableName <TABLE> --maxRowCount 1`：只选非敏感表并使用最小行数；返回的 `resultSetId` 可用于翻页。
- `execute_sql_query --connectionId <CID> --databaseName "" --schemaName <SCHEMA> --queryText "SELECT 1 AS mcp_smoke"`：默认仅执行有限 `SELECT`、`SHOW`、`DESCRIBE`；任何非空 `errorMessage` 都是失败。
- `fetch_query_result --resultSetId <RESULT_ID> --offset 0`：`resultSetId` 来自查询或预览，offset 从 0 开始。
- `list_recent_sql_queries --connectionId <CID>`：只按任务需要查看本工具发起的查询、`sessionId` 和状态，不披露无关历史 SQL。
- `cancel_sql_query --sessionId <SESSION_ID>`：只取消确认属于自己的运行中查询，不猜测 ID；同一数据源串行化时，取消可能排在查询完成后。
- `create_database_connection --name "MCP Smoke" --dbms MySQL --url "jdbc:mysql://host:3306/db" --needToCheckDs false`：会持久化新增数据源；执行前确认名称、DBMS、完整 JDBC URL 和是否立即检查连接。
- `edit_database_connection --connectionId <CID> --dbms MySQL --url "jdbc:mysql://host:3306/db" --needToCheckDs true`：会持久化修改数据源，只编辑明确指定的测试连接。

## 运行

- `get_run_configurations`：先读取已有运行配置、动态覆盖支持和工作目录；可传 `--filePath module/src/main/java/A.java` 查找可运行行。
- `execute_run_configuration --configurationName "YudaoServerApplication" --waitForExit false`：使用 `configurationName`，或同时使用 `filePath + line`，二者不可混用。只在配置支持时临时覆盖 `workingDirectory`；`waitForExit=true` 超时后进程可能继续运行，启动前应有停止或继续获取结果的方案。

## 文件、编辑与格式化

- `create_new_file --pathInProject docs/mcp-smoke.txt --text "hello" --overwrite false`：使用项目相对路径，默认不覆盖。只有创建和删除均已获相应授权时，才创建临时测试文件并在删除后用 `git_status` 复核。
- `get_all_open_file_paths`：返回活动编辑器和已打开文件，只读取 IDE 当前状态。
- `list_directory_tree --directoryPath yudao-server/src/main/java --maxDepth 2`：`directoryPath` 必填且相对项目根，使用较小 `maxDepth` 控制输出。
- `open_file_in_editor --filePath module/src/main/java/A.java`：改变 IDE 活动编辑器，不修改文件内容。
- `reformat_file --files ["module/src/main/java/A.java"]`：`files` 必须是 JSON 数组，只格式化本次明确目标文件。
- `read_file --file_path module/src/main/java/A.java --limit 200`：参数是 `file_path`；offset 从 1 开始，limit 最大 5000。
- `rename_refactoring --pathInProject module/src/main/java/A.java --symbolName oldName --newName newName`：符号名精确且区分大小写，执行前确认引用影响范围。

`apply_patch` 的 `--input` 使用 Codex patch 或 unified diff，必须包含真实换行，不能用反斜杠加 `n` 代替；补丁内双引号在 `command` 外层双引号中写为 `\"`，并检查全部 operation 均已 applied。

```text
apply_patch --input "*** Begin Patch
*** Update File: docs/mcp-smoke.txt
@@
-String value = \"old\";
+String value = \"new\";
*** End Patch"
```

若整个 `command` 又写在 JavaScript 双引号字符串中，还要按 JavaScript 规则再转义一层。

## 搜索

- `search_file --q YudaoServerApplication.java --limit 5`：`q` 是 glob；可用 `--paths ["module/**"]` 限制范围。
- `search_regex --q "public\s+static\s+void\s+main" --limit 10`：`q` 是正则；实际 `command` 中使用一个反斜杠，复杂表达式用双引号并限制路径和数量。JavaScript 双引号字符串写作 `command: "search_regex --q \"public\\s+static\\s+void\\s+main\" --limit 10"`。
- `search_symbol --q YudaoServerApplication --limit 10`：用于定位类、方法、字段，再将精确签名交给 `analyze_calls`。
- `search_text --q "SpringApplication.run" --limit 10`：用于固定文本，不能替代调用链分析。

## 终端与版本控制

- `get_repositories`：读取项目 VCS 根，多仓库项目先确定目标仓库；不能单独作为所有项目的通用身份依据。
- `git_status`：编辑、构建或测试前后记录 staged、unstaged、untracked 和冲突状态，不自动回退用户改动。
