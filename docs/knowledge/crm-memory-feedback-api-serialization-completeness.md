# 后端序列化函数必须输出前端用到的所有字段

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-20T14:00:00Z

## 摘要

_user_to_dict 漏了 manager_id，前端「上级」列永远为空的教训

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `feedback_api_serialization_completeness.md`，本次按侯爷要求同步到飞书知识库。

## 内容
## 根因（NUP-429）

前端「上级」列用 `record.manager_id` 做 id→name 映射。
后端 `_user_to_dict()` 没有输出 `manager_id` 字段。
结果：数据库里有值，API 返回的 JSON 里没有，前端列永远显示 `-`。

## 铁律

**新增前端列/字段 → 必须同时检查后端序列化函数是否输出该字段。**

```python
# ❌ 漏掉 manager_id
def _user_to_dict(u: SysUser) -> dict:
    return {"id": u.id, "name": u.name, "role": u.role, ...}

# ✅ 正确：前端用什么，后端就输出什么
def _user_to_dict(u: SysUser) -> dict:
    return {
        "id": u.id,
        "name": u.name,
        "phone": u.phone,
        "role": u.role,
        "manager_id": u.manager_id,   # 前端上级列需要
        "team_id": u.team_id,
        "is_active": u.is_active,
        "last_login_at": str(u.last_login_at) if u.last_login_at else None,
        "created_at": str(u.created_at),
    }
```

## 检查方法

前端新增字段时，立刻 grep 后端序列化函数：
```bash
grep -n "manager_id\|新字段名" backend/app/api/v1/admin.py
```
确认序列化 dict 里有该字段才算完成。

## 适用范围

所有手写序列化函数（`_user_to_dict`、`_account_to_dict`、`to_dict()` 方法等）。
如果用 Pydantic response_model 自动序列化，则不受此影响（model 字段 = 输出字段）。

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

