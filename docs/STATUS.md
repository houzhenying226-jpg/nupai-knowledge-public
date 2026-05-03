# NuPai 系统状态快照
更新时间：2026-05-03T14:02:10Z（OpenClaw status-snapshot cron）

## 生产服务状态
❌ CRM 39.106.83.79: timed out
❌ 门店 39.96.216.33: HTTP Error 404: Not Found
❌ Dify 123.57.224.35: HTTP Error 404: Not Found

## 任务台账摘要（过去 24h）
| 任务名 | 状态 | 最近日志 |
|--------|------|----------|
| crm-patrol | 未知 |  |
| pos-patrol | 未知 |  |
| daily-health | 未知 |  |
| weekly-memory | 未知 |  |
| weekly-report | 未知 |  |
| feishu-wiki-sync | 正常 | 2026-05-03T13:36:00Z 同步 9 文件 |
| task-pulse | 正常 | 2026-05-03T14:00:01Z ok=13 issues=0 |
| status-snapshot | 正常 | 2026-05-03T14:00:01Z STATUS.md 已更新 |
| wiki-lint | 异常 | 2026-05-02T19:00:00Z gateway 失败: [Errno  |
| openclaw-gateway | 未知 |  |
| webhook-server | 未知 |  |
| watchdog | 未知 |  |
| feishu-ws | 未知 |  |

## 开放 PR
**houzhenying226-jpg/nupai-crm**
  #450 fix(db): NUP-331/332/333/359/360 rollback cancelle @houzhenying226-jpg
  #439 fix(ops): task_pulse.py use absolute path for open @houzhenying226-jpg
  #420 fix(ai): NUP-415 suppress cancelled chat SSE noise @houzhenying226-jpg
  #419 fix(ai): NUP-414 handle cancelled chat SSE session @houzhenying226-jpg
  #418 fix(bi): NUP-413 修复 WF13 月报 period 入参 @houzhenying226-jpg
  #417 fix(battle-card): NUP-411 guard null network score @houzhenying226-jpg
  #416 fix(battle-card): NUP-412 avoid missing account de @houzhenying226-jpg
  #395 fix(auth): NUP-346 clear legacy nupai_token on log @houzhenying226-jpg
  #388 fix(k9): NUP-387 harden Sentry issue handler @houzhenying226-jpg
  #332 fix: NUP-331 suppress WeCom IP whitelist Sentry no @houzhenying226-jpg
  #330 fix(dispatcher): throttle SIGTERM alert (sigterm k @houzhenying226-jpg
  #329 fix(cortex): NUP-324 harden morning report ID hand @houzhenying226-jpg
  #328 fix(cortex): NUP-320 harden morning report queries @houzhenying226-jpg
  #327 fix(cortex): NUP-322 isolate morning report remind @houzhenying226-jpg
  #326 fix(cortex): NUP-323 recover aborted scanner trans @houzhenying226-jpg
  #325 fix(cortex): NUP-321 isolate morning report remind @houzhenying226-jpg
  #318 fix(visits): NUP-317 pre-briefing sentry crash @houzhenying226-jpg
  #316 fix(api): NUP-315 修复钉钉 webhook HMAC 验签 @houzhenying226-jpg
  #314 fix(workflow): NUP-313 harden sentry auto review r @houzhenying226-jpg
  #285 fix: resolve crm_knowledge_cards table name mismat @houzhenying226-jpg
  #282 fix(ci): ruff exclude non-Python files @houzhenying226-jpg
  #258 fix(deploy): deploy.sh failure path 全面加固 @houzhenying226-jpg
  #195 test: trigger CodeRabbit @houzhenying226-jpg
  #194 ci: test PR Gate trigger @houzhenying226-jpg
  #134 infra(ci): PR-B 基建修复 — ruff + mypy + pytest + CI w @houzhenying226-jpg
  #133 chore(docs): PR-A 铁律写入 + 71个Technical Specs @houzhenying226-jpg
  #131 chore(validation): NUP-219 restore verification ba @houzhenying226-jpg
  #110 fix: config.py补充Dify WF字段定义 @houzhenying226-jpg
  #70 fix: [Sentry #69] [Sentry] TypeError: Cannot read  @app/github-actions
  #67 fix: [Sentry #65] [Sentry] MissingGreenlet error i @app/github-actions
**houzhenying226-jpg/-nupai-store**
  #87 fix: STO-202 (OpenClaw auto-commit, task=e39c23b7- @houzhenying226-jpg
  #86 fix(modelops): NUP-394 修复成本报告聚合类型 @houzhenying226-jpg
  #74 fix(ci): ruff exclude non-Python files @houzhenying226-jpg
  #73 feat(ops): STO-165 WeworkMsg 注册 systemd 服务 + 健康检查 @houzhenying226-jpg
  #71 docs(ops): handoff-2026-04-20 追加晚间真修记录（STO-167 闭环） @houzhenying226-jpg
  #54 docs: STO-148/149/160 蓝图回写v1.8.0 + 审计基线 @houzhenying226-jpg
  #42 feat(store): STO-159 门店详情员工列表增加最后登录时间列 @houzhenying226-jpg

## 知识库条目数
- 架构决策：26
- 故障复盘：27
- 操作规程：27

> 详细知识：读 docs/index.md 获取目录
> 历史快照：docs/snapshots/{YYYY-MM-DD}.md（保留 90 天）
