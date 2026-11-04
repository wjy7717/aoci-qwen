---
description: 通过 AOCI MCP 为当前仓库建立或补齐认知索引
argument-hint: [范围限定，如“只处理 internal/ 下的对象”]
---

为当前仓库建立或补齐 AOCI 认知。**只通过 AOCI MCP 工具创作**，不要手改任何 `aoci.*.txt`
或 `.aoci/` 下的文件。

0. 前置检查：运行 `aoci status --json`。未初始化就停止，建议 `/aoci:init`；
   已初始化但 MCP 工具不可用，报告连接问题，不要用直接编辑文件来绕过。
1. 调用 `aoci_rules`，读取当前项目规则与创作约束。
2. 调用 `aoci_overview` 加载完整 Whole-Index。有 `next_cursor` 时按它读到最后一个 Chunk，
   然后提交一次 Attestation。不要用 `check_only` 替代、不要抽样、不要跳过 Chunk。
3. 调用无参数的 `aoci_maintain`，按它返回的实时 Guide 决定批次范围。
4. 调查源码、测试、配置与已有证据，为 `index` 角色的受管对象创作 FRAS 候选，
   用 `aoci_update_entry` 按批提交。每批提交后以 Guide 的下一步为准，不要把内部状态机写成脚本。
5. 全部批次完成后收口：`aoci_maintain` → `aoci verify` → `aoci check`。
   报告最终是 `aligned` 还是仍有缺口，以及每步的真实输出摘要。
6. 报告覆盖了多少对象、分了几批、终态是什么、服务自报的版本与仓库根。
   覆盖不足或预算耗尽时如实说明，只有 `current_system_cognition_reliable=true`
   才可以说已完整掌握当前系统认知。

执行期间保持连续：不要因为索引大就退回局部搜索冒充完整索引；遇到 `stopped` 或
`repair_required` 先看 `failed_step` 与 Guide，不要盲目重试。

范围限定（可为空）：{{args}}
