# nupai-oc-router架构与三工具分工

> 类型：架构决策 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-05T14:00:01Z

## 摘要

架构定型：所有通用派工/确认/进度走 GPT-5.5 \+ 工具自主决策；只把生产部署留在 before_dispatch 强制审批。任何修改这套架构前必须先读上面 5 个真实文件，不靠记忆。架构图见复盘文档第 2 节 mermaid。

## 正文

> 记录日期：2026-05-05  风险等级：L3

## 背景
飞书自然语言（"确认" / "汇报进度" / "派 worker 修 STO-186"）需要走 GPT-5.5 理解意图后派发，但 OpenClaw 原先只稳定识别固定 /oc ... 指令。同时部署类自然语言不能让 LLM 直接执行，必须走审批卡片。

## 内容
nupai-oc-router 插件架构通过 api.registerTool\(\) 暴露 3 个工具给 GPT-5.5 自主决策，部署单独留在 before_dispatch 走审批。3 个工具职责与触发：   工具名 触发词 参数 底层动作    dispatch_worker "派 worker / 修 issue / 帮我修 STO-xxx" issue_id（必填）\+ repo（可选） POST dispatcher /command verb=fix  confirm_task_plan "确认 / 好 / ok / 全部派 / 都派了" plan_id（可空，自动从 session transcript 和 pending-task-plans.json 找最近 taskplan-*） POST dispatcher /command verb=TASK_CONFIRM  query_task_status "进度 / worker 怎么样 / 完成了吗 / 汇报一下" 无 执行 openclaw tasks list --jsonbefore_dispatch 当前只保留部署审批：识别"部署 / 上线 / 发布 / deploy / release / 上生产 / 推到 prod"等部署词，能推断 env\+target\+ref 时转为 /oc deploy plan \{env\} \{target\} \{ref\}，否则让 dispatcher/LLM 继续澄清；不会直接部署生产。关键文件路径：\~/.openclaw/openclaw.json（主配置）\~/.openclaw/extensions/nupai-oc-router/index.mjs（工具实现）\~/.openclaw/extensions/nupai-oc-router/openclaw.plugin.json（插件清单）\~/.openclaw/.learnings/[ERRORS.md](ERRORS.md) / [LEARNINGS.md](LEARNINGS.md)（历史踩坑）\~/.openclaw/workspace-shared/[MEMORY.md](MEMORY.md)（共享记忆）dispatcher endpoints：/command（POST，verb=fix / TASK_CONFIRM / deploy）端口 18789（gateway） \+ 18790（dispatcher）

## 结论
架构定型：所有通用派工/确认/进度走 GPT-5.5 \+ 工具自主决策；只把生产部署留在 before_dispatch 强制审批。任何修改这套架构前必须先读上面 5 个真实文件，不靠记忆。架构图见复盘文档第 2 节 mermaid。

## 影响
所有飞书自然语言路由、所有 dispatcher 调用入口、所有插件工具开发（新增工具必须走 5 处配置同步规则，见独立卡）、部署审批流程、/oc 命令别名行为。

## 下次注意
新增工具优先走 api.registerTool 不要碰 before_dispatch；除非是部署/生产红线类，必须 LLM 不能自主决策；接手前必读 5 个真实文件路径；不要把自然语言能力退回到 before_dispatch 关键词堆砌。

