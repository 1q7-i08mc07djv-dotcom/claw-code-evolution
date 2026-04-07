# Claw Code 50 课精要

> 来源：Claw Code (by UltraWorkers) Rust 实现源码分析
> 时间：2026-04-07

---

## 第1课：系统提示词核心逻辑

- `SystemPromptBuilder` 构建器模式，分段有序注入 prompt
- `__SYSTEM_DYNAMIC_BOUNDARY__` 标记动态内容起点，作为压缩锚点
- 指令文件发现链：从当前目录向上遍历找 `CLAUDE.md`、`.claw/instructions.md`
- Git 上下文自动发现：`git status`、`git diff`、`git log -n 5`、staged files
- Session 持久化：256KB rotation，最多 3 个 `.rot-*.jsonl` 归档

**借鉴**：已更新 AGENTS.md，加入 Git 上下文发现机制

---

## 第2课：权限系统

- 五级权限模式：ReadOnly → WorkspaceWrite → DangerFullAccess → Prompt → Allow
- 三层过滤：PermissionPolicy（规则引擎）→ PermissionEnforcer（执行前检查）→ bash_validation（命令语义分析）
- ReadOnly 模式：完整白名单命令列表，拦截写命令和状态修改命令
- 危险命令检测：`rm -rf /`、`mkfs`、`dd`、`fork bomb`、`chmod -R 777`
- Hook Override：`PermissionContext` 可在运行时覆盖权限决策

**借鉴**：已更新 AGENTS.md，加入"权限操作规范"章节

---

## 第3课：Session 压缩机制

- 触发条件：`preserve_recent_messages=4`，`max_estimated_tokens=10000`
- 压缩后：旧消息合并为摘要 system message，保留最近 4 条原文
- 多次压缩：合并而非替换，保留 `Previously compacted context` + `Newly compacted context`
- 摘要内容结构：Scope统计、Tools mentioned、Pending work、Key files、Key timeline
- `summary_compression.rs`：行级压缩（1200 chars / 24 lines / 160 chars per line），优先级选择

**借鉴**：memory/YYYY-MM-DD.md 可以做类似的历史叠加而非覆盖

---

## 第4课：Task 子系统

- TaskPacket 契约完整：objective、scope、repo、branch_policy、acceptance_tests、commit_policy、reporting_contract、escalation_policy
- 任务状态机：Created → Running → Completed/Failed/Stopped（终态）
- TaskRegistry：内存中的任务表，支持 create/get/list/stop/update/output/assign_team

**借鉴**：subagent 可以增加 description 和 acceptance_tests 概念

---

## 第5课：MCP 生命周期管理

- 11阶段有限状态机：ConfigLoad → ServerRegistration → SpawnConnect → InitializeHandshake → ToolDiscovery → ResourceDiscovery → Ready ↔ Invocation → ErrorSurfacing → Shutdown → Cleanup
- 阶段转换严格验证，不可跳跃
- McpDegradedReport：优雅降级，部分服务器挂了自己工作

**借鉴**：工具调用可以返回"部分成功"状态

---

## 第6课：Hooks 系统

- 三种事件：PreToolUse、PostToolUse、PostToolUseFailure
- Hook 可以：覆盖权限决策（allow/deny/ask）、修改工具输入、注入消息
- Exit code 语义：0=allow、2=deny、1=failure
- AbortSignal：防止 hook 无限阻塞

**借鉴**：工具调用前后可加标准化检查

---

## 第7课：Recovery Recipes

- 六种失败场景：TrustPromptUnresolved、PromptMisdelivery、StaleBranch、CompileRedCrossCrate、McpHandshakeFailure、PartialPluginStartup、ProviderFailure
- 核心原则：一次自动恢复，强制升级（max_attempts=1）
- 三种结果：Recovered、PartialRecovery、EscalationRequired

**借鉴**：subagent 加"失败后最多重试一次"规范

---

## 第8课：Green Contract

- 四级质量门禁：TargetedTests < Package < Workspace < MergeReady
- 偏序关系：高级自动满足低级要求
- 用于构建质量验证

