# Bug 修复流程复盘知识库

> 类型：故障复盘 | 项目：nupai-crm | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-24T09:00:00Z

## 摘要

覆盖 NUP-579/NUP-587/NUP-588 修复周期，沉淀测试证据、hotfix 叠加、合并流程、会话核查等 Bug 修复流程铁律。

## 正文

> 记录日期：2026-05-24  风险等级：L3

## 背景
用 mcp__playwright__browser_* 工具做 UI 测试，和 npx playwright test 跑脚本文件是两条不同路径。

CRM 系统特有知识：
```PLAIN_TEXT
// CRM 的 localStorage token 键名不是 "token"，是 "nupai_token"
const token = localStorage.getItem('nupai_token').replace(/"/g, '');

// 存储键全集（截至 2026-05-24）：
// nupai_auth → Zustand auth store（含 user 对象）
// nupai_token → JWT Bearer token（用于 API 请求）
// nupai_refresh_token → refresh token
```

标准测试步骤：
```PLAIN_TEXT
Step 1: browser_navigate → 生产 URL
Step 2: browser_take_screenshot → 确认页面状态（已登录/未登录）
Step 3: browser_run_code_unsafe → 用 localStorage.nupai_token 调 API 验证接口
Step 4: 交互操作（click/fill/select）
Step 5: browser_take_screenshot → 记录操作结果
Step 6: 检查页面是否白屏（截图中只有导航栏+空白 = 白屏）
```

白屏判断标准：
```PLAIN_TEXT
// 截图后检查页面内容
const bodyText = await page.locator('body').innerText();
const isBlankScreen = bodyText.trim().length < 50; // 内容少于50字符视为白屏
```

## 二、Hotfix 叠加反模式

### P3 · 同一个 Bug 连续修 3 次 = 根因定位没找对，必须归零重查

症状：NUP-587 /activities 500 经历了 hotfix1/2/3/4，前三次均在"周边代码"改动，真正根因行（nulls_last()）一直未被触碰。

错误路径：hotfix1 回滚列选择 → hotfix2 回滚 .order_by(None) → hotfix3 重写 count_stmt，仍 500；每次基于上次方向叠加，从未回到真实日志。

正确路径：第2次失败后 STOP → docker logs 提取真实 SQL 错误 → MySQL client 手动执行 → 确认 NULLS LAST 语法错误 → 定位 nulls_last() 根因行 → hotfix4 根治。

铁律：同一接口 hotfix 失败 2 次 → 强制执行归零程序：
1. 不再基于"上次假设"改代码
2. `docker logs nupai-fastapi --tail=500 | grep -A10 "ERROR"`
3. 提取完整 SQL 错误语句
4. 在 MySQL client 里执行该 SQL，肉眼确认错误类型
5. 只有看到真实错误后才开始 hotfix N+1

### P4 · 修复前必须复述根因，不能直接改代码

每个 hotfix 开始前，必须写出：
- 根因：具体到代码行/SQL语句
- 修复：改哪个文件第几行，改什么
- 验证：改完后执行什么命令/SQL确认修复

没有"验证"步骤的 hotfix 不允许提交。

## 三、合并流程失误

### P5 · PR 合并前必须检查所有 review thread 是否 resolved

症状：PR #726 全部 CI 通过，执行合并时返回 405 All comments must be resolved，原因是 Gemini bot 的 inline comment thread 未 resolve（虽然已回复）。

区别：回复 comment ≠ resolve thread。即使 bot 的建议性 comment 已按建议修改并回复，thread 必须手动 resolve。

合并前检查脚本：
```PLAIN_TEXT
gh api graphql -f query='
{
 repository(owner:"houzhenying226-jpg", name:"nupai-crm") {
 pullRequest(number: PR号) {
 reviewThreads(first: 20) {
 nodes { id isResolved comments(first:1){nodes{body}} }
 }
 }
 }
}' | python3 -c "..."
```

Resolve 命令：
```PLAIN_TEXT
gh api graphql -f query='
mutation {
 resolveReviewThread(input: {threadId: "PRRT_xxx"}) {
 thread { isResolved }
 }
}'
```

### P6 · CI Watcher 脚本 required checks 不能硬编码

症状：自动合并脚本硬编码 REQUIRED = {'test', 'lint', ...}，实际 CI job 名称是 lint-and-test，导致 watcher 永久等待不存在的 check。

正确做法：从 GitHub branch protection API 动态读取 required checks：
```PLAIN_TEXT
def get_required_checks(owner, repo, branch='main'):
 result = subprocess.run([
 'gh', 'api',
 f'repos/{owner}/{repo}/branches/{branch}/protection',
 '--jq', '.required_status_checks.checks[].context'
 ], capture_output=True, text=True)
 return set(result.stdout.strip().split('\n'))
```

## 四、知识库维护

### P7 · 知识库必须在当次会话内提交，不能依赖"下次提交"

知识库文件写完后，必须在同一个会话内：
```PLAIN_TEXT
git add docs/knowledge/*.md
git commit -m "docs(knowledge): ..."
git push origin {branch}
```
"等下次提交"= 永远不会提交。

