# PAT空粘贴排查与len校验

> 类型：故障复盘 | 项目：_global | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-02T07:06:06Z

## 摘要

read -s 必须加 len 校验：read -s NEW_PAT && echo "len={#NEW_PAT}"。粘贴后立即看到长度数字，fine-grained PAT 通常 93 字符左右。len=0 说明粘贴失败立即重做，不会再走到 deploy 链路才发现。重做完用 echo " NEW_PAT" | gh secret set ... 写入，跑 unset NEW_PAT 清环境变

## 正文

> 记录日期：2026-05-02  风险等级：L3

## 背景
v2.1 版本治理 PR #402 合并后，post-deploy job 失败，actions/checkout@v4 报 Input required and not supplied: token。secret OPENCLAW_BOT_PAT 之前已通过 read -s 写入。

## 内容
诊断路径：① gh secret list 显示 OPENCLAW_BOT_PAT 存在（updated_at 时间戳正确）② Actions log 里 with: 块完全没有 token: 字段（GitHub 对真实值会显示 ***，对空字符串会完全省略）③ 排除"PAT 被撤销"假设（撤销会过 input 校验后报 401，不是 not supplied）。结论是 secret 写入了空字符串。根因：read -s 模式输入不可见，用户 Cmd+V 粘贴时屏幕没反应误以为粘了，实际剪贴板未真正粘入或粘入空。

## 结论
read -s 必须加 len 校验：read -s NEW_PAT && echo "len={#NEW_PAT}"。粘贴后立即看到长度数字，fine-grained PAT 通常 93 字符左右。len=0 说明粘贴失败立即重做，不会再走到 deploy 链路才发现。重做完用 echo " NEW_PAT" | gh secret set ... 写入，跑 unset NEW_PAT 清环境变量。完整 runbook 见 docs/runbooks/pat-rotation.md。

## 影响
所有需要敏感字符串写入（PAT / secret / API key / 数据库密码）的流程，必须加长度校验或回声确认。NuPai 系统涉及：OPENCLAW_BOT_PAT rotate、飞书 App Secret rotate、Anthropic API key 配置等。

## 下次注意
铁律：任何用 read -s 接收敏感字符串的命令，必须配套 echo "len=${#VAR}" 立刻验证长度。如果忘了校验直接进生产链路，等于把"调试时间"押后到链路失败时——成本翻 10 倍。

