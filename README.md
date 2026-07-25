# ChunMengDie-1.0-0.4B

## 关于作者

https://huggingface.co/XingChina/ChunMengDie-1.0-0.4b
一个热爱 AI 的八年级学生。欢迎交流学习。

## 模型简介

ChunMengDie-1.0-0.4B 是一个基于 Transformer 架构的中文对话实验模型，参数量 4 亿（0.4B）。

**当前版本状态**：
- 早期研究阶段，输出质量不稳定，存在大量乱码和语义不连贯现象。
- 仅用于学术探索和实验验证，**不建议用于任何生产环境**。
- 后续版本将持续迭代优化。
- 由于其他分词器死活下不下来，使用内置的gpt2分词器进行训练（我自己尝试过训练自定义分词器，但是llama.cpp转换出现错误，尝试把分词器加进llama.cpp也不行，所以选择内置的分词器）
- 使用动态学习率

---

## 📂 仓库文件结构

本仓库同时提供 **Hugging Face 原生格式** 和 **llama.cpp GGUF 量化格式** 两种权重，请根据你的部署场景选择：

### 🤗 Hugging Face 格式（适用于 `transformers` 库）

| 文件名 | 说明 |
|--------|------|
| `config.json` | 模型结构配置文件 |
| `generation_config.json` | 文本生成参数配置 |
| `model.safetensors` | BF16 原始精度权重（约 0.8 GB） |
| `tokenizer.json` | 分词器词表 |
| `tokenizer_config.json` | 分词器配置 |

### 🦙 llama.cpp GGUF 格式（适用于 `llama.cpp` 或 `ollama` 部署）

| 精度 | 文件名 | 文件大小 | 适用场景 |
|------|--------|----------|----------|
| **BF16（原始）** | `chunmengdie_f16.gguf` | 501 MB | 追求最佳效果，适合高端显卡 |
| **8bit 量化** | `chunmengdie_q8_0.gguf` | 275 MB | 常规推理，质量损失极小（推荐） |
| **4bit 量化** | `chunmengdie_q4_km.gguf` | 181 MB | 低显存设备（如 4GB 显卡）或移动端部署 |

> 💡 **选择建议**：如果你是 Python 开发者，用 `transformers` 加载 HF 格式最方便；如果你在本地命令行或边缘设备部署，用 GGUF 格式配合 `llama.cpp` 更高效。

---

## 🚀 推理使用指南

### 方式一：Transformers（HF 格式）

安装依赖：
```bash
pip install transformers torch
```

加载模型并生成文本：
```python
from transformers import AutoModelForCausalLM, AutoTokenizer

model = AutoModelForCausalLM.from_pretrained("XingChina/ChunMengDie-1.0-0.4B")
tokenizer = AutoTokenizer.from_pretrained("XingChina/ChunMengDie-1.0-0.4B")

inputs = tokenizer("你好", return_tensors="pt")
outputs = model.generate(**inputs, max_new_tokens=128)
print(tokenizer.decode(outputs[0]))
```

### 方式二：llama.cpp（GGUF 格式）

下载对应的 `.gguf` 文件后，使用 `llama.cpp` 进行推理：

```bash
# BF16 原始版本
./main -m chunmengdie_f16.gguf -p "你好" -n 128

# 8bit 量化版本
./main -m chunmengdie_q8_0.gguf -p "你好" -n 128

# 4bit 量化版本（低显存首选）
./main -m chunmengdie_q4_km.gguf -p "你好" -n 128
```

**使用 ollama 部署（可选）**：
```bash
ollama create chunmengdie -f ./Modelfile
ollama run chunmengdie
```

---

## 🛡️ 训练环境与权重状态声明

本模型基于 **NVIDIA RTX Pro 6000（96GB 显存）** 专业卡训练，采用针对大显存优化的高吞吐配置（大 Batch Size + 长序列）。

