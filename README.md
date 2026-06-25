# HW3: 深度学习与空间智能

期末作业代码仓库，包含两个任务：
- **Task 1**：基于 3DGS 与 AIGC 的多源资产生成与真实场景融合
- **Task 2**：基于 LeRobot 的 ACT 策略跨环境泛化

实验报告见 `report/main.pdf`，作业说明见 `HW3_深度学习与空间智能.pdf`。

> **数据集与模型权重**单独打包，请从网盘下载后按下方说明放置。
>
> **注意**：由于文件体积限制，`task1_3dgs_aigc/threestudio/load/zero123/stable_zero123.ckpt` 已从代码包中移除。使用 Zero123 单图到3D 功能前，请手动下载该权重文件并放置到对应路径：
> ```
> task1_3dgs_aigc/threestudio/load/zero123/stable_zero123.ckpt
> ```
> 下载方式（任选其一）：
> - 官方 HuggingFace：`huggingface-cli download stabilityai/stable-zero123 stable_zero123.ckpt --local-dir task1_3dgs_aigc/threestudio/load/zero123/`
> - 或从本作业网盘的模型权重包中获取。

---

## 目录结构

```
HW3_code/
├── task1_3dgs_aigc/
│   ├── colmap/               # COLMAP 位姿估计
│   ├── gaussian-splatting/   # 3DGS 重建与渲染
│   └── threestudio/          # 文本/单图到3D生成
├── task2_lerobot_act/
│   └── lerobot/              # LeRobot ACT 策略训练与测试
└── report/                   # LaTeX 实验报告
```

---

## Task 1：3DGS + AIGC

### 环境配置

```bash
# 3D Gaussian Splatting
cd task1_3dgs_aigc/gaussian-splatting
conda env create -f environment.yml
conda activate gaussian_splatting
pip install submodules/diff-gaussian-rasterization
pip install submodules/simple-knn

# ThreeStudio
cd ../threestudio
pip install -r requirements.txt
```

### 数据准备

将数据集解压后放置如下（数据包单独提供）：

```
task1_3dgs_aigc/data/
├── object_A/          # 自拍物体 A 的多视角图像
├── object_C/          # 物体 C 的单张图片
└── garden/            # Mip-NeRF 360 garden 背景场景
```

### 训练命令

**物体 A（多视角重建）**
```bash
# 1. COLMAP 位姿估计（可用 gaussian-splatting 自带 convert.py）
cd task1_3dgs_aigc/gaussian-splatting
python convert.py -s ../data/object_A

# 2. 3DGS 训练
python train.py -s ../data/object_A --model_path ../output/object_A
```

**物体 B（文本到3D，DreamFusion）**
```bash
cd task1_3dgs_aigc/threestudio
python launch.py --config configs/dreamfusion-sd.yaml --train \
  system.prompt_processor.prompt="a 3D model of <your object>"
```

**物体 C（单图到3D，Zero123）**
```bash
cd task1_3dgs_aigc/threestudio
python launch.py --config configs/zero123.yaml --train \
  data.image_path=../data/object_C/object_C_rgba.png
```

**背景场景（garden）**
```bash
cd task1_3dgs_aigc/gaussian-splatting
python train.py -s ../data/garden --model_path ../output/garden
```

### 渲染命令

```bash
cd task1_3dgs_aigc/gaussian-splatting
# 渲染单场景
python render.py -m ../output/garden

# 渲染融合场景
python render.py -m ../output/fused_scene --skip_train
```

---

## Task 2：LeRobot ACT

### 环境配置

```bash
cd task2_lerobot_act/lerobot
pip install -e ".[act]"
```

### 数据准备

将 CALVIN 数据集解压后放置如下（数据包单独提供）：

```
task2_lerobot_act/data/
└── calvin_task_ABC_D/
    ├── calvin_task_ABC_D_lerobot_0_4/   # 仅环境A数据
    └── calvin_task_envABC/              # 环境A+B+C混合数据
```

### 训练命令

**仅环境 A 训练**
```bash
cd task2_lerobot_act/lerobot
python lerobot/scripts/train.py \
  policy=act \
  env=calvin \
  dataset_repo_id=data/calvin_task_ABC_D/calvin_task_ABC_D_lerobot_0_4 \
  hydra.run.dir=../../output/task2_act/env_A
```

**多环境 A+B+C 联合训练**
```bash
cd task2_lerobot_act/lerobot
python lerobot/scripts/train.py \
  policy=act \
  env=calvin \
  dataset_repo_id=data/calvin_task_ABC_D/calvin_task_envABC \
  hydra.run.dir=../../output/task2_act/run_envABC
```

### 测试命令（Zero-shot 跨环境）

```bash
cd task2_lerobot_act/lerobot
python lerobot/scripts/eval.py \
  -p ../../output/task2_act/env_A \
  eval.n_episodes=50
```

---

## 子项目 README

各子项目含详细原始文档：

| 子项目 | README |
|--------|--------|
| COLMAP | [task1_3dgs_aigc/colmap/README.md](task1_3dgs_aigc/colmap/README.md) |
| 3D Gaussian Splatting | [task1_3dgs_aigc/gaussian-splatting/README.md](task1_3dgs_aigc/gaussian-splatting/README.md) |
| ThreeStudio | [task1_3dgs_aigc/threestudio/README.md](task1_3dgs_aigc/threestudio/README.md) |
| LeRobot | [task2_lerobot_act/lerobot/README.md](task2_lerobot_act/lerobot/README.md) |
