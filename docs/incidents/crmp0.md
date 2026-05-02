# CRM无数据库备份P0风险

> 类型：故障复盘 | 项目：nupai-crm | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T11:06:02Z

## 摘要

立即落地：① 复制 Store deploy/backup.sh 到 CRM，把 pg_dump 改 mysqldump（用 .env 里的 MYSQL_USER/MYSQL_PASSWORD/MYSQL_DATABASE）→ 输出到 /opt/nupai-crm/backups/。② 加 cron 每天 3:00 跑。③ 异地复制到职业装1（123.57.224.35）。④ 还原演练验证脚本能跑

## 正文

> 记录日期：2026-04-17  风险等级：L4

## 背景
2026-04-17 整理基础设施清单时审计 CRM 服务器，发现 ❌ MySQL 完全没有自动备份。/opt/backup/ 目录里只有两个旧代码备份不是数据库备份。MySQL 数据丢失=全部客户数据丢失。基础设施清单 §15 标 P0。

## 内容
CRM MySQL 容器 nupai-mysql（mysql:8.0.45），数据本地挂载 /opt/nupai-crm/data/mysql。当前没有 cron、没有 mysqldump 脚本、没有异地复制、没有任何还原路径。 对比 Store：Store 有完整每日 pg_dump + 异地复制 + 30 天日备 + 周归档永久（参考“Store每日备份+异地复制”卡）。 风险面：客户主数据、合同、订单、跟进记录、AI 分析历史全部在 MySQL，磁盘损坏/误删/勒索病毒任一发生=数据归零。

## 结论
立即落地：① 复制 Store deploy/backup.sh 到 CRM，把 pg_dump 改 mysqldump（用 .env 里的 MYSQL_USER/MYSQL_PASSWORD/MYSQL_DATABASE）→ 输出到 /opt/nupai-crm/backups/。② 加 cron 每天 3:00 跑。③ 异地复制到职业装1（123.57.224.35）。④ 还原演练验证脚本能跑通。⑤ 把 .env 加进每日异地同步。

## 影响
CRM 全部客户数据安全底线；灾备能力；服务器迁移恢复路径；保险/合规审计；新人 onboarding 流程（不能上线就能写数据）。

## 下次注意
P0 必须立刻处置不能再拖；脚本上线后做一次完整还原演练（不是只看脚本不报错就放过）；备份盘使用率定期看；新服务上线前先建好备份再投产，不允许“先上线后补备份”。

