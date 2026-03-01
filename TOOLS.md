# TOOLS.md - Local Notes
Skills define *how* tools work. This file is for *your* specifics — the stuff that's unique to your setup.

## What Goes Here
Things like:
- Camera names and locations
- SSH hosts and aliases
- Preferred voices for TTS
- Speaker/room names
- Device nicknames
- Anything environment-specific

## Examples
```markdown
### Cameras
- living-room → Main area, 180° wide angle
- front-door → Entrance, motion-triggered

### SSH
- home-server → 192.168.1.100, user: admin

### TTS
- Preferred voice: "Nova" (warm, slightly British)
- Default speaker: Kitchen HomePod
```

## Why Separate?
Skills are shared. Your setup is yours. Keeping them apart means you can update skills without losing your notes, and share skills without leaking your infrastructure.

---

### memory-enhanced - 记忆增强系统 (v2.0)
- 本地技能路径: `skills/memory-enhanced/`
- 功能: 极速搜索(<100ms)、深度搜索、智能推荐
- 特点: 预加载索引、分层语义搜索

### depth-search - 深度搜索引擎
- 本地技能路径: `skills/depth-search/`
- 功能: 语义扩展、知识图谱、多层级穿透搜索
- 演示: `@depth-search demo "查询词"`

### gateway-guardian - Gateway守护
- 本地技能路径: `skills/gateway-guardian/`
- 功能: 进程守护、健康监控、自动重启、状态持久化
- 状态: `openclaw guardian status`

### auto-repair - 自动修复系统
- 本地技能路径: `skills/auto-repair/`
- 功能: 错误捕获、模式学习、自动修复
- 支持: 技能故障、配置错误、网络问题自愈

### speed-optimizer - 响应速度优化
- 本地技能路径: `skills/speed-optimizer/`
- 功能: 流式输出、智能缓存、并行工具执行
- 目标: 响应时间 <2秒，首字 <500ms

### antigravity-config - Antigravity API配置 (🆕)
- API Key: `sk-b97b3cf1074f434cabfacdebe3476608`
- Base URL: `https://api.antigravity.prod/v1`
- 模型: Gemini 3 Pro / Gemini 3 Pro Image
- 状态: 待手动配置到 ~/.openclaw/openclaw.json

---

### 10x-chat
- 本地技能路径: `skills/10x-chat/SKILL.md`
- 配置文件: `~/.10x-chat/profiles/`
- 状态: Gemini ✅ / ChatGPT ✅（双 AI 配置）

#### 快捷命令
```powershell
# Gemini
npx 10x-chat@latest chat --provider gemini -p "问题"

# ChatGPT（带文件）
npx 10x-chat@latest chat --provider chatgpt -p "问题" --file "path/to/file"

# 查看会话
npx 10x-chat@latest status
```

---
Add whatever helps you do your job. This is your cheat sheet.
