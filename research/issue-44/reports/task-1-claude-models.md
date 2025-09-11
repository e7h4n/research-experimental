# Claude Models Research Report

## Claude 4 Series

### Claude Opus 4
- **Input Price**: $15 per million tokens
- **Output Price**: $75 per million tokens  
- **Token Limits**: Not specified in official docs
- **Multimodal**: Yes (text, images, audio)
- **Key Features**: Best coding model, leads SWE-bench at 72.5%, can work continuously for hours
- **Availability**: API, Amazon Bedrock, Google Cloud Vertex AI

### Claude Sonnet 4  
- **Input Price**: $3 per million tokens
- **Output Price**: $15 per million tokens
- **Token Limits**: Not specified in official docs
- **Multimodal**: Yes (text, images, audio)
- **Key Features**: Leads SWE-bench at 72.7%, hybrid reasoning modes, parallel tool use
- **Availability**: API, Amazon Bedrock, Google Cloud Vertex AI, free users

## Claude 3.7 Sonnet
- **Input Price**: $3 per million tokens  
- **Output Price**: $15 per million tokens (includes thinking tokens)
- **Output Token Limit**: 128K tokens
- **Input Token Limit**: Not explicitly specified
- **Multimodal**: Yes (text, images, audio)
- **Token Speed**: Output tokens ~8x more expensive due to slower generation (993/second vs 7,934/second input)
- **Key Features**: First hybrid reasoning model, extended thinking mode, 45% reduction in unnecessary refusals
- **Availability**: Claude.ai (all plans), API, Amazon Bedrock, Google Cloud Vertex AI

## Claude 3.5 Series

### Claude 3.5 Sonnet
- **Input Price**: $3 per million tokens (reduced by 25%)
- **Output Price**: $15 per million tokens (reduced by 25%)  
- **Output Token Limit**: 8,192 tokens (beta) / 4,096 tokens (standard)
- **Input Token Limit**: 200K tokens context window
- **Multimodal**: Yes (text, images)
- **Availability**: API, Amazon Bedrock, Google Cloud Vertex AI

### Claude 3.5 Haiku
- **Input Price**: $0.80 per million tokens
- **Output Price**: $4 per million tokens
- **Token Limits**: Standard limits apply
- **Multimodal**: Yes (text, images)
- **Key Features**: Ultra-low latency and price optimized for scale workloads
- **Availability**: API, Amazon Bedrock, Google Cloud Vertex AI

## Rate Limits and Tiers

Claude uses automatic tier progression:
- **Tier 1**: $100/month usage, $5 deposit, 20 RPM, 4K tokens/min
- **Tier 2**: $500/month usage, $40 deposit, 40 RPM, 8K tokens/min  
- **Tier 4**: $5,000/month usage, $400 deposit, 200 RPM, 40K tokens/min

## Cost Optimization Features
- Cache reads: 0.1x base price ($0.30 per 1M tokens for Sonnet 4)
- Batch processing: Half price for non-real-time requests (24h delivery)

## References
- [Claude 4 Official Announcement](https://www.anthropic.com/news/claude-4)
- [Claude 3.7 Sonnet Official Announcement](https://www.anthropic.com/news/claude-3-7-sonnet)
- [Anthropic API Documentation](https://docs.anthropic.com/)