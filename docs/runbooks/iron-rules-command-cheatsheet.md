# 铁律命令速查表

> 类型：操作规程 | 项目：crm | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T03:41:20Z

## 摘要

开发铁律速查：禁止命令清单及替代方案、数据库操作规范、Linear 前缀规则、Codex 分工规则。

## 正文

# 铁律命令速查表

## 绝对禁止
- git push origin main（走 PR）
- git push --force（用 --force-with-lease）
- docker build --no-cache（占磁盘）
- alembic upgrade head 未审查
- python init_db.py（用 alembic）
- DROP TABLE / TRUNCATE（迁移文件+备份）

## 部署
```bash
# 触发 CI/CD
gh pr merge {PR号} --squash
# 验证
curl http://39.106.83.79:8001/health
```

## Linear 铁律
- NUP- = CRM，STO- = 门店
- commit 格式：type(scope): NUP-N 描述
- 建错 Team → 删除重建

## Codex 分工
- ≤3 文件 → 自己实现
- >3 文件 → /codex:rescue
- 前后端同时改 → 两个 Codex 并行