验证命令：
```PLAIN_TEXT
git log origin/$(git branch --show-current) --oneline -3 | grep "knowledge"
```

### P8 · 会话切换后必须先核查上次声称"已完成"的工作

恢复会话后的核查清单：
```PLAIN_TEXT
# 1. 检查上次声称的 git 提交是否真实存在
git log --oneline -5
# 2. 检查上次声称的测试是否有 artifact
cat test-results/.last-run.json
ls test-results/ | grep -E "\.png$"
# 3. 检查上次声称的文件是否落盘
ls docs/knowledge/
# 4. 检查上次声称的 PR 是否真实合并
gh pr list --state merged --json number,title --limit 5
```

原则：摘要是 AI 的主观声称，不是事实。每次恢复会话，用命令行验证，不信摘要。

## 五、总结：Bug 修复流程标准

正确的 Bug 修复完整流程：
1. Phase 0 根因定位：docker logs 提取真实错误、MySQL client 复现 SQL 错误、复述根因才允许进 Phase 1。
2. Phase 1 单次修复：只改根因行，容器内验证 SQL 通过，写 Spec。
3. Phase 2 测试验证：真实浏览器操作、关键步骤截图、检查 .last-run.json passed、截图附 PR。
4. Phase 3 合并：review thread 全 resolved、required checks API 动态获取、合并后验证生产接口。
5. Phase 4 知识库：当次会话内写复盘文档、git commit + push、提供飞书原文。

## 六、守卫脚本

```PLAIN_TEXT
# 1. 测试 artifact 验证
cat test-results/.last-run.json | python3 -c "
import sys,json; d=json.load(sys.stdin)
assert d['status']=='passed', f'测试未通过: {d}'
print('✅ 测试状态: passed')
"

# 2. 截图文件存在
COUNT=$(find test-results -name "*.png" 2>/dev/null | wc -l)
[ "$COUNT" -gt 0 ] && echo "✅ 截图存在: ${COUNT}张" || echo "❌ ERROR: 无截图文件"

# 3. 知识库已推送
git log origin/$(git branch --show-current) --oneline -1 | grep -q "knowledge" \
 && echo "✅ 知识库已推送" \
 || echo "⚠️ WARNING: 知识库未推送到远端"
```

两份知识库都已入 GitHub：
- `docs/knowledge/2026-05-22-crm-launch-r1-postmortem.md` — 技术根因（MySQL/SQLAlchemy/索引/前端级联）
- `docs/knowledge/2026-05-24-bugfix-process-postmortem.md` — 修复流程（测试证据/hotfix叠加/合并流程/会话核查）

## 内容
# Bug 修复流程复盘知识库

> 覆盖 NUP-579 / NUP-587 / NUP-588 修复周期（2026-05-21 ～ 2026-05-24）
> 本文专注**修复过程本身的失误**，与 `2026-05-22-crm-launch-r1-postmortem.md`（技术根因）互补

## 一、测试声明必须有可核查 Artifact

### P1 · "Playwright 测试通过"但无截图 = 未测试

症状：第一轮修复完成后，声称"Playwright E2E 测试全部通过"，但：
- `test-results/.last-run.json` 显示 `{"status": "failed", "failedTests": []}`
- `test-results/` 目录下零 PNG 截图文件
- 第二个会话重新核查时，不得不重跑所有测试

根因：
1. `npx playwright test` 因配置问题提前退出，状态写入 `failed` 但未报告具体失败项
2. 声称"测试通过"时只看了命令行输出，未检验 artifact 文件是否落盘
3. 会话压缩后，无法还原"测试到底有没有跑"的事实

铁律（不可违反）：
```
测试声明通过的充要条件（缺一不算通过）：
✅ test-results/.last-run.json → "status": "passed"
✅ test-results/ 下存在 .png 截图文件（至少 1 张）
✅ 截图内容与被测功能匹配（不是空白页/登录页）
```

正确的验证命令：
```bash
# 跑完测试后立即验证 artifact
cat test-results/.last-run.json
ls test-results/**/*.png | wc -l # 必须 > 0
```

预防规则：
1. 任何"E2E 测试通过"的声明，必须附上截图路径或截图内嵌展示
2. 截图存入 test-results/screenshots/，提 PR 时附在 PR description 里
3. AI 自动执行测试后，必须读 .last-run.json，不能只看终端 exit code

### P2 · Playwright MCP 浏览器测试的正确姿势

## 结论
Bug 修复流程必须以真实日志、SQL 复现、可核查测试 artifact、resolved review threads、动态 CI checks 和当次会话内提交知识库为硬门槛。摘要不是事实，恢复会话后必须命令核查。

## 影响
后续 CRM/门店 Bug 修复、hotfix、PR 合并、E2E 测试声明和知识库维护均必须按该流程执行，避免反复 hotfix、伪测试通过、PR 合并阻塞和知识未提交。

## 下次注意
文档类写入不需要 Spec/reviewer/冒烟；但 Bug 修复声明必须有截图与 .last-run.json artifact。同一接口 hotfix 失败 2 次必须归零重查，不允许基于上次假设叠加修改。

