# 2026-07-09 日报：Qwen3.5-35B-A3B 推理环境调研

## 一、模型概况

Qwen3.5-35B-A3B：阿里 2026.02 发布，MoE 架构，**35B 总参数 / 3B 激活参数/ token**，128 experts，GQA+Gate DeltaNet。BF16 原版 ~67GB，支持 262K 上下文。

手上两个模型：BF16 原版（/llm/modelscope/Qwen/）和社区微调版 AWQ-INT4（/llm/huggingface/cyankiwi/）。

---

## 二、遇到的问题（5 个坑，已梳理清楚）

1. **transformers 版本**：4.x 不支持 qwen3_5_moe，需 ≥5.13
2. **INT4 量化不兼容**：5.13 把 gate_proj+up_proj 合并为 gate_up_proj（nn.Parameter），量化元数据没跟着合并 → 报 "Asymmetric quant requires zero-point values"。这是 transformers v5 已知 bug（issue #43284，Intel 1 月就提了，3 月部分修复，未完成）
3. **torch._grouped_mm 不支持 RTX 5090 (SM 120)**：NVIDIA 定制版 PyTorch 限制了 CC 9.0/10.0，正式版 2.9.0 已支持

结论：BF16 原版在 5.13 上可跑（~26GB 显存因为只加载激活 expert），INT4 量化版暂不可用。

---

## 三、核心概念理解

### 3.1 为什么新模型需要 transformers 新支持？
模型架构各不相同（层数、注意力类型、FFN 类型、Norm、RoPE…），transformers 无法自动推断，必须为每个模型写专属 Python 类。训练用的是阿里内部 Megatron 体系，和 HF 推理代码是完全不同的项目，靠人工 PR 桥接。Qwen3.5 2 月发布，5 月还在修 bug。

### 3.2 AutoModel 如何识别模型？
通过 `config.json` 的 `architectures` 和 `model_type` 字段查内部映射表 → 找到对应 Python 类 → 加载权重。

### 3.3 模型发布标准格式（HF safetensors）
核心文件：`config.json`（身份证）+ `model-xxxx-of-yyyy.safetensors`（权重）+ `tokenizer.json/config.json`（分词器）+ `vocab.json/merges.txt`（BPE 词表）+ `tokenizer_config.json`（chat_template）。

### 3.4 safetensors vs GGUF vs MLX vs ONNX
safetensors = 纯数据，依赖 Python 类描述模型结构；GGUF/MLX/ONNX = 自包含格式。GGUF 核心优势是 mmap（零拷贝，5GB RAM 跑 67GB 模型），safetensors 默认需要全量读入 CPU RAM 再搬 GPU。

### 3.5 BPE 分词器
Byte-level BPE（BBPE）是当前开源模型最主流分词器（~70%）。字符级、无 `<unk>`、多语言友好。合并规则全在 `merges.txt` 和 `vocab.json`，推理时无需训练。

### 3.6 CUTLASS kernel
NVIDIA 开源的 CUDA 模板库，用于 GPU 高性能线性代数。按编译时的 SM 架构预编译 SASS，不支持 SM 120 就 fallback/报错。vs Triton 的区别：CUTLASS AOT 编译（极致性能但新卡可能不兼容），Triton JIT 编译（兼容性好但略慢）。

### 3.7 AWQ 量化
Activation-aware Weight Quantization：观察校准数据激活分布 → 识别重要通道 → per-channel scaling 保护 → INT4 量化为对称量化（无 zero-point）。核心前提：被量化的层必须是 `nn.Linear`，但 5.13 的 MoE 合并成了 `nn.Parameter` + `torch._grouped_mm` → AutoAWQ 无法识别，也无法处理合并后的权重切分和量化元数据拼接。

---

## 四、当前可用方案

**GGUF 量化**：llama.cpp 转换脚本不依赖 transformers 模型结构，自己解析权重 → 转换成 GGUF → quantize 到 Q4_K_M（~18GB）→ RTX 5090 24GB 单卡可跑。这也是目前唯一绕过 transformers 5.13 bug 的量化方案。

---

## 五、一键流程

```bash
# 1. 装 llama.cpp
git clone https://github.com/ggerganov/llama.cpp && cd llama.cpp
mkdir build && cd build
cmake .. -DGGML_CUDA=ON && cmake --build . --config Release -j

# 2. 转换 safetensors → FP16 GGUF
python convert_hf_to_gguf.py /llm/modelscope/Qwen/Qwen3.5-35B-A3B \
    --outtype f16 --outfile Qwen3.5-35B-A3B-F16.gguf

# 3. 量化到 Q4_K_M
./build/bin/quantize Qwen3.5-35B-A3B-F16.gguf Qwen3.5-35B-A3B-Q4_K_M.gguf Q4_K_M

# 4. 推理
./build/bin/llama-cli -m Qwen3.5-35B-A3B-Q4_K_M.gguf -p "你好" -ngl 99
```
