---
title: "在LlamaFactory中使用vLLM推理"
date: 2025-07-07
description: "介绍如何在LLaMA Factory中配置并使用vLLM作为推理后端，包括LoRA模型合并与API启动步骤"
tags: ["vLLM", "LLaMA Factory", "推理", "大模型", "Qwen"]
categories: ["大模型技术"]
---

# 在llamafactory中使用vllm推理

> 本文时撰写时的相关依赖版本
>
> `llamafactory=0.9.4.dev0`
> `nvidia-cudnn-cu12=9.5.1.17`
> `torch=2.7.1`

1. **配置llama factory**

   ```python
   git clone https://github.com/hiyouga/LLaMA-Factory.git
   conda create -n llama_factory python=3.10
   conda activate llama_factory
   cd LLaMA-Factory
   pip install -e '.[torch,metrics]'
   ```

2. **安装vllm**

   ```python
   pip install 'vllm>=0.4.3,<=0.10.0' 'numpy<2.0.0,>=1.18.2'
   ```

   可能会出现该报错：

   ```
   ERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
   opencv-python-headless 4.12.0.88 requires numpy<2.3.0,>=2; python_version >= "3.9", but you have numpy 1.26.4 which is incompatible.
   ```

   不影响纯文本推理

3. **vllm推理**

   如果模型经过lora微调，则需要先合并（全量微调不需要）

   ```bash
   llamafactory-cli export \
       --model_name_or_path /root/autodl-tmp/qwen3-8b/ \
       --adapter_name_or_path ./saves/qwen3-8b/lora/0829  \
       --template qwen3 \
       --finetuning_type lora \
       --export_dir ./saves/qwen3-8b/merged/0829 \
       --export_size 2 \
       --export_device cpu \
       --export_legacy_format False
   ```

   启动推理

   ```bash
   export API_PORT=6008
   export CUDA_VISIBLE_DEVICES=0,1
   nohup llamafactory-cli api \
       --model_name_or_path ./saves/qwen3-8b/merged/0829 \
       --template qwen3 \
       --infer_backend vllm \
       --vllm_maxlen 40960 \
       >/dev/null 2>nohup.out \
       &
   ```

   发送请求时默认模型名为`gpt-3.5-turbo`
