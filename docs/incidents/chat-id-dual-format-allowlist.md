# chat_id 双格式白名单修法

> 类型：故障复盘 | 项目：crm | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T03:41:20Z

## 摘要

飞书 chat_id 有 ou_xxx（用户）和 oc_xxx（群聊）两种格式，白名单只配一种导致另一种被静默拒绝。修法：allowFrom 同时包含两种格式。

## 正文

# chat_id 双格式白名单修法

## ID 格式
- ou_xxx：用户 open_id（DM 场景）
- oc_xxx：群聊 chat_id（群消息场景）

## 修法
在 openclaw.json channels.feishu.allowFrom 同时加入两种 ID:
```json
{"allowFrom": ["ou_振英个人ID", "oc_运维群ID"]}
```

## 获取群 chat_id
飞书开放平台 → 事件订阅 → 查看消息原始内容中的 chat_id 字段

## 下次注意
新增接入场景先确认 ID 类型，变更白名单后测试 DM + 群聊两路径
