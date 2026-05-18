# 角色权限分组必须与业务语义对齐

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-18T12:20:30Z

## 摘要

director 被错误分组到 manager 级别导致看不到全公司数据的教训

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `feedback_role_permission_scope.md`，本次按侯爷要求同步到飞书知识库。

## 内容
## 根因（NUP-419）

`backend/app/core/permissions.py` 的 `apply_data_scope()` 中，
`director`（销售总监）被错误地放入 `manager` 分组，
导致 director 只能看到下属数据，无法像 CEO 一样看到全公司数据。

## 正确分组

```python
# ✅ 正确：director 与 CEO/admin 同级，看全公司
if user.role in ("ceo", "admin", "director"):
    return query  # 全公司数据

# manager 只看下属
if user.role in ("manager",):
    sub_ids = await get_subordinate_ids(user.id, db)
    return query.where(owner_column.in_(sub_ids))

# sales 只看自己
return query.where(owner_column == user.id)
```

## 铁律

每次新增角色或修改权限时，必须同时：
1. 在 `apply_data_scope()` 确认分组正确
2. 在 `check_write_permission()` 确认写权限正确
3. 写测试验证角色分组（见 `backend/tests/test_permissions_nup419.py` 示范）

## 角色层级（NuPai CRM）

```
ceo / admin / director  →  全公司数据（apply_data_scope 直接 return）
manager                 →  本人 + 下属数据
sales / user            →  仅本人数据
```

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

