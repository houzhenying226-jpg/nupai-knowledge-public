# deploy脚本与patch脚本nginx配置互相覆盖

> 类型：故障复盘 | 项目：nupai-store | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-13T04:00:00Z

## 摘要

将正确配置永久写入 deploy-sidebar.sh，删掉 fix-api-tunnel.sh（或不再使用）：
```PLAIN_TEXT
# deploy-sidebar.sh 中正确的 nginx 配置段location /api/ {    proxy_pass http://10.255.13.1:8081/api/;  # ← 必须是这个，不是 127.0.0.1    proxy_s

## 正文

> 记录日期：2026-05-13  风险等级：L3

## 背景
deploy-sidebar.sh 写入 nginx 配置用 proxy_pass [http://127.0.0.1:8081（错误），用](http://127.0.0.1:8081%EF%BC%88%E9%94%99%E8%AF%AF%EF%BC%89%EF%BC%8C%E7%94%A8/) fix-api-tunnel.sh 临时修成 [10.255.13.1:8081](10.255.13.1:8081)（正确）。但下次跑 deploy 脚本，又回到 [127.0.0.1](127.0.0.1)，502 复现。

## 内容
- 覆盖路径：
  a. deploy-sidebar.sh 第4步写入 dev-sidebar.conf（[127.0.0.1:8081](127.0.0.1:8081)）
  b. fix-api-tunnel.sh 手动修正（[10.255.13.1:8081](10.255.13.1:8081)）
  c. 再次跑 deploy-sidebar.sh → 第4步再次写入（[127.0.0.1:8081](127.0.0.1:8081)）→ 覆盖修正
- 诊断方法：查 API 返回 502 时，看 Playwright network requests 里 index-*.js 的哈希值
  - 旧哈希 index-Du7Ko8Zt.js = 部署前（不含 WecomSidebar 路由）
  - 新哈希 index-tXKRKNBc.js = 正确版本

## 结论
将正确配置永久写入 deploy-sidebar.sh，删掉 fix-api-tunnel.sh（或不再使用）：
```PLAIN_TEXT
# deploy-sidebar.sh 中正确的 nginx 配置段location /api/ {    proxy_pass http://10.255.13.1:8081/api/;  # ← 必须是这个，不是 127.0.0.1    proxy_set_header Host $host;    proxy_read_timeout 30;}
```

## 影响
- /Users/james/projects/nupai-store/deploy-sidebar.sh（已修正）
- fix-api-tunnel.sh（已废弃，不再使用）

## 下次注意
不要用临时 patch 脚本修配置，要直接改 deploy 脚本的源头。有多个脚本管理同一份配置 = 必然冲突。配置的唯一真相来源是 deploy-sidebar.sh。

