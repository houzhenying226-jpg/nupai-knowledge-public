# Self-hosted Runner 清单

> 类型：操作规程 | 项目：_global | 风险：L3 | 状态：活跃
> 最后更新：2026-05-30（振英 + Claude Code，新增 VPS 自托管 Linux + 远程通道 + $PATH 坑）

## 摘要

NuPai 全量 self-hosted runner 清单，含所有机器 SSH 信息、runner 注册明细、服务管理命令。
共 4 台机器 · 4 个仓库 · 96+ 条 runner。

---

## 机器清单

### 北京 Mac M4 Max（houzhenying）

| 项目 | 值 |
|------|---|
| 用户 | `houzhenying` |
| hostname | `houzhenyingdeMacBook-Pro-10.local` |
| Tailscale IP | `100.106.82.24` |
| 局域网 IP | `192.168.100.245` |
| 芯片 | Apple M4 Max · 16 核 · 128 GB |
| macOS | 26.3 (Build 25D125) |
| Runner 版本 | 2.334.0 |
| VPN | Clash 端口 7897 |

```bash
ssh houzhenying@100.106.82.24   # Tailscale（跨网络推荐）
ssh houzhenying@192.168.100.245 # 局域网直连
```

---

### 北京 Intel Mac（zhangxiaomi）

| 项目 | 值 |
|------|---|
| 用户 | `zhangxiaomi` |
| hostname | `zhangximideiMac` |
| 局域网 IP | `192.168.31.212` |
| 芯片 | Intel · X64 |
| SSH 密码 | `1234`（已配置免密，密码备用）|
| VPN | Clash 端口 7897 |
| Runner 版本 | 2.334.0 |
| 睡眠设置 | 全部禁用（sleep/standby/autopoweroff/hibernate 均=0）|

```bash
ssh zhangxiaomi@192.168.31.212  # 局域网直连（免密）
```

**睡眠修复命令（如需重设）：**
```bash
sudo pmset -a sleep 0 disksleep 0 displaysleep 0 standby 0 autopoweroff 0 hibernatemode 0 powernap 0
```

---

### 北京 Windows（39081）

| 项目 | 值 |
|------|---|
| 用户 | `39081` |
| 局域网 IP | `192.168.100.138` |
| PowerShell 提示 | `PS C:\Users\39081>` |
| VPN | Clash Verge 系统代理 端口 7897 |
| Runner 版本 | 2.334.0 |
| Runner 目录 | `C:\actions-runners\` |
| SSH | 需手动开启 OpenSSH Server |

```powershell
# 开启 SSH（管理员 PowerShell）
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
```

> ⚠️ 注意：Windows 上注册 runner 脚本**禁止**手动设置 `$env:https_proxy`，
> 否则与 Clash Verge 系统代理冲突，报 "Invalid configuration provided for token"。
> 让 Clash Verge 系统代理自动接管即可。

---

### 新加坡 Mac（james）

| 项目 | 值 |
|------|---|
| 用户 | `james` |
| Tailscale IP | `100.69.20.113` |
| 局域网 IP | `192.168.1.9` |
| 芯片 | Apple M4 · 10 核 · 16 GB |
| macOS | 26.3 |

```bash
ssh james@100.69.20.113   # 或别名 ssh nupai
```

---

## Runner 全览（按机器）

### 北京 M4 Max（houzhenying）— ARM64

| Repo | Runner 名称 | 数量 | Labels |
|------|------------|------|--------|
| nupai-crm | runner-crm-bj-1~8 | 8 | crm,bj,ARM64 |
| fesun-mos | runner-mos-bj-01~06 | 6 | mos,bj,ARM64 |
| -nupai-store | runner-store-bj-01~06 | 6 | store,bj,ARM64 |
| fesun-platform | runner-platform-bj-01~06 | 6 | platform,bj,ARM64 |

Runner 目录：`~/actions-runners/runner-{prefix}-bj-{N}/`
LaunchAgents：`~/Library/LaunchAgents/actions.runner.houzhenying226-jpg-*.plist`

---

### 北京 Intel Mac（zhangxiaomi）— X64

| Repo | Runner 名称 | 数量 | Labels |
|------|------------|------|--------|
| nupai-crm | runner-crm-bj-intel-1~6 | 6 | self-hosted,macOS,X64,crm,bj |
| fesun-mos | runner-mos-bj-intel-1~6 | 6 | self-hosted,macOS,X64,mos,bj |
| -nupai-store | runner-store-bj-intel-1~6 | 6 | self-hosted,macOS,X64,store,bj |
| fesun-platform | runner-platform-bj-intel-1~6 | 6 | self-hosted,macOS,X64,platform,bj |

Runner 目录：`~/actions-runners/runner-{prefix}-bj-intel-{1-6}/`
代理配置：每个 runner 目录下 `.env`（含 `https_proxy=http://127.0.0.1:7897`）
LaunchAgents：`~/Library/LaunchAgents/actions.runner.houzhenying226-jpg-*.plist`

