# MaGS 复现项目

本仓库用于记录在远程 RTX 3090 服务器上复现 **MaGS** (Mesh-adsorbed Gaussian Splatting, ICCV 2025 Highlight) 的全过程：环境搭建、问题、解决方案、训练 / 推理结果。

> 工程目录虽叫 `DG_mesh`（早期命名遗留），但实际复现的是浙江大学的 **MaGS**。

## 项目介绍

- 论文：**MaGS: Reconstructing and Simulating Dynamic 3D Objects with Mesh-adsorbed Gaussian Splatting**, ICCV 2025 Highlight
- 作者：Shaojie Ma, Yawei Luo, Wei Yang, Yi Yang（浙江大学）
- 项目主页：https://wcwac.github.io/MaGS-page/
- 论文：https://arxiv.org/abs/2406.01593
- 官方代码：https://github.com/wcwac/MaGS

### 核心思想

MaGS 把 **3D Gaussian Splatting**（渲染灵活）与 **Mesh**（结构清晰、几何先验强）结合，提出"网格吸附式高斯"表示，单目视频输入即可重建并模拟动态 3D 物体：

1. **RMD-Net**（Relative Mesh Deformation Network）—— 从视频学习运动先验，细化 mesh 形变；
2. **RGD-Net**（Relative Gaussian Deformation Network）—— 让 Gaussian 在 mesh 表面附近"游走"，建模 Gaussian 与 mesh 的相对位移，兼顾渲染保真和形变合理性；
3. 3D Gaussians 被吸附在 mesh 上，避开"渲染精度 vs. 形变合理性"的传统取舍。

## 远程服务器

- 主机：`ssh-cn-xinan1.ebcloud.com:30026`
- 用户：`root`

### 硬件 / 软件

| 项目 | 配置 |
|---|---|
| GPU | 1× NVIDIA RTX 3090 24 GB |
| GPU 驱动 | 580.65.06 / CUDA Driver 13.0（向下兼容 11.x） |
| CPU | Intel Xeon Gold 6330 @ 2.0 GHz, 112 cores |
| 内存 | 503 GiB |
| OS | Ubuntu 22.04.4 LTS |
| Python | 3.10.12（系统） / miniconda3 25.1.1 |
| 系统盘 | 30 GB（剩 17 GB） |
| `/root/data` | 50 GB |
| `/public` | 25 TB |

### 已有 conda 环境
- `base`、`janusvln`（与本项目无关）

## 3090 是否满足 MaGS 推理需求？✅ 满足

| 资源 | MaGS 需求 | 3090 24GB | 结论 |
|---|---|---|---|
| 显存（推理） | ~3–5 GB | 24 GB | 极其充裕 |
| 显存（训练，batch=1–2，res=400×400，max 150K Gaussians） | ~6–10 GB | 24 GB | 充裕 |
| CUDA 兼容 | CUDA 11.x / 12.x | 驱动 580 兼容 | OK |
| 数据集大小 | D-NeRF + meshes < 1 GB | 50 GB 工程盘 | OK |

依据：
- `config/3dgs.yaml` 中 `max_n_gauss=150000`
- DeformModel：MLP D=8 W=256 → 几 MB 参数
- D-NeRF 分辨率 800×800，再 ÷2 → 400×400
- batch_size=1–2，3DGS 原版渲染器，单物体合成场景

## 实施计划

1. ✅ 检查服务器环境
2. ✅ 评估 3090 可行性
3. **本地仓库写好 README/PLAN/PROGRESS** ← 当前
4. 服务器 clone MaGS + submodules（simple-knn、diff-gaussian-rasterization）
5. conda 创建 `mags` 环境（Python 3.9 + PyTorch + CUDA 11.8）
6. 安装依赖
7. 下载 D-NeRF 数据 + `D-NeRF_meshes.7z`
8. 运行 `python train.py config/3dgs.yaml,config/dnerf/jumpingjacks.yaml`
9. 监控显存 + loss + PSNR / LPIPS / SSIM
10. 每个关键节点 commit 到 https://github.com/CHANGJianshuo/mags

## 运行命令（参考）

```bash
# 进入服务器
ssh -p 30026 root@ssh-cn-xinan1.ebcloud.com

# 克隆
cd /root/data && git clone --recursive https://github.com/wcwac/MaGS.git
cd MaGS
git clone https://gitlab.inria.fr/bkerbl/simple-knn.git submodules/simple-knn
git clone --recursive https://github.com/graphdeco-inria/diff-gaussian-rasterization.git submodules/diff-gaussian-rasterization

# 环境
conda create -n mags python=3.9 -y
conda activate mags
pip install torch==2.1.0 torchvision==0.16.0 --index-url https://download.pytorch.org/whl/cu118
pip install -r requirements.txt
pip install -e submodules/simple-knn
pip install -e submodules/diff-gaussian-rasterization

# 训练
python train.py config/3dgs.yaml,config/dnerf/jumpingjacks.yaml
```
