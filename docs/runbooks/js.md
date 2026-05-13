# 通过JS包哈希判断服务器是否运行旧前端

> 类型：操作规程 | 项目：nupai-store | 风险：L2 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-13T04:00:00Z

## 摘要

看 Network 里的 index-*.js 哈希是最快的"服务器版本指纹"。部署后第一步永远是刷新页面查这个哈希是否更新。

## 正文

> 记录日期：2026-05-13  风险等级：L2

## 背景
本地重新构建前端并部署后，服务器可能仍在服务旧版 dist，导致新路由/新组件不生效。需要快速判断服务器运行的是哪个版本的构建。

## 内容
Vite 构建每次生成带内容哈希的文件名（如 index-tXKRKNBc.js），哈希变了 = 新构建。
判断步骤：
1. 本地确认新构建哈希：
```PLAIN_TEXT
ls dist/assets/index-*.js# 输出例：dist/assets/index-tXKRKNBc.js
```
2. 用 Playwright 或浏览器访问站点，看 Network 请求的 index-*.js 文件名：
  - 与本地一致 → 服务器已是新版
  - 哈希不同（如 index-Du7Ko8Zt.js）→ 服务器仍是旧版，需重新部署
3. 如果是旧版，检查特定组件是否在服务器上：
  - Network 里是否出现 WecomSidebar-BjaQQcRQ.js → 有=新版，无=旧版
4. 重新部署：sh /Users/james/projects/nupai-store/deploy-sidebar.sh

## 结论
看 Network 里的 index-*.js 哈希是最快的"服务器版本指纹"。部署后第一步永远是刷新页面查这个哈希是否更新。

## 影响
- 所有前端部署验证步骤
- Playwright 自动化测试中的版本检查

## 下次注意
deploy 成功（脚本输出 ✅）≠ 服务器一定在运行新版。必须在浏览器/Playwright Network 里验证 index-*.js 哈希与本地一致，才算部署验证通过。

