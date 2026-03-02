# Sparkrun Recipes - Llama.cpp Conversion

This repository contains sparkrun recipes converted from the `llama-manager.sh` script.

## Conversion Summary

Converted the `llama-manager.sh` bash script into individual sparkrun YAML recipes for the llama.cpp runtime. This provides a cleaner, more maintainable way to manage and deploy large language models.

## What Was Converted

✅ **Main LLM Models** (20 recipes)
- GPT-OSS 120B & 20B
- Qwen3-VL 30B
- Qwen3-Next 80B (Thinking & Instruct variants)
- Devstral-2 123B
- GLM-4.5 Air Q3
- Qwen3 Coder 30B
- GLM-4.6V Flash
- Olmo 3.1/3 32B (Instruct & Think variants)
- RNJ-1 8B
- Nemotron 3 Nano 30B
- Ministral 3 14B Instruct
- GLM-4.7 Flash (llama.cpp solo)
- GLM-4.7 Flash (vLLM 2-node cluster)
- Step 3.5 Flash
- Qwen3 Coder Next 80B
- GLM-4.7 Flash Grande Heretic (Uncensored)

❌ **Not Converted** (as requested)
- Rerankers (BGE Reranker v2-m3)
- Embeddings (Qwen3 Embedding 0.6B & 8B)
- OCR (DeepSeek OCR)

## Recipe Structure

Each recipe follows the sparkrun GGUF/llama.cpp format:

```yaml
model: <repo>:<quant>
runtime: llama-cpp
max_nodes: 1
container: scitrera/dgx-spark-llama-cpp:b8094-cu131
defaults:
  port: 8000
  host: 0.0.0.0
  n_gpu_layers: 999
  ctx_size: 262144
  threads: 20
  parallel: 4
  ubatch: 2048
  flash_attn: 1
  jinja: true
  reasoning_format: auto
  reasoning_budget: -1
```

## Usage

```bash
# Run a specific model (solo mode)
sparkrun run recipes/gpt-oss-120b.yaml --solo

# Run a 2-node vLLM cluster
sparkrun run recipes/glm-4.7-flash-vllm-2node.yaml -H host1,host2

# Override port
sparkrun run recipes/qwen3-vl-30b.yaml -o port=9000

# Override context size
sparkrun run recipes/glm-4.7-flash.yaml -o ctx_size=131072
```

## Open WebUI

Open WebUI provides an OpenAI-compatible web interface for interacting with your models.

### Starting Open WebUI

First, ensure your model is running, then start Open WebUI:

```bash
# Start Open WebUI (connects to localhost:8000)
sparkrun run recipes/open-webui.yaml

# Connect to remote spark backend
sparkrun run recipes/open-webui.yaml -o api_host=192.168.1.100

# Custom web port
sparkrun run recipes/open-webui.yaml -o port=8080

# Custom API port (if your model runs on a different port)
sparkrun run recipes/open-webui.yaml -o api_port=9000
```

### Usage Example

```bash
# 1. Start your model
sparkrun run recipes/qwen3-vl-30b.yaml --solo

# 2. In another terminal, start Open WebUI
sparkrun run recipes/open-webui.yaml

# 3. Access the web UI at http://localhost:3000
```

### Configuration Options

- `port`: Port for the Open WebUI web interface (default: 3000)
- `api_host`: Host where your model's API is running (default: localhost)
- `api_port`: Port where your model's API is running (default: 8000)

## Key Differences from Original Script

1. **No interactive menu** - Each model is now a separate recipe file
2. **Consistent port 8000** - All recipes use port 8000 (can be overridden)
3. **Standard GGUF variants** - Uses standard quants unless only MXFP4 is available
4. **No auxiliary services** - Rerankers, embeddings, and OCR are not included
5. **No idle timeout** - Removed the auto-unload monitor
6. **No HF Token management** - Token must be set via environment variable if needed

## Hugging Face Token Access

Some models in these recipes may require Hugging Face access (gated models):

```bash
export HF_TOKEN=hf_your_token_here
sparkrun run recipes/qwen3-vl-30b.yaml --solo
```

## Original Script

The original `llama-manager.sh` script has been moved to the `to-convert/` directory for reference.

## Notes

- Most recipes use the `llama-cpp` runtime with `max_nodes: 1` (solo mode only)
- The `glm-4.7-flash-vllm-2node.yaml` recipe uses the `vLLM` runtime with `min_nodes: 2` for multi-node tensor parallelism
- Models with specific quantization variants use the `:quant` syntax (e.g., `:Q4_K_XL`)
- Standard configuration includes flash attention, Jinja templates, and reasoning format support
- Context size is set to 262144 tokens by default
