# MemoryOps v1.1 架构决策：Bitable 为正文真源

> 类型：架构决策 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T03:41:20Z

## 摘要

放弃 Feishu Wiki 真源方案（Route 1），改为 Bitable 表 A 存储完整正文（Route 2）。原因：Wiki API 权限复杂，Bitable PoC 成功，字段上限 80KB 可接受。

## 正文

# MemoryOps v1.1 架构决策：Bitable 为正文真源

## 决策
正文真源 = 飞书 Bitable 表 A「正文」字段
Wiki = 可选人类阅读层（不再是系统数据源）

## 背景
原方案 Wiki 真源在批 2 遇阻：wiki:wiki 权限需 Wiki 空间管理员额外配置，Bitable 写回 PoC 已验证成功。

## 对比
| 维度 | Route 1 Wiki | Route 2 Bitable |
|------|-------------|----------------|
| 权限复杂度 | 高 | 低 |
| PoC 状态 | 未验证 | ✅ 已验证 |
| 字段上限 | 无限制 | 80KB 安全线 |

## 影响
1. C1 从 Bitable 表 A 读「正文」渲染为 Markdown
2. knowledge-writer 只写表 A，不写 Wiki
3. 超 80KB 自动分 part（-part1/-part2 后缀）

实施日期：2026-04-30
