# doctor砍tools.allow导致agent全挂

> 类型：故障复盘 | 项目：openclaw | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-05T14:00:01Z

## 摘要

OpenClaw 启动失败 + 飞书反复 abort 时,**第一件查 tools.allow 完整性**,不是查飞书入口。Embedded agent 起不来会拖死整个 gateway。doctor 类自动修复工具真账上有可能砍配置导致连锁失败。

## 正文

> 记录日期：2026-05-05  风险等级：L4

## 背景
2026-05-05 02:00-02:42 OpenClaw 飞书完全失联 40+ 分钟。表象:gateway 启动 7 plugin 含 feishu,但 feishu WebSocket 连上后 4 分钟内必 abort signal 被 SIGTERM。最初误判为飞书后台没勾"长连接"或 watchdog 误杀,排查偏向飞书入口和 launchctl plist。

## 内容
真根因 — `~/.openclaw/openclaw.json` 里 `tools.allow` 被 doctor 工具砍剩 ["exec", "message"],缺 write/edit/read/apply_patch/process/sessions_*/subagents 等 13 项。

真账行为链:
1. agent 真账 lane(main / crm-dev / pos-dev / security)启动时报 "No callable tools remain after resolving explicit tool allowlist (tools.allow: exec, message; sandbox tools.allow: ...); no registered tools matched"
2. embedded_run_failover_decision 真账 surface_error → 模型 fallback 也失败
3. lane task 反复 surface_error → gateway 整体被 SIGTERM 杀掉
4. feishu plugin 是 gateway 子组件,gateway 死它跟着 abort
5. launchd 自动拉起新 gateway → 4 分钟后 agent 又起不来 → 又死,死循环

排查偏向真账(踩坑):
- 误以为 feishu WebSocket 不通 → 排查飞书后台长连接(其实长连接已勾)
- 误以为 watchdog 误杀 → 查 watchdog.log 全 OK
- 误以为 launchctl plist 路径不对 → bootout/bootstrap 反复折腾
- 真根因日志在 `embedded_run_failover_decision` 和 `[tools] No callable tools remain` 两行,但被淹没在 SIGTERM / abort signal 假象里

修复 SOP:
1. python3 写 `tools.allow` 完整 15 项:
   ['exec','message','write','edit','read','apply_patch','image','process','sessions_list','sessions_history','sessions_send','sessions_spawn','sessions_yield','subagents','session_status']
2. launchctl kickstart -k gui/$(id -u)/ai.openclaw.gateway
3. 等 health-monitor 5 分钟超时拉起 feishu(不能急着 kickstart,新进程需要时间稳)
4. 真账核 grep -iE "feishu.*websocket|callable tools" /tmp/openclaw/openclaw-*.log | tail -20
5. 4 分钟内不再出现 "callable tools remain" + 出现 "WebSocket client started" → 真活

## 结论
OpenClaw 启动失败 + 飞书反复 abort 时,**第一件查 tools.allow 完整性**,不是查飞书入口。Embedded agent 起不来会拖死整个 gateway。doctor 类自动修复工具真账上有可能砍配置导致连锁失败。

## 影响
- ~/.openclaw/openclaw.json tools.allow 字段
- 任何 doctor / dashboard / 自动诊断工具运行后必须真账核 tools.allow
- gateway 启动失败诊断顺序:embedded agent → tools.allow → plugins → channels(不要倒序)

## 下次注意
- 任何 OpenClaw 启动失败 / channel 反复 abort → 第一行 grep "No callable tools remain" 看 tools.allow 真账
- 任何 doctor 类工具跑完必备份 + diff openclaw.json,看砍了什么
- abort signal received 不是真根因,是 SIGTERM 的下游表现,要查上游谁发的 SIGTERM(系统 / launchd / 还是 lane error 触发的进程退出)
- doctor 已知行为:可能砍 tools.allow / disable skills / 删 plugins.installs。每跑必 diff
- Embedded agent 启动失败 → 整个 gateway 被拉下水 → 所有 channel 跟着死。这是 OpenClaw 架构特性,不是 bug

