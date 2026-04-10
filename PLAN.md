# DG-Mesh 实施计划

## 阶段 0：本地准备
- [x] 在本地 `/home/chang/DG_mesh` 初始化 git 仓库
- [x] 配置 remote → `https://github.com/CHANGJianshuo/mags`
- [x] SSH 登录服务器，确认硬件 / OS / Python / conda
- [x] 编写 README、PLAN
- [ ] 首次 commit & push

## 阶段 1：服务器环境搭建
- [ ] 在 `/root/data/DG-Mesh` 克隆官方仓库
- [ ] `conda create -n dgmesh python=3.9 -y`
- [ ] 安装 PyTorch 2.1 + cu118
  ```
  pip install torch==2.1.0 torchvision==0.16.0 --index-url https://download.pytorch.org/whl/cu118
  ```
- [ ] 安装 CUDA Toolkit 11.8（仅 nvcc，conda 渠道）
  ```
  conda install -c "nvidia/label/cuda-11.8.0" cuda-toolkit -y
  ```
- [ ] 安装 pytorch3d（FORCE_CUDA=1）
- [ ] 安装 nvdiffrast、tiny-cuda-nn
- [ ] 安装 diff-gaussian-rasterization、simple-knn 子模块
- [ ] `pip install -r requirements.txt`
- [ ] `python -c "import torch; print(torch.cuda.is_available())"` 验证

## 阶段 2：数据准备
- [ ] 下载 DG-Mesh 官方 synthetic 数据集（先选 1 个场景，例如 `beagle`）
- [ ] 数据放置在 `/public/dgmesh_data/`，软链接到工程目录
- [ ] 检查数据完整性

## 阶段 3：跑通训练
- [ ] 阅读 README 找到训练脚本
- [ ] 用最小配置启动训练，验证 forward/backward 不报错
- [ ] 监控显存占用、loss 曲线
- [ ] 训练若干迭代验证可收敛

## 阶段 4：Mesh 提取与可视化
- [ ] 运行 mesh 提取脚本
- [ ] 检查输出 mesh 文件
- [ ] （可选）渲染对比图

## 阶段 5：记录与提交
- 每完成一个阶段：
  - 更新 PROGRESS.md
  - `git add . && git commit -m "step N: ..."`
  - `git push`

## 风险与应对
| 风险 | 应对 |
|---|---|
| pytorch3d 编译慢/失败 | 用预编译 wheel 或 conda 渠道 |
| tiny-cuda-nn 编译失败 | 检查 nvcc 是否在 PATH |
| 系统盘 17GB 不够 | 工程放 /root/data，conda 缓存放 /public |
| 训练卡住 | 加 timeout，后台跑 + tail 日志 |
| 网络下载慢 | 用国内镜像或 hf-mirror.com |
