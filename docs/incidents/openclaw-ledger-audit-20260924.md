# OpenClaw 台账核查与修复（2026-09-24）

背景：「📊 NuPai 日报 2026-09-23」（status-snapshot cron 22:00 生成）出现 4 处疑点：
门店/Dify 生产服务报 ❌、两条 CRM 部署通知（⚠️ 真实登录 UI 验证失败 / ❌ 部署失败）。
本文为 2026-09-24 逐项核查结论与处置记录。

## 1. 生产服务三行

| 项 | 日报 | 实际判定 | 处置 |
|---|---|---|---|
| CRM 39.106.83.79 | ✅ | 真实正常 | — |
| 门店 39.96.216.33 | ❌ | **假警报**：检查地址指到不存在的路径（80 端口 /health），自建立起从未对过 | 已修正检查地址 |
| Dify 123.57.224.35 | ❌ | **假警报**：同上 | 已修正检查地址 |

- 门店：原检查 `http://39.96.216.33/health` 返回 404；实测健康端点 `https://store.fesuntailor.net/api/v1/health` → `{"status":"ok","db":"ok","redis":"ok",...}`。
- Dify：原检查 `http://123.57.224.35/health` 返回 404；`http://123.57.224.35/signin` 返回 200（服务正常）。
- 修正：`~/.openclaw/scripts/status_snapshot.py` 的 SERVERS 两项地址（同目录 `.bak.20260924-*` 备份）；改后 `--dry-run` 实测三行全 ✅。2026-09-24 22:00 起日报恢复正常。

## 2. 两条 CRM 部署通知

两条文案逐字出自 nupai-crm 仓库 `.github/workflows/deploy.yml` 的飞书通知段（部署流水线推送，非服务本体故障指示）。

- **「❌ OpenClaw CRM 部署失败」**：最近实例 09-20 / 09-21 / 09-22 / 09-23 各一次。根因（以 09-23 为例）：**部署触发早于镜像构建完成**——07:28 触发部署、等目标镜像 15 分钟超时判失败；而该镜像 07:26 才启动构建、08:24 才完成；08:26 的下一轮部署成功补上该版本。**非代码/服务器故障，无停机，生产未受影响。**
- **最新部署（09-24 02:39 +08）成功**：版本 0e2750497；线上容器（fastapi / cortex / nextjs / visit-thumb）均运行该版本（已核对）。
- **「⚠️ 部署健康检查通过，但真实登录 UI 验证失败」**：对应每日登录冒烟（E2E Login Smoke）于 09-24 09:38 失败一次——当时 HTTPS 证书刚过期（浏览器拦截）。证书修复后，09-24 11:48 手动复跑 → **成功**（run 35952969960，登录进 /today，构建 sha 与线上一致）。
- 建议（OpenClaw 侧）：部署触发增加「等待镜像就绪」重试，替代固定 15 分钟超时。

## 3. 任务台账 9 行逐项

| 任务 | 日报 | 实际 | 处置 |
|---|---|---|---|
| feishu-wiki-sync | 异常 | 真故障（两级），详见 §4 | 已修复 + 推送恢复 |
| task-pulse | 异常 | 已主动停用（crontab 标注 DISABLED — noise）；台账行为 6/4 存量 | 无需处理 |
| status-snapshot | 正常 | 正常（每日 22:00 更新） | — |
| wiki-lint | 异常 | 真失败：`openclaw infer` 因 OpenClaw 主配置含未识别字段（migrations / agentRuntime 等）拒跑 | **OpenClaw 侧自查** |
| openclaw-gateway / webhook-server / watchdog / feishu-ws / nl-hash-guard | 混合 | **陈旧记录**：该 5 行由 task_pulse.py 更新，其停用后定格于 6/4；服务本体均在运行（gateway / dispatcher / watchdog / nl-hash-guard 今日仍有活动） | 台账刷新机制待 OpenClaw 评估 |

## 4. 知识库同步链修复（feishu-wiki-sync）

三级故障逐层暴露并处置：

1. **程序缺失**：脚本调用 gitleaks 时报 FileNotFoundError（cron 环境 PATH 找不到 Homebrew 程序），长期崩溃、推送从未执行。
   → 修复：脚本内改为绝对路径 `/opt/homebrew/bin/gitleaks`（同目录 `.bak.20260924-*` 备份）。
2. **安全拦截（真实保护）**：gitleaks 恢复后扫出 76 处疑似凭据，全部位于 `.claude/worktrees/sleepy-jones-842be6/`（6 月残留的旧会话工作目录）中的 2 个文件：
   - `.claude/bash-commands.log`（54 处）
   - `_fesun-deploy-configs/SSOT审查-Codex.md`（22 处）
   该目录不在 git 跟踪内（push 内容实际不含它），但扫描范围包含工作区的未跟踪文件，导致自 7 月起全部知识库推送被拦（远端 origin/main 停在 07-01）。
   → 处置：两文件移出至隔离区 `/Users/james/.openclaw/quarantine/20260924/`（原样保留、可回溯）；全仓复扫 rc=0，将推送内容预扫 rc=0。
3. **推送恢复**：积压 93 个提交（84 个每日快照 + 9 个周巡检）全部推送成功（d62b3e9 → 4ad14a4 → 8c6bef1 → be9d997）；脚本全链路复跑 rc=0（`feishu-wiki-sync done: 68 file(s)`）。
   注：12:00 自动轮次的推送曾受环境瞬态影响未完成，已即时补推；后续轮次持续观察。

## 5. 其他发现

- CRM 服务器磁盘 92%（87G/99G，余 7.6G）；共 68 个 nupai 镜像 tag（≈23.8GB）。本次仅执行安全清理（dangling 回收 0B）。建议后续制定镜像保留策略。
- `.claude/worktrees/sleepy-jones-842be6` 为已登记 worktree（分支 claude/sleepy-jones-842be6，6 月起无活动）；本次未删除目录（仅移出 2 个敏感文件）。建议由对应系统评估是否清退。

## 附：本次改动清单

| 文件/对象 | 动作 | 备份/隔离位置 |
|---|---|---|
| ~/.openclaw/scripts/status_snapshot.py | 门店/Dify 检查地址 2 处 | 同目录 `.bak.20260924-*` |
| ~/.openclaw/scripts/feishu_wiki_sync.py | gitleaks 改绝对路径 | 同目录 `.bak.20260924-*` |
| 知识库仓（nupai-knowledge） | 93 提交补推 + 修复后复跑 | — |
| 疑似凭据残留（2 文件） | 隔离 | /Users/james/.openclaw/quarantine/20260924/ |
