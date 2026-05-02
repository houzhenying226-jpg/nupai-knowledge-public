# CRM MySQL数据库

> 类型：操作规程 | 项目：nupai-crm | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:06:02Z

## 摘要

立即配 mysqldump 每日 3:00 cron + 异地复制到职业装1（参考 Store deploy/backup.sh 改 mysqldump 即可）。库名不改（迁移成本高），但新项目必须遵守命名规范。

## 正文

> 记录日期：2026-04-17  风险等级：L4

## 背景
CRM 主库是 MySQL 8，所有客户/订单数据落地。当前最大风险：❌ 没有任何自动备份（基础设施清单 §15 P0）。

## 内容
引擎 MySQL 8.0.45，容器 nupai-mysql，端口 127.0.0.1:3306→3306。库名 nupaidb（历史遗留命名，新项目应按 nupai{项目} 规范）。迁移工具 alembic。数据目录本地挂载 /opt/nupai-crm/data/mysql。开发机连接：Mac Mini 走 SSH 隧道 localhost:3307 → CRM 3306。MCP 包用 mcp-server-mysql。 ❌ 备份现状：/opt/backup/ 只有两个旧代码备份不是数据库备份。MySQL 数据丢失=全部客户数据丢失。 ⚠️ 迁移规范偏差：库名 nupaidb 是历史遗留，新数据库必须按 nupai{项目} 命名（基础设施清单 §16）。

## 结论
立即配 mysqldump 每日 3:00 cron + 异地复制到职业装1（参考 Store deploy/backup.sh 改 mysqldump 即可）。库名不改（迁移成本高），但新项目必须遵守命名规范。

## 影响
全部 CRM 客户/订单/合同数据安全底线；alembic 迁移；Claude Code MCP；新增表/字段流程；灾备恢复；服务器迁移路径。

## 下次注意
mysqldump cron 必须立即配（P0）；新项目库名遵循 nupai_{项目} 不要再用 nupaidb 这种；改库走 alembic；连库前确认引擎是 MySQL；备份脚本上线后做一次还原演练。

