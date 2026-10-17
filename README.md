# bitbrowser-agent-kit

独立仓库：[by10457/bitbrowser-agent-kit](https://github.com/by10457/bitbrowser-agent-kit)。采用 [MIT 许可证](LICENSE)。本项目为社区封装，与比特浏览器官方无隶属关系。

面向 AI Agent 的比特浏览器自动化工具集，基于比特浏览器官方 API 封装。支持标准 stdio MCP 的 Agent 宿主可以接入 MCP 服务，也可以通过 CLI 或共享 JavaScript 核心调用；轻量 Skill 提供需求识别、操作编排和结果交接规范。普通 JS ESM，无编译或压缩步骤。

## 快速接入

准备 Node.js >=22.12（含 npm）和已启用本地 API 的比特浏览器。获取项目并安装依赖：

```powershell
git clone https://github.com/by10457/bitbrowser-agent-kit.git
cd bitbrowser-agent-kit
npm ci
```

按 Agent 宿主支持的方式接入：

- **MCP**：参考 [通用配置示例](examples/mcp-config.json)，将 `node adapters/mcp/stdio.js` 注册为 stdio 服务，使用项目入口的实际绝对路径。
- **Skill**：让 Agent 阅读 [SKILL.md](SKILL.md)，或按宿主的 Skill 安装机制接入；发现目录和调用语法以宿主为准。
- **CLI / JS**：按 [CLI 说明](references/commands.md) 调用现有命令，或按 [JS API](references/api.md) 导入共享函数。

接入后可请求 Agent：“使用比特浏览器工具检查在线状态并列出窗口，只做只读查询，不启动、关闭或修改任何窗口。”应以当前宿主实际发现工具并成功调用为准；独立 SDK 验证成功不代表宿主已经加载 MCP。

### Codex 接入示例

可以将以下提示词交给 Codex：

> 请拉取 https://github.com/by10457/bitbrowser-agent-kit 到本地，阅读项目的 AGENTS.md、SKILL.md 和 references/setup.md，检查所需依赖，使用现有 setup.js 完成 Codex 的 Skill 和 MCP 接入。保留已有配置，遇到同名冲突不要覆盖。完成后通过当前宿主的 MCP 检查在线状态并列出窗口，只做只读查询，不启动、关闭或修改任何窗口。如果需要新开会话或重新加载宿主，请明确说明，并区分独立 SDK 验证结果与宿主实际调用结果。

当前 `setup.js` 自动配置的是 **Codex**，需要 Codex CLI；其他宿主按上面的通用接入方式配置。在项目目录运行 `node setup.js` 即可安装缺失依赖、注册 Skill/MCP 并执行只读验证。支持重复运行，保留已有配置，冲突时停止。参数及故障处理见 [安装与接入](references/setup.md)。

## 能力与边界

- 查询在线状态、窗口详情及运行状态、标签页和显示器。
- 创建、启动、关闭、删除窗口，修改名称备注，访问 URL 和操作标签。
- 随机指纹，宫格、对角线及自适应排列，清理 Cookie/HTTP 缓存。
- 管理写操作采用 prepare → execute → inspect；结果不确定时核对原计划，不自动重放写请求。
- 从标签快照生成包含窗口 ID、targetId、URL 和账号证据的交接文件，直接复用工具返回的计划参数和文件链接。

操作限定用户授权目标。新窗口隔离会话不等于已登录；账号状态需要网站业务证据。排列仅返回 API 确认，指纹不承诺全局唯一。官方 API 未提供的安装检测和桌面客户端启动不在封装范围内。

## 使用说明

- [轻量 Skill](SKILL.md)：Agent 使用入口。
- [任务编排与交接](references/workflows.md)：多窗口、独立登录及结果交付。
- [MCP 契约](references/mcp.md)、[JS API](references/api.md)、[CLI](references/commands.md)。
- [管理操作](references/management.md)、[连接生命周期](references/connection.md)、[诊断](references/diagnostics.md)。
- [通用 MCP 配置](examples/mcp-config.json)、[Codex 配置](examples/codex-config.toml)。

## 项目结构与开发

| 目录 | 职责 |
| --- | --- |
| src/client、src/contracts | 官方 API、参数及结果契约 |
| src/services、src/runtime | 业务服务、连接、锁、限频与持久化 |
| adapters | CLI 与 MCP 适配 |
| scripts | 稳定 JS 导入入口和开发检查 |
| references、examples、agents | 使用说明、请求示例与 Skill 元数据 |
| tests | 自动化回归与协议集成测试 |

```powershell
npm ci
npm test
npm run check
```

测试默认不操作真实浏览器，临时产物使用系统临时目录，可通过 BITBROWSER_TEST_ROOT 指定其他位置。运行状态独立于源码保存，位置及覆盖方式见接入说明。分发保留回归测试，排除依赖、运行数据和本地开发档案。
