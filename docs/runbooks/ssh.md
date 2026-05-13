# SSH反向隧道暴露本地后端到远程服务器

> 类型：操作规程 | 项目：nupai-store | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-13T04:00:00Z

## 摘要

完整一键重建命令：sh /Users/james/projects/nupai-store/deploy-sidebar.sh（包含隧道\+iptables\+nginx全部步骤）。

## 正文

> 记录日期：2026-05-13  风险等级：L3

## 背景
本地开发阶段，FastAPI 后端跑在 Mac localhost:8001，需要让服务器上的 nginx 能访问到，用 SSH 反向隧道实现，不需要把后端部署上服务器。

## 内容
命令：
```PLAIN_TEXT
# 建立反向隧道（Mac 本地执行，-f 后台运行，-N 不执行命令）ssh -fN -R 127.0.0.1:8081:localhost:8001 root@39.96.216.33
# 查看隧道是否在跑ps aux | grep "ssh.*8081" | grep -v grep
# 杀掉旧隧道（重建前）pkill -f "ssh.*-R.*8081:localhost:8001"
```
端口关系：
- 服务器 [127.0.0.1:8081](127.0.0.1:8081) ← SSH反向隧道 → Mac localhost:8001
- nginx 通过 iptables DNAT 把 [10.255.13.1:8081](10.255.13.1:8081) 重定向到 [127.0.0.1:8081](127.0.0.1:8081)
GatewayPorts 限制：
- 默认 ssh -R 只绑定服务器 [127.0.0.1](127.0.0.1)（非 [0.0.0.0](0.0.0.0)）
- Docker 容器无法直接访问宿主机 [127.0.0.1:8081](127.0.0.1:8081)，需配合 iptables（见"Docker→宿主机路由打通三件套"）
持久化：
- Mac 重启后隧道断，需重跑 sh deploy-sidebar.sh 重建
- 服务器重启后 iptables 规则消失，也需重跑

## 结论
完整一键重建命令：sh /Users/james/projects/nupai-store/deploy-sidebar.sh（包含隧道\+iptables\+nginx全部步骤）。

## 影响
- deploy-sidebar.sh 第6步
- Mac 每次重启后需手动重跑

## 下次注意
侧边栏 API 502 时，第一步查隧道：ps aux \| grep "ssh.*8081"，没输出 = 隧道断了，跑 deploy-sidebar.sh 重建。

