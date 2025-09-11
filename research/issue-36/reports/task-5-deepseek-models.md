# DeepSeek Models Research Report

## Overview
DeepSeek offers a comprehensive family of open-source large language models featuring advanced reasoning capabilities, cost-effective pricing, and state-of-the-art performance. The latest models combine traditional language understanding with sophisticated chain-of-thought reasoning.

## Model Specifications

### DeepSeek-V3.1 (Latest Flagship - August 2025)

#### deepseek-chat (Non-thinking Mode)
- **Parameters**: 671B total / 37B activated (MoE architecture)
- **Input Pricing**: $0.56 per million tokens (cache miss) / $0.07 per million tokens (cache hit)
- **Output Pricing**: $1.68 per million tokens
- **Context Window**: 128,000 tokens
- **Output Limit**: Default 4K, Maximum 8K tokens
- **Multimodal Support**: ❌ Text only (multimodal in development)
- **Processing Speed**: Fast (standard mode)
- **Features**: JSON Output, Function Calling, FIM Completion (Beta), Chat Prefix Completion (Beta)

#### deepseek-reasoner (Thinking Mode)
- **Parameters**: 671B total / 37B activated (MoE architecture)
- **Input Pricing**: $0.56 per million tokens (cache miss) / $0.07 per million tokens (cache hit)
- **Output Pricing**: $1.68 per million tokens
- **Context Window**: 128,000 tokens
- **Output Limit**: Default 32K, Maximum 64K tokens (includes CoT reasoning)
- **Multimodal Support**: ❌ Text only
- **Processing Speed**: Slower due to reasoning process
- **Features**: Chain-of-thought reasoning, self-verification, reflection, JSON Output, Chat Prefix Completion (Beta)

### DeepSeek-V3 (Previous Generation)
- **Parameters**: 671B total / 37B activated (MoE architecture)
- **Input Pricing**: $0.14 per million tokens
- **Output Pricing**: $0.28 per million tokens
- **Context Window**: 64,000 - 128,000 tokens
- **Output Limit**: Up to 8,000 tokens
- **Training Data**: 14.8 trillion tokens
- **Training Cost**: 2.788M H800 GPU hours
- **Performance**: Strong across all context lengths

### DeepSeek-R1 Series (Reasoning Specialist)

#### DeepSeek-R1 (Full Model)
- **Parameters**: 671B total / 37B activated (based on V3-Base)
- **Context Window**: Variable, supports long contexts
- **Output Limit**: Extended for reasoning tasks
- **Multimodal Support**: ✅ Multimodal learning (text, images, speech, video)
- **Processing Speed**: Variable depending on reasoning complexity
- **Architecture**: Pure Reinforcement Learning (RL) based
- **Features**: Chain-of-thought reasoning, self-verification, reflection

#### DeepSeek-R1-0528 (Updated May 2025)
- **Parameters**: Available in 671B and 8B variants
- **Improvements**: Enhanced reasoning and inference capabilities
- **Features**: Advanced post-training optimizations
- **Performance**: Comparable to OpenAI-o1 on math, code, reasoning tasks

#### DeepSeek-R1 Distilled Models
- **Available Sizes**: 1.5B, 7B, 8B, 14B, 32B, 70B
- **Base Models**: Qwen2.5 and Llama3 series
- **Use Case**: Resource-constrained deployments
- **Features**: Retained reasoning capabilities at smaller scale

## Key Features and Capabilities

### Hybrid Architecture (V3.1)
- **Mode Switching**: Single model can switch between thinking and non-thinking modes
- **Template Control**: Mode selection via chat template changes
- **Versatility**: Covers both general-purpose and reasoning-heavy use cases

### Advanced Reasoning (R1 Series)
- **Chain-of-Thought**: Native CoT reasoning capabilities
- **Self-Verification**: Built-in response validation
- **Reflection**: Ability to reconsider and improve responses
- **Long-Context Reasoning**: Maintains reasoning quality across extended contexts

### Cost Optimization Features
- **Context Caching**: 99% cost reduction for cached responses ($0.014 per million tokens)
- **MoE Efficiency**: Only 37B parameters activated per token
- **Free Tier**: 5 million tokens monthly for active accounts
- **Competitive Pricing**: 85% cost savings vs GPT-4

### Extended Context Training (V3.1)
- **32K Extension**: 630B training tokens (10x increase)
- **128K Extension**: 209B training tokens (3.3x increase)
- **Context Performance**: Strong performance across all context lengths

## Performance Benchmarks

### Coding Performance
- **HumanEval**: 82.6% (V3.1) vs 80.5% (GPT-4)
- **SWE-bench**: Competitive with closed-source models
- **Code Generation**: Strong multi-language support

### Reasoning Performance
- **Math Tasks**: Comparable to OpenAI-o1
- **Science Reasoning**: State-of-the-art open-source performance
- **Complex Problem Solving**: Enhanced through RL training

### Multimodal Performance (R1)
- **Vision Understanding**: Basic multimodal capabilities
- **Document Analysis**: Text + image processing
- **Multi-Input Processing**: Text, images, speech, video support

## Training and Development

### Training Scale
- **V3 Pre-training**: 14.8 trillion tokens
- **V3.1 Extended**: Additional context-specific training
- **R1 Training**: Pure reinforcement learning approach
- **Data Quality**: Diverse, high-quality training data

### Architecture Innovation
- **Mixture of Experts**: Efficient parameter utilization
- **Reinforcement Learning**: Advanced reasoning capabilities
- **Context Scaling**: Progressive context window expansion
- **Distillation**: Knowledge transfer to smaller models

## Deployment Options

### API Access
- **Official API**: DeepSeek API platform
- **Third-Party**: Various cloud providers
- **Pricing Model**: Pay-per-token with caching discounts

### Open Source Availability
- **Hugging Face**: Model weights and documentation
- **GitHub**: Official repositories and code
- **Ollama**: Local deployment support
- **License**: Apache 2.0 (most models)

### Hardware Requirements
- **Full Models**: Requires significant GPU resources
- **Distilled Models**: Various sizes for different hardware constraints
- **Edge Deployment**: Smaller variants available

## Future Developments
- **Multimodal Expansion**: Full multimodal support planned
- **Model Scaling**: Larger parameter variants in development
- **Specialized Variants**: Domain-specific fine-tuned models
- **Performance Optimization**: Continued efficiency improvements

## References
- [DeepSeek V3.1 Release](https://api-docs.deepseek.com/news/news250821)
- [DeepSeek V3 Introduction](https://api-docs.deepseek.com/news/news1226)
- [DeepSeek API Pricing](https://api-docs.deepseek.com/quick_start/pricing/)
- [DeepSeek R1 Research Paper](https://arxiv.org/pdf/2501.12948)
- [DeepSeek V3 on Hugging Face](https://huggingface.co/deepseek-ai/DeepSeek-V3)
- [DeepSeek R1 GitHub Repository](https://github.com/deepseek-ai/DeepSeek-R1)