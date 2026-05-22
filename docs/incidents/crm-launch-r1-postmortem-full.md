# CRM 5.20 用户反馈全批次深度复盘（L1-L15，含守卫清单）

> 类型：故障复盘 | 项目：nupai-crm | 风险：L1 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-22T09:00:00Z

## 摘要

最大时间损耗来源：L1 产品方向判断失误（新建孤立页又删掉）+ L3 重复 hotfix 而不查 docker logs。最便宜预防：写用户剧本 5 分钟防 L1；docker logs --tail=200 | grep ERROR 30 秒防 L3。E2E 验证 5/5 通过闭环。

## 正文

> 记录日期：2026-05-22  风险等级：L1

## 背景
2026-05-20 收到 5 条用户反馈，引发 11 个 Issue（NUP-579~NUP-589）修复周期（2026-05-21 ~ 2026-05-22）。期间发生一次方向性错误（建孤立页又删掉）、一个 Bug 被 4 次 hotfix 才根治、多个流程细节失误。最终消耗约 8 小时无效时间。

## 内容
## 事件时间线
05-20: 5 条用户反馈（Excel 5.20）
05-21 16:13: NUP-583 建了 /activities 孤立页面（方向错误）
05-21 23:14: NUP-580 修默认排序
05-21 23:24: NUP-579 修手机端白屏
05-21 23:40: NUP-581 加联系人快捷跟进入口
05-22 00:10: NUP-582 修返回卡死
05-22 01:58: NUP-584 P0 修路由 422
05-22 02:30~10:17: NUP-587 hotfix ×4 次才根治（MySQL NULLS LAST）
05-22 11:54: NUP-588 补 account_id 索引
05-22 15:08: NUP-589 删孤立页 + 合并快记到 /visits（真意修复）
05-22 16:10: 生产真账号 E2E 验证 5/5 通过

## 15 条教训逐条根因

### L1 · 产品方向失误：新建孤立页而非改造现有页
症状：建 /activities 孤立页。销售从不进入，因工作流是"打开拜访列表"而非"活动记录列表"。
预防：任何新入口必须写用户剧本（3 步内从首页到达）；优先改造销售每天用的页面（/visits, /accounts）。

### L2 · FastAPI 路由顺序：字符串路径必须在 {id} 前声明
症状：GET /visits/activities 返回 422（"activities" 不是整数）。
修复：将 /activities 移到 /{visit_id} 之前声明。
预防：所有具名子路径（/calendar, /activities, /export）必须在 /{id} 之前。

### L3 · MySQL 方言：nulls_last() 在 MySQL < 8.0.26 不支持（hotfix ×4）
症状：部署后 500，连续 4 次 hotfix 未命中根因。
修复：order_by((col == None).asc(), col.desc()) 替代 nulls_last()。
预防：禁止 nulls_last/nulls_first/NULLS LAST；修 2 次没好 → 先看 docker logs。

### L4 · SQLAlchemy 类型标注：混合列表必须声明 list[Any]
症状：mypy 报 Argument to append incompatible type。
修复：from typing import Any; a_filters: list[Any] = []。

### L5 · 前端级联 Select：父值变更必须 reset 子 Select
症状：选客户后联系人下拉白屏。
修复：onChange 第一行 setFieldValue('contact_id', undefined)。

### L6 · 默认排序错误：列表接口默认必须 DESC
修复：order_by((col==None).asc(), col.desc(), created_at.desc())。

### L7 · 数据模型：活动记录缺 contact_id 绑定
规则：活动/跟进记录建表时必须同时支持 account_id 和 contact_id。

### L8 · SWR 缓存 + page_size=200：返回列表卡死
修复：revalidateOnFocus: false, keepPreviousData: true; page_size ≤ 50。

### L9 · 两表 schema 不同：应用层归并替代 DB UNION
数学证明：各取 page×page_size 行，内存归并切页，覆盖全局前 k 条。

### L10 · Spec 必须含"蓝图来源"+"数据校验"章节
CI spec-format check 会验证。模板必含：背景/用户剧本/API设计/数据模型/蓝图来源/数据校验/验收标准/禁止做。

### L11 · PR body 必含蓝图引用 + 7项验证清单
CI blueprint-check 验证，缺失直接打回。

### L12 · E2E 盲区：PLAYWRIGHT_PASSWORD 未注入 CI → 全部 skip
测试 skip ≠ 功能通过。无密码时 E2E job 应 fail。

### L13 · AI 工具链：Codex 无余额需降级方案
降级链：Codex → Claude Code 编码 + Gemini/DeepSeek review。

### L14 · Linear API：无 Bearer 前缀；stateId 必须 UUID
Authorization: lin_api_xxx（不加 Bearer）。stateId 先调 list_issue_statuses。

### L15 · CLAUDE.md CRM 服务器 IP 写错
CRM 生产：39.106.83.79（不是 39.96.216.33，那是 Store）。每次配置先查 MACHINE.md。

## 结论
最大时间损耗来源：L1 产品方向判断失误（新建孤立页又删掉）+ L3 重复 hotfix 而不查 docker logs。最便宜预防：写用户剧本 5 分钟防 L1；docker logs --tail=200 | grep ERROR 30 秒防 L3。E2E 验证 5/5 通过闭环。

## 影响
建立 7 项守卫检查清单覆盖 MySQL 方言禁用词/SQLAlchemy is None/FastAPI 路由顺序/级联 Select 清空/SWR 配置/mock 数据残留/Alembic downgrade 实现。每次 PR 前必须执行。PR body 含蓝图引用 + 7 项验证清单 + 用户剧本。

## 下次注意
1) 写用户剧本（3 步内从首页到达）→ 防方向错误；2) 500 修 2 次没好停来看 docker logs → MySQL client 手动执行真 SQL；3) 禁止 nulls_last/nulls_first；4) 列表接口 page_size ≤ 50 + revalidateOnFocus: false；5) 所有具名子路径在 /{id} 前声明；6) 级联 Select onChange 第一行清空子值。

