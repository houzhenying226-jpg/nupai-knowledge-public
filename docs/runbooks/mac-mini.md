# Mac Mini开发机配置

> 类型：操作规程 | 项目：_global | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:00:00Z

## 摘要

所有 Claude Code 操作通过 Tailscale 远程进入这台机器执行。两套仓库 CLAUDE.md 总计 747 行，包含项目铁律和上下文。

## 正文

> 记录日期：2026-04-17  风险等级：L3

## 背景
新加坡 Mac Mini 是振英远程跑 Claude Code 的开发机，所有 SSH/Hooks/Skills/Agent 都在这上面，需要一卡看清环境。

## 内容
主机名 192.168.1.9，Tailscale IP 100.69.20.113，macOS 26.3，10 核 CPU，16GB 内存，磁盘 228G（已用 11G / 可用 62G / 占比 16%）。Node v25.7.0，Python 3.9.6，SSH Key 在 ~/.ssh/deploy_key（ed25519）。 项目路径：~/-nupai-store/（CLAUDE.md 343 行）；~/nupai-crm/（CLAUDE.md 404 行）。 Claude Code 配置：Hooks（14 个）→ ~/.claude/hooks/；Skills（6 个 × 2 仓库）→ {仓库}/.claude/skills/；Agent → {仓库}/.claude/agents/frontend-engineer.md。启动命令：CRM → ~/start-claude.sh crm；Store → ~/start-claude.sh store；紧急停止 → ~/stop-claude.sh。 SSH 隧道（MCP 配置）：localhost:3307 → CRM MySQL；localhost:5433 → Store PostgreSQL。MCP 包用 mcp-server-mysql + mcp-postgres（@modelcontextprotocol/server-postgres 已废弃）。

## 结论
所有 Claude Code 操作通过 Tailscale 远程进入这台机器执行。两套仓库 CLAUDE.md 总计 747 行，包含项目铁律和上下文。

## 影响
所有 Claude Code 远程开发流程、Hooks 五层防护、Skills 触发、Agent 调度、SSH 隧道连接生产数据库的位置。

## 下次注意
新增 Hook/Skill 后同步更新本卡和基础设施清单 §12；Tailscale 断线整台机器开发能力归零，要保证多设备 Tailscale 在线；macOS 升级前先备份 ~/.claude 整目录。

