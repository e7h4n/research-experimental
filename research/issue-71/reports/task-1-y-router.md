# Task 1: y-router 深度研究报告

## 概述

y-router 是一个轻量级的代理解决方案，通过 Cloudflare Worker 实现 Anthropic Claude API 与 OpenAI 兼容 API 之间的协议转换。根据 [GitHub 仓库](https://github.com/luohy15/y-router) 的描述，它是"一个简单的代理，使 Claude Code 能够与 OpenRouter 配合使用"。

## 核心功能与架构

### 工作原理

y-router 的核心机制包括：
- **请求接收**：接受 Anthropic API 格式的请求
- **协议转换**：将请求转换为 OpenAI chat completions 格式
- **请求转发**：转发到 OpenRouter 或其他兼容 API
- **响应转换**：将响应转换回 Anthropic 格式
- **流式支持**：同时支持流式和非流式响应

### 技术架构

基于 Cloudflare Workers 构建，提供边缘计算能力，确保低延迟和高可用性。

## 安装与部署

### 方法一：一键安装（推荐）

```bash
bash -c "$(curl -fsSL https://cc.yovy.app/install.sh)"
```

### 方法二：手动配置

```bash
# 安装 Claude Code
npm install -g @anthropic-ai/claude-code

# 设置环境变量
export ANTHROPIC_BASE_URL="https://cc.yovy.app"
export ANTHROPIC_API_KEY="your-openrouter-api-key"
export ANTHROPIC_CUSTOM_HEADERS="x-api-key: $ANTHROPIC_API_KEY"
```

### 方法三：Docker 部署

```bash
git clone https://github.com/luohy15/y-router
cd y-router
docker-compose up -d
```

### 方法四：Cloudflare Workers 部署

```bash
git clone https://github.com/luohy15/y-router
cd y-router
npm install -g wrangler
wrangler deploy
```

## 配置选项

主要配置参数：
- `OPENROUTER_BASE_URL`：目标 API 的基础 URL（默认为 OpenRouter）
- 模型配置：通过环境变量指定具体模型
- 支持多个配置别名

## 使用示例

### 基本 API 调用

```bash
curl -X POST https://cc.yovy.app/v1/messages \
  -H "Content-Type: application/json" \
  -H "x-api-key: your-openrouter-key" \
  -d '{
    "model": "claude-sonnet-4-20250514",
    "messages": [{"role": "user", "content": "Hello, Claude"}],
    "max_tokens": 1024
  }'
```

### 在 Claude Code 中使用

配置环境变量后，直接使用 Claude Code：

```bash
claude "请帮我编写一个 Python 脚本"
```

## 局限性与注意事项

### 模型兼容性问题

根据社区反馈，存在以下限制：
- **工具能力支持**：只有 Claude、GPT-4o 系列和部分 Gemini 模型支持工具调用功能
- **纯文本模型**：许多免费模型（如 Qwen、DeepSeek）只能处理文本生成，不支持工具执行
- **错误提示**："No endpoints found that support" 错误通常表示模型不兼容高级功能

### 使用建议

根据 [y-router 文档](https://github.com/luohy15/y-router)：
- 适合测试非 Anthropic 模型
- 密集使用（超过 $200）时，建议使用 claude-relay-service 以获得更好的性价比
- 日常使用建议部署自己的实例以确保可靠性

### 性能考虑

- **并发限制**：使用免费服务时可能存在并发限制
- **超时错误**：对于较慢的模型，需要增加 `API_TIMEOUT_MS`
- **连接问题**：需要确保路由器正在运行且配置正确

### 安全性声明

y-router 明确声明：
- 是独立的非官方工具
- 与 Anthropic PBC、OpenAI 或 OpenRouter 无关联
- 按"原样"提供，不提供任何保证

## 实际应用场景

1. **成本优化**：使用更便宜的模型进行简单任务
2. **模型测试**：快速测试不同 LLM 的能力
3. **混合部署**：在不同任务中使用不同的模型

## 社区反馈

根据 GitHub issues 和讨论，用户报告的主要问题包括：
- API key 配置问题
- 模型名称不匹配
- 某些高级功能的兼容性问题

## 参考链接

- [y-router GitHub 仓库](https://github.com/luohy15/y-router)
- [OpenRouter 模型列表](https://openrouter.ai/models)
- [y-router 安装脚本](https://cc.yovy.app/install.sh)