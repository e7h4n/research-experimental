# LLaMA Models Research Report

## LLaMA 3.2 Series (Latest - Released September 2024)

### LLaMA 3.2 1B
- **Input Price**: $0.03 per million tokens (varies by provider)
- **Output Price**: $0.05 per million tokens (varies by provider)  
- **Input Token Limit**: 128K tokens
- **Output Token Limit**: Not specified
- **Token Speed**: Not specified
- **Multimodal**: Text-only
- **Key Features**: Lightweight model for edge deployment
- **AWS Bedrock**: $0.0001 per 1,000 tokens (input/output)

### LLaMA 3.2 3B  
- **Input Price**: $0.06 per million tokens
- **Output Price**: $0.08 per million tokens
- **Input Token Limit**: 128K tokens
- **Output Token Limit**: Not specified
- **Token Speed**: Not specified
- **Multimodal**: Text-only
- **Key Features**: Balanced performance and efficiency

### LLaMA 3.2 11B (Multimodal)
- **Input Price**: Not specified (varies by provider)
- **Output Price**: Not specified (varies by provider)
- **Input Token Limit**: 128K tokens
- **Output Token Limit**: Not specified
- **Token Speed**: Not specified  
- **Multimodal**: Yes (text, images) - document understanding, charts, image captioning
- **AWS Bedrock**: $0.00035 per 1,000 tokens (input/output)
- **Key Features**: Image reasoning, visual grounding tasks

### LLaMA 3.2 90B (Multimodal)
- **Input Price**: $0.12 per million tokens
- **Output Price**: $0.15 per million tokens
- **Input Token Limit**: 128K tokens  
- **Output Token Limit**: Not specified
- **Token Speed**: Not specified
- **Multimodal**: Yes (text, images) - advanced vision capabilities
- **AWS Bedrock**: $0.002 per 1,000 tokens (input/output)
- **Key Features**: Document-level understanding, scene analysis, image captioning

## LLaMA 3.1 Series (Released July 2024)

### LLaMA 3.1 8B
- **Input Price**: Varies by provider (competitive rates)
- **Output Price**: Varies by provider (competitive rates)
- **Input Token Limit**: 128K tokens (16x increase from LLaMA 3)
- **Output Token Limit**: Not specified
- **Token Speed**: Up to competitive industry rates
- **Multimodal**: Text-only
- **Key Features**: Efficient mid-size model
- **Training**: Over 15 trillion tokens from public sources

### LLaMA 3.1 70B  
- **Input Price**: Varies by provider
- **Output Price**: Varies by provider
- **Input Token Limit**: 128K tokens
- **Output Token Limit**: Not specified
- **Token Speed**: Up to 249 tokens/second (Groq)
- **Multimodal**: Text-only
- **Key Features**: High performance, AWS optimized latency
- **AWS Bedrock**: Requires 8 Custom Model Units

### LLaMA 3.1 405B
- **Input Price**: Higher tier pricing (enterprise focused)
- **Output Price**: Higher tier pricing (enterprise focused)
- **Input Token Limit**: 128K tokens
- **Output Token Limit**: Not specified
- **Token Speed**: AWS optimized (fastest major cloud provider)
- **Multimodal**: Text-only
- **Key Features**: Frontier-level capabilities, GPT-4 competitive performance
- **Training**: Over 15 trillion tokens

## LLaMA 2 Series (Legacy)

### LLaMA 2 7B/13B/70B
- **Input Price**: Generally lower than LLaMA 3 series
- **Output Price**: Generally lower than LLaMA 3 series
- **Input Token Limit**: 4K tokens (standard)
- **Output Token Limit**: Not specified
- **Token Speed**: Baseline speeds
- **Multimodal**: Text-only
- **Key Features**: Open-source availability, various fine-tuned versions

## Provider-Specific Pricing

### AWS Bedrock
- Latency-optimized inference available
- Fastest performance for 405B and 70B models among major clouds
- Custom Model Units required for larger models

### Groq  
- Highly competitive token-based pricing
- Exceptional speed: 249 tokens/second for 70B model
- Strong price-performance ratio

### Azure AI
- Available through Models-as-a-Service (serverless endpoints)
- Marketplace pricing for all model sizes
- Fine-tuned model options available

### Together AI
- Free tier with limited tokens
- Moderate pricing balanced for cost and performance
- Developer-friendly pricing structure

## Cost Optimization Features
- **Free Tiers**: Together AI and AWS Bedrock offer initial credits
- **Enterprise Discounts**: Custom rates for high-volume usage (10M+ tokens/day)
- **Volume Pricing**: Negotiated rates for large-scale deployments

## Licensing
- **LLaMA 3.2**: Llama 3.2 Community License (custom commercial license)
- **LLaMA 3.1**: Llama 3.1 Community License (custom commercial license)
- **LLaMA 2**: More permissive commercial license

## Training Data
- **LLaMA 3.2**: Up to 9 trillion tokens from public sources
- **LLaMA 3.1**: Over 15 trillion tokens from public sources
- **Tokenizer**: 128K vocabulary for efficient encoding

## References
- [LLaMA 3.2 Official Blog](https://ai.meta.com/blog/llama-3-2-connect-2024-vision-edge-mobile-devices/)
- [LLaMA 3.1 Official Blog](https://ai.meta.com/blog/meta-llama-3-1/)
- [AWS Bedrock Pricing](https://aws.amazon.com/bedrock/pricing/)
- [LLaMA Documentation](https://llama.meta.com/doc/overview)