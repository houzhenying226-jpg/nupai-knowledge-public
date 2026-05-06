# before_dispatch关键词路由弃用决策

> 类型：架构决策 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-05T16:00:00Z

## 摘要

通用派工、确认、进度类自然语言全部从 before_dispatch 移除，改用 api.registerTool 暴露工具给 GPT-5.5 自主调用；before_dispatch 只保留生产部署红线。

## 正文

> 记录日期：2026-05-05  风险等级：L3

## 背景
nupai-oc-router 早期方案试图通过给 before_dispatch 不断补关键词来识别飞书自然语言意图，例如“部署 / 上线 / 发布 / 派 worker / 确认 / 汇报”等。补一轮通一种说法，换种说法又失效；“确认”还可能重新生成新计划而不是确认已有的。

## 内容
关键词路由的失败模式：
1. 同义词覆盖不全：加“派 worker”能识别，但“帮我修一下”“处理 STO-186”等表达又失效。
2. 上下文缺失：关键词分支无法利用 session transcript / 历史任务 / 待确认 checkpoint，“确认”无法定位是确认哪个 plan。
3. 多意图冲突：“确认派 worker 修 STO-186”同时含确认和派工，关键词分支按顺序匹配会错路由。
4. 不可恢复：“确认”匹配后直接走新建 plan 流程，把已有 pending plan 丢了。

根因：关键词路由是硬编码意图分类，绕开了 GPT-5.5 的语义理解和工具选择能力。

部署单独保留 before_dispatch 的原因：生产部署有不可逆风险，必须强制走审批卡片，不能依赖 LLM 判断“是不是应该部署”。LLM 可被 prompt injection 欺骗，硬编码关键词 + 审批卡片是更稳的红线。

## 结论
通用派工/确认/进度类自然语言全部从 before_dispatch 移除，改用 api.registerTool 暴露 3 个工具给 GPT-5.5 自主调用。判断标准：“这个意图错了会不会造成不可逆后果”——是则留 before_dispatch + 审批；否则走工具注册让 LLM 决策。

## 影响
nupai-oc-router index.mjs 删除大量关键词分支；新增工具不再走 before_dispatch；未来其他插件设计也按红线类硬编码、意图类工具注册的标准执行。

## 下次注意
不要再给 before_dispatch 加关键词补丁；新意图先想能否注册成工具让 LLM 决策；只有不可逆操作如部署、删数据、推 prod 才用关键词 + 审批卡片硬卡；工具 description 要写清楚。

