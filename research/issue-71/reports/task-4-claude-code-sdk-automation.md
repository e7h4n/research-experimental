# Task 4: Claude Code SDK 与自动化工作流集成

## Claude Code SDK 概述

根据 [Anthropic 官方文档](https://docs.anthropic.com/en/docs/claude-code/sdk)，Claude Code SDK 提供了构建生产就绪 AI 代理所需的所有构建块，允许从应用程序中以非交互模式与 Claude Code 进行接口交互。

## 核心功能

### 1. SDK 选项

根据 [SDK 文档](https://docs.anthropic.com/en/docs/claude-code/sdk)，支持多种模式：
- **Headless Mode**：用于 CLI 脚本和自动化
- **TypeScript SDK**：用于 Node.js 和 Web 应用
- **Python SDK**：用于数据科学和 Python 应用

### 2. 自动化能力

Claude Code 提供了强大的自动化支持：
- 直接编辑文件、运行命令和创建提交
- MCP (Model Context Protocol) 支持，可读取 Google Drive 文档、更新 Jira 票据
- 遵循 Unix 哲学 - 可组合和可脚本化

## GitHub Actions 集成

### 快速设置

根据 [GitHub Actions 文档](https://docs.anthropic.com/en/docs/claude-code/github-actions)：

```bash
# 在终端中运行
/install-github-app
```

### 手动配置

1. 安装 Claude GitHub 应用
2. 添加 `ANTHROPIC_API_KEY` 到仓库 secrets
3. 创建工作流文件 `.github/workflows/claude.yml`：

```yaml
name: Claude Code
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]

jobs:
  claude:
    runs-on: ubuntu-latest
    if: contains(github.event.comment.body, '@claude')
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          claude_args: |
            --max-turns 10
            --model claude-3-sonnet-20240620
```

### 高级配置选项

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    claude_args: |
      --system "You are a helpful coding assistant"
      --allowed-tools Read,Write,Edit,Bash
      --disallowed-tools WebSearch
      --max-turns 15
      --permission-mode strict
```

## 非交互式模式使用

### 基本命令行使用

```bash
# 使用 --print 或 -p 标志进入非交互模式
claude --print "分析代码并生成测试"

# 带系统提示
claude --print \
  --system "你是一个专注于代码质量的助手" \
  "审查这个项目的代码质量"
```

### 脚本集成示例

```bash
#!/bin/bash
# automated-review.sh

# 设置环境变量
export ANTHROPIC_API_KEY="your-api-key"

# 执行代码审查
result=$(claude --print \
  --allowed-tools Read,Grep \
  --max-turns 5 \
  "审查 src/ 目录下的所有 TypeScript 文件")

# 将结果保存到文件
echo "$result" > review-report.md
```

## CI/CD 管道集成

### NPM Scripts 集成

根据 [社区实践](https://collabnix.com/claude-code-the-complete-developers-guide-to-getting-started-with-anthropics-revolutionary-ai-coding-assistant/)，可以配置 npm scripts：

```json
{
  "scripts": {
    "ai:review": "claude --print 'Review code quality'",
    "ai:test": "claude --print 'Generate unit tests'",
    "ai:fix": "claude --print 'Fix ESLint errors'"
  }
}
```

### 权限配置

在 `.claude/settings.json` 中配置：

```json
{
  "allowedTools": [
    "Read",
    "Write",
    "Edit",
    "Bash(npm run lint)",
    "Bash(npm run test:*)",
    "Bash(npm run build)"
  ],
  "disallowedTools": [
    "Bash(rm -rf *)",
    "Bash(git push)"
  ]
}
```

## 高级自动化模式

### 1. 自动化日报生成

```yaml
name: Daily Report
on:
  schedule:
    - cron: '0 9 * * *'

jobs:
  report:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          claude_args: |
            --print
            --system "生成项目进度日报"
            "分析最近24小时的提交和 PR，生成进度报告"
```

### 2. PR 自动审查

```yaml
name: PR Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          claude_args: |
            --allowed-tools Read,Grep
            "审查这个 PR 的代码变更"
```

### 3. 自动化测试生成

```bash
# test-generation.sh
claude --print \
  --allowed-tools Read,Write \
  --system "你是一个测试专家" \
  "为 src/components 目录生成单元测试"
```

## 配置管理

### CLAUDE.md 文件

在项目根目录创建 `CLAUDE.md` 定义持久指令：

```markdown
# Claude Code 项目规范

## 代码标准
- 使用 TypeScript
- 遵循 ESLint 规则
- 所有函数需要 JSDoc 注释

## 测试要求
- 测试覆盖率 > 80%
- 使用 Jest 框架
- 遵循 AAA 模式

## Git 规范
- 使用 conventional commits
- PR 需要通过所有测试
```

### 环境变量配置

```bash
# .env.claude
export ANTHROPIC_API_KEY="sk-xxx"
export ANTHROPIC_MODEL="claude-3-sonnet-20240620"
export CLAUDE_MAX_TURNS="10"
export CLAUDE_PERMISSION_MODE="strict"
```

## Hooks 系统

### 配置 Hooks

在 `.claude/settings.json` 中：

```json
{
  "hooks": {
    "afterEdit": "npm run format",
    "beforeCommit": "npm run test",
    "afterPython": "black {file}"
  }
}
```

## 企业级功能

根据 [官方文档](https://docs.anthropic.com/en/docs/claude-code/overview)：

- **多云支持**：AWS Bedrock、Google Vertex AI
- **Git 集成**：GitHub、GitLab 原生支持
- **安全性**：细粒度权限控制
- **可扩展性**：MCP 协议支持自定义工具

## 最佳实践建议

### 1. 成本控制

```yaml
claude_args: |
  --max-turns 10  # 限制交互次数
  --timeout 300   # 设置超时（秒）
```

### 2. 安全配置

```json
{
  "permissionMode": "strict",
  "disallowedTools": [
    "Bash(sudo *)",
    "Bash(rm -rf /)",
    "Write(/etc/*)"
  ]
}
```

### 3. 监控和日志

```bash
# 启用详细日志
claude --print --verbose \
  --log-file claude.log \
  "执行任务"
```

## 与第三方路由集成

虽然 Claude Code SDK 主要设计用于 Anthropic API，但可以通过环境变量配置与第三方路由结合：

```bash
# 使用 y-router
export ANTHROPIC_BASE_URL="https://cc.yovy.app"
export ANTHROPIC_API_KEY="openrouter-key"

# 使用 Claude Code Router
ccr code --print "执行自动化任务"
```

## 参考链接

- [Claude Code SDK 文档](https://docs.anthropic.com/en/docs/claude-code/sdk)
- [GitHub Actions 集成](https://docs.anthropic.com/en/docs/claude-code/github-actions)
- [Claude Code 概览](https://docs.anthropic.com/en/docs/claude-code/overview)
- [最佳实践](https://www.anthropic.com/engineering/claude-code-best-practices)