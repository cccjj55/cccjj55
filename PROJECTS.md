# Projects — 架构笔记与负结果记录

> 主页 `README.md` 只放索引，技术选型表见 [PORTFOLIO.md](PORTFOLIO.md)。下面按方向记录每个公开仓库**解决了什么、踩过什么坑、哪些数字已被直接测出**。

---

## 1. 具身仿真与调度边界

### [mujoco-omp-rollout](https://github.com/cccjj55/mujoco-omp-rollout) ✅ 可复现（`v1.1.0`）

把同一个 MuJoCo XML 模型、同一初始条件放进同一套校验脚本（每个模式跑完核对终态 `qpos` checksum，或调用方验证与 `mujoco.mj_step` 在 `atol=1e-12` 下逐元素一致），在 Intel i7-14650HX（WSL2：`6 物理核 × 2 SMT`）上拆解 6 种并行边界：

- `serial` vs `threads`（线程本地预建模型以剥离建模耗时）：纯 Python GIL 争用让轻量 `1body` 从 `968k` 跌到 `51k env-steps/s`（**`0.05×`**）。
- `proc_cold` vs `proc_reuse`：复用持久进程省掉重复 XML 解析后，从 `3.41×` 提到 `5.62×`。
- `mj_rollout`（官方多线程 C++）vs 自写 C++17 OpenMP（`cpp/omp_rollout.cpp` + C ABI `libomp_rollout.so` + Python `RolloutPool` 封装）：消除 Python 胶水、IPC 序列化和整条轨迹内存写回后，`1body` 达到 **`10.75M env-steps/s`（`11.11×`）**。
- **硬件天花板对照**：当单步计算从极轻的 `1body` 换成耦合重的 `12body` 链，C++ OpenMP 与持久多进程同时收敛在 `~5.5×–5.7×`（`566k–587k/s`），因为实际只有 **6 个物理浮点执行核**。

### [sim-dt-pareto](https://github.com/cccjj55/sim-dt-pareto) 🟡 CPU 基线 + 精度扫描

用统一契约（`Backend.step(n)`）把"每秒步数"和"积分步长 `dt` 引起的 RMSE 漂移"画在同一张 Pareto 视图上：

- **MuJoCo Python 循环几乎不随并行环境数扩展**：`8 → 64 envs` 稳在 `209k → 212k env-steps/s`。
- **MJX 即使在 CPU 端也随 batch 放大摊平派发开销**：`8 → 64 envs` 从 `316k` 升到 `522k env-steps/s`。
- GPU 端 `--device cuda` 待完整 CUDA wheel 环境落地后补齐，不估算。

### [sim-claims-ledger](https://github.com/cccjj55/sim-claims-ledger) ✅ 持续维护（`v1.1.0`）

从 COMSOL 多物理场转向机器人仿真时最容易被误导的，是脱离 GPU 型号、commit 和任务场景的加速倍数宣传。这个仓库用机器可校验的 `data/claims.yaml` + `scripts/check_claims.py` + 自动生成的 `docs/claims_overview.svg` 把来源拆成 `UPSTREAM` / `3P` / `MEASURED` 三档，没有硬件指纹和可复现链接就不入表。

---

## 2. 高性能计算与数值求解（HPC / CAE）

### [fem-diffusion-cuda](https://github.com/cccjj55/fem-diffusion-cuda) ✅ CPU + CUDA 实测（`v1.2.0`）

2D P1 三角形非结构化有限元泊松/扩散求解器（C11 + OpenMP + CUDA CSR）：

- **装配 vs 求解耗时真相**：用 **8-color 图着色** 消除 OpenMP 装配锁竞争后，`nx=400` 整体达到 `5.92×`；到 `nx=800`（~64 万自由度），CSR 装配只占 `1.1%`，SpMV 与向量点积占 **`98.8%`** 并撞上 `61.9 GB/s` 的 CPU DRAM 带宽墙。
- **GPU 主机同步陷阱与设备侧循环**：RTX 5070 Ti 上 CSR SpMV 跑满 **`392.9 GB/s`**（基本等于显存拷贝速率 `392.3 GB/s`）；但第一版每步 CG 三次回传标量同步主机，让 `159k DOF` 比 CPU 还慢（**`0.44×`**）。把整条 CG 循环搬进设备侧（`cg_cuda_dev.cu`）后，`159k` 翻回 `1.15×`（`56 ms`），`638k` 提到 `2.91×`（`331 ms`）。
- **当 SpMV 撞墙后，预条件才是最值钱的杠杆**：开启空间非均匀扩散系数 `--graded`（`k(x) = 1 + 49x²`）时，`nx=400`（12 线程）无预条件 CG 需要 `3,136` 次迭代（`453.1 ms`）；加上轻量的 `--precond` Jacobi 对角缩放后，在完全相同的误差精度（`L2 = 3.1154e-06`）下迭代降到 **`1,061` 次（减少 `2.96×`）**，总耗时缩至 **`167.7 ms`（`2.70×` 提速）**。

