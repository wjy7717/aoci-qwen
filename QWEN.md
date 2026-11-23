# AOCI 代码认知层

本扩展通过 stdio 启动随包携带的 `bin/aoci`（0.2.0-rc3，linux/amd64），为当前工作区提供 AOCI 认知工具。
它只在**已初始化**的仓库里工作：仓库中必须已有 `aoci.txt` 与 `.aoci/`
（由 `aoci init` 建立、`aoci scan` 建立基线）。

MCP 工具共九个：

- 读取：`aoci_rules`、`aoci_overview`、`aoci_get_entries`、`aoci_search`
- 维护：`aoci_maintain`、`aoci_update_entry`、`aoci_remove_entry`
- 辅助：`aoci_header`、`aoci_report`

工具描述、JSON Schema 与返回的机器状态值是权威；本文件和 README 只是入口。

## 开工顺序

1. 新会话开始、或已知发生上下文压缩后，先用 `aoci_rules` 读取项目规则，再加载完整 `aoci_overview`。
2. Overview 分块时，按 `next_cursor` 一直读到最后一个 Chunk，然后只提交一次 Attestation。
   `check_only=true` 只是紧凑检查点，不能替代完整交付。
3. 用 Whole-Index 定位相关对象，再读源码、测试、配置和证据核对。索引帮助你认识系统，不替代源码。
4. 完成代码与测试修改并跑过项目检查后，再维护受影响的认知。

## 维护认知

- 先调用无参数的 `aoci_maintain` 获取实时 Guide，不要凭记忆猜下一步。
- 按 Guide 调查证据，用 `aoci_update_entry` 分批提交完整批次；批次内容一次给全。
- 收口顺序：`aoci_maintain` → `aoci verify` → `aoci check`，直到正式认知与基线重新 `aligned`。
- `aoci_remove_entry` 只在对象确实被删除、且 Guide 要求时使用。
- 写入终态可能是 `applied`、`repair_required` 或 `stopped`。`stopped` 不是成功，也不代表没有写入；
  先看 `failed_step`、正式写入证据和 Guide 给出的恢复动作再继续。

## 硬规则

- 不要手工编辑 `aoci.txt`、`aoci.meta.txt`、`aoci.code.txt`、`aoci.database.txt` 或 `.aoci/baseline.json`。
- 不要把 Plan、Stage、Check、Diff、CAS、Apply 等内部状态机复制进脚本或包装器。
- 不要用局部搜索、旧记忆或直接读文件冒充一次完整 Overview 交付。
- Attestation 通过不等于语义绝对正确；只有 `current_system_cognition_reliable=true` 时，
  才可以无保留地声称掌握当前完整系统认知。
- 覆盖不足、预算耗尽或索引与源码不一致时，如实报告 `source-bound` 结论，不要补写成"已完全掌握"。

## 工具不可用时

- 服务器显示未连接或工具报索引缺失：当前工作区多半未初始化，或工作区不在仓库内。
  说明原因，并建议运行 `/aoci:init`（初始化）或 `/aoci:status`（查看状态）。
- 需要不经 MCP 的只读事实时，可用 CLI：`aoci status --json`、`aoci capabilities`、`aoci doctor`。
- 命令与技能里写的 `aoci` 指的是扩展自带的 `bin/aoci`（扩展根路径见 `qwen extensions list` 的
  `aoci-code` 条目）。如果 `PATH` 里的 `aoci` 是另一个版本，不要混用：CLI 与 MCP 应用同一份二进制。
- 如果随包二进制不可执行，先如实报告，不要改用别的二进制继续维护同一个索引。
- 数据库认知只通过显式 `aoci database` 命令接入，凭据只以环境变量名引用；不要读取业务数据。
