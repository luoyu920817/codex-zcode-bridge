# Zcode Bridge

把开发任务从 Codex 交给本机 ZCode Agent，在 Codex 中跟进进度并审查实际改动。

## 使用前

需要 Node.js 22.18+、Git、已安装并登录的 ZCode，以及支持插件 marketplace 的 Codex。

添加本项目 GitHub marketplace 并安装 **Zcode Bridge**：在桌面应用中先用 CLI 注册 marketplace，再到 **Plugins Directory** 选择并安装；CLI 用户可直接按下方命令安装。之后开启新对话并检查、信任插件的 `SessionStart` hook。Windows 上 hook 会检查本机环境并报告配置问题。Marketplace 安装无需克隆源码或运行 `npm install`；Node.js 和 ZCode 需另行安装。

CLI 安装示例：

```sh
codex plugin marketplace add https://github.com/Sandyzzx/codex-zcode-bridge.git --ref master
codex plugin add codex-zcode-bridge@codex-zcode-bridge
```

## 派发任务

默认策略为 `explicit_only`：普通开发任务由 Codex 独立完成。需要委派时，用自然语言说明任务和验收条件，例如“这个 Bug 交给 ZCode 修复”或“把实现交给 ZCode，你负责 Review”，无需固定命令。仅提及、询问或比较 ZCode 不构成委派授权；要求 Codex 独立执行时不委派。含糊的“使用子代理处理”不直接调用 ZCode，必须委派但目标不明时才询问。

确认委派后，Codex 准备 TaskPackage 并跟踪执行；小范围、低冲突任务优先当前分支 In-Place，大范围、高风险或存在冲突时按需准备 worktree。未提供 worktree 时在项目目录执行；提供时在该 worktree 执行。完成后由 Codex 独立检查 diff 和测试，决定 Accept / Retry / Replan。`completed` 不等于 Codex 已接受改动。

插件安装、MCP 注册和启动环境检查不构成新任务的委派授权；环境检查和工具注册能力保持可用。此策略由 Skill 指导 Codex 语义判断，并非 Bridge 服务端的硬性开关；已有任务的恢复、跟踪、续作、取消和结果获取机制保留。仓库根目录的 `AGENTS.md` 只约束其适用工作区，不会自动应用到安装插件的其他项目。

遇到安装或启动问题时，可调用 MCP 工具 `zcode_doctor` 查看只读诊断；它不会启动 ZCode session。

可按任务选择 provider/model，也可配置用户默认模型。执行模式默认是 `yolo`，它会放行普通工具操作并使用当前操作系统账户权限。需要审批时，将 `ZCODE_BRIDGE_MODE` 设为 `build`；记录版本上已完成单次 43,014-byte Write 的真实审批往返实测。超过单事件行上限的大审批请求会以显式标记的预览事件交付，并附私有完整记录的 `full_request` 引用（路径、字节数、SHA256，按 `JSON.stringify(params)` 紧凑序列化计量）；预览不是全文，批准前需要全文时读引用记录。审批 params 完整保存上限为 1 MiB，超过时会收到含 actual/limit 与拆分建议的明确拒绝。worktree 不是沙箱。

可选浏览器宿主（ADR-007）：`ZCODE_BRIDGE_BROWSER_USE` 默认 `off`；设为 `headless` 后，任务内 ZCode 原生浏览器工具经 `interaction/browserList`/`interaction/browserExecute` 反向 RPC 使用 worker 托管的 headless Chromium/Edge（独立临时 profile，不下载浏览器、不连接用户窗口）。可选 `ZCODE_BRIDGE_BROWSER_EXECUTABLE` 指定系统浏览器绝对路径。插件包内 `worker/vendor/` 自带所需运行资源，不依赖 Codex 的 node_modules。真实模型链路验收与边界见 仓库中的 `../../docs/reports/2026-10-15-browser-and-transport-fixes.md`。

## 安全边界

ZCode 以当前操作系统用户权限运行。Git worktree 不是 OS 沙箱；`allowed_paths` 和 `forbidden_paths` 是任务指令，不能强制限制文件或命令访问。任务 prompt、日志、可见模型输出和结果会保存在本机 `.codex/codex-zcode-bridge/` 目录。派发前请检查任务可见的工作区内容。

Marketplace 用户配置自定义 ZCode 路径或默认模型的说明见[中文安装指南](https://github.com/Sandyzzx/codex-zcode-bridge/blob/master/README.zh-CN.md#安装)。
