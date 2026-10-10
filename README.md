# WorldVLA-2: Action World Model 训练流程

基于 [RynnVLA-002](https://github.com/alibaba-damo-academy/RynnVLA-002) 框架，在 LIBERO-Goal 基准上进行 Action World Model 的训练与发布。本仓库记录从数据准备到模型生成的完整流程，并提供训练好的模型权重。

## 项目概述

本项目在 LIBERO-Goal 任务套件上训练了一个统一的 Action World Model，将 VLA 动作模型与世界模型融合在单一框架中。模型输入包含三组数据：

| 数据组 | 说明 |
| :---: | :--- |
| **Third View** | 第三视角相机图像（外部观察） |
| **Wrist** | 手腕相机图像（第一人称观察） |
| **State** | 机器人末端执行器状态（位姿 + 夹爪） |

模型基于 7B 参数的 Chameleon 架构，通过 Action Transformer 输出连续动作，同时具备图像生成（世界模型）能力。

## 预训练模型

训练前需下载以下预训练权重，放置于 `rynnvla-002/ckpts/` 目录下：

| 模型 | 用途 | 下载地址 |
| :--- | :--- | :--- |
| Starting Point | 训练初始化权重 | [WorldVLA/chameleon/starting_point](https://huggingface.co/Alibaba-DAMO-Academy/WorldVLA/tree/main/chameleon/starting_point) |
| Chameleon Tokenizer | 图像 VQGAN 分词器 | [WorldVLA/chameleon/tokenizer](https://huggingface.co/Alibaba-DAMO-Academy/WorldVLA/tree/main/chameleon/tokenizer) |
| Base Model | 基础语言模型 | [WorldVLA/base_model](https://huggingface.co/Alibaba-DAMO-Academy/WorldVLA/tree/main/base_model) |
| Lumina-mGPT-7B-768 | 文本分词器 | [Alpha-VLLM/Lumina-mGPT-7B-768](https://huggingface.co/Alpha-VLLM/Lumina-mGPT-7B-768) |

> 注：本仓库不包含上述预训练模型代码与权重，请按上表地址自行下载。

## 训练流程

### Step 1: 数据准备

数据准备流程（过滤 no-op、提取图像/动作/状态、生成对话格式、分词）详见 [RynnVLA-002](https://github.com/alibaba-damo-academy/RynnVLA-002) 官方文档。本项目数据采集至三组数据的流程见此前发布的 [数据采集仓库](https://github.com/xiangcongyanjiusuo)。

训练使用的数据配置（Pretokenize 管线）：

```
rynnvla-002/configs/libero_goal/his_2_third_view_wrist_w_state_5_256_pretokenize.yaml
```

数据由 Action Model 数据与 World Model 数据拼接而成：
- **Action Model**: `libero_goal_his_2_third_view_wrist_w_state_5_256`（历史 2 帧 + 第三视角 + 手腕 + 状态，5 步动作）
- **World Model**: `libero_goal_his_1_third_view_wrist_a2i_256`（历史 1 帧，action-to-image 生成）

### Step 2: 启动训练

```bash
cd rynnvla-002/exps_pretokenize
bash libero_goal_his_2_third_view_wrist_w_state_5_256_abiw.sh
```

### 训练超参数

| 参数 | 值 |
| :--- | :--- |
| 模型规模 | 7B |
| 分辨率 | 256 × 256 |
| 历史帧数 | 2 |
| 动作步数 (time_horizon) | 5 |
| 动作维度 | 7 |
| Batch Size | 8 |
| Epochs | 40（保存 epoch 8） |
| 学习率 | 5e-6 |
| Weight Decay | 0.15 |
| Dropout | 0.08 |
| Max Seq Len | 4096 |
| GPU 数量 | 8 |
| 数据并行 | FSDP (bf16) |

## 模型权重

训练完成的模型权重（epoch 8）：

| 项目 | 详情 |
| :--- | :--- |
| 路径 | `outputs/libero_goal/his_2_third_view_wrist_w_state_5_256_abiw/epoch8/` |
| 大小 | ~14 GB |
| 格式 | Safetensors（3 个分片） |
| 模型配置 | [config.json](./outputs/libero_goal/his_2_third_view_wrist_w_state_5_256_abiw/epoch8/config.json) |

> 模型文件较大，请从 [Releases](../../releases) 页面下载，或参考下方官方模型权重。

### 官方模型权重参考

完整的官方训练模型（含 4 个任务套件）可在 HuggingFace 获取：
[Alibaba-DAMO-Academy/RynnVLA-002](https://huggingface.co/Alibaba-DAMO-Academy/RynnVLA-002)

## 评估

```bash
cd rynnvla-002/evals_libero
# 连续动作评估
bash eval_libero_goal_his_2_third_view_wrist_w_state_5_256_abiw_continous.sh
# 离散动作评估
bash eval_libero_goal_his_2_third_view_wrist_w_state_5_256_abiw_discrete.sh
```

> 评估 7B 模型需要 GPU 显存 ≥ 16GB。

## 环境配置

```bash
git clone https://github.com/alibaba-damo-academy/RynnVLA-002.git
cd RynnVLA-002
pip install -r requirements.txt
pip install flash-attn --no-build-isolation
pip install -e .
git clone https://github.com/Lifelong-Robot-Learning/LIBERO.git
cd LIBERO && pip install -e .
```

## 致谢

本项目基于以下开源工作构建：
- [RynnVLA-002](https://github.com/alibaba-damo-academy/RynnVLA-002) / [WorldVLA](https://github.com/alibaba-damo-academy/WorldVLA)
- [Lumina-mGPT](https://github.com/Alpha-VLLM/Lumina-mGPT)
- [Chameleon](https://github.com/facebookresearch/chameleon)
- [OpenVLA](https://github.com/openvla/openvla)
- [LIBERO](https://github.com/Lifelong-Robot-Learning/LIBERO)

## 引用

```bibtex
@article{cen2025rynnvla,
  title={RynnVLA-002: A Unified Vision-Language-Action and World Model},
  author={Cen, Jun and Huang, Siteng and Yuan, Yuqian and Li, Kehan and Yuan, Hangjie and Yu, Chaohui and Jiang, Yuming and Guo, Jiayan and Li, Xin and Luo, Hao and Wang, Fan and Wang, Fan and Zhao, Deli},
  journal={arXiv preprint arXiv:2511.17502},
  year={2025}
}

@article{cen2025WorldVLA,
  title={WorldVLA: Towards Autoregressive Action World Model},
  author={Cen, Jun and Yu, Chaohui and Yuan, Hangjie and Jiang, Yuming and Huang, Siteng and Guo, Jiayan and Li, Xin and Song, Yibing and Luo, Hao and Wang, Fan and Zhao, Deli and Chen, Hao},
  journal={arXiv preprint arXiv:},
  year={2025}
}
```

## License

[Apache 2.0](./LICENSE)
