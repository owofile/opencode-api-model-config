---
name: api-model-config
description: |
  添加、更新、管理模型API配置。当用户提到以下情况时使用：
  - "添加API"、"配置API"、"添加模型"、"添加provider"
  - "更新模型列表"、"添加新模型"
  - "修改API key"、"更新key"、"更换API"
  - "查看当前配置"、"查看已有模型"
  - "xx模型的API怎么添加"、"我想用xx模型"
  - 中转API、代理API配置
  - 任何关于 opencode 模型的配置需求
---

# API Model Config Skill

管理 OpenCode 的模型 API 配置，包括添加、更新、修改 provider 和模型。

## 配置文件

| 文件 | 路径 | 用途 |
|------|------|------|
| provider 配置 | `~/.config/opencode/opencode.json` | baseURL、模型列表 |
| 凭证存储 | `~/.local/share/opencode/auth.json` | API keys |

## Provider ID 命名规范

### 命名规则

| 类型 | 规则 | 示例 |
|------|------|------|
| 普通/基础版 | 简洁小写单词 | `minimax`, `deepseek`, `zhipu` |
| 套餐/特殊版 | 驼峰命名 | `minimaxPlan`, `openaiPro` |
| 中转/代理 | 描述性名称 | `siliconflow`, `baishan` |

### 命名禁忌

- 避免模糊词汇：`coding`, `pro`, `plus`, `plan`（单独使用）
- 避免全小写连字符：`minimax-plan` → 应为 `minimaxPlan`
- 避免过长名称

### 正反对比

```json
// 错误
"minimax-cn-coding-plan": { }
"openai-pro": { }
"zhipu-plan": { }

// 正确
"minimax": { }
"minimaxPlan": { }
```

## 核心操作

### 查询当前配置

```bash
cat ~/.config/opencode/opencode.json
cat ~/.local/share/opencode/auth.json | jq 'map_values(.key = "***")'
```

### 添加 Provider

**opencode.json** 添加 provider：
```json
{
  "provider": {
    "provider-id": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "显示名称",
      "options": {
        "baseURL": "https://api.example.com/v1"
      },
      "models": {
        "model-id": { "name": "模型显示名" }
      }
    }
  }
}
```

**auth.json** 添加凭证：
```json
{
  "provider-id": {
    "type": "api",
    "key": "your-api-key"
  }
}
```

### 更新模型列表

在已有 provider 下添加模型：

```json
{
  "provider": {
    "existing-provider": {
      "models": {
        "new-model": { "name": "新模型" }
      }
    }
  }
}
```

### 修改 API Key

编辑 `auth.json`：
```json
{
  "provider-id": {
    "type": "api",
    "key": "new-key"
  }
}
```

### 删除 Provider

从两个文件中**同时**删除对应条目。

### 重命名 Provider（重要）

**必须同步修改两个文件：**
1. `opencode.json` 中的 provider ID
2. `auth.json` 中的 provider ID（必须一致）
3. `opencode.json` 中的 `model` 字段（如引用了该 provider）

## 常见 Provider 模板

### Anthropic 兼容 API（套餐版）

```json
{
  "provider": {
    "minimaxPlan": {
      "npm": "@ai-sdk/anthropic",
      "name": "MiniMax 套餐",
      "options": {
        "baseURL": "https://api.minimaxi.com/anthropic/v1"
      },
      "models": {
        "MiniMax-M2.7": { "name": "MiniMax M2.7" }
      }
    }
  }
}
```

### OpenAI 兼容 API（普通版）

```json
{
  "provider": {
    "minimax": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "MiniMax 普通API",
      "options": {
        "baseURL": "https://api.minimax.chat/v1"
      },
      "models": {
        "MiniMax-M2.7": { "name": "MiniMax M2.7" },
        "MiniMax-M2.5": { "name": "MiniMax M2.5" },
        "MiniMax-M2.1": { "name": "MiniMax M2.1" }
      }
    }
  }
}
```

### 国内常用 Provider

**智谱 AI**：baseURL `https://open.bigmodel.cn/api/paas/v4`，npm `@ai-sdk/openai-compatible`

**DeepSeek**：baseURL `https://api.deepseek.com/v1`，npm `@ai-sdk/openai-compatible`

**硅基流动**：baseURL `https://api.siliconflow.cn/v1`，npm `@ai-sdk/openai-compatible`

**Moonshot**：baseURL `https://api.moonshot.cn/v1`，npm `@ai-sdk/openai-compatible`

## 获取 Provider 支持的模型

1. 访问官方文档的 API/Models 页面
2. 确认 baseURL 和模型 ID

常用文档：
| Provider | 文档地址 |
|----------|----------|
| MiniMax | https://platform.minimaxi.com/docs |
| 智谱 AI | https://open.bigmodel.cn/dev/api |
| DeepSeek | https://platform.deepseek.com/docs |
| 硅基流动 | https://docs.siliconflow.cn |

## 验证配置

```bash
opencode
/models
```

确认 provider 和模型出现在列表中。

## 故障排除

| 错误现象 | 可能原因 | 解决方案 |
|----------|----------|----------|
| 模型不出现 | JSON 语法错误、provider ID 不一致 | 检查两个文件的 provider ID 是否一致 |
| 认证失败 | API key 错误或过期 | 检查 auth.json 中的 key |
| 连接超时 | baseURL 不可访问 | 浏览器打开 baseURL 确认 |
| 重复的 provider | 手动编辑时未删除旧配置 | 清理重复配置 |
| 套餐API不显示 | provider ID 命名不规范 | 用驼峰如 `minimaxPlan` |

### 环境变量检查

某些 provider（如 MiniMax 套餐）需清除 Anthropic 环境变量：
```bash
env | grep -i anthropic
# 如有 ANTHROPIC_AUTH_TOKEN 或 ANTHROPIC_BASE_URL，需清除
```

## 配置检查清单

- [ ] provider ID 命名符合规范
- [ ] 两个文件中的 provider ID 完全一致
- [ ] 无重复的 provider 配置
- [ ] 无多余括号或逗号
- [ ] baseURL 格式正确（`/v1` 或 `/anthropic/v1`）
- [ ] npm 包类型正确（`@ai-sdk/openai-compatible` 或 `@ai-sdk/anthropic`）