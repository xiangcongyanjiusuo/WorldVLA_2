# 对话文件生成说明

## 文件列表

1. **VLA模型对话文件** (动作生成)
   - 文件名: `libero_vla_data_his_1_train_img_state_abs_ck_1_256.json`
   - 对话数: 839条
   - 用途: 训练动作生成模型（图像+状态 → 动作序列）
   - 输入: 前向相机图像 + 腕部相机图像 + 机器人状态
   - 输出: 未来20步动作序列

2. **World Model对话文件 - 前向相机** (图像预测)
   - 文件名: `libero_world_model_data_his_1_train_a2i_512_abs_front_all_data.json`
   - 对话数: 836条
   - 用途: 训练前向相机预测模型（动作 → 图像）
   - 输入: 历史图像 + 动作
   - 输出: 下一帧前向相机图像

3. **World Model对话文件 - 腕部相机** (图像预测)
   - 文件名: `libero_world_model_data_his_1_train_a2i_512_abs_wrist_all_data.json`
   - 对话数: 836条
   - 用途: 训练腕部相机预测模型（动作 → 图像）
   - 输入: 历史图像 + 动作
   - 输出: 下一帧腕部相机图像

## 数据来源

- 原始数据: 3个episodes (episode_000000, episode_000001, episode_000002)
- 每个episode: 320帧数据
- 任务名称: Pick and Place (抓取和放置)
- 动作维度: 6维 (x, y, z, roll, pitch, yaw)
- 动作序列长度: 20步

## 生成时间

- 生成日期: 2026-05-11
- 总耗时: 约2分钟

## 下一步操作

请参考 README.md 中的 Step 4: 计算归一化参数 & Tokenize数据
