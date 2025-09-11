# Claude Models Research Report

## Overview
Anthropic's Claude models represent some of the most advanced AI systems available, with a range of options from high-performance reasoning to fast, cost-effective processing.

## Model Specifications

### Claude Opus 4.1 (Flagship Model)
- **Input Pricing**: $15 per million tokens
- **Output Pricing**: $75 per million tokens
- **Context Window**: 200,000 tokens (standard)
- **Output Limit**: Standard (model-specific)
- **Multimodal Support**: ✅ Vision/image input, text output
- **Special Features**: Highest intelligence level, extended thinking capability
- **Processing Speed**: High-quality (slower than Sonnet/Haiku)

### Claude Sonnet 4 (Balanced Performance)
- **Input Pricing**: $3 per million tokens
- **Output Pricing**: $15 per million tokens
- **Context Window**: 200,000 tokens (1M tokens in beta with `context-1m-2025-08-07` header)
- **Extended Context Pricing**: $6 input / $22.50 output per million tokens (for >200K tokens)
- **Output Limit**: Standard (128K tokens available with beta header)
- **Multimodal Support**: ✅ Vision/image input, text output
- **Processing Speed**: Fast response times, balanced performance

### Claude Sonnet 3.7
- **Input Pricing**: $3 per million tokens
- **Output Pricing**: $15 per million tokens
- **Context Window**: 200,000 tokens
- **Output Limit**: 128,000 tokens (with `output-128k-2025-02-19` beta header)
- **Multimodal Support**: ✅ Vision/image input, text output
- **Processing Speed**: Fast

### Claude Haiku 3.5 (Speed Optimized)
- **Input Pricing**: $0.80 per million tokens (some sources show $0.25)
- **Output Pricing**: $4 per million tokens (some sources show $1.25)
- **Context Window**: 200,000 tokens
- **Output Limit**: Standard
- **Multimodal Support**: ✅ Vision/image input, text output
- **Processing Speed**: "Blazing speeds" - fastest model in the lineup

## Cost Optimization Features

### Prompt Caching
- **Cache Read Cost**: 0.1x base input price
- **Cache Duration**: 5-minute and 1-hour options
- **Savings**: Up to 90% for repeated prompts

### Batch API
- **Discount**: 50% on both input and output tokens
- **Use Case**: Non-time-sensitive bulk processing
- **Access**: Available for all models

## Token Calculation
- **Conversion**: ~1 token = 4 characters = 0.75 words (English)
- **Example**: 1,000 tokens ≈ 750 words ≈ 2-3 pages of text

## Additional Costs
- **Web Search**: $10 per 1,000 searches (Claude.ai feature)
- **Tool Use**: Additional system prompt tokens

## Enterprise Features
- Custom rate limits and pricing available
- 500K context window for Enterprise Claude Sonnet 4 users
- Enhanced support and SLA

## References
- [Anthropic Official Pricing](https://docs.anthropic.com/en/docs/about-claude/pricing)
- [Claude Models Overview](https://docs.anthropic.com/en/docs/about-claude/models/overview)
- [Context Windows Documentation](https://docs.anthropic.com/en/docs/build-with-claude/context-windows)
- [1M Token Context Announcement](https://www.anthropic.com/news/1m-context)