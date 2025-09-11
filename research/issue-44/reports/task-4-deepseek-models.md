# DeepSeek Models Research Report

## DeepSeek V3.1 Series (August 2025)

### DeepSeek-V3.1 Chat (deepseek-chat)
- **Input Price**: $0.56 per million tokens (cache miss), $0.07 per million tokens (cache hit)
- **Output Price**: $1.68 per million tokens
- **Input Token Limit**: 128K tokens (4x increase from previous version)
- **Output Token Limit**: Default 4K, Maximum 8K tokens
- **Token Speed**: Not specified
- **Multimodal**: Text-only (separate multimodal models exist)
- **Key Features**: Generalist model, 685 billion parameters, 82.6% HumanEval score (beats GPT-4's 80.5%)
- **Context Caching**: Up to 90% cost reduction for repeated queries

### DeepSeek-V3.1 Reasoner (deepseek-reasoner)
- **Input Price**: $0.56 per million tokens (cache miss), $0.07 per million tokens (cache hit)
- **Output Price**: $1.68 per million tokens
- **Input Token Limit**: 128K tokens
- **Output Token Limit**: Default 32K, Maximum 64K tokens
- **Token Speed**: Not specified
- **Multimodal**: Text-only
- **Key Features**: Thinking mode, integrated deep reasoning, advanced math and coding capabilities

## DeepSeek V3 Series (December 2024)

### DeepSeek-V3 (deepseek-chat legacy)
- **Input Price**: $0.14 per million tokens
- **Output Price**: $0.28 per million tokens
- **Input Token Limit**: 128K tokens
- **Output Token Limit**: Not specified
- **Token Speed**: Not specified
- **Multimodal**: Text-only
- **Architecture**: 671B total parameters, 37B activated per token
- **Training**: 14.8 trillion high-quality tokens with MLA and DeepSeekMoE architectures
- **Key Features**: Cost-effective general chat and conversation model

## DeepSeek-R1 Series (January 2025)

### DeepSeek-R1 (deepseek-reasoner)
- **Input Price**: $0.55 per million tokens (cache miss), $0.14 per million tokens (cache hit)
- **Output Price**: $2.19 per million tokens
- **Input Token Limit**: Not specified
- **Output Token Limit**: Not specified
- **Token Speed**: Not specified
- **Multimodal**: Text-only
- **Performance**: On par with OpenAI o1 model
- **Key Features**: Advanced reasoning, math, and coding tasks
- **License**: MIT License (allows free distillation and commercial use)
- **Training**: Large-scale RL in post-training with minimal labeled data

### DeepSeek-R1 Distilled Models
- **32B Model**: Performance on par with OpenAI o1-mini
- **70B Model**: Performance on par with OpenAI o1-mini
- **Licensing**: Open-source under MIT License
- **Availability**: 6 distilled small models released

## Cost Optimization Features
- **Context Caching**: Up to 90% cost reduction for repeated queries
- **Cache Hit Pricing**: Significantly reduced costs for cached inputs
- **JSON Output**: Structured output capabilities
- **Function Calling**: Available on select models

## Performance Benchmarks
- **HumanEval**: 82.6% (V3.1) vs GPT-4's 80.5%
- **Context Performance**: Excellent performance across all context lengths up to 128K
- **Reasoning Tasks**: R1 performs comparably to OpenAI o1

## Technical Architecture
- **V3 Architecture**: Multi-head Latent Attention (MLA) and DeepSeekMoE
- **Parameters**: 671B total (V3), 37B activated per token
- **Training Data**: 14.8 trillion diverse, high-quality tokens (V3)

## API Access
- **Base URL**: api.deepseek.com
- **Models**: 
  - deepseek-chat (V3.1 non-thinking mode)
  - deepseek-reasoner (V3.1 thinking mode, R1)
- **Web Interface**: chat.deepseek.com

## Multimodal Capabilities
- **Current Models**: Text-only
- **Future Plans**: Multimodal support mentioned in roadmap
- **Separate Offering**: Janus-Pro multimodal model (separate from main API)

## Licensing and Availability
- **R1 Models**: MIT License (commercial use allowed)
- **V3 Series**: Commercial API licensing
- **Open Source**: Technical reports and smaller distilled models available
- **Distillation**: Free distillation allowed for R1 models

## Competitive Positioning
- **Cost Advantage**: $1.68/million tokens vs GPT-4's $10/million tokens
- **Performance**: Matches or exceeds GPT-4 on key benchmarks
- **Open Source**: Unique in offering high-performance open-source alternatives
- **Transparency**: Full technical reports and open model weights

## References
- [DeepSeek V3.1 Release](https://api-docs.deepseek.com/news/news250821)
- [DeepSeek-R1 Release](https://api-docs.deepseek.com/news/news250120)
- [DeepSeek Pricing](https://api-docs.deepseek.com/quick_start/pricing/)
- [DeepSeek V3 Introduction](https://api-docs.deepseek.com/news/news1226)