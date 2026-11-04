---
description: 初始化当前仓库的 AOCI 认知骨架并建立基线
argument-hint: [aoci init 的额外参数，如 --locale zh-CN --scope-profile full]
---

为当前仓库初始化 AOCI。这个命令会写入文件，按下面的顺序执行。

1. 用 `git rev-parse --show-toplevel` 找到仓库根。若不在 git 仓库内，停止并说明原因。
2. 检查仓库根是否已存在 `aoci.txt` 或 `.aoci/`。任一存在就**停止**，报告"仓库已初始化"，
   建议改用 `/aoci:status`；不要重复 init，也不要覆盖现有认知资产。
3. 向用户列出将要创建的内容：`aoci.txt`、`aoci.meta.txt`、`aoci.code.txt`、`.aoci/`（含 `config.json`）、
   `AGENTS.md`。然后执行初始化：
   `aoci init` + 用户提供的参数。
   不要加 `--agent`：本仓库接入 Qwen Code 的方式是本扩展的 MCP 注册，不需要 init 再写宿主配置文件。
4. 执行 `aoci scan`，建立第一份 Baseline。若返回未初始化或结构错误，停止并报告原始错误。
5. 报告：实际执行的两条命令、生成的文件、受管文件数与 Baseline 状态。
6. 说明下一步：运行 `/aoci:index` 让 Agent 通过 MCP 创作第一条认知。
   `init` 只是骨架，Zero Entry 不代表已有可用认知。

不要把这三步合并成脚本，也不要跳过第 2 步的存在性检查。

用户参数（可为空）：{{args}}
