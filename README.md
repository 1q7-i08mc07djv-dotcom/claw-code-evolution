# claw-code-evolution

Claw Code 架构分析与进化指南。

## 是什么

来自 [Claw Code](https://github.com/UltraWorkers/claw-code) (by UltraWorkers) Rust 实现的系统设计学习，用于 AI Agent 的架构进化。

分析了其 50 个核心设计知识点，涵盖系统提示词、权限模型、会话压缩、子任务管理、MCP 生命周期、Hook 拦截层、错误恢复、质量门禁、CLI 架构、渲染系统、遥测、API 抽象等。

## 结构

```
claw-code-evolution/
├── SKILL.md              ← 入口（触发条件 + 快速导航）
└── references/
    └── 50-lessons.md     ← 50 课精要完整内容
```

## 核心内容

### 已验证可借鉴的设计

| 设计 | 来源 | 说明 |
|------|------|------|
| Git 上下文自动发现 | 第1课 | 每次启动读取 git status/log/diff |
| 危险操作申报 | 第2课 | rm -rf 等高危操作主动告知 |
| 工作区保护 | 第2课 | 禁止修改系统目录 |
| Git 操作分级 | 第2课 | read-only vs 需确认 vs 禁止 |
| Session 压缩合并 | 第3课 | 历史叠加而非覆盖 |

### 知识点分类

| 类别 | 数量 | 代表设计 |
|------|------|---------|
| 核心系统 | 8 | 权限五级模式、压缩合并、Hook 拦截 |
| Runtime | 7 | ConversationRuntime 管道、MCP 生命周期 |
| 工具链 | 8 | FileOps 安全、Worker 状态机 |
| 架构基础设施 | 8 | Config 发现链、Sandbox 隔离、OAuth |
| 生态系统 | 10 | CLI/Plugin/Telemetry/LSP/Skill |
| API 层 | 8 | Multi-Provider、Error 分类、Telemetry |

## 核心感悟

Claw Code 工程度极高但**工具化**定位——"You are an interactive agent that helps users with software engineering tasks"。

自身优势在于：**人格灵魂**（SOUL.md）、**主动巡查**（Heartbeat）、**群聊感知**（知道何时说话何时沉默）、**技能系统**（Skills 热插拔）。

## 安装

将 `.skill` 文件放入 OpenClaw 的 skills 目录，或使用 clawhub 安装。

## 来源

- Claw Code: https://github.com/UltraWorkers/claw-code
- 分析时间: 2026-04-07
- 分析方式: Rust 源码全文阅读
