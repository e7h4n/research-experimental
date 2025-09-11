# Chinese AI Models Research Report

## Kimi K2 (Moonshot AI)

### Kimi K2
- **Input Price**: $0.15 per million tokens (cache hit), $0.60 per million tokens (cache miss)
- **Output Price**: $2.50 per million tokens
- **Input Token Limit**: 128K tokens (130K tokens)
- **Output Token Limit**: Not specified (API examples show 256 tokens max)
- **Token Speed**: 53.3 tokens per second (output), 0.53s TTFT
- **Multimodal**: Text-only currently (images/audio planned)
- **Architecture**: 32B activated parameters, 1T total parameters (MoE)
- **Key Features**: State-of-the-art performance in frontier knowledge, math, coding
- **License**: Modified MIT License (open-source)
- **API**: OpenAI/Anthropic-compatible

## GLM4 Series (Zhipu AI)

### GLM-4.5
- **Input Price**: Not specified for main model
- **Output Price**: Not specified for main model
- **Input Token Limit**: 1M tokens context
- **Output Token Limit**: 96K tokens output
- **Token Speed**: Not specified
- **Multimodal**: Yes (text, images, other media types)
- **Architecture**: 355B total parameters, 32B active (MoE)
- **Key Features**: Agentic AI, function calling, web browsing, code execution
- **Tool Success Rate**: 90.6% in coding tasks
- **Training**: 15 trillion tokens

### GLM-4-Long
- **Input Price**: ¥0.001 per 1,000 tokens ($0.14 per million tokens)
- **Output Price**: ¥0.001 per 1,000 tokens ($0.14 per million tokens)
- **Input Token Limit**: 128K to 1M tokens (expanded context)
- **Output Token Limit**: Not specified
- **Token Speed**: Not specified
- **Multimodal**: Yes (text, images with GLM-4-9B variant)
- **Key Features**: Ultra-low pricing, economical solution

### GLM-4-9B (Multimodal)
- **Languages**: 26 languages supported
- **Multimodal**: Yes (text and image without additional visual expert)
- **Performance**: Comparable to GPT-4V
- **Key Features**: High-resolution inputs, integrated multimodal processing

## QWEN Series (Alibaba)

### QWEN3-Max-Preview
- **Input Price**: Tiered pricing
  - 0-32K tokens: $0.861 per million tokens
  - 32K-128K tokens: $1.434 per million tokens  
  - 128K-252K tokens: $2.151 per million tokens
- **Output Price**: Tiered pricing
  - 0-32K tokens: $3.441 per million tokens
  - 32K-128K tokens: $5.735 per million tokens
  - 128K-252K tokens: $8.602 per million tokens
- **Input Token Limit**: 258,048 tokens (262,144 context window)
- **Output Token Limit**: 32,768 tokens
- **Token Speed**: Not specified
- **Multimodal**: Not specified
- **Architecture**: 1+ trillion parameters
- **Key Features**: Context caching (25% of original price for cache hits)

### QWEN3 Family
- **Model Sizes**: 0.6B, 1.7B, 4B, 8B, 14B, 32B, 30B-A3B, 235B-A22B
- **Context Window**: Up to 1 million tokens (ultra-long context)
- **Release**: Open-source scheduled for April 2025
- **Architecture**: Dense and MoE variants available

### QWEN3-Coder
- **Input Price**: $0.084 per million tokens
- **Output Price**: $1.575 per million tokens
- **Input Token Limit**: 262K tokens context
- **Output Token Limit**: Not specified
- **Token Speed**: Not specified
- **Multimodal**: Not specified
- **Key Features**: Code generation, Agent capabilities, tool calling, environment interaction

### QWEN2.5-Coder-32B
- **Input Price**: $0.84 per million tokens
- **Output Price**: $0.84 per million tokens
- **Input Token Limit**: 16K tokens
- **Output Token Limit**: Not specified
- **Token Speed**: Not specified
- **Multimodal**: Not specified

## Doubao Series (ByteDance)

### Doubao-1.5-Pro-32K
- **Input Price**: ¥0.8 per million tokens ($0.11 per million tokens)
- **Output Price**: ¥2 per million tokens ($0.28 per million tokens)
- **Input Token Limit**: 256K tokens context window
- **Output Token Limit**: 12K tokens maximum
- **Token Speed**: Not specified
- **Multimodal**: Yes (text, images, video, music generation)
- **Key Features**: 99.8% cheaper than GPT-4, industry-leading price point
- **Market Share**: 46.4% of China's public cloud LLM market

### Doubao-1.5-Lite-32K
- **Input Price**: ¥0.3 per million tokens ($0.042 per million tokens)
- **Output Price**: Not specified
- **Input Token Limit**: 32K tokens
- **Output Token Limit**: Not specified
- **Token Speed**: Not specified
- **Multimodal**: Yes
- **Key Features**: Ultra-lightweight, lowest cost option

### Doubao Enterprise
- **Input Price**: ¥0.0008 per 1,000 tokens ($0.00011 per 1,000 tokens)
- **Output Price**: Not specified
- **Token Speed**: Not specified
- **Multimodal**: Yes
- **Key Features**: 99.3% less than industry average, enterprise-focused

## Multimodal Capabilities Summary

### Doubao Advanced Features
- **Visual Understanding**: Exceptional image interpretation and analysis
- **3D Generation**: Integrated with veOmniverse platform for digital assets
- **Video Generation**: Multi-shot narrative, 1080p HD, cinematic quality
- **Music Generation**: 3-minute complete pieces, multimodal inputs (text, audio, sheet music)
- **Text-to-Image**: v2.1 with Chinese character precision

### GLM-4 Multimodal Features
- **All Tools Module**: Autonomous tool selection
- **Built-in Tools**: Python interpreter, web browser, image generation
- **Agent Capabilities**: Full-stack development, frontend to backend

### Usage Statistics
- **Doubao**: 16.4 trillion daily tokens (137x increase since May 2024)
- **Kimi**: Competitive pricing driving adoption
- **QWEN**: Available through multiple platforms (Qwen Chat, Alibaba Cloud, OpenRouter)

## Cost Optimization Features
- **Context Caching**: QWEN offers 25% pricing for cache hits
- **Free Tiers**: Volcano Engine offers 2 trillion free tokens
- **Volume Discounts**: Enterprise pricing available for high-volume usage
- **API Compatibility**: OpenAI/Anthropic-compatible APIs reduce integration costs

## References
- [Kimi K2 Official](https://moonshotai.github.io/Kimi-K2/)
- [GLM-4.5 Official](https://glm45.org/)
- [Alibaba Cloud Model Studio](https://www.alibabacloud.com/help/en/model-studio/models)
- [ByteDance Volcano Engine](https://www.volcengine.com/)