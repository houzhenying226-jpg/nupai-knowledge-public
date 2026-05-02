# Wiki真源迁移到Bitable

> 类型：架构决策 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T07:06:06Z

## 摘要

v1.1 方案 4.4 节"Wiki 是正文真源"修订为"Bitable 表 A 是正文真源"。Wiki 角色降级为人类阅读层，可空着不维护。C1 feishu-wiki-sync 改为从表 A「正文」字段读 → push GitHub。knowledge-writer skill 写入表 A 时「正文」字段必填。

## 正文

> 记录日期：2026-04-30  风险等级：L3

## 背景
v1.1 方案 4.4 节原本设计"飞书 Wiki 是正文真源"，C1 cron 同步 Wiki Markdown 到 GitHub。批 2 落地时发现飞书 Wiki 空间不能直接给应用授权，逐篇文档授权违反自动化原则。

## 内容
评估三条路径：路 1 每篇 Wiki 文档手动加应用（违反自动化）；路 2 把"正文"字段加到 Bitable 表 A，作为新真源；路 3 用 user_access_token 而非 tenant_access_token（需 OAuth 用户登录态，复杂度高）。最终选路 2。表 A 加「正文」字段（fldOkKZico，80KB 上限），>80KB 自动分 part 文件，不用 Gist/附件。

## 结论
v1.1 方案 4.4 节"Wiki 是正文真源"修订为"Bitable 表 A 是正文真源"。Wiki 角色降级为人类阅读层，可空着不维护。C1 feishu-wiki-sync 改为从表 A「正文」字段读 → push GitHub。knowledge-writer skill 写入表 A 时「正文」字段必填。

## 影响
v1.1 → v1.2 修订：所有提到"Wiki 是真源"的地方都要更新；C1 cron 实现路径变更；knowledge-writer skill 协议变更；用户体验"知识库在哪看"答案变了（飞书表 A 而非 Wiki）。

## 下次注意
未来设计涉及"飞书写回"的能力时，先确认目标载体（Wiki / Bitable / 文档）的应用授权能力是否支持自动化，再写方案。Wiki 不适合作为应用自动写入的目标，Bitable / 独立云文档是更好选择。

