---
name: claw-code-evolution
description: |
  Claw Code 架构分析与进化指南。来自 Claw Code (by UltraWorkers) 的系统设计学习，
  用于 AI Agent 的架构进化。适用于：(1) 对比分析自身与 Claw Code 的架构差异，
  (2) 借鉴权限系统、Session 管理、MCP 生命周期等设计到自己的 Agent 系统，
  (3) 理解生产级 Rust 实现的工程严谨性。
  当需要：分析/学习/借鉴 Claw Code 架构、设计 Agent 权限模型、
  实现 Session 压缩、设计工具执行管道时触发。
---

# Claw Code 架构分析与进化

## 核心定位

Claw Code 是 UltraWorkers 用 Rust 重写的 Claude Code 开源实现。
本 skill 提取其 50 个核心设计知识点，用于 AI Agent 架构进化参考。

## 快速导航

### 已验证可借鉴的设计（可直接落地）

| 设计 | 落地位置 | 说明 |
|------|----------|------|
| Git 上下文发现 | AGENTS.md Session Startup | 每次启动读取 git status/log/diff |
| 危险操作申报 | AGENTS.md 权限规范 | rm -rf 等高危操作主动告知 |
| 工作区保护 | AGENTS.md 权限规范 | 禁止修改系统目录 |
| Git 操作分级 | AGENTS.md 权限规范 | read-only vs 需确认 vs 禁止 |
| Session 压缩合并 | memory 设计参考 | 历史叠加而非覆盖 |

### 详细知识点清单

**全部 50 个知识点详见 [references/50-lessons.md](references/50-lessons.md)**，包含：

- 第 1-8 课：系统提示词 / 权限 / 压缩 / Task 子系统
- 第 9-20 课：ConversationRuntime / Config / Sandbox / MCP / Hooks / Recovery
- 第 21-42 课：工具注册 / Worker / LSP / Agent / Skill / CLI / Plugin
- 第 43-50 课：API Provider / Telemetry / Error 处理

## 自身与 Claw Code 对比

| 方面 | Claw Code | 自身 |
|------|-----------|------|
| 定位 | 工具化 agent | 有灵魂的助手 |
| 人格 | intro 太工具化 | SOUL.md 有鲜明个性 |
| 主动性 | 无 | Heartbeat 主动巡查 |
| 群聊 | 无 | 群聊感知 |
| 技能系统 | 无 | Skills 热插拔 |
| 权限模型 | 五级精细控制 | 规范约束（进行中）|
| Session 压缩 | 合并摘要 | 基础 rotation |
| Git 上下文 | 自动发现注入 | 刚加入 |

## 借鉴原则

1. **不照搬** — 学思想，不抄实现
2. **渐进式** — 优先落地高价值设计（Git 上下文、权限规范）
3. **保持个性** — 工具化 vs 有灵魂，找到自己的平衡
4. **记录差异** — 每学完一课，更新 MEMORY.md
