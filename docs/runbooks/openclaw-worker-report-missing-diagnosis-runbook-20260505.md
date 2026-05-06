# OpenClaw worker 汇报缺失四步诊断法

> 类型：操作规程 | 项目：openclaw | 风险：L1 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-05T16:00:00Z

## 摘要

飞书没有收到 worker 进度时，必须按 heartbeat、worker log、planTool、heartbeat config 四层排查。

## 正文

> 记录日期：2026-05-05  风险等级：L1

## 背景
2026-05-05，用户发现飞书没有收到 worker 进度汇报，要求只读日志，不改配置，判断是 heartbeat 没触发、worker 没回调 gateway、飞书 WebSocket 断了，还是 planTool 注册但 GPT-5.5 没调用。

## 内容
标准诊断四步：
1. 查 gateway heartbeat/planTool/update_plan/notifyOnExit 日志。
2. 查最新 worker log，看 worker 和 gateway 之间有没有进度回调。
3. grep gateway.log 中 update_plan / planTool 调用。
4. 查 `agents.defaults.heartbeat` 和 `channels.feishu.heartbeat` 配置是否生效。

## 结论
飞书 worker 汇报缺失不能直接猜“飞书坏了”。必须先区分：调度没跑、worker 没回调、GPT 没调用 planTool、Feishu WS 断链、还是独立脚本绕过 GPT。

## 影响
以后所有 worker 汇报问题，先跑四步只读诊断，再决定改配置或脚本。禁止先改配置后找证据。

## 下次注意
日志判断必须贴真实输出；空 grep 也是证据。Feishu WS alive 不代表 worker 进度链路正常。

