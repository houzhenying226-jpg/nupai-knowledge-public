# brain 抢路由真根因

> 类型：架构决策 | 项目：openclaw | 风险：L4 | 状态：已审核
> 同步自 Feishu Bitable 表 A | 2026-04-30T03:41:20Z

Linear: NUP-284

## 摘要

brain.py 在 before_dispatch 钩子触发之前抢占路由，导致路由器优先级混乱。修复：before_dispatch 内加 brain 优先级检查，仅明确匹配才接管，其余走正常调度链路。教训：钩子执行顺序决定 brain 抢占窗口，任何新 skill 必须在 before_dispatch 后注册。

## 正文

（测试完成，已清空）