**重启所有 runner（SSH 进去后）：**
```bash
launchctl unload ~/Library/LaunchAgents/actions.runner.*.plist 2>/dev/null
sleep 2
launchctl load ~/Library/LaunchAgents/actions.runner.*.plist 2>/dev/null
```

---

### 北京 Windows（39081）— X64

| Repo | Runner 名称 | 数量 | Labels |
|------|------------|------|--------|
| nupai-crm | runner-crm-bj-win-1~6 | 6 | self-hosted,Windows,X64,crm,bj |
| fesun-mos | runner-mos-bj-win-1~6 | 6 | self-hosted,Windows,X64,mos,bj |
| -nupai-store | runner-store-bj-win-1~6 | 6 | self-hosted,Windows,X64,store,bj |
| fesun-platform | runner-platform-bj-win-1~6 | 6 | self-hosted,Windows,X64,platform,bj |

Runner 目录：`C:\actions-runners\runner-{prefix}-bj-win-{1-6}\`

**重启所有 runner（管理员 PowerShell）：**
```powershell
Get-Service -Name "actions.runner.*" | Restart-Service
```

**检查状态：**
```powershell
Get-Service -Name "actions.runner.*" | Select-Object Name, Status
```

---

### 新加坡 Mac（james）— ARM64

| Repo | Runner 名称 | 数量 | Labels |
|------|------------|------|--------|
| nupai-crm | runner-crm-1, runner-crm-2 | 2 | crm,sg-mini |
| nupai-crm | runner-crm-sg64-1~2 | 2 | crm,sg-64g |
| fesun-mos | runner-mos-mini-01~06 | 6 | mos,sg-mini |
| fesun-mos | runner-mos-sg64-01~06 | 6 | mos,sg-64g |
| -nupai-store | runner-store-sg64-01~04 | 4 | store,sg-64g |

---

## 快速诊断命令

```bash
# 一键查所有仓库 runner 在线状态
for repo in nupai-crm fesun-mos fesun-platform -nupai-store; do
  echo "=== $repo ==="
  gh api repos/houzhenying226-jpg/$repo/actions/runners \
    --jq '.runners[] | "\(.name) \(.status)"'
done

# 只查 Intel Mac runner
gh api repos/houzhenying226-jpg/nupai-crm/actions/runners \
  --jq '.runners[] | select(.name|test("intel")) | "\(.name) \(.status)"'

# 只查 Windows runner
gh api repos/houzhenying226-jpg/nupai-crm/actions/runners \
  --jq '.runners[] | select(.name|test("win")) | "\(.name) \(.status)"'
