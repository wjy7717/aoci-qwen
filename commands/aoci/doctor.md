---
description: 诊断 AOCI 环境、仓库资产与 MCP 接入
---

对当前仓库做 AOCI 诊断，只读，不修改正式资产。

1. 运行 `aoci doctor --json`，按它返回的三组信息汇报：仓库与核心资产、Agent 集成、平台能力。
2. 运行 `aoci capabilities`，报告版本、布局与 MCP 工具数。
3. 运行 `aoci status --json`，确认治理状态；未初始化就说明这一点。
4. 如果诊断指出扩展接入异常，同时检查 `qwen mcp list` 里 `aoci` 的连接状态，
   并说明本扩展要求 `aoci` 在 PATH 中、且当前工作区位于已初始化仓库内。
5. 只报告事实与建议，不要自动修复。需要写入的动作先说明再征得用户同意。

用户附加关注点（可为空）：{{args}}