**发布权重说明**：
1. 本仓库仅提供 **纯推理权重**（HF 格式 + GGUF 格式），**不包含优化器（Optimizer）和调度器（Scheduler）状态**。
2. 由于训练环境依赖 **96GB 显存级的内存分配策略**，在消费级显卡（如 24GB 4090）或标准 A100（80GB）上**无法直接加载续训**。（由于我手滑删了最终调度器，你拿什么卡都不能续训，即使能继续训也很容易爆显存并且发挥不出最大实力，中间调度器不能和最终模型混用，没有上传）
3. 如需进行二次微调，建议使用本仓库的 BF16 权重作为基座，自行挂载新的优化器开始训练。
4. 项目根目录train[1].py已针对20~24G显存显卡进行微调，实测在4090上全程不会爆显存，请以最新版train[1].py为准

**我们鼓励二次创新，但尊重算力投入，请合理使用开源资源。**

---

## 📚 训练数据声明

本模型在训练过程中参考/使用了以下公开数据集进行统计学习（点击可查看原始版权信息）：

| 数据集 | 许可证 | 使用方式 | 版权归属 |
|--------|--------|----------|----------|
| [BelleGroup/train_0.5M_CN](https://huggingface.co/datasets/BelleGroup/train_0.5M_CN) | GPL-3.0 | 全量用于训练并通过统计学习从数据中提取对话模式 | 版权归 BelleGroup 所有 |
| [liumindmind/NekoQA-10K](https://huggingface.co/datasets/liumindmind/NekoQA-10K) | Apache-2.0 | 全量用于训练并通过统计学习从数据中提取对话模式 | 版权归 MindsRiverPonder 所有 |
| [cyberlangke/Nana-catgirl-dataset-110k](https://huggingface.co/datasets/cyberlangke/Nana-catgirl-dataset-110k) | MIT | 全量用于训练并通过统计学习从数据中提取对话模式 | 版权归 cyberlangke 所有 |
| [XingChina/ChunMengDie-1.0-User-Data](https://huggingface.co/datasets/XingChina/ChunMengDie-1.0-User-Data) | CC BY 4.0 | 本数据集为原创角色对话数据，全量用于训练并通过统计学习从数据中提取对话模式 | 版权归 XingChina 所有 |

**重要说明**：
1. 本模型权重**并非**上述任何数据集的“衍生代码副本”或“复制品”，模型仅通过梯度下降从数据中提取统计模式。
2. 本模型的输出内容由 AI 随机生成，**不代表对训练数据集的检索或分发**。
3. 在极低概率下（<0.1%），模型可能因统计噪声而产生与训练数据某一段落高度相似的文本，该现象属于概率性巧合，建议使用者对输出进行抽样审核。
4. 验证loss与训练loss均已降到1以下，但不能理解语义

---

## ⚠️ 免责声明

本模型按“原样”（AS IS）提供，**不附带任何形式明示或默示的担保**。

使用者应自行承担使用本模型产生的一切后果，包括但不限于：
- 输出的准确性、安全性、合规性；
- 对第三方知识产权的潜在侵犯（若发生，属极小概率事件，我方不承担责任）。

建议在生产环境部署前配合敏感词过滤和输出重复检测模块。

---

## 🚫 使用限制

- 本模型目前质量不佳，**不建议用于任何生产环境**。
- 商业使用需自行评估输出内容的合规性。

---

## 📄 许可证

本模型权重采用 **BSD 3-Clause License** 发布。详见 [LICENSE](./LICENSE) 文件。

---

## 📖 引用

如果你在研究中使用本模型，请引用：

```bibtex
@misc{ChunMengDie-1.0-0.4B,
  author = {XingChina},
  title = {ChunMengDie-1.0-0.4B: A Chinese Conversational AI Model},
  year = {2026},
  publisher = {Hugging Face},
  url = {https://huggingface.co/XingChina/ChunMengDie-1.0-0.4B}
}
```

---

**感谢你对 ChunMengDie 项目的关注！** 🎉