```

---

## macOS runner 休眠断网问题

**根因**：`sleep 0` 不够，macOS 的 `standby` 参数会在系统 sleep 数小时后触发深度待机，切断所有网络。

| 参数 | 作用 | 是否断网 |
|------|------|---------|
| `sleep` | 系统闲置睡眠 | 是 |
| `standby` | 睡眠N小时后进深度待机 | **是（主犯）** |
| `autopoweroff` | 长时间后自动断电 | 是 |
| `hibernatemode` | 休眠时内存写盘 | 是 |
| `displaysleep` | 仅屏幕关闭 | **否，不断网** |

**永久修复（两台 Mac 都已执行，如需重设）：**
```bash
sudo pmset -a sleep 0 disksleep 0 displaysleep 0 standby 0 autopoweroff 0 hibernatemode 0 powernap 0
```

**验证（所有值应为 0）：**
```bash
pmset -g | grep -E "sleep|standby|autopoweroff|hibernate|powernap"
```

> 屏幕可以关闭，不影响 runner 在线。关键是 standby=0。

---

## 注册新 Runner 标准流程

### macOS ARM64（M4 Max，北京本机执行）

```bash
TOKEN=$(gh api -X POST \
  repos/houzhenying226-jpg/{REPO}/actions/runners/registration-token \
  --jq '.token')
RUNNER_NAME="runner-{prefix}-bj-{N}"
mkdir -p ~/actions-runners/$RUNNER_NAME
tar xzf ~/actions-runner-bj-5/actions-runner-osx-arm64-2.334.0.tar.gz \
    -C ~/actions-runners/$RUNNER_NAME
cd ~/actions-runners/$RUNNER_NAME
./config.sh \
  --url https://github.com/houzhenying226-jpg/{REPO} \
  --token "$TOKEN" --name "$RUNNER_NAME" \
  --labels "self-hosted,macOS,ARM64,{prefix},bj" \
  --unattended --replace
./svc.sh install && ./svc.sh start
```

### macOS X64（Intel Mac，SSH 进去后执行）

```bash
# SSH 进入
ssh zhangxiaomi@192.168.31.212

# 然后执行（替换 TOKEN、prefix、REPO）
TOKEN="<registration-token>"
RUNNER_NAME="runner-{prefix}-bj-intel-{N}"
BASE="$HOME/actions-runners"
TPL="$HOME/runners-temp/tpl"

rm -rf "$BASE/$RUNNER_NAME" && mkdir -p "$BASE/$RUNNER_NAME"
cp -r "$TPL/." "$BASE/$RUNNER_NAME/"
rm -f "$BASE/$RUNNER_NAME/.runner" "$BASE/$RUNNER_NAME/.credentials"
cd "$BASE/$RUNNER_NAME"
./config.sh --unattended \
  --url "https://github.com/houzhenying226-jpg/{REPO}" \
  --token "$TOKEN" --name "$RUNNER_NAME" \
  --labels "self-hosted,macOS,X64,{prefix},bj" --replace
