# Projects — 详细版

> README 只放一屏，细节放这里；分类总表见 [PORTFOLIO.md](PORTFOLIO.md)。

## 1. 机器人仿真 × 并行计算

### [parallel-envs](https://github.com/cccjj55/parallel-envs) ✅ 可跑（`v0.2.0`）

同一 MuJoCo XML 模型、同一初始条件（每模式校验终态 `qpos` checksum 一致），在 6 核 12 线程 CPU 上对比 6 种执行方式：

- `serial` vs `threads`（线程本地预建模型，隔离出纯 GIL 争用导致 `1body` 跌至 `0.05×`）
- `proc_cold` vs `proc_reuse`（持久化 worker 消除每次 XML 解析，从 `3.41× → 5.62×`）
- `mj_rollout`（官方 C++ 多线程 `mujoco.rollout`）vs 自写 **`cpp_openmp`**（`cpp/omp_rollout.cpp`，零 Python / 零 IPC / 零整段轨迹写回，`1body` 达到 `10.75M env-steps/s` / `11.11×`）
- 揭示硬件边界：`lscpu` 显示宿主机为 `6 物理核 × 2 SMT`，因此重物理模型 `12body` 在所有 C++/多进程方案上统一收敛到 `~5.5×–5.7×`

### [sim-throughput-benchmark](https://github.com/cccjj55/sim-throughput-benchmark) ✅ CPU 基线完成（`v0.1.0`）

仿真吞吐基准与精度-速度前沿，统一 backend 接口（`Backend.step(n)`）：

- `bench/harness.py`：扫 env 数 → `env_steps_per_s` / `steps_per_s` / `spread_pct` → CSV + Markdown + `docs/throughput.svg`
- `bench/accuracy.py`：相对 `dt=1e-4` 参考解计算轨迹 RMSE（回答“误差 ≤1 mm 时最快跑多少”）
- 已测：MuJoCo CPU 稳在 ~212k steps/s（受限于 Python 循环），MJX CPU 从 8→64 envs 涨至 522k steps/s；GPU 行待本机 CUDA 就绪后补齐

### [robot-sim-benchmarks](https://github.com/cccjj55/robot-sim-benchmarks) ✅ 策展维护中（`v0.1.0`）

带硬件、版本、出处与证据分级（`UPSTREAM` / `3P` / `MEASURED`）的机器人仿真声明对照表，含 `data/claims.yaml` 与 `docs/evidence-rules.md` 拒收规则，已收入 EnvPool 上游声明及 `sim-throughput-benchmark`、`parallel-envs` 实测行。

---

## 2. 高性能计算与数值求解（HPC / CAE）

### [cae-parallel-solver](https://github.com/cccjj55/cae-parallel-solver) ✅ 可跑（`v1.1.0`）

2D P1 三角形有限元泊松方程求解器（C11 + OpenMP + CUDA，非结构化 CSR 稀疏矩阵装配 + CG 迭代）：

- CPU 侧：`serial` vs `omp_atomic` vs **8-color 无锁并行装配 `omp_colored`**；`make test` 验证 `4.00×` `O(h²)` 收敛且差异 `< 2.5e-15`；`nx=400` 达到 `5.92×`（`157.8 GB/s`），`nx=800` 装配仅占 `1.1%`、SpMV+向量点积占 `98.8%` 且撞上 `61.9 GB/s` DRAM 墙
- GPU 侧（RTX 5070 Ti）：CSR SpMV 达 `392.9 GB/s`（等于设备拷贝带宽 `392.3 GB/s`）；v1 每步三次主机同步导致 `159k DOF` 只有 `0.44×`（`638k` `2.07×`），v2 设备侧循环（`cg_cuda_dev.cu`）将 `159k` 拉回 `1.15×`（`56 ms`）、`638k` 提到 `2.91×`（`331 ms`）

### [mpi-heat](https://github.com/cccjj55/mpi-heat) ✅ 可跑（`v1.0.0`）

2D 显式 Jacobi 热传导（C11 + OpenMPI 1D 行分解 + `MPI_Sendrecv` + `MPI_PROC_NULL` 边界）：

