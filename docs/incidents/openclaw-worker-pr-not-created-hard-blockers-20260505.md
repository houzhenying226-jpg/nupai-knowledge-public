# worker 不开 PR 的三类硬阻塞归因

> 类型：故障复盘 | 项目：openclaw | 风险：L1 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-05T16:00:00Z

## 摘要

worker 不开 PR 的主因不是执行脑没干，而是 GitHub DNS、worktree 并发锁、D3/生产确认点三类硬阻塞。

## 正文

> 记录日期：2026-05-05  风险等级：L1

## 背景
用户追问为什么执行脑不开 PR。日志显示部分任务已本地修复，但 PR 未更新；部分旧 worker 未进入实现；部分任务停在确认点。

## 内容
三类硬阻塞：
1. GitHub 网络/DNS：日志原文 `Could not resolve host: github.com`，导致 push/PR create 失败。
2. worktree 并发锁：STO-175、STO-183 首轮 `worktree_add_failed`。
3. D3/生产确认点：部分 worker 因生产、安全组、恢复演练边界等待确认，后来才补 repo-only 规则。

## 结论
“不出 PR”不能只判 worker 懒或 GPT 没执行。必须看 worker log 的最后有效信号，区分本地已修但无法 push、worker 未进入实现、需要确认、PR 已推分支但未创建 URL。

## 影响
汇报 PR 阻塞必须用表格列出 Issue、cmd_id、状态、阻塞原因，不能用文字墙堆日志。

## 下次注意
repo-only 可自动继续；生产、安全组、恢复演练必须明确授权。GitHub 网络问题恢复前，本地补丁无法进入 PR。

