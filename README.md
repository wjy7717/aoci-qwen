# aoci-code Qwen Code 扩展

把 [AOCI-CODE](https://github.com/aoci-spec/aoci-code)（`aoci`）的 stdio MCP 服务接入 Qwen Code。
扩展**随包携带预构建的 aoci 二进制**，负责注册 `aoci mcp`，并附带常驻上下文、斜杠命令和一个操作手册技能。

安装后无需编译、无需另装 `aoci`；但当前随包版本只覆盖 **linux/amd64**，其他平台需要替换二进制（见下文）。

## 前置条件

1. 平台为 **linux/amd64**（当前随包二进制）。其他平台请先替换 `bin/aoci` 为该平台的官方发布二进制。
2. 目标仓库已经用 `aoci init` 初始化，并有 `.aoci/` 与 `aoci.txt`。
   扩展本身不会替你初始化仓库；未初始化的仓库里 MCP 服务会连不上，这是预期行为。
3. Qwen Code 支持扩展（`qwen extensions --help` 可用）。

## 随包二进制

| 项目 | 值 |
| --- | --- |
| 版本 | `0.2.0-rc3`（commit `0a00a2b952ca6f6c40556b0fc37d4230baac98ae`，构建于 2026-11-20T11:43:51Z） |
| 平台 | linux / x86-64，静态链接（无 CGO 依赖） |
| 路径 | `bin/aoci` |
| 大小 | 29,868,158 字节 |
| SHA256 | `7f034d99de12812cd35011929bf949313c34de32f43abd4aef3c77be1eb98c77` |

校验：

```bash
cd /path/to/aoci-code-extension/bin
sha256sum -c aoci.sha256
./aoci --version
```

替换为其他平台的二进制时：

1. 从[官方 Releases](https://github.com/aoci-spec/aoci-code/releases) 取对应平台的 `aoci`，按发布页完成校验；
2. 覆盖 `bin/aoci` 并 `chmod +x bin/aoci`；
3. 重新生成 `bin/aoci.sha256`；
4. 重启 Qwen Code 会话，让 MCP 进程重新拉起。

二进制是 **FSL-1.1-MIT** 许可，未作任何修改；许可证、NOTICE、专利与第三方声明随包放在 `licenses/aoci/`。


## 安装

从 Git 仓库安装：

```bash
qwen extensions install <本仓库的 git URL>
```

从本地目录安装（开发或离线分发）：

```bash
qwen extensions install /absolute/path/to/aoci-code-extension
```

安装后**新开一个 Qwen Code 会话**，扩展与 MCP 服务器才会加载。

## 避免同名 MCP 重复

本扩展注册的服务器名是 `aoci`。如果 `~/.qwen/settings.json`（或工作区设置）里已经有同名条目，
请二选一，不要同时保留：

```bash
# 查看
qwen mcp list

# 移除旧的用户级条目
qwen mcp remove aoci
```

移除后重开会话。两处同时定义同名服务器会让实际生效的配置变得不确定。

## 验证

```bash
qwen extensions list          # 应出现 aoci-code
qwen mcp list                 # 在已初始化仓库内，aoci 应为 Connected
```

在已初始化的仓库里让 Agent 调用一次 `aoci_overview`（`check_only=true`），
回执中的 `runtime_repository_root` 应指向当前仓库，`cognition_receipt.mcp_service_version`
应是实际运行的 aoci 版本。

> 在未初始化仓库或非仓库目录里看到 `Disconnected` 属于正常现象：
> `aoci mcp` 找不到仓库会直接退出。

## 提供的能力

| 资源 | 名称 | 用途 |
| --- | --- | --- |
| MCP 服务器 | `aoci` | 随包 `bin/aoci mcp`，九个认知工具（读取/维护/辅助） |
| 常驻上下文 | `QWEN.md` | 开工顺序、维护时机、硬规则、工具不可用时的处置 |
| 命令 | `/aoci:status` | 只读查看治理状态与 MCP 身份 |
| 命令 | `/aoci:init` | `aoci init` + `aoci scan`（带已初始化检查） |
| 命令 | `/aoci:index` | 通过 MCP 建立或补齐 Whole-Index |
| 命令 | `/aoci:doctor` | `aoci doctor` / `capabilities` 只读诊断 |
| 命令 | `/aoci:ui` | 启动只读面板并返回链接 |
| 技能 | `aoci-cognition` | 首次建索引、增量维护、压缩重载、失败恢复手册 |

## 常见问题

- **`qwen mcp list` 显示 Disconnected**：先确认当前目录在已初始化仓库内；
  再确认随包二进制可执行（`ls -l bin/aoci`、`bin/aoci --version`）；
  最后看用随包二进制执行的 `bin/aoci doctor --json` 输出。
- **Agent 说索引缺失**：仓库还没 `aoci init` / `aoci scan`，或 `.aoci/` 被忽略/删除。
- **安装后提示扩展二进制不可执行**：确认 `bin/aoci` 有可执行位（`chmod +x bin/aoci`），
  并确认扩展所在分区没有挂 `noexec`。
- **替换了 `bin/aoci` 但版本没变**：已在运行的 MCP 进程仍持有旧二进制。
  重开会话让扩展重新拉起 `aoci mcp`，再用 `aoci_overview` 回执核对版本。
- **其他平台/架构**：当前只带 linux/amd64。按上一节的替换步骤换成官方对应平台二进制；
  不要用交叉平台的二进制硬跑。
- **不想让 Agent 读取完整索引**：可以停用扩展（`qwen extensions disable aoci-code`），
  MCP 服务器会随之停用；不要只删 `QWEN.md` 而保留同名 MCP 条目。

## 卸载

```bash
qwen extensions uninstall aoci-code
```

卸载只移除扩展注册，不会删除任何仓库里的 AOCI 认知资产。

## 来源与边界

- AOCI-CODE 是 [aoci-spec/aoci-code](https://github.com/aoci-spec/aoci-code) 的产物；
  本扩展只做宿主接入并原样分发其二进制，不修改二进制，也不复制其治理逻辑。
- 随包二进制以 **FSL-1.1-MIT** 分发，许可证与声明见 `licenses/aoci/`；
  升级或替换二进制时，请同步更新该目录下的许可证文件与 `bin/aoci.sha256`。
- AOCI 离线运行，不联网；数据库认知只读取表结构，凭据只以环境变量名引用。
- 工具 Description、JSON Schema 与机器状态值以运行中的 `aoci` 为准，本仓库文档只是入口。