### [cuda-omp-roofline](https://github.com/cccjj55/cuda-omp-roofline) ✅ CPU + CUDA Roofline（`v1.2.0`）

`axpy` / `reduce` / `softmax` / `matmul` 从 C11 到 OpenMP 再到 CUDA（`sm_120`），坚持**非零确定性输入 + 计时前对 CPU 参考解做 `max|diff|` 校验**，并由 `scripts/plot_roofline.py` 自动绘制 `docs/roofline.svg`：

- **带宽斜率**：CPU `axpy` 测得 `58.35 GB/s`，GPU `axpy` 达 `402.40 GB/s`，warp-shuffle `reduce` 达 `417.9 GB/s`。
- **算力屋顶与 FP64 裁剪**：GPU 分块 `matmul` 测得 `2,285.91 GFLOP/s`（FP32）vs `444.42 GFLOP/s`（FP64，受限于消费级显卡 FP64 单元比例，约 `5.1×` 落差）；CPU 缓存分块 + OpenMP `matmul` 为 `70.62 GFLOP/s`。
- **`softmax` 反面教材**：初版单 block 归约只有 `0.2679 Gelem/s`（慢于 12 核 CPU）；重写为 grid-stride 两遍归约 + 设备端标量后跳到 **`8.6532 Gelem/s`**（相对初版 **`32.3×`**、相对 CPU **`7.2×`**，`max|diff| = 1.19e-18`）。

### [jacobi-halo-mpi](https://github.com/cccjj55/jacobi-halo-mpi) ✅ 本机 + CI + 反直觉实测（`v1.1.0`）

2D 显式 Jacobi 域分解（1D 行条带、`MPI_Sendrecv` 与 `MPI_Isend/Irecv` 计算通信重叠双模式）：

- **不变量验证**：CI 门禁与本机 1/2/4/8 rank、双通信模式下全局 checksum（如 `256×256, 50 steps` = `26285.462047`）逐位一致。
- **L3 缓存边界**：同一套代码 8 rank 扩展到 `1024²`（缓存友好）拿 **`6.43×`**，一扩到 `2048²`（32 MB/数组溢出 L3）就掉回 **`3.25×`**。
- **单机节点的重叠负结果**：在共享内存 WSL2 单机上，`MPI_Isend/Irecv` 内外圈拆分反而不如 `MPI_Sendrecv`（`1024²`: `0.110` vs `0.050 ms/step`；`4096²`: `6.730` vs `5.757 ms/step`），直观验证了计算-通信重叠必须配对跨节点真实网络延迟或大消息窗口。

---

## 3. 机器人系统、实时接口与数据清洗

### [nav2-mission-fsm](https://github.com/cccjj55/nav2-mission-fsm) ✅ 可跑（`v0.2.0`）

仓储 AMR 的 ROS 2 Humble 软件栈——导控本体包装官方 `nav2_bringup`，把易碎的任务层与可观测性抽出来单测：

- `mission_fsm.py`：显式迁移表 + 重试预算 + 看门狗超时，零 ROS 依赖，24 个 pytest 在 0.2 s 内跑完。
- `mission_server.py`：`NavigateToPose` 异步链 + `1 Hz` 看门狗，规避定时器里阻塞 spin 的死锁陷阱。
- `bench_goals.py` + `metrics.py`：随机目标压测落 JSONL，自动化汇总 P50/P95/失败原因回写 README。

### [servo-frame-probe](https://github.com/cccjj55/servo-frame-probe) ✅ 协议与度量工具就绪

面向 1 kHz 关节伺服链路的帧解析与串口抖动诊断：`0xAA55` + `seq` + `CRC-16/CCITT-FALSE`（校验覆盖 `seq` 与 payload 以拦截重放/乱序帧）、`iter_frames` 字节流重同步，以及 `jitter_report.py` 复算脚本（9 个单测；在真机逻辑分析仪抓取前明确仅用合成 CSV 验证工具链）。

### [lerobot-qc-pipeline](https://github.com/cccjj55/lerobot-qc-pipeline) 🟡 合成数据链路打通

`profile.py` + `quality.py`（定频重采样 + `k×MAD` 异常动作段过滤）+ `dataset_card.md` 模板，在策略训练前先暴露缺帧、timestamp 突发抖动与异常幅值。

---

## 4. 工程效率与早期积累

- [dsh-plugin-bench](https://github.com/cccjj55/dsh-plugin-bench) & [dsh-plugin-wsl](https://github.com/cccjj55/dsh-plugin-wsl)：带 `spread` + `cv` 双闸门、上次运行 delta 对比及环境指纹的命令行跑分工具（31 tests，三平台 CI）。
- [rrt-dwa-ppo-planner](https://github.com/cccjj55/rrt-dwa-ppo-planner)：RRT 全局规划 + DWA 局部避障 + PPO 在线权重调节。
- [fleet-sim-multiairport](https://github.com/cccjj55/fleet-sim-multiairport)：多机场机队离散事件调度仿真（makespan / 延误 / 吞吐）。
