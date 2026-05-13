# Docker→宿主机SSH隧道路由打通三件套

> 类型：操作规程 | 项目：nupai-store | 风险：L4 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-05-13T04:00:00Z

## 摘要

三步缺一不可。route_localnet 控制内核路由，iptables DNAT 拦截 Docker 发往 [10.255.13.1:8081](10.255.13.1:8081) 的包并重写目标为 [127.0.0.1:8081](127.0.0.1:8081)，SSH 隧道把宿主机 8081 转发到 Mac 8001。

## 正文

> 记录日期：2026-05-13  风险等级：L4

## 背景
本地 Mac 后端（localhost:8001）通过 SSH 反向隧道暴露为服务器 [127.0.0.1:8081](127.0.0.1:8081)，但 Docker 容器内的 nginx 无法直接访问宿主机 loopback。需要三步打通。

## 内容
架构：
```PLAIN_TEXT
WeCom侧边栏 → nginx(Docker) → 10.255.13.1:8081                                  ↓ iptables DNAT                              宿主机 127.0.0.1:8081                                  ↓ SSH反向隧道                              Mac localhost:8001
```
三件套命令（在宿主机 [root@39.96.216.33](root@39.96.216.33) 上执行）：
```PLAIN_TEXT
# 第1步：允许从非loopback接口路由到loopbackecho 1 > /proc/sys/net/ipv4/conf/all/route_localnet
# 第2步：iptables DNAT（先删旧规则再加，幂等）iptables -t nat -D PREROUTING -p tcp --dport 8081 -j DNAT --to-destination 127.0.0.1:8081 2>/dev/null || trueiptables -t nat -A PREROUTING -p tcp --dport 8081 -j DNAT --to-destination 127.0.0.1:8081
# 第3步：SSH反向隧道（在Mac本地执行）pkill -f "ssh.*-R.*8081:localhost:8001" 2>/dev/null || truesleep 1ssh -fN -R 127.0.0.1:8081:localhost:8001 root@39.96.216.33
```
验证方法：
浏览器访问 [https://store.ifeishen.com/api/v1/health，返回](https://store.ifeishen.com/api/v1/health%EF%BC%8C%E8%BF%94%E5%9B%9E) 200 或 401 即为通（不是 502）

## 结论
三步缺一不可。route_localnet 控制内核路由，iptables DNAT 拦截 Docker 发往 [10.255.13.1:8081](10.255.13.1:8081) 的包并重写目标为 [127.0.0.1:8081](127.0.0.1:8081)，SSH 隧道把宿主机 8081 转发到 Mac 8001。

## 影响
- deploy-sidebar.sh 的第5、6步
- 每次 Mac 重启后需重新执行第3步（SSH隧道）
- 服务器重启后需重新执行第1、2步（iptables规则不持久化）

## 下次注意
服务器重启后 route_localnet 会归零、iptables 规则会清空，需重跑 deploy-sidebar.sh。Mac 关机后 SSH 隧道断，重开后跑 ssh -fN -R [127.0.0.1:8081](127.0.0.1:8081):[root@39.96.216.33](root@39.96.216.33)。

