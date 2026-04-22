---
name: api-model-config
description: |
  Add, update, and manage model API configurations for OpenCode. Use when the user mentions:
  - "add API", "configure API", "add model", "add provider"
  - "update model list", "add new model"
  - "change API key", "update key", "switch API"
  - "view current config", "list existing models"
  - "how to add API for xx model", "I want to use xx model"
  - proxy API, gateway API configuration
  - any OpenCode model configuration needs
---

# API Model Config Skill

Manage OpenCode's model API configurations, including adding, updating, and modifying providers and models.

## Config Files

| File | Path | Purpose |
|------|------|---------|
| Provider config | `~/.config/opencode/opencode.json` | baseURL, model list |
| Credentials | `~/.local/share/opencode/auth.json` | API keys |

## Provider ID Naming Rules

### Naming Conventions

| Type | Rule | Example |
|------|------|---------|
| Standard/Basic | Simple lowercase word | `minimax`, `deepseek`, `zhipu` |
| Plan/Premium | CamelCase | `minimaxPlan`, `openaiPro` |
| Proxy/Gateway | Descriptive name | `siliconflow`, `baishan` |

### Naming Restrictions

- Avoid vague words: `coding`, `pro`, `plus`, `plan` (standalone)
- Avoid lowercase with hyphens: `minimax-plan` → use `minimaxPlan`
- Avoid overly long names

### Good vs Bad

```json
// Wrong
"minimax-cn-coding-plan": { }
"openai-pro": { }
"zhipu-plan": { }

// Correct
"minimax": { }
"minimaxPlan": { }
```

## Core Operations

### Query Current Config

```bash
cat ~/.config/opencode/opencode.json
cat ~/.local/share/opencode/auth.json | jq 'map_values(.key = "***")'
```

### Add Provider

**opencode.json** - add provider:
```json
{
  "provider": {
    "provider-id": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Display Name",
      "options": {
        "baseURL": "https://api.example.com/v1"
      },
      "models": {
        "model-id": { "name": "Model Display Name" }
      }
    }
  }
}
```

**auth.json** - add credentials:
```json
{
  "provider-id": {
    "type": "api",
    "key": "your-api-key"
  }
}
```

### Update Model List

Add models to existing provider:

```json
{
  "provider": {
    "existing-provider": {
      "models": {
        "new-model": { "name": "New Model" }
      }
    }
  }
}
```

### Modify API Key

Edit `auth.json`:
```json
{
  "provider-id": {
    "type": "api",
    "key": "new-key"
  }
}
```

### Delete Provider

Delete corresponding entries from **both** files simultaneously.

### Rename Provider (Important)

**Must update both files synchronously:**
1. Provider ID in `opencode.json`
2. Provider ID in `auth.json` (must match)
3. `model` field in `opencode.json` (if it references this provider)

## Common Provider Templates

### Anthropic-compatible API (Plan/Premium)

```json
{
  "provider": {
    "minimaxPlan": {
      "npm": "@ai-sdk/anthropic",
      "name": "MiniMax Plan",
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

### OpenAI-compatible API (Standard)

```json
{
  "provider": {
    "minimax": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "MiniMax Standard API",
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

### Common Domestic Providers

| Provider | baseURL | npm |
|----------|---------|-----|
| Zhipu AI | `https://open.bigmodel.cn/api/paas/v4` | `@ai-sdk/openai-compatible` |
| DeepSeek | `https://api.deepseek.com/v1` | `@ai-sdk/openai-compatible` |
| SiliconFlow | `https://api.siliconflow.cn/v1` | `@ai-sdk/openai-compatible` |
| Moonshot | `https://api.moonshot.cn/v1` | `@ai-sdk/openai-compatible` |

## Get Models Supported by Provider

1. Visit official documentation's API/Models page
2. Confirm baseURL and model ID

Common docs:
| Provider | Documentation |
|----------|---------------|
| MiniMax | https://platform.minimaxi.com/docs |
| Zhipu AI | https://open.bigmodel.cn/dev/api |
| DeepSeek | https://platform.deepseek.com/docs |
| SiliconFlow | https://docs.siliconflow.cn |

## Verify Config

```bash
opencode
/models
```

Confirm provider and models appear in the list.

## Troubleshooting

| Symptom | Cause | Solution |
|---------|-------|----------|
| Model not showing | JSON syntax error, provider ID mismatch | Check if provider IDs match in both files |
| Auth failed | API key incorrect or expired | Check key in auth.json |
| Connection timeout | baseURL unreachable | Open baseURL in browser to confirm |
| Duplicate provider | Old config not removed when manually editing | Clean up duplicate configs |
| Plan API not showing | Provider ID naming non-standard | Use CamelCase like `minimaxPlan` |

### Environment Variable Check

Some providers (like MiniMax Plan) require clearing Anthropic env vars:
```bash
env | grep -i anthropic
# If ANTHROPIC_AUTH_TOKEN or ANTHROPIC_BASE_URL exists, clear them
```

## Config Checklist

- [ ] Provider ID naming follows convention
- [ ] Provider IDs match exactly in both files
- [ ] No duplicate provider configs
- [ ] No extra brackets or commas
- [ ] baseURL format correct (`/v1` or `/anthropic/v1`)
- [ ] npm package type correct (`@ai-sdk/openai-compatible` or `@ai-sdk/anthropic`)