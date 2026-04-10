# DG-Mesh 复现进度日志

按时间顺序记录每一步操作、遇到的问题、解决方案。

---

## 2026-04-10

### 步骤 1：环境检查
- SSH 登录 `root@ssh-cn-xinan1.ebcloud.com:30026` 成功
- 硬件：1× RTX 3090 24GB / Xeon Gold 6330 (112c) / 503GB RAM
- 软件：Ubuntu 22.04, Python 3.10, miniconda3 25.1.1
- 已有 conda env: `base`, `janusvln`
- 磁盘：系统盘 17GB 剩余，`/root/data` 50GB，`/public` 20TB

**结论**：硬件充足，磁盘需要把工程放到 `/root/data`，数据集放到 `/public`。

### 步骤 2：本地工程初始化
- `/home/chang/DG_mesh` 初始化 git
- 添加 remote → `github.com/CHANGJianshuo/mags`
- 编写 README.md / PLAN.md / PROGRESS.md
- 首次 commit & push（待执行）

### 步骤 N：(待续)
