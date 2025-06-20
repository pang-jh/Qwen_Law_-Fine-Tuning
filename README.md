# Qwen法律微调
本项目旨在对Qwen2.5-3B-Instruct模型进行法律领域的微调，以提升其在法律文本处理和理解方面的能力。

## 相关技术
- Qwen2.5-3B-Instruct模型： Qwen2.5-3B-instruct是Qwen2.5系列中的一个SFT版本，具有3.09亿参数，采用了先进的Transformer架构，支持32,768个token的上下文长度，适用于多种自然语言处理任务。 
- LoRA技术： LoRA（Low-Rank Adaptation）是一种高效的模型微调技术，通过引入低秩矩阵来减少微调过程中需要更新的参数数量，从而降低计算资源需求。LoRA在保持模型原有能力的前提下，实现了任务特定的高效适应。 
- LLaMA-Factory： LLaMA-Factory是一个用于大模型微调的工具，支持多种微调方法，如LoRA微调、全量微调（full）以及冻结微调（freeze）等。它提供了灵活的配置选项，方便用户根据需求进行模型调优。 

## 数据预处理
- 数据集地址：https://huggingface.co/datasets/ShengbinYue/DISC-Law-SFT
- 对数据进行预处理
```bash
import json
with open('DISC-Law-SFT-Pair-QA-released.jsonl','r') as f:
     json_objects=[ json.loads(line) for i ,line in enumerate(f) if i<8000 ]
with open('DISC_small.json','w') as f:
     json.dump(json_objects,f,indent=4,ensure_ascii=False)

```
## 克隆LLaMA Factory仓库
- 仓库地址 https://github.com/hiyouga/LLaMA-Factory
- 克隆并安装依赖的linux命令
```bash
git clone --depth 1 https://github.com/hiyouga/LLaMA-Factory.git
cd LLaMA-Factory
pip install -e ".[torch,metrics]"
```
## 配置数据集
- 将数据放到data目录下
- 在data的data_info.json中注册数据集信息，添加如下信息
```bash
  "DISC_small": {
    "file_name": "DISC_small.json",
    "columns": {
      "prompt": "input",
      "response": "output"
    }
  }
```
## 下载模型文件
- 执行命令
```bash
modelscope download --model Qwen/Qwen2.5-3B-Instruct --local_dir qwen2.5-3b-instruct
```
## 配置训练yaml文件
- yaml文件位于examples/train_lora/llama3_lora_sft.yaml
- 配置代码块：llama3_lora_sft.yaml
## 执行训练命令
```bash
llamafactory-cli train examples/train_lora/llama3_lora_sft.yaml
```
## 推理
- 找到saves文件夹中想推理的checkpoint
- 修改yaml文件，位于examples/inference/llama3_lora_sft.yaml
```bash
model_name_or_path: qwen2.5-3b-instruct
adapter_name_or_path: saves/qwen2.5-3b-instruct/lora/1_try/checkpoint-20
template: qwen
infer_backend: huggingface  # choices: [huggingface, vllm]
trust_remote_code: true
```
- 执行推理命令llamafactory-cli webchat examples/inference/llama3_lora_sft.yaml
