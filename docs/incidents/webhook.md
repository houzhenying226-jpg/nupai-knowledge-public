# 飞书webhook命名差异坑

> 类型：故障复盘 | 项目：openclaw | 风险：L1 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T07:06:06Z

## 摘要

确认 FEISHU_WEBHOOK = 心跳 webhook，跳过 Step 5（不动它），继续后续步骤。新增的 FEISHU_RELEASE_WEBHOOK 是版本发布通知专用，跟心跳是两个独立 secret。

## 正文

> 记录日期：2026-05-02  风险等级：L1

## 背景
v2.1 版本治理方案 Step 5 写"确认 FEISHU_HEARTBEAT_WEBHOOK secret 仍有效"，CC 实施时 gh secret list 没找到此 secret，只有 FEISHU_WEBHOOK。

## 内容
方案文档命名错误：实际历史上振英用的心跳 webhook secret 名就是 FEISHU_WEBHOOK（创建于 2026-04-10），不叫 FEISHU_HEARTBEAT_WEBHOOK。deploy.yml 里飞书通知用的也是 FEISHU_WEBHOOK。CC 自查发现命名差异，没盲目按方案文档去新建 secret，而是先问振英确认。

## 结论
确认 FEISHU_WEBHOOK = 心跳 webhook，跳过 Step 5（不动它），继续后续步骤。新增的 FEISHU_RELEASE_WEBHOOK 是版本发布通知专用，跟心跳是两个独立 secret。

## 影响
v2.1 方案文档下次修订时纠正命名。NuPai 系统中飞书相关 secret 命名规则：FEISHU_WEBHOOK（心跳）、FEISHU_RELEASE_WEBHOOK（版本发布）、未来可能的 FEISHU_ALERT_WEBHOOK（告警）。

## 下次注意
方案文档中的资源名（secret/文件/服务）可能与现状有出入。CC 实施时遇到命名差异，正确做法是先 gh secret list 确认现状再问振英，而不是按方案文档盲建。任何方案落地都要"现状核对"步骤。

