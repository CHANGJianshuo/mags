# DG-Mesh 复现项目

本仓库用于记录 **DG-Mesh** (Dynamic Gaussian Mesh, ICLR 2025) 项目在远程服务器上的环境搭建、运行过程、问题解决方案以及实验结果。

## 项目介绍

**DG-Mesh** (Dynamic Gaussian Mesh): Consistent Mesh Reconstruction from Monocular Videos
- 论文: ICLR 2025
- 官方仓库: https://github.com/Isabella98Liu/DG-Mesh
- 任务：从单目视频中重建出**时序一致的高保真三角网格**，可处理拓扑变化和细薄结构（如鸟翅膀）。

### 核心思想

DG-Mesh 把 3D Gaussian Splatting 与可微分网格化相结合：
1. 使用 Dynamic Gaussian Splatting 重建动态场景；
2. 通过 Gaussian-Mesh 绑定，把 Gaussian 拉到 mesh 表面附近；
3. 借助可微分渲染优化 mesh 顶点的运动轨迹；
4. 输出每一帧时序一致的 mesh。

## 远程服务器

- 主机：`ssh-cn-xinan1.ebcloud.com:30026`
- 用户：`root`

### 硬件 / 软件清单

| 项目 | 配置 |
|---|---|
| GPU | 1× NVIDIA RTX 3090 24GB |
| GPU 驱动 | 580.65.06，CUDA Driver 13.0 |
| CPU | Intel Xeon Gold 6330 @ 2.00GHz, 112 cores |
| 内存 | 503 GiB |
| OS | Ubuntu 22.04.4 LTS |
| 系统 Python | 3.10.12 |
| Conda | miniconda3 25.1.1（已安装） |
| 系统盘 | 30 GB（剩 17 GB） |
| `/root/data` | 50 GB（剩 50 GB）— 可用作工程目录 |
| `/public` | 25 TB（剩 20 TB）— 可用作数据/缓存 |

### 已有环境

- 已有 conda 环境：`base`, `janusvln`
- 系统未装 PyTorch、CUDA Toolkit（nvcc）
- DG-Mesh 官方推荐 Python 3.9 + PyTorch + cuda 11.8

## 需求满足度评估

| 需求 | 是否满足 | 备注 |
|---|---|---|
| GPU 显存 ≥ 12GB | ✅ 24GB | 充足 |
| CUDA 11.8 兼容 | ✅ | 驱动 580 向下兼容 11.8 |
| 依赖（pytorch3d, nvdiffrast, tiny-cuda-nn, diff-gaussian-rasterization, simple-knn） | ⚠️ 待装 | 通过 conda 创建独立环境 |
| 磁盘空间 | ✅ | 工程放 /root/data，数据放 /public |
| 网络访问 GitHub / pip | 待验证 | |

## 实施计划

1. **环境检查与本地仓库初始化** ✅
2. **首次 commit & push** — 将 README/PLAN 推送到 https://github.com/CHANGJianshuo/mags
3. **服务器侧环境搭建**
   - 在 `/root/data` 下 clone DG-Mesh
   - 用 conda 创建 `dgmesh` 环境（Python 3.9）
   - 安装 PyTorch (cu118)、pytorch3d、nvdiffrast、tiny-cuda-nn
   - 安装 diff-gaussian-rasterization、simple-knn 子模块
   - `pip install -r requirements.txt`
4. **数据准备**
   - 下载 DG-Mesh synthetic 数据集（一个场景先试跑）
   - 放在 `/public/dgmesh_data` 节省系统盘
5. **跑通训练 / 推理**
   - 运行训练脚本，监控显存与 loss
   - 提取 mesh、渲染验证
6. **每个关键节点 commit 进度日志**

## 目录结构（本地）

```
DG_mesh/
├── README.md          # 本文件，项目介绍 + 环境
├── PLAN.md            # 详细执行计划
├── PROGRESS.md        # 过程记录（按步骤）
└── scripts/           # 自己写的辅助脚本
    └── remote_setup.sh
```
