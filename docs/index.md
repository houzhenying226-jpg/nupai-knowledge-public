# NuPai 知识目录

知识目录由 C1 feishu-wiki-sync cron 自动维护（每 30 分钟同步飞书 Wiki）。

更新时间：2026-04-30（批 1 Gate 通过）

## 飞书多维表（NuPai-AI-底座管理台账）

| 表 | 名称 | 说明 |
|----|------|------|
| A  | 知识条目索引 | 公开可引用（table_id 见私有仓） |
| B  | 任务台账     | 公开可引用（table_id 见私有仓） |
| C  | 凭据索引     | **私有仓专属，不在此列出** |

> APP_TOKEN、table_id、访问 URL 仅存于私有仓 `docs/index.md`。
> 凭据索引（表 C）任何标识符不出现在公开仓。

## 目录结构

- `docs/decisions/` — 架构决策
- `docs/incidents/` — 故障复盘
- `docs/runbooks/` — 操作规程
- `docs/specs/` — 功能规格
- `docs/projects/nupai-crm/STATUS.md` — CRM 项目状态
- `docs/projects/nupai-store/STATUS.md` — 门店项目状态
- `docs/tasks/ledger.md` — 任务台账镜像
- `docs/snapshots/` — STATUS.md 历史快照（90 天）
- `docs/credentials/` — 凭据位置索引（私有仓）
