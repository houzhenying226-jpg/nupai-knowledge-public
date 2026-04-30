# Codex CLI ChatGPT Pro device-auth 认证 SOP

> 类型：操作规程 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T11:00:00Z

## 摘要

GPT5.5 全栈激活需要通过 ChatGPT Pro 订阅接入 Codex CLI。deviceauth 路径是 OpenAI 官方支持的合规方式，但操作有细节要求。

## 正文

## 背景

GPT-5.5 全栈激活需要通过 ChatGPT Pro 订阅接入 Codex CLI。device-auth 路径是 OpenAI 官方支持的合规方式，但操作有细节要求。

## 前提条件

- ChatGPT Pro / Business / Enterprise 订阅
- Codex CLI >= 0.125.0（`codex --version` 确认）
- 北京手机（北京 IP 扫码，避免新加坡 IP 风控）
- 新加坡 Mac 执行 CLI 命令

## 操作步骤

**Step 1**：在 ChatGPT 设置中开启 device-auth 权限
```
ChatGPT → 设置 → 数据控制 → 为 Codex 启用设备代码授权（默认关闭，必须手动打开）
```

**Step 2**：Mac 上执行登录，获取设备码
```bash
codex auth login
# 输出一个 URL + 8 位设备码
```

**Step 3**：北京手机打开 URL，扫码 / 输入设备码确认授权

**Step 4**：Mac 上验证登录成功
```bash
codex exec --model gpt-5.5 "说你好"
# 期望：短句回复，不是 5 段格式
```

## 常见问题

| 问题 | 原因 | 解决 |
|---|---|---|
| 反复杀进程重试 | 无影响，OpenAI 服务器记住授权 | 任一次扫码成功即可 |
| 新加坡 IP 扫码失败 | 风控 | 改用北京手机扫码 |
| 设备代码授权不可见 | 设置未开启 | ChatGPT 设置显式开启 |

## 经验教训

- OAuth 过期时重走此 SOP 即可，不需要其他操作。过期凭据存于 `~/.codex/auth.json`，可以直接删除后重新认证。
- 此认证对应 K8 路由器中的「Codex」执行器，认证失败时 K10 健康检查会自动降级到备选 LLM（Qwen/DeepSeek）。
