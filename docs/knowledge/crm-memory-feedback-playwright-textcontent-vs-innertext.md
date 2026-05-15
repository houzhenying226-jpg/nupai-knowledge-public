# Playwright 测试用 innerText 不用 textContent

> 类型：知识条目 | 项目：nupai-crm | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-15T05:25:34Z

## 摘要

textContent 抓取隐藏 Tab 内容导致误判的教训

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
来源 Claude CRM memory 文件 `feedback_playwright_textcontent_vs_innertext.md`，本次按侯爷要求同步到飞书知识库。

## 内容
## 根因（E6.1 情报Tab JSON检测持续失败）

Playwright 测试使用 `page.textContent('body')` 检测情报Tab是否显示原始JSON。
Ant Design 的非激活 Tab pane 用 `display:none` 隐藏，但 `textContent` 仍能读取隐藏DOM节点的文本。
AI分析Tab / AI攻关Tab 中有JSON格式的内容 → 即使情报Tab正确，测试仍然失败。

## 铁律

**检测「用户可见内容」必须用 `innerText`，不用 `textContent`：**

```javascript
// ❌ 错误：抓取所有 DOM 文本，包括 display:none 的隐藏 Tab
const body = await page.textContent('body') || '';

// ✅ 正确：只抓取 CSS 可见内容
const body = await page.evaluate(() => document.body.innerText) || '';

// ✅ 更精准：只检测当前激活 Tab 的内容
const tabContent = await page.locator('.ant-tabs-tabpane-active').textContent() || '';
```

## 已修复（PR #544）

E6 测试已改为 `page.evaluate(() => document.body.innerText)`。

## renderIntelContent 兜底规则

JSON 渲染函数的兜底逻辑**禁止使用** `JSON.stringify(parsed, null, 2)`，
因为它输出含 `{"` 的原始 JSON 格式。
正确兜底：提取所有字段值为纯文本字符串：
```typescript
// ❌ 禁止
return JSON.stringify(parsed, null, 2);

// ✅ 正确：提取值，不输出JSON语法
Object.entries(parsed).forEach(([, v]) => {
  if (v !== null && v !== undefined && typeof v !== 'object') lines.push(String(v));
  if (Array.isArray(v)) lines.push((v as string[]).join('、'));
});
return lines.join('\n');
```

## 结论
该条 CRM 记忆已整理为 NuPai 知识库条目；敏感 token / API key / IP 已在入库前脱敏。

## 影响
后续 OpenClaw / 执行脑可从飞书知识库检索这些 CRM 历史规则、踩坑与参考信息。

## 下次注意
同步外部 memory 目录时先扫描并脱敏 secret、token、API key、服务器 IP，再写入知识库。

