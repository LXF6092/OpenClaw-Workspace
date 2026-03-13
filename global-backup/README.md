# OpenClaw 全局配置备份

备份时间：2026-03-13

## 备份内容

- `openclaw.json-*` - 主配置文件及历史备份
- 包含：模型配置、Agent 默认配置、Gateway 认证

## 当前配置状态

### ✅ 已配置
- 阿里云百炼模型 (qwen3.5-plus, qwen3-max, glm-5, kimi-k2.5 等)
- Gateway 本地模式

### ❌ 未配置
- 飞书 (Feishu/Lark) 渠道
- 多 Agent bindings 路由
- 其他渠道 (Discord/Telegram/WhatsApp 等)

## 恢复方法

```powershell
# 停止 Gateway
openclaw gateway stop

# 恢复配置
Copy-Item C:\Users\Mac\.openclaw\backup\openclaw.json-YYYYMMDD C:\Users\Mac\.openclaw\openclaw.json -Force

# 重启 Gateway
openclaw gateway restart
```

## 下次备份

```powershell
$timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
Copy-Item C:\Users\Mac\.openclaw\openclaw.json C:\Users\Mac\.openclaw\backup\openclaw.json-$timestamp
```
