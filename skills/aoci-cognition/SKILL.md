---
name: aoci-cognition
description: AOCI 认知索引的操作手册：首次建立 Whole-Index、日常增量维护、压缩后重载、verify/check 收口与失败恢复。在为已初始化仓库建立或维护 AOCI 认知时使用。
---

# AOCI 认知索引操作手册

AOCI 把代码、配置、测试（以及可选的数据库结构）压缩成受治理的文本认知层，随仓库一起进 Git。
模型负责语义（写 Entry），`aoci` 负责治理（Scope、Baseline、Plan、CAS、Apply、Ledger）。

## 何时使用

- 仓库已初始化（有 `aoci.txt` 与 `.aoci/`），用户要求"建立索引/补齐认知/维护 AOCI"。
- 会话上下文被压缩后，需要重新获得系统级认知再继续业务任务。
- `aoci status` 报出漂移或治理状态不是 `aligned`。

未初始化时先用 `/aoci:init`（= `aoci init` + `aoci scan`）；本技能不负责初始化骨架。

## 认知资产布局

- `aoci.txt`：Root，只声明当前 CognitionSet 有哪些 Volume。
- `aoci.meta.txt`：标签字典、FRAS 规则、预算、创作合同。
- `aoci.code.txt`：Code Volume 的对象 Entry。
- `aoci.database.txt`：启用 Database Cognition 时才存在。
- `.aoci/`：`config.json`、`baseline.json`、可选 `curation.json`，以及草稿、Ledger、事务与恢复证据。

Root + Meta + 参与的对象 Volume 才是 Whole-Index。只读 `aoci.txt` 不等于读到完整认知。

这些文件要跟代码一起提交；不要把它们加进 `.gitignore` 或 `.git/info/exclude`，
否则 `scan` 按 git 忽略权威取文件时会静默跳过，索引建不起来。
`init --agent` 写出的宿主配置（`.mcp.json`、`.claude/settings.json`、`.codex/config.toml`、`opencode.json`）
是本机绑定的绝对路径，应加入忽略，不要提交。

## 首次建立索引

1. `aoci_rules`：读取当前项目规则、标签字典与创作约束。
2. `aoci_overview`：拉取完整 Whole-Index。返回 `next_cursor` 时循环读取到最后一个 Chunk，
   然后提交**一次** Attestation。禁止用 `check_only`、局部搜索、旧记忆或直接读文件替代。
3. 无参数 `aoci_maintain`：拿到当前批次的实时 Guide、目标对象与状态机下一步。
4. 调查证据：读源码、测试、配置；不要只看文件名或 AST 猜语义。S 字段只写非显而易见、
   且能防止错误的信息，遵守 Meta 的准入与配额。
5. `aoci_update_entry`：按 Guide 的目标集合提交完整批次。每批结束后重新 `aoci_maintain`，
   以返回的下一步为准，不要把 Plan/Stage/Diff/CAS 的流程复制成脚本。
6. 收口：`aoci_maintain` → `aoci verify` → `aoci check`，直到 `aligned`。
7. 报告：覆盖对象数、批次数、终态、`cognition_receipt.mcp_service_version` 与
   `runtime_repository_root`。

## 日常任务中的维护

业务开发照常进行，不需要在每个需求后重复"维护索引"的提示。合适时机是代码与测试都稳定之后：

1. 确认受影响对象仍在 Managed Scope 内（`aoci_search` / `aoci_get_entries` 辅助定位）。
2. 调用无参数 `aoci_maintain`，按 Guide 只更新真正变化的 Entry。
3. 对象消失才用 `aoci_remove_entry`；不要用它清理"看起来过时"的内容。
4. `aoci verify` + `aoci check` 收口，报告是否重新 `aligned`。

## 上下文压缩之后

1. 若 Rules 已不可靠，先 `aoci_rules`。
2. 用新的事件 ID 声明 `context_compaction`。
3. 完成一次普通完整 `aoci_overview`（cursor、确认、Attestation 全序列）。
4. `check_only` 与认知 probe 不能替代这次重载。
5. 即使 Attestation 是 `partial` 或 `fail`，也继续按 source-bound 规则执行，
   不要自动发起第二次重载；重载被打断时按 Rules 的恢复要求重来，不要自行改配置。

## 终态与失败恢复

写入结果只有三种：

- `applied`：批次已提交，继续 Guide 的下一步。
- `repair_required`：需要按返回的证据修复，不要重跑同一批次。
- `stopped`：不是成功，也不代表没有写入。检查 `failed_step`、正式写入证据与 Guide 的恢复动作。

其它约束：

- 完整批次在正式写入前被拒绝时，索引不变。
- 正式写入开始后中断，会留下不可变 Intent、写入证据与 Recovery；按可证明的后像 Resume，
  或按精确前像回滚。
- 第三方字节冲突时失败关闭，不要用"继续写完"覆盖外部修改。

## 诊断与只读事实

```bash
aoci status --json        # 紧凑对齐状态
aoci status --deep        # 计算当前磁盘差异（会哈希整个仓库，可能较慢）
aoci capabilities         # 版本、布局、MCP 工具数
aoci doctor               # 仓库/接入/平台三组只读诊断
```

`verify`、`check`、`index score`、`index inventory` 的"只读"只表示不改正式索引与 Baseline；
启用 Ledger 时仍可能追加本地审计数据。需要严格零写入时使用隔离副本。

需要手工排查时可用 CLI：`aoci index agent guide --agent <name> --json` 读取实时 Guide。
不要用它把状态机固化进包装脚本。

## 硬规则

- 不手改 `aoci*.txt`、`.aoci/baseline.json` 或任何正式认知资产。
- 不复制内部状态机；以运行中二进制的 Tool Description、Schema、Guide 与机器状态为准。
- 不用局部结果冒充完整 Overview 交付。
- 只有 `current_system_cognition_reliable=true` 才可声称无保留掌握当前完整系统认知。
- 数据库只读系统目录、只取表结构；不读业务数据，凭据只用环境变量名引用。
- 默认不联网、不上传索引；认知语义由宿主模型通过 MCP 撰写。
