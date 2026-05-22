# CRM 上线 R1 批次 Bug 复盘知识库

> 类型：故障复盘 | 项目：nupai-crm | 风险：L1 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-22T05:00:01Z

## 摘要

11 条教训（L1-L11）覆盖 MySQL 方言兼容、SQLAlchemy ORM 陷阱、前端级联选择、CI/CD 流程改进、调试方法论。全部根因已定位并修复（PR #726/727/728），建立 4 项守卫脚本 + CI 检查 + PR review checklist。

## 正文

> 记录日期：2026-05-22  风险等级：L1

## 背景
覆盖 NUP-579 / NUP-580 / NUP-581 / NUP-582 / NUP-583 / NUP-584 / NUP-587 / NUP-588，事件周期 2026-05-21 ～ 2026-05-22。CRM 上线后 /activities 页面反复 500（先后 4 次 hotfix），拜访表单白屏，activity 表缺索引等问题。

## 内容
## 一、最高优先级教训
### L1 · MySQL 方言和 SQLAlchemy ORM 语法必须在本地 MySQL 验证
症状：/activities 页面反复 500，4 次 hotfix 未命中根因。
根因：desc(CrmActivity.activitydate).nulls_last() 生成的 NULLS LAST 语法在 MySQL < 8.0.26 不支持。
修复：用 (col IS NULL) ASC 替代 .nulls_last()。
预防：grep "nulls_last\|nullslast\|NULLS LAST" backend/app/ → 必须 0 行。

### L2 · SQLAlchemy 过滤 NULL 必须用 .is_(None)
症状：accounts.py:163 公海池过滤 CrmAccount.owner_id is None 永远不生效。
修复：改为 .is_(None)。
预防：grep "\.where(.*is None\|\.filter(.*is None" backend/app/ → 必须 0 行。

### L3 · 同一 Bug 修 3 次没好 = 根因定位错
规则：同一接口 500 修 2 次没好 → 停下来 → docker logs 提取真实错误 → MySQL client 复现 → 确认后开修。禁止基于"上次分析"叠加。

## 二、数据库与迁移
### L4 · 高频关联查询外键必须显式创建索引
症状：crm_activity 表 account_id 列全表扫描。
修复：op.create_index('idx_activity_account', 'crm_activity', ['account_id'])
预防：任何 ForeignKey 列出现在 WHERE/JOIN → Alembic 迁移显式 create_index。

### L5 · Alembic 迁移 downgrade() 必须实现
创建 index → downgrade 写 op.drop_index；添加 column → downgrade 写 op.drop_column。

## 三、前端
### L6 · 拜访表单白屏：级联 Select onChange 回调触发渲染中断
症状：NUP-579，选客户后联系人下拉展开白屏。
根因：account_id Select onChange 触发 fetch 前未清空 contact_ids。
修复：父 Select 变更时 form.setFieldValue('child_id', undefined)。
预防：所有级联选择必须在父值变更时 reset 子 Select。

## 四、CI/CD 流程
### L7 · 自动合并 REQUIRED checks 从 branch protection API 获取
PR #726 watcher 硬编码 check 名称与 CI job 名称不一致（test vs lint-and-test）。
修复：从 gh api branch protection 动态读取。

### L8 · "All comments must be resolved" → GraphQL resolveReviewThread
修复：gh api graphql mutation resolveReviewThread。

### L9 · self-hosted runner 网络不稳定 deploy 失败
detect-release-docs-only job checkout 连不上 github.com:443。
处理：gh run rerun --failed，非代码问题不改代码。

## 五、调试方法论
### L10 · 正确的 500 调试顺序
Step 1: docker logs nupai-fastapi --tail=200 | grep ERROR
Step 2: 提取真实 SQL → mysql client 手动执行
Step 3: 根据真实错误定位根因 → 修根因不修周边
Step 4: 本地 MySQL 验证修复
Step 5: 提 PR，CI 验证，部署
禁止：猜测根因、基于"上次 hotfix 方向"叠加、不看后端日志。

### L11 · Playwright E2E 测试必须覆盖二次操作场景
拜访表单测试必含 3 场景：直接选/换客户再选/有联系人换客户清空。

## 六、总结
| Issue | 根因类型 | 修复 PR |
| NUP-587 /activities 500 | MySQL 方言 NULLS LAST | PR #726 |
| NUP-579 拜访表单白屏 | 级联 Select 未 reset | 本批次 |
| NUP-588 activity 索引缺失 | 外键无显式索引 | PR #727 |

## 七、守卫脚本（每次部署前跑）
1. grep nulls_last backend/app/ → 必须 0 行
2. grep "\.where(.*is None" backend/app/ → 必须 0 行
3. grep "def downgrade" backend/alembic/versions/*.py → downgrade 不能 pass
4. grep "onChange.*account_id" frontend/ → 人工确认 setFieldValue('contact')

## 结论
11 条教训（L1-L11）覆盖 MySQL 方言兼容、SQLAlchemy ORM 陷阱、前端级联选择、CI/CD 流程改进、调试方法论。全部根因已定位并修复（PR #726/727/728），建立 4 项守卫脚本 + CI 检查 + PR review checklist。

## 影响
建立部署前守卫脚本（grep 检查 NULLS LAST / is None / downgrade pass / 级联 reset）；CI/CD 流程改进（自动合并 watcher 从 branch protection API 动态获取 required checks）；Playwright E2E 测试覆盖二次操作场景；数据库迁移规则强化（downgrade 必须成对实现、外键必须有显式索引）。

## 下次注意
每次部署前跑守卫脚本；MySQL 方言必须本地 MySQL 验证；同一 Bug 修 2 次没好停下来重新查 docker logs；SQLAlchemy 过滤 NULL 必须用 .is_(None)；禁止使用 nulls_last()/nullslast()/NULLS LAST。