- 不变量门禁：CI 自动验证 1/2/4 rank 输出完全一致的全局 checksum
- 12 线程实测：`1024²`（缓存友好）8 rank 达 `6.43×`，同一代码在 `2048²`（32 MB/数组，越过 L3）跌至 `3.25×`

### [hpc-kernels](https://github.com/cccjj55/hpc-kernels) ✅ CPU + GPU 跑通（`v1.1.0`）

`axpy` / `reduce` / `softmax` / `matmul`（C11 / OpenMP / CUDA），**非零输入并在计时前对 CPU 参考解做 `max|diff|` 校验**：

- CPU 侧：`matmul` 分块+OpenMP 12 线程达 `70.62 GFLOP/s`；`axpy` 测得 DRAM 带宽上限 `58.35 GB/s`
- GPU 侧（RTX 5070 Ti）：`axpy` `402.40 GB/s`、`reduce`（warp shuffle）`417.9 GB/s`、`matmul` `2,285.91`（FP32）/ `444.42`（FP64）GFLOP/s；`softmax` 从单 block `0.2679 Gelem/s`（CPU `0.22×`）重写为 grid-stride + 设备端标量后达到 `8.6532 Gelem/s`（`32.3×` 于自身、`7.2×` 于 CPU，`max|diff|=1.19e-18`）

---

## 3. 机器人系统、控制与数据

### [ros2-warehouse-amr](https://github.com/cccjj55/ros2-warehouse-amr) ✅ 可跑（`v0.2.0`）

仓储 AMR 的 ROS 2 Humble 软件栈。**我写的是任务层与度量层**，导航本体包装官方 `nav2_bringup`：

| 模块 | 文件 | 说明 |
| --- | --- | --- |
| 任务状态机 | `warehouse_amr_mission/mission_fsm.py` | 显式迁移表 + 每段重试预算 + 任务级看门狗，零 ROS 依赖 |
| 执行器 | `mission_server.py` | Nav2 `NavigateToPose` 异步链，不阻塞 executor；JSON 状态发布 |
| 基准 | `bench_goals.py` + `metrics.py` | 随机目标压测 → JSONL → P50/P95/失败原因 → 自动回写 README |
| 修复 | `launch/*.launch.py` | 修掉 3 个导致 Humble 上起不来的缺陷（`spawner.py`、Foxy 节点列表、AMCL 初始位姿落在墙里） |

- 测试：24 个 pytest，CI 里 0.2 s 跑完；Gazebo/CI 构建 + xacro 展开 + YAML + `ros2 launch` 校验

### [rt-joint-control](https://github.com/cccjj55/rt-joint-control) 🚧 协议与工具已就绪

1 kHz 关节伺服的实时性证据链：二进制帧（`0xAA55` + seq + CRC-16）、`jitter_report.py` 从原始 CSV 复算 P50/P90/P99、9 个单测；待补真机/逻辑分析仪三配置实测。

### [embodied-data-pipeline](https://github.com/cccjj55/embodied-data-pipeline) 🚧 合成数据跑通

`profile.py` + `quality.py`（重采样 + MAD 自适应异常帧剔除）+ `dataset_card.md` 模板；待补 20 条真机示教与 BC baseline。

---

## 4. 工具插件与早期项目

- [dsh-plugin-bench](https://github.com/cccjj55/dsh-plugin-bench) (`v0.1.0`) & [dsh-plugin-wsl](https://github.com/cccjj55/dsh-plugin-wsl) (`v0.1.0`)：命令 N 次跑分与双噪声闸门（`spread` + `cv`）、上一次基线对比及机器指纹落盘（31 tests，三平台 CI）。
- [rrt-dwa-ppo-planner](https://github.com/cccjj55/rrt-dwa-ppo-planner)：RRT 全局规划 + DWA 局部避障 + PPO 在线调权验证。
- [fleet-sim-multiairport](https://github.com/cccjj55/fleet-sim-multiairport)：多机场机队离散事件调度仿真（makespan / 延误 / 吞吐）。

---

## 5. 一条主线

所有项目都在回答同一个工程问题：**“你说的这个性能/精度数字，有没有基线、硬件指纹和自动化校验？”**
