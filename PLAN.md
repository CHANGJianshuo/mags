# MaGS 复现执行计划

## 阶段 0：本地准备 ✅
- [x] 本地 git 仓库初始化、remote → `github.com/CHANGJianshuo/mags`
- [x] SSH 登录服务器、确认硬件 / OS / Python / conda
- [x] 研究 MaGS 仓库结构（`train.py`、`config/`、submodules）
- [x] 评估 3090 24GB 对 MaGS 的推理 / 训练可行性 → **充足**
- [x] 编写 README、PLAN、PROGRESS

## 阶段 1：本地首次 commit & push
- [ ] `git add . && git commit -m "switch to MaGS reproduction"`
- [ ] `git push`

## 阶段 2：服务器克隆 MaGS
- [ ] 在 `/root/data/MaGS` 克隆主仓库
- [ ] 克隆 submodules：
  - `submodules/simple-knn` ← https://gitlab.inria.fr/bkerbl/simple-knn.git
  - `submodules/diff-gaussian-rasterization` ← https://github.com/graphdeco-inria/diff-gaussian-rasterization.git

## 阶段 3：搭建 Python 环境
- [ ] `conda create -n mags python=3.9 -y`
- [ ] 安装 PyTorch 2.1 + cu118
  ```
  pip install torch==2.1.0 torchvision==0.16.0 --index-url https://download.pytorch.org/whl/cu118
  ```
- [ ] `pip install -r requirements.txt`
  - opencv-python, omegaconf, torchmetrics, open3d, plyfile, roma
  - 还需补：tensorboard, tqdm, lpips（torchmetrics 自带）, scikit-image
- [ ] 安装 submodules（CUDA 编译）
  ```
  pip install -e submodules/simple-knn
  pip install -e submodules/diff-gaussian-rasterization
  ```
- [ ] 验证：`python -c "import torch; print(torch.cuda.is_available()); from diff_gaussian_rasterization import GaussianRasterizer; print('OK')"`

## 阶段 4：数据准备
- [ ] 下载 D-NeRF 原始数据（[官方链接](https://github.com/albertpumarola/D-NeRF) 或 fixed: [Deformable-3D-Gaussians](https://github.com/ingra14m/Deformable-3D-Gaussians)）
- [ ] 下载 mesh 档案：`https://github.com/wcwac/MaGS/releases/download/v0.0.1/D-NeRF_meshes.7z`（15 MB）
- [ ] 数据放 `/public/mags_data/dnerf/`，软链接到 `MaGS/dataset/dnerf/`
- [ ] 解压后目录：
  ```
  dataset/dnerf/jumpingjacks/
  ├── train/  *.png
  ├── train_meshes/  *.ply
  ├── test/  *.png
  └── test_meshes/  *.ply
  ```

## 阶段 5：跑通训练 / 推理
- [ ] 启动一次最小训练：`python train.py config/3dgs.yaml,config/dnerf/jumpingjacks.yaml`
- [ ] 用 nvidia-smi 监控显存峰值
- [ ] 观察 tensorboard 中 PSNR / loss
- [ ] 跑到至少出 test 评估（每 1000 iter）
- [ ] 把 outputs/ 中的最佳 PSNR / SSIM / LPIPS 写到 PROGRESS

## 阶段 6：周期性 commit
- 每完成一个阶段：
  1. 更新 PROGRESS.md
  2. `git add . && git commit -m "step N: <概要>"`
  3. `git push`

## 风险 & 缓解

| 风险 | 缓解 |
|---|---|
| diff-gaussian-rasterization 编译失败（无 nvcc） | 装 `cuda-toolkit=11.8`（conda 渠道） |
| pytorch CUDA 与系统驱动不匹配 | 用 cu118，驱动 580 向下兼容 |
| 数据下载慢 / 国外超时 | 用 hf-mirror、镜像或先 wget 然后 rsync |
| 训练长（40k iter） | 先训 5k 验证可行，再决定要不要跑完 |
| 进程 hang 不输出 | 用后台 + tail 监控，加超时 |
| 系统盘 17GB 不够 | 工程放 /root/data，数据放 /public |
