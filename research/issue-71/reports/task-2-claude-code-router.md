# Task 2: Claude Code Router 深度研究报告

## 概述

Claude Code Router 是一个功能更加强大的路由解决方案，根据 [官方 GitHub 仓库](https://github.com/musistudio/claude-code-router) 的描述，它允许"使用 Claude Code 作为编码基础设施的基础，让您决定如何与模型交互，同时享受来自 Anthropic 的更新"。

## 核心特性

### 主要功能

根据 [Claude Code Router 文档](https://github.com/musistudio/claude-code-router)，主要特性包括：

1. **模型路由**：根据不同需求（后台任务、思考、长上下文）路由到不同模型
2. **多提供商支持**：支持 OpenRouter、DeepSeek、Ollama、Gemini、Volcengine、SiliconFlow 等
3. **请求/响应转换**：使用 transformers 自定义请求和响应
4. **环境变量插值**：安全管理 API 密钥
5. **子代理路由**：通过特殊标签在子代理中指定模型

## 安装与配置

### 基础安装

```bash
# 安装 Claude Code
npm install -g @anthropic-ai/claude-code

# 安装 Claude Code Router
npm install -g @musistudio/claude-code-router
```

### 完整配置示例

根据 [配置文档](https://github.com/musistudio/claude-code-router)，创建 `~/.claude-code-router/config.json`：

```json
{
  "Providers": [
    {
      "name": "openrouter",
      "api_base_url": "https://openrouter.ai/api/v1/chat/completions",
      "api_key": "sk-xxx",
      "models": [
        "google/gemini-2.5-pro-preview",
        "anthropic/claude-sonnet-4",
        "anthropic/claude-3.5-sonnet",
        "anthropic/claude-3.7-sonnet:thinking"
      ],
      "transformer": {
        "use": ["openrouter"]
      }
    },
    {
      "name": "deepseek",
      "api_base_url": "https://api.deepseek.com/chat/completions",
      "api_key": "sk-xxx",
      "models": ["deepseek-chat", "deepseek-reasoner"],
      "transformer": {
        "use": ["deepseek"],
        "deepseek-chat": {
          "use": ["tooluse"]
        }
      }
    },
    {
      "name": "ollama",
      "api_base_url": "http://localhost:11434/v1/chat/completions",
      "api_key": "ollama",
      "models": ["qwen2.5-coder:latest"]
    },
    {
      "name": "gemini",
      "api_base_url": "https://generativelanguage.googleapis.com/v1beta/models/",
      "api_key": "sk-xxx",
      "models": ["gemini-2.5-flash", "gemini-2.5-pro"],
      "transformer": {
        "use": ["gemini"]
      }
    }
  ],
  "Router": {
    "default": "openrouter,anthropic/claude-3.5-sonnet",
    "background": "ollama,qwen2.5-coder:latest",
    "think": "deepseek,deepseek-reasoner",
    "longContext": "gemini,gemini-2.5-pro",
    "longContextThreshold": 60000
  }
}
```

## 高级功能

### API 密钥轮换

支持多个 API 密钥轮换以避免限制：

```json
{
  "Providers": [
    {
      "name": "gemini",
      "api_base_url": "https://generativelanguage.googleapis.com/v1beta/models/",
      "api_keys": [
        "YOUR_GEMINI_API_KEY_1",
        "YOUR_GEMINI_API_KEY_2",
        "YOUR_GEMINI_API_KEY_3"
      ],
      "enable_rotation": true,
      "rotation_strategy": "round_robin",
      "retry_on_failure": true,
      "max_retries": 3,
      "models": ["gemini-2.5-pro", "gemini-2.5-flash"]
    }
  ]
}
```

### 自定义路由逻辑

使用 JavaScript 编写自定义路由规则：

```javascript
// custom-router.js
module.exports = async function router(req, config) {
  const userMessage = req.body.messages.find((m) => m.role === "user")?.content;
  
  if (userMessage && userMessage.includes("explain this code")) {
    // 代码解释使用强大的模型
    return "openrouter,anthropic/claude-3.5-sonnet";
  }
  
  if (userMessage && userMessage.includes("write unit test")) {
    // 单元测试使用快速模型
    return "ollama,qwen2.5-coder:latest";
  }
  
  // 回退到默认配置
  return null;
};
```

### 子代理路由

在子代理提示中指定特定模型：

```markdown
<CCR-SUBAGENT-MODEL>openrouter,anthropic/claude-3.5-sonnet</CCR-SUBAGENT-MODEL>

分析这段代码并提供优化建议...
```

## 运行命令

```bash
# 启动后台服务
ccr start

# 停止服务
ccr stop

# 查看状态和 API 密钥轮换信息
ccr status

# 查看详细的 API 密钥轮换状态
ccr rotation

# 执行 Claude Code（自动启动服务）
ccr code "编写一个 Hello World 程序"

# 打开配置界面
ccr ui
```

## 动态模型切换

在运行时可以使用命令切换模型：

```
/model deepseek,deepseek-chat
/model openrouter,anthropic/claude-3.5-sonnet
/model ollama,qwen2.5-coder:latest
```

## 免费模型使用

根据 [iFlow Platform 集成](https://github.com/musistudio/claude-code-router)，可以免费使用以下模型：
- GLM-4.5
- Kimi-K2
- Qwen3-Coder-480B-A35B
- DeepSeek v3.1

## 已知限制

### 并发限制

根据文档，iFlow Platform 限制每个用户的并发为 1，这意味着需要将后台请求路由到其他模型。

### 配置冲突

当同时设置令牌（`ANTHROPIC_AUTH_TOKEN`）和 API 密钥时，可能导致认证冲突。

### 模型兼容性

某些模型不支持工具调用功能，需要使用相应的 transformer 来适配。

## Transformers 说明

内置 transformers 包括：
- `maxtoken`：处理最大令牌限制
- `tooluse`：处理工具使用能力
- `deepseek`：DeepSeek 特定优化
- `openrouter`：OpenRouter 协议适配
- `gemini`：Gemini API 适配

## 最佳实践

1. **模型选择策略**：
   - 复杂推理任务使用高端模型
   - 简单代码生成使用快速模型
   - 长文本处理使用专门的长上下文模型

2. **成本优化**：
   - 使用 API 密钥轮换避免单一账户限制
   - 合理配置路由规则，平衡性能和成本

3. **错误处理**：
   - 配置重试机制
   - 设置合理的超时时间

## 社区集成案例

根据 [GitHub 上的项目](https://github.com/St1ma/claude-code-openrouter-code-reviewer)，有开发者结合 Claude Code Router 和 OpenRouter 创建了完全免费的代码审查解决方案。

## 参考链接

- [Claude Code Router GitHub](https://github.com/musistudio/claude-code-router)
- [Claude Code Router 官网](https://claudecoderouter.com/)
- [配置示例](https://github.com/musistudio/claude-code-router/blob/main/config.example.json)