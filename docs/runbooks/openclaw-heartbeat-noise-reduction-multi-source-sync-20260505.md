# OpenClaw heartbeat 降噪配置必须多处同步

> 类型：操作规程 | 项目：openclaw | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-05T16:00:00Z

## 摘要

心跳降噪不能只改 AGENTS.md，还必须同步 HEARTBEAT.md、当前 sandbox、runtime config，并验证 gateway reload。

## 正文

> 记录日期：2026-05-05  风险等级：L2

## 背景
用户要求 worker 状态汇报必须改成 Markdown 表格。初始只改 AGENTS.md 不足以覆盖运行时，后来发现 heartbeat prompt、sandbox 文件、showOk 配置也会影响飞书表现。

## 内容
必须同步的位置：
1. `~/.openclaw/workspace/AGENTS.md`
2. `~/.openclaw/workspace/HEARTBEAT.md`
3. 当前 agent sandbox 的 AGENTS.md / HEARTBEAT.md
4. `agents.defaults.heartbeat.prompt`
5. `channels.defaults.heartbeat.showOk=false`
6. gateway hot reload 或重启后的日志验证。

## 结论
AGENTS.md 是上下文约束，不是唯一运行时配置。飞书心跳体验由 workspace 文档、sandbox 文档、openclaw config、gateway reload、session 上下文共同决定。

## 影响
以后修改 OpenClaw 行为规则，必须说明改了哪一层，并用配置读取、gateway log、真实 session 输出三类证据验证。

## 下次注意
不要把 HEARTBEAT_OK 主动发飞书；无任务可静默，有任务不可无限静默。