---

## 第9课：ConversationRuntime — 主模型循环

- `run_turn()`: 用户输入 → 模型调用 → 工具调用循环 → 返回 TurnSummary
- 工具调用链路：PreHook → PermissionCheck → ToolExecutor → PostHook/FailureHook → 结果合并
- `maybe_auto_compact()`: 每次 turn 后检查累积 input tokens 是否超过阈值（默认100k）
- `fork_session()`: Session Fork 支持分支
- SessionTracer: 结构化事件日志
- max_iterations 保护：防止无限循环

**重要**：工具执行流程的完整管道值得借鉴

---

## 第10课：Config 加载机制

- 配置优先级链：User Legacy (.claw.json) → User Settings → Project Legacy → Project Settings → Local Settings
- `deep_merge_objects()`: 递归合并嵌套 JSON 对象
- `ProviderFallbackConfig`: 模型提供商的降级链（primary + fallbacks 顺序尝试）
- Model aliases: 用户定义的模型别名映射
- Sandbox 配置：`filesystemMode`（off/workspace-only/allow-list）
- OAuth 配置：完整的 OAuth 流程支持

**借鉴**：ConfigLoader 发现链设计值得参考

---

## 第11课：Sandbox 隔离机制

- 基于 Linux namespace 的进程隔离（`unshare --user --map-root-user`）
- 三级文件系统隔离模式：Off / WorkspaceOnly / AllowList
- 容器检测：通过 cgroup、/.dockerenv、/run/.containerenv、环境变量
- `SandboxStatus`: 详细报告各功能支持状态和激活状态
- `build_linux_sandbox_command()`: 生成 unshare 命令行

**重要**：提供了生产级的进程隔离参考实现

---

## 第12课：Usage 追踪与成本估算

- TokenUsage: input / output / cache_creation / cache_read 四类计数
- UsageTracker: 跨 turn 累积计数
- ModelPricing: Haiku($1/$5) / Sonnet($15/$75) / Opus($15/$75) 三档定价
- `summary_lines_for_model()`: 生成带成本明细的 usage 报告

**重要**：Cache 成本分离统计很值得参考

---

## 第13课：Branch Lock 冲突检测

- `BranchLockIntent`: lane_id / branch / worktree / modules 定义
- `detect_branch_lock_collisions()`: 检测不同 lane 意图修改同一 branch+module 的冲突
- 模块重叠检测：`runtime/mcp` 和 `runtime` 视为重叠

**重要**：多 agent 并发工作时防止修改同一分支的模块

---

## 第14课：Lane Events 事件系统

- 完整的 lane（工作流/任务通道）生命周期事件体系
- 16种事件类型：started, ready, prompt_misdelivery, blocked, red, green, commit_created, pr_opened, merge_ready, finished, failed, reconciled, merged, superseded, closed, branch_stale_against_main
- 11种失败分类：prompt_delivery, trust_gate, branch_divergence, compile, test, plugin_startup, mcp_startup, mcp_handshake, gateway_routing, tool_runtime, infra
- `dedupe_superseded_commit_events()`: 根据 canonical_commit 去重被取代的 commit 事件

**重要**：提供了生产级多 agent 协作事件模型参考

---

## 第15课：Stale Base 检测

- `.claw-base` 文件存储期望的 base commit
- `--base-commit` CLI flag 优先级高于 `.claw-base` 文件
- `BaseCommitState`: Matches / Diverged(expected, actual) / NoExpectedBase / NotAGitRepo
- Diverged 时输出警告："Session may run against a stale codebase"

**重要**：防止在代码已过时的情况下运行会话

---

## 第16课：Team & Cron Registry

- TeamRegistry: 内存中的团队注册表，支持 create/get/list/delete/remove
- CronRegistry: 定时任务注册表，支持 create/get/list/delete/disable/record_run
- 跟踪 `run_count` 和 `last_run_at`
- team_id 格式: `team_{timestamp}_{counter}`；cron_id 格式: `cron_{timestamp}_{counter}`

