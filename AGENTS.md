# AGENTS.md

本仓库提供 Codex 按需委派任务给本机 ZCode 的 Bridge。任何 agent 动手前先读这份文件。

## ZCode Delegation Policy

- 默认策略为 `explicit_only`：Codex 独立完成普通开发任务，只有用户通过自然语言明确要求将具体任务交给 ZCode 或可确定指向它的 Worker 时才委派，无需固定命令。
- 仅提及、询问或比较 ZCode、分析 Bridge 源码不构成委派授权；用户要求 Codex 独立执行时不得委派。含糊请求默认自行处理；必须委派但目标 Agent 不明时才询问，不把通用“子代理”自动当作 ZCode。
- 安装插件、注册 MCP 和工具可用不构成启动新 Worker 任务的理由。Codex 始终负责拆分、TaskPackage、跟踪与独立验收，决定 Accept / Retry / Replan，不把 `completed` 当作验收通过。
- 本文件只约束其适用范围内的工作区，不会自动控制外部项目；插件在其他项目中的触发说明由 `plugins/codex-zcode-bridge/skills/zcode-bridge/SKILL.md` 提供。不修改用户全局 Codex 配置。

## 阅读顺序

1. `docs/README.md` —— 文档路由、状态等级、语言约定。
2. `docs/PROJECT_STATE.md` —— 当前状态快照：稳定、实验、不支持、调查中。
3. 按任务类型读权威文档：架构改动读 `docs/ARCHITECTURE.md`，接口改动读 `docs/INTERFACES.md`，宿主接入读 `docs/SHARED_CORE.md`，运行配置读 `docs/ZCODE_RUNTIME.md`。
4. 需要知道"为什么这样定"时读 `docs/decisions/`。
5. `docs/archive/` 默认不读，只有调查历史决策时才读。

## 文档等级

每份文档顶部标注四种状态之一：`AUTHORITATIVE` 当前事实或合同、`DECISION` 已批准决策、`RESEARCH` 研究结论、`ARCHIVED` 历史材料。研究结论和归档材料不能当作当前实现使用。

权威文档的顶部至少包含：

```markdown
> Status: AUTHORITATIVE
> Last updated: YYYY-MM-DD
> Last verified: YYYY-MM-DD（核对了哪几条；只核对了一部分就写明范围）
> Verified against: <commit SHA>
```

`Last updated` 和 `Last verified` 是两件事：改过不等于核对过。`Verified against` 记录核对时代码停在哪个提交，指码已经前进很多时应当重新核对。索引汇总在 `docs/README.md`，但不能代替文档自身的标注。

## 硬规则

- 改变 `docs/ARCHITECTURE.md`、`docs/INTERFACES.md`、`docs/SHARED_CORE.md`、`docs/ZCODE_RUNTIME.md` 描述的行为前，先有 `DECISION` 记录。实现不能反向改写合同。
- 公共 MCP 工具名与 schema、任务状态语义、隐私边界、新增运行时依赖，都属于合同变更。
- 研究材料放 `docs/research/`，草稿放 `docs/_draft/`（已忽略）。不要把未定稿留在 `docs/` 顶层。
- 语言：`README.md` 英文，技术文档中文，`docs/README.md` 双语索引。
- 核心权威文档控制在 10 份以内。单份超过约 15 KB，或同一主题被三个以上独立读者分读，才拆成多份。
- 写结论时区分"已核实"和"未运行"。不要把 NOT RUN 写成通过，也不要把尝试过的方法写成可行方法。
- 引用 ZCode 内部结构时标明观测版本，用 Observed 措辞，不要写成官方合同。

## 提交与验证

- 代码改动跑 `npm run typecheck`、`npm test`、`npm run build`、`npm run validate:plugin`，并保证 `git diff --check` 干净。
- `plugins/codex-zcode-bridge/dist/bridge.mjs` 与 `worker/worker-main.mjs` 是提交物，必须与源码同步；CI 会校验它们没有落后。
- 不默认提交、推送或发布。发布走 release-please，合并到 `master` 之后由它生成版本 PR。
