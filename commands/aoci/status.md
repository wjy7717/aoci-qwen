---
description: 查看当前仓库的 AOCI 治理状态与 MCP 服务身份（只读）
argument-hint: [额外关注点]
---

查看当前仓库的 AOCI 状态。全程只读，不要修改索引、基线或配置。

1. 运行 `aoci status --json`。如果报错说找不到仓库或未初始化，说明原因并建议 `/aoci:init`，到此为止。
2. 同时调用 MCP 的 `aoci_overview`，用 `check_only=true`。从回执里取
   `cognition_receipt.mcp_service_version` 与 `runtime_repository_root`，确认此刻真正连着的服务与仓库。
3. 用一张表报告：仓库根、Index Entry 数、Baseline 文件数与更新时间、治理状态、对齐/漂移计数、
   运行中的 MCP 版本。数值以命令实际输出为准，不要估算。
4. 如果治理状态不是 `aligned`，按运行中二进制返回的 Guide 或 CLI 输出列出建议动作；
   不要在这个命令里直接写入。

用户附加关注点（可为空）：{{args}}