**重要**：提供了多 agent 团队和定时任务的基础架构参考

---

## 第17课：FileOps 文件操作模块

- Read: 10MB 上限，二进制检测（NUL字节），行窗口（offset+limit）
- Write: 10MB 上限，保留 original_file，生成 structured_patch
- Edit: 字符串替换，replace_all 支持
- Glob: 按修改时间排序，最多 100 结果
- Grep: 正则搜索，上下文行（-B/-A/-C），输出模式控制
- Workspace boundary: `validate_workspace_boundary()` 防止 `../` 穿越
- Symlink escape: `is_symlink_escape()` 检测符号链接是否逃逸工作区

**重要**：生产级文件操作模块，安全性设计完善

---

## 第18课：OAuth 认证模块

- OAuth 2.0 + PKCE (S256) 完整流程
- Token 持久化：`~/.claw/credentials.json`，与其他配置分离
- Loopback redirect: `http://localhost:{port}/callback`
- `generate_pkce_pair()`: 32字节随机 verifier，SHA256 摘要生成 challenge

**重要**：完整的生产级 OAuth2+PKCE 实现

---

## 第19课：Trust Resolver

- 检测屏幕是否出现"do you trust these files"提示
- 5种 cue: "do you trust the files in this folder" 等
- 三级策略：AutoTrust / RequireApproval / Deny
- 配置：allowlisted + denied roots，支持路径前缀匹配

**重要**：Deny 优先级高于 Allowlist，防止危险路径被错误信任

---

## 第20课：Remote / Upstream Proxy

- `CCR_UPSTREAM_PROXY_ENABLED` + `CLAUDE_CODE_REMOTE` 双因素启用
- WebSocket 代理：`wss://{base}/v1/code/upstreamproxy/ws`
- CA 证书管理：`CCR_CA_BUNDLE_PATH`、`CCR_SYSTEM_CA_BUNDLE`
- NO_PROXY 白名单：localhost、RFC1918 私有地址等
- subprocess_env: 自动注入 HTTPS_PROXY/SSL_CERT_FILE 等环境变量

**重要**：生产级远程代理和 CA 证书管理方案

---

## 第21-27课：工具注册 / Worker / LSP / Agent / Skill

详见 SKILL.md 导航，核心是：
- 50+ 内置工具完整注册与权限体系
- Global 注册表（OnceLock 单例模式）
- 子 agent 类型：Explore/Plan/Verification/claw-guide
- Skill 路径解析：多根目录搜索链
- 多语言 LSP 统一注册与派发

---

## 第28-42课：CLI / Plugin / Renderer / SSE

- 70+ slash 命令完整架构
- Terminal Renderer：Markdown + 语法高亮 + Spinner
- LineEditor：rustyline 补全 + history
- Plugin 系统：Builtin/Bundled/External + Hooks + Lifecycle
- Telemetry：完整事件追踪系统

---

## 第43-50课：API / Provider / Error

- Multi-Provider：Anthropic / Xai / OpenAi 统一抽象
- Model alias 解析 + context window 校验
- API Error 分类：retryable / context_window / auth / transport
- Dotenv 解析：支持 export 前缀、引号去除
- Preflight request 校验

---

## 总结

| 类别 | 数量 | 最有价值的设计 |
|------|------|--------------|
| 核心系统 | 8 | 权限五级模式、压缩合并、Hook 拦截 |
| Runtime | 7 | ConversationRuntime 管道、MCP 生命周期 |
| 工具链 | 8 | FileOps 安全、Worker 状态机 |
| 架构基础设施 | 8 | Config 发现链、Sandbox 隔离、OAuth |
| 生态系统 | 10 | CLI/Plugin/Telemetry/LSP/Skill |
| API 层 | 8 | Multi-Provider、Error 分类、Telemetry |

**核心感悟**：Claw Code 工程度极高但工具化，缺乏人格灵魂。
自身优势在于：SOUL.md 人格、Heartbeat 主动性、群聊感知、Skills 热插拔。
