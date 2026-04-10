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

### Step N — (待续)
