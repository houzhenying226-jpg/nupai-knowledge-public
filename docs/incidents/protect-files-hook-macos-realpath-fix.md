# protect-files hook macOS realpath 修复记录

> 类型：故障复盘 | 项目：openclaw | 风险：L3 | 状态：草稿
> 同步自 Feishu Bitable 表 A | 2026-04-30T03:41:20Z

## 摘要

realpath -m 在 macOS 不可用 → 路径标准化失效 → 符号链接绕过风险。修法：realpath（无-m）→ python3 abspath → raw 三段 fallback 链，同时解决兼容性和 symlink 问题。

## 正文

# protect-files hook macOS realpath 修复记录

## 两次问题
1. 初版 realpath -m：macOS BSD 不支持 -m，静默 fallback 到原始字符串
2. 二版 python3 abspath：不解析 symlink，ln -s ~/.claude /tmp/trap 可绕过

## 最终修法（三段 fallback）
```bash
file_abs=$(realpath "$file" 2>/dev/null \
  || python3 -c "import os,sys; print(os.path.abspath(sys.argv[1]))" "$file" 2>/dev/null \
  || echo "$file")
```
- realpath（无-m）：对存在文件解析 symlink ✓
- python3 abspath：文件不存在时兜底 ✓
- raw：极端环境 ✓

## 验证
- 白名单路径 → 豁免 exit 0 ✅
- ~/.claude/CLAUDE.md → 拦截 exit 2 ✅
- ln -s ~/.claude /tmp/trap + write settings.json → 拦截，显示解析后真实路径 ✅

## 教训
跨平台 shell 必须分别验证 macOS/Linux，安全相关路径标准化必须做 symlink 攻击测试
