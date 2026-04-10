# MaGS 复现进度日志

按时间顺序记录每一步操作、问题、解决方案。

---

## 2026-04-10

### Step 1 — 服务器环境检查 ✅
- 主机：`ssh -p 30026 root@ssh-cn-xinan1.ebcloud.com`
- 1× RTX 3090 24GB / Xeon Gold 6330 (112c) / 503 GB RAM
- 驱动 580.65.06，CUDA Driver 13.0
- Ubuntu 22.04，Python 3.10，miniconda3 25.1.1
- 已有环境：`base`, `janusvln`（与本项目无关）
- 磁盘：系统盘剩 17 GB，`/root/data` 50 GB（工程目录），`/public` 20 TB（数据/缓存）

**结论**：硬件远超 MaGS 需求，磁盘把工程放 `/root/data`、数据放 `/public` 即可。

### Step 2 — 研究 MaGS 仓库 & 评估可行性 ✅
仓库：https://github.com/wcwac/MaGS （ICCV 2025 Highlight）

关键发现：
- 入口是 `train.py`（README 写的是 main.py，实际仓库里是 train.py）
- 配置：`config/3dgs.yaml` + 场景 yaml（D-NeRF / DG-Mesh / vrig / instantavatar）
- D-NeRF jumpingjacks: batch_size=2, resolution=2 (即 400×400), max_n_gauss=150000
- Deform MLP: D=8, W=256
- Submodules: `simple-knn`、`diff-gaussian-rasterization`
- requirements.txt: opencv-python, omegaconf, torchmetrics, open3d, plyfile, roma

**3090 24GB 评估**：训练峰值预计 6–10 GB，推理 < 5 GB → **完全够用**。

### Step 3 — 重写本地文档 ✅
- 删除早前误把项目当成 DG-Mesh 的 README/PLAN/PROGRESS
- 重写为 MaGS 内容
- 待 commit & push

### Step 4 — 服务器 clone MaGS & submodules ✅
- `/root/data/MaGS` 克隆主仓 OK
- submodules：
  - `simple-knn` 从 gitlab.inria.fr 直连 OK
  - `diff-gaussian-rasterization` GitHub 直连 **卡死**
    - **解决**：用 `https://ghfast.top/` 代理克隆 → OK
  - 内嵌 `third_party/glm` 也走同样代理 → OK

### Step 5 — 文件系统踩坑 ✅
- `/public` 是 **只读** ceph HDD，原计划数据放那里不行
- `/dev/mapper/data-data`（3.7T）只挂在 `/etc/hosts` 等文件上，不可写入目录
- **结论**：唯一大容量可写目录是 `/root/data`（50 GB ceph NVMe），工程 + env + 数据全放这里

### Step 6 — Conda 环境 ✅
- 系统盘只剩 17 GB，conda env 不能放默认的 `~/miniconda3/envs`
- **解决**：`conda create --prefix /root/data/conda_envs/mags python=3.9 -y` → 放 `/root/data`
- 配好清华 pip 镜像（`~/.config/pip/pip.conf`）

### Step 7 — PyTorch 安装踩坑 ✅
- 第一次：`pip install torch==2.1.0+cu118 --index-url https://download.pytorch.org/whl/cu118`
  - **卡死**，pytorch.org 境内连通性差
- 第二次：`pip install ... -i tsinghua --extra-index-url aliyun-pytorch-wheels/cu118/`
  - **失败**，aliyun 是纯文件服务器，不是 PyPI 索引格式
- 第三次（成功）：直接 `wget` 下载 wheel，再本地 `pip install`
  - `torch-2.1.0+cu118-cp39-cp39-linux_x86_64.whl` ~2.3 GB，aliyun 镜像 ~20 MB/s
  - `torchvision-0.16.0+cu118-cp39-cp39-linux_x86_64.whl` ~6 MB
- 验证：`torch 2.1.0+cu118, cuda=True, device=NVIDIA GeForce RTX 3090` ✅
- numpy 首次装成 2.0.2，torch 2.1 不兼容 → 降级 `numpy<2` → 1.26.4

### Step 8 — 数据准备 ✅
- D-NeRF 数据集（258 MB）
  - 第一次 `wget github.com/.../D-NeRF-Deformable-GS.zip` → 卡死
  - **解决**：`wget https://ghfast.top/https://github.com/...` → 35 秒完成
- 解压到 `/root/data/mags_data/`，8 个场景（bouncingballs, hellwarrior, hook, jumpingjacks, lego, mutant, standup, trex）
- `D-NeRF_meshes.7z`（15 MB）同样走 ghfast → OK
- 安装 `p7zip-full`，`7z x` 解压到同目录，meshes 文件合并到每个场景下
- 验证 `jumpingjacks/` 有 `train/` `train_meshes/` `test/` `test_meshes/`，200 帧 + 200 mesh

### Step 9 — MaGS Python 依赖安装
- `pip install opencv-python omegaconf torchmetrics open3d plyfile roma tensorboard tqdm`
- 顺带装了 matplotlib、scipy、pandas、scikit-learn、ipython 等上游依赖
- **坑**：open3d 0.19.0 把 numpy 再次升到 2.0.2 → torch 又会报错
  - 修复：再次 `pip install 'numpy<2'` 强制降级
  - 教训：以后 numpy pin 要写死在 requirements 里

### Step 10 — CUDA toolkit (nvcc) 安装
- 系统 PATH 里没有 nvcc，simple-knn / diff-gaussian-rasterization 编译需要
- 方案：conda 装 `nvidia/label/cuda-11.8.0` 频道里的 `cuda-nvcc cuda-cudart-dev cuda-cccl cuda-libraries-dev`
- 仅装 dev 组件而非整包 cuda-toolkit，避免空间膨胀
- 进行中（env 从 4.6 G 涨到 6.3 G）

### Step 11 — 子模块编译（待做）
- `pip install -e submodules/simple-knn`
- `pip install -e submodules/diff-gaussian-rasterization`
- 验证 `from diff_gaussian_rasterization import GaussianRasterizer; from simple_knn._C import distCUDA2`

### Step 12 — 运行训练验证（待做）
- `cd /root/data/MaGS && python train.py config/3dgs.yaml,config/dnerf/jumpingjacks.yaml`
- 边跑边 `nvidia-smi` 看显存，确认 3090 24 GB 够用
- 收集 PSNR / LPIPS / SSIM 指标