printf 'https_proxy=http://127.0.0.1:7897\nhttp_proxy=http://127.0.0.1:7897\nHTTPS_PROXY=http://127.0.0.1:7897\nHTTP_PROXY=http://127.0.0.1:7897\nno_proxy=localhost,127.0.0.1\nPATH=/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin\nSSL_CERT_FILE=/etc/ssl/cert.pem\nSSL_CERT_DIR=/etc/ssl/certs\nPIP_CERT=/etc/ssl/cert.pem\nREQUESTS_CA_BUNDLE=/etc/ssl/cert.pem\nCURL_CA_BUNDLE=/etc/ssl/cert.pem\nNODE_EXTRA_CA_CERTS=/etc/ssl/cert.pem\n' > .env
./svc.sh install && ./svc.sh start
```

> ⚠️ **macOS runner 的 `.env` 必须同时含 `PATH` + 6 个 CA 证书变量**（见下方"SSL 证书故障"一节），否则任务里 `python3` 会找不到（PATH 缺 homebrew）或 SSL 握手失败。

---

## SSL 证书故障（CERTIFICATE_VERIFY_FAILED）— 2026-05-29 根治

### 现象
CI 任务里 `pip install ruff/pytest/bandit` 或连 `api.github.com` 报：
```
[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:1028)
```
表现为「间歇性」——落到某些 runner 必失败，重跑落到别的 runner 又过。**不是网络抖动，是确定性故障。**

### 根因（可复现）
runner 的 launchd 环境 PATH 把 `python3` 解析到 `/usr/local/bin/python3`，它指向
`/Library/Frameworks/Python.framework/Versions/3.13/bin/python3.13`（python.org 安装包）。
该安装包**装时没运行 "Install Certificates.command"**，framework 里 `etc/openssl/cert.pem`
不存在 → CA bundle = `None` → 任何 Python TLS 校验必失败。

诊断命令（在故障机器上）：
```bash
# 看 runner PATH 下 python3 是谁
PATH=/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin which -a python3
# 看它的 CA bundle —— 输出 None 即中招
/usr/local/bin/python3 -c "import ssl; print(ssl.get_default_verify_paths().cafile)"
# 复现（带代理）
https_proxy=http://127.0.0.1:7897 /usr/local/bin/python3 -c "import urllib.request; urllib.request.urlopen('https://pypi.org/simple/',timeout=15)"
```
对比：`/usr/bin/python3`（系统 3.9）CA=`/etc/ssl/cert.pem` 正常；`/usr/local/bin/python3` CA=None 失败。

### 根治（环境变量层，无需 sudo，非绕过）
给**每个** runner 的 `.env` 注入指向系统有效 CA bundle 的变量（macOS 自带 `/etc/ssl/cert.pem`，root 管理、系统自动更新）：
```bash
CERT=/etc/ssl/cert.pem
for f in ~/actions-runners/*/.env ~/actions-runner-bj-5/.env ~/actions-runner-bj-6/.env; do
  [ -f "$f" ] || continue
  sed -i '' -E '/^(SSL_CERT_FILE|SSL_CERT_DIR|PIP_CERT|REQUESTS_CA_BUNDLE|CURL_CA_BUNDLE|NODE_EXTRA_CA_CERTS)=/d' "$f"
  printf 'SSL_CERT_FILE=%s\nSSL_CERT_DIR=/etc/ssl/certs\nPIP_CERT=%s\nREQUESTS_CA_BUNDLE=%s\nCURL_CA_BUNDLE=%s\nNODE_EXTRA_CA_CERTS=%s\n' "$CERT" "$CERT" "$CERT" "$CERT" "$CERT" >> "$f"
done
# 重启生效
launchctl unload ~/Library/LaunchAgents/actions.runner.*.plist 2>/dev/null
sleep 2
launchctl load ~/Library/LaunchAgents/actions.runner.*.plist 2>/dev/null
```
- **不是 `--trusted-host` / `verify=False` 那种关闭校验的绕过** —— 指向的是合法 CA，TLS 校验照常进行，安全。
- 仅改 runner 机器运行时配置，零仓库代码 / 零 workflow 改动。

### 验证
```bash
# 用 runner 完整 env 跑一次装包，应零 SSL 错误
PATH=/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin SSL_CERT_FILE=/etc/ssl/cert.pem PIP_CERT=/etc/ssl/cert.pem \
  https_proxy=http://127.0.0.1:7897 /usr/local/bin/python3 -m pip install --dry-run bandit
# 期望末行：Would install bandit-x.y.z ...
```

### 彻底根治（可选，需 sudo）
补上 framework 缺失的证书软链（修好后该 python 自带 CA，连环境变量都不需要）：
```bash
sudo mkdir -p /Library/Frameworks/Python.framework/Versions/3.13/etc/openssl
sudo ln -sf /Library/Frameworks/Python.framework/Versions/3.13/lib/python3.13/site-packages/certifi/cacert.pem \
  /Library/Frameworks/Python.framework/Versions/3.13/etc/openssl/cert.pem
```

### 关键教训
- **新增 macOS runner 必须在 `.env` 里同时配 6 个 CA 变量**（已并入上方注册流程的 `.env` 模板）
- 看到「间歇性 SSL 失败、重跑有时过」别当网络问题——查 `python3 -c "import ssl; print(ssl.get_default_verify_paths().cafile)"` 是不是 None

---

## 排查 Checklist

| 现象 | 处理 |
|------|------|
| macOS runner offline | `launchctl unload ~/Library/LaunchAgents/actions.runner.*.plist && sleep 2 && launchctl load ~/Library/LaunchAgents/actions.runner.*.plist` |
| macOS runner 反复掉线 | 检查 standby：`pmset -g \| grep standby`，执行睡眠修复命令 |
| Windows runner offline | 确认 Clash Verge 系统代理开启；`Get-Service "actions.runner.*" \| Restart-Service` |
| 注册 token 无效 | 重新获取：`gh api -X POST repos/.../registration-token --jq '.token'` |
| launchctl stop 后无法重启 | 用 `unload+load`，不要用 `stop+start`（stop 留 stop marker 阻止自动重启）|
| Windows 注册报 "Invalid token" | 删除脚本里的 `$env:https_proxy` 设置，让 Clash Verge 系统代理自动处理 |
| 任务报 `CERTIFICATE_VERIFY_FAILED`（间歇/重跑有时过）| macOS framework python 缺 CA。查 `/usr/local/bin/python3 -c "import ssl;print(ssl.get_default_verify_paths().cafile)"`，None 即中招。给所有 runner `.env` 加 6 个 CA 变量后重启（见"SSL 证书故障"一节）|
| 任务报 `python3: command not found` | runner `.env` 缺 `PATH`，加 `PATH=/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin` 后重启 |

---

## 注意事项

- **不要删除/停止** runner-crm-bj-5、runner-crm-bj-6（旧目录 `~/actions-runner-bj-5/6/`）
- ARM64 tarball：`~/actions-runner-bj-5/actions-runner-osx-arm64-2.334.0.tar.gz`（可复用）
- X64 模板：Intel Mac 上 `~/runners-temp/tpl/`
- launchd 服务开机自启，不依赖登录 session
- Windows 服务通过 `svc.ps1 install` 注册为 Windows Service，开机自启

---

## 远程接入通道（2026-05-30 建立）

三台机器现在都能远程 SSH 维护，不用本人到电脑旁。

### Tailscale 网（账号 `houzhenying226@`）
| 机器 | Tailscale IP | SSH |
|------|-------------|-----|
| M4 Max（macbook-pro）| `100.68.35.88` | 本机 |
| Intel Mac（imac）| `100.123.78.7` | `ssh zhangxiaomi@100.123.78.7`（已加公钥免密）|

> 公钥：`houzhenying226@gmail.com` 的 ed25519 已加到 Intel Mac `~/.ssh/authorized_keys`。
> Intel Mac 自动登录已开（重启后 Tailscale+runner 自动回来）。

### 新加坡 VPS 跳板（账号 `zhenyinghou2@`，跨网到 james Mac）
- VPS：`root@185.194.54.64`（s54811.vps.hosting，Ubuntu 24.04，已加公钥免密）
- 新加坡 james Mac 不在 `houzhenying226@` 网，只能经 VPS 跳：
  ```bash
  ssh -J root@185.194.54.64 james@100.69.20.113   # james Mac，已加公钥免密
  ```
- james Mac 登录密码（备用）：`372925`；自动登录已开，FileVault 关。
- VPS Tailscale 账号 `zhenyinghou2@` 里有 james Mac（`100.69.20.113`，hostname `192`）。

---

## macOS runner `.env` 的 `$PATH` 不展开坑（2026-05-30）

**现象**：某 runner 的 job 报 `tar: command not found`（或其它 /usr/bin 下的命令），checkout 前就崩。

**根因**：runner 的 `.env` 文件是**逐行字面解析、不做 shell 变量展开**。若写了
`PATH=/opt/homebrew/bin:/usr/local/bin:$PATH`，`$PATH` 不会展开，被当成字面目录名，
结果实际 PATH **不含 `/usr/bin`** → tar 等系统命令找不到。

**修复**：`.env` 里 PATH 必须写**绝对路径、不能带 `$PATH`**：
```
PATH=/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```
（不写 PATH 行也行，用 launchd 默认 PATH 反而正常。）案例：新加坡 `runner-crm-6`。

---

## 新加坡 VPS：自托管 Linux runner（2026-05-30 新建）

**背景**：`-nupai-store` 的 workflow 全部 `runs-on: ubuntu-latest`（成本治理策略，
`runner_policy_guard.py` 只允许 ubuntu），但 GitHub 托管 runner 对该仓库不可用 →
job 调度即失败（runner 为空、1 秒失败、无日志）。store 22 条自托管 runner 全 macOS/Windows，
零 Linux。决策：在 VPS 上建自托管 Linux runner + 把 store workflow 迁到 self-hosted。

### VPS runner 信息
| 项 | 值 |
|----|---|
| 机器 | `root@185.194.54.64`（Ubuntu 24.04，2 vCPU，1.9G+4G swap）|
| runner 用户 | `actions`（非 root，NOPASSWD sudo）|
| runner 目录 | `/home/actions/actions-runner-{1,2}/` |
| 服务 | systemd `actions.runner.houzhenying226-jpg--nupai-store.runner-store-vps-{1,2}.service`（开机自启）|
| 名称/标签 | `runner-store-vps-{1,2}` → `self-hosted,Linux,X64,store,vps,deploy` |
| 工具链 | node20 / npm / python3+pip / git / jq / gh / bc（都在 PATH）|

### 注册新 Linux runner（VPS 上，root）
```bash
TOKEN=$(gh api -X POST repos/houzhenying226-jpg/-nupai-store/actions/runners/registration-token --jq '.token')  # 在有 gh 的机器生成
sudo -u actions bash -c "cd /home/actions/actions-runner-N && ./config.sh --unattended \
  --url https://github.com/houzhenying226-jpg/-nupai-store --token $TOKEN \
  --name runner-store-vps-N --labels self-hosted,Linux,X64,store,vps --replace"
cd /home/actions/actions-runner-N && ./svc.sh install actions && ./svc.sh start
```

### store workflow runner 策略（runner_policy_guard.py）
- 守卫脚本 `DEFAULT_ALLOWED = {ubuntu-latest, ubuntu-22.04, ubuntu-24.04}`，
  self-hosted/macOS **必须**附 `# runner-policy: allow-self-hosted` 注释（上 4 行/下 2 行内）才只算 warning。
- **deploy 类 job**（job 名含 deploy/release/publish/prod）的自托管 runner **必须带 `deploy` 或 `prod` 标签**，否则 error。
- store 已全部迁移：`runs-on: [self-hosted, Linux, X64, store]  # runner-policy: allow-self-hosted`，deploy/release job 带 `deploy` 标签（PR #166 已合并）。
- 改 workflow 后本地先验证：`python3 .github/scripts/runner_policy_guard.py --format markdown --fail-on error`（退出码 0 才推）。

### 踩坑
- VPS 缺 `gh` → `pr-size` / `enable-auto-merge` job 报 `gh: command not found`（exit 127）。装 `gh` 二进制到 `/usr/local/bin` 解决。
- 自托管 Linux runner 标签是 `[self-hosted, Linux, X64]`，**不匹配 `ubuntu-latest`** —— 想用自托管必须同时改 workflow 的 runs-on。
