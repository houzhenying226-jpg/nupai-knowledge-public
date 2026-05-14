# Pydantic model_fields_set 区分"未传"与"传 null"

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-14T18:22:07Z

## 摘要

更新接口中 Optional 字段需要支持"清除"语义时的正确做法

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `feedback_pydantic_model_fields_set.md`，本次按侯爷要求同步到飞书知识库。

## 内容
## 问题场景（NUP-429）

`PUT /admin/users/{id}` 的 `manager_id` 字段：
- 客户端不传 → 不修改上级（保持原值）
- 客户端传 `null` → 清除上级关系（设为 None）
- 客户端传数字 → 设置新上级

如果直接用 `if req.manager_id is not None`，无法区分"未传"与"传 null"。

## 正确做法：model_fields_set

```python
class UserUpdateRequest(BaseModel):
    name: Optional[str] = None
    manager_id: Optional[int] = None  # None = 清除，不传 = 不动

# 在 update_user 中：
if "manager_id" in req.model_fields_set:
    # 客户端明确传了这个字段（无论是数字还是 null）
    if req.manager_id is not None:
        # 校验 manager 存在
        manager = await db.get(SysUser, req.manager_id)
        if not manager:
            raise HTTPException(400, detail="上级用户不存在")
        # 防止自引用
        if req.manager_id == user_id:
            raise HTTPException(400, detail="不能将自身设为上级")
    user.manager_id = req.manager_id  # 可以是 None（清除）
# 否则不动 manager_id
```

## 铁律

凡是 PATCH/PUT 接口中存在"支持清除"语义的 Optional 字段，
必须用 `model_fields_set` 判断而不是 `if value is not None`。

适用场景：manager_id、team_id、任何外键关联字段的"取消关联"。

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

