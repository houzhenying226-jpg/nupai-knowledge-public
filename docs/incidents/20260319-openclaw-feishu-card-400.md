# 飞书卡片消息 HTTP 400 错误

> 类型：故障复盘 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T11:00:00Z

## 摘要

OpenClaw 发送飞书消息时持续报 HTTP 400 错误，deliveryqueue 积压多条失败记录。错误日志关键条目：feishu streaming start failed: Error: Create card request failed with HTTP 400。

## 正文

## 背景

OpenClaw 发送飞书消息时持续报 HTTP 400 错误，delivery-queue 积压多条失败记录。错误日志关键条目：`feishu streaming start failed: Error: Create card request failed with HTTP 400`。

## 根因分析

OpenClaw 使用了「卡片消息」格式，但飞书机器人未开通卡片消息权限，或卡片模板版本不匹配。飞书 400 = 消息卡片格式被飞书拒绝。

## 处理方式

**Step 1**：清空积压的失败队列：`rm -f ~/.openclaw/delivery-queue/*.json`

**Step 2**：关闭卡片消息模式，改用普通文本：
```bash
openclaw config set channels.feishu.streaming false
openclaw config set channels.feishu.cardMessages false
openclaw reload
```

**Step 3**：验证：`tail -5 ~/.openclaw/logs/gateway.err.log` 确认无新 400 错误。

## 经验教训

- 飞书机器人卡片消息需要在飞书开放平台单独开通权限；默认配置只支持普通文本消息。
- 开启新通知功能前，先确认机器人应用权限是否包含该消息类型。
- delivery-queue 积压时不要重试，先清空队列再修复根因，否则重试会继续失败并污染日志。
- 此问题与 delivery.mode=announce 是不同问题，但容易混淆：400 是权限/格式问题，announce 是配置行为问题。
