# OpenClaw 23 项落地清单（Sprint-12 + 批次 1 + 批次 2）

> 类型：操作规程 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T11:00:00Z

## 摘要

20260426 凌晨 → 20260427 下午，13+ 小时连续完成 OpenClaw 基础设施全部落地。本清单是实际交付凭据，可作为未来类似部署的参考模板。

## 正文

## 背景

2026-04-26 凌晨 → 2026-04-27 下午，13+ 小时连续完成 OpenClaw 基础设施全部落地。本清单是实际交付凭据，可作为未来类似部署的参考模板。

## Sprint-12（12/12）

| 项目 | PR / 凭据 |
|---|---|
| 14 状态机 + Spec/Test Worker | nupai-crm PR #305 + dispatcher PR #27-30 |
| 三铁律 M1+M2+M3 | PR #304 |
| auto-merge + 风险分类 + 修复轮数升级 | PR #304 |
| webhook 替代 polling（代码层） | dispatcher PR #29 |
| Sentry / 预算 / skip-spec | dispatcher PR #27 #28 #30 |
| 双通道（心跳告警 + 命令机器人 DM） | PR #306 |

## 批次 1（6/6，K3 砍掉）

| 项目 | 凭据 |
|---|---|
| K1 老 PR 闭环 | PR #291 #308 |
| K2 命令机器人 DM + webhook_server.py | PR #306，port 18790 |
| K4 Mac 本地韧性 | launchd com.nupai.k2-webhook + com.nupai.k2-monitor |
| K5 蓝图回写 + 3 份审计文档 | PR #307 |
| K6 蓝图漂移检查 | launchd cron 周一 09:00 |
| K7 D3 部署审批卡片 | GitHub Deployments SoT |

## 批次 2（5/5）

| 项目 | 凭据 |
|---|---|
| K8 LLM 路由器 | PR #312 |
| K9 Sentry → OpenClaw 切换 | smee.io 中继 + HMAC 验签 |
| K10 健康检查 + 自动降级 | executor-health.json |
| K11 Linear webhook | PR #312 |
| K12 飞书群聊批量派工 | PR #312 |

## 经验教训

- 13h 时间分配：6h Sprint-12 → 4h 批次 1 → 2h 批次 2 → 1h K9 切换。越到后期每项工时越短，说明前期基础设施落地后后续集成明显加速。
- K3（阿里云迁移）被砍是最重要的减法决策，节省了 1-2h + ¥30-50/月。
- 振英全程 0 次复制粘贴 bash，仅 5 次浏览器点击（飞书审批 + GitHub 审批），验证「指挥官」角色完成转型。
