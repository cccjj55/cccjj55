# Portfolio — by category

> **技术栈 / 证据落点 / 能不能独立复现**。  
> 状态记号：**✅ 实测与代码闭环** · **🟡 工具链/CPU 基线跑通** · **⬜ 计划或排队中**（不写"熟悉/精通"这类空词）。

---

## A. 高性能仿真 & HPC / Solvers

| 项目 | 技术栈 | 证据与关键读数 | 状态 |
| --- | --- | --- | --- |
| C++/OpenMP MuJoCo 并行与 `RolloutPool` | C++17 · OpenMP · `ctypes` · MuJoCo 4.0.1 | [mujoco-omp-rollout](https://github.com/cccjj55/mujoco-omp-rollout) (`v1.1.0`) · `docs/parallel_scaling.svg`：线程 `0.05×`、进程复用 `5.62×`、C++ OpenMP `11.11×` (`10.75M env-steps/s`)；`RolloutPool` 与 `mj_step` 逐元素对拍 `atol=1e-12` | ✅ 本机 + CI |
| 2.5D Chiplet 热/IR-Drop Signoff 求解器 | C++17 · CUDA · `cuSPARSE` · RB-GS · 混精度迭代精化 · `ncu` | [chiplet-signoff-cuda](https://github.com/cccjj55/chiplet-signoff-cuda) (`v1.0.0`) · `docs/chiplet_signoff.svg`：87× TIM 跳变、`1024²` 热学 `1,343 ms`（`25.9×` CPU / `2.02×` `cuSPARSE`）、IR-Drop `846 ms`（`2.27×` `cuSPARSE`），基尔霍夫/能量守恒 `< 2e-11` | ✅ 本机 + CI |
| CUDA/OpenMP Roofline 与 `cuBLAS` 对标 | C11 · OpenMP · CUDA (`sm_120`) · `cuBLAS` TC · `ncu` | [cuda-omp-roofline](https://github.com/cccjj55/cuda-omp-roofline) (`v1.3.0`) · `docs/roofline.svg` · `docs/gemm_cublas_ncu.svg`：`4×4` 寄存器分块 `7.14 TFLOP/s`（`5.0×` 于 32×32）vs `cuBLAS` FP32 `18.64` / FP16 TC `55.33 TFLOP/s`，`ncu` MIO stall `2965% → 7.6%` | ✅ CPU + GPU |
| P1 有限元 CUDA/OpenMP 求解器 | C11 · OpenMP · CUDA · 8-color CSR · Jacobi PCG | [fem-diffusion-cuda](https://github.com/cccjj55/fem-diffusion-cuda) (`v1.2.0`) · `docs/fem_breakdown.svg` · `docs/gpu_cg.svg`：CPU `nx=400` `5.92×`、SpMV 撞 `61.9 GB/s` 墙；GPU SpMV `392.9 GB/s`、设备循环 CG `1.15×` (`159k`) / `2.91×` (`638k`)；非均匀介质 `--precond` 迭代 `2.96×` 缩减、耗时 `2.70×` 加速 | ✅ CPU + GPU |
| 2D Jacobi MPI 域分解 | C11 · OpenMPI · `Sendrecv` vs `Isend/Irecv` | [jacobi-halo-mpi](https://github.com/cccjj55/jacobi-halo-mpi) (`v1.1.0`) · `docs/scaling.svg` · CI 门禁：1/2/4/8 rank 与双通信模式全局 checksum 完全一致（`26285.462047`）；`1024²` `6.43×` vs `2048²` `3.25×`；单机实测 `isend` 慢于 `sendrecv` (`0.110` vs `0.050 ms/step`) | ✅ 本机 + CI |
| 仿真步长 `dt` × 吞吐 Pareto | Python · MuJoCo · MJX · JAX | [sim-dt-pareto](https://github.com/cccjj55/sim-dt-pareto) · `docs/throughput.svg` · `results/latest.md` · CI：MuJoCo Python 循环稳在 `212k`，MJX CPU 从 `8→64 envs` 扩到 `522k` | 🟡 CPU + Pareto |
| 仿真声明核验账本 | YAML schema · SVG 生成 · GitHub Actions | [sim-claims-ledger](https://github.com/cccjj55/sim-claims-ledger) (`v1.1.0`) · `docs/claims_overview.svg` · `data/claims.yaml` (`UPSTREAM` / `3P` / `MEASURED` 证据分级 + CI 校验) | ✅ 维护中 |
| 可复现跑分插件 `dsh-plugin-bench` | Node.js · 双噪声闸门 (`spread` + `cv`) · 机器指纹 | [dsh-plugin-bench](https://github.com/cccjj55/dsh-plugin-bench) · 31 tests · 三平台 CI（含真实超时杀进程单测） | ✅ 跑通 |
| `ncu` 硬件计数器深度剖析 | Nsight Compute 2025.1.0 (`CC 12.0`) | [chiplet-signoff-cuda](https://github.com/cccjj55/chiplet-signoff-cuda) & [cuda-omp-roofline](https://github.com/cccjj55/cuda-omp-roofline)：实采 `DRAM%`、`L1/L2` 命中、`Occupancy`、`Tensor Pipe`、`MIO Throttle` 与 `Long Scoreboard` | ✅ 实测闭环 |

**现在能用自己实测数据讲清的六件事**：
1. **语言与并发边界**：为什么 Python `threading` 跑 MuJoCo 会跌到 `0.05×`、持久化多进程只能救到 `5.62×`、而 C++ OpenMP 线程私有 `mjData` 能把 `1body` 推到 `10.75M env-steps/s`（`11.11×`）以及 `6 物理核 / 12 线程` 对 `12body` 的硬上限。
2. **规模反转与混精度迭代精化（`chiplet-signoff-cuda`）**：在真实 Chiplet `87×` TIM 跳变问题上，为什么 `512²` 下 `cuda_fused_fp64_jacobi` 最快（`123.9 ms`），而溢出 L2 的 `1024²` 下 **`cuda_mixed_fp32_rbgs`（FP32 RB-GS 内循环 + FP64 残差精化）** 通过 `ncu` 证实的 `-33.4%` SpMV 读流量（`75.5 → 50.3 MB`）反超 `cuSPARSE` `2.02×–2.27×` 且保持 `10⁻¹¹` 物理守恒精度。
3. **高 Occupancy 不等于高算力（`ncu` 硬件证据）**：为什么 `gemm_smem_32x32` 有 `66.7%` occupancy 却卡死在 `2,965.6%` `MIO Throttle`（共享内存管线堵塞，只有 `1.33 TFLOP/s`），`4×4` 寄存器分块砍掉 `2.6×` MIO stall 翻到 `5.90–7.14 TFLOP/s`，而 CUTLASS `cuBLAS` 主动降到 `8.3%–16.6%` occupancy 来换超大寄存器分块与 async 流水，把 MIO stall 压到 `2%–8%`、跑出 **`55.33 TFLOP/s` FP16 Tensor Core**。
4. **内存墙与条件数预条件（`fem-diffusion-cuda`）**：为什么 FEM 装配只占 `1.1%`、SpMV 越过 L3 后撞上 CPU `~58–62 GB/s` 和 GPU `392.9 GB/s` 带宽墙；消除主机同步可救回小规模 CG（`0.44× → 1.15×`），而在病态 `k(x)=1+49x²` 下 Jacobi 预条件可再砍掉 `2.96×` 迭代（`2.70×` 提速）。
5. **单机 MPI 非阻塞重叠的反直觉（`jacobi-halo-mpi`）**：在共享内存宿主机内，`MPI_Isend/Irecv` 内外圈拆分因软件状态机开销输给紧凑的 `MPI_Sendrecv`（`0.110` vs `0.050 ms/step`）。
6. **GPU kernel 归约边界**：naive 单 block `softmax` 比 12 核 CPU 慢 `4.5×`，改成 grid-stride + warp shuffle 后提升 `32.3×` 且保持双精度误差 `< 1.2e-18`。

---

## B. 机器人仿真栈 / Robotics Simulation

| 引擎 / 工具 | 用途 | 证据 | 状态 |
| --- | --- | --- | --- |
| **Gazebo Classic + Nav2** | 仓储 AMR 全栈（定位·规划·控制 + 任务 FSM） | [nav2-mission-fsm](https://github.com/cccjj55/nav2-mission-fsm) (`v0.2.0`) · 24 pytest + ROS 2 Humble 容器 CI（colcon + xacro + launch 校验） | ✅ |
| **MuJoCo (Python & C API)** | 刚体动力学、GIL/进程/OpenMP rollout 与 C ABI 封装 | [mujoco-omp-rollout](https://github.com/cccjj55/mujoco-omp-rollout) (`v1.1.0`) & `backends/mujoco_cpu.py` | ✅ |
| **MuJoCo MJX** | 向量化仿真 (`vmap` + `jit` + `fori_loop`) | [sim-dt-pareto](https://github.com/cccjj55/sim-dt-pareto) (`CPU 522k steps/s` 已测) | 🟡 CPU 跑通，待 GPU 采数 |
| **Isaac Sim / Isaac Lab / Genesis** | 统一 `Backend.step(n)` 契约下的跨引擎对标 | 协议与基准表结构已就位 | ⬜ 计划中 |

---

## C. 实时控制、数据与规划 / Control, Data & Planning

| 项目 | 技术栈 | 证据 | 状态 |
| --- | --- | --- | --- |
| 伺服帧协议与抖动剖析 | Python · `CRC-16/CCITT-FALSE` · P50/P95/P99 CSV 复算 | [servo-frame-probe](https://github.com/cccjj55/servo-frame-probe) · 9 个单测 · `jitter_report.py`（固件上板前仅用合成 CSV 验证工具链） | ✅ 协议与工具 |
| 示教轨迹清洗管线 | Python · 重采样 · `k×MAD` 离群剔除 · Dataset Card | [lerobot-qc-pipeline](https://github.com/cccjj55/lerobot-qc-pipeline) · 合成数据端到端报告 + 卡模板 | 🟡 合成数据跑通 |
| RRT + DWA + PPO 混合规划器 | Python · 全局 RRT · 局部 DWA · PPO 在线调权 | [rrt-dwa-ppo-planner](https://github.com/cccjj55/rrt-dwa-ppo-planner) | ✅ |
| 多机场机队离散事件仿真 | 多智能体 · 离散事件调度 | [fleet-sim-multiairport](https://github.com/cccjj55/fleet-sim-multiairport) | ✅ |

---

## D. CAE / 多物理场研究背景

| 方向 | 工具与物理问题 | 成果与对应代码库 | 状态 |
| --- | --- | --- | --- |
| PEM 电解水多物理场建模与实验对标 | COMSOL Multiphysics · Simulink · 两相流/电化学/热耦合 | **第一作者 IJHE 2025**（Q1, IF 8.1）+ 合作 Q1 论文 2 篇 + 已授权发明专利 2 项 | ✅ |
| 非结构化有限元并行求解 | C11 · OpenMP · 8-color CSR 装配 · CUDA CG/PCG | [fem-diffusion-cuda](https://github.com/cccjj55/fem-diffusion-cuda) | ✅ |
| 结构/热力学与车体集成 | ANSYS Mechanical/Thermal · 车体多工况力学 | 大学生方程式（FSAE）与工程车辆集成背景 | ✅ |

---

## 技术栈诚实分级（面试对照）

| 类别 | ✅ 有代码与实测数据 | 🟡 跑通过基线/工具链 | ⬜ 学习与规划中 |
| --- | --- | --- | --- |
| **仿真与机器人** | MuJoCo (Python & C API) · Gazebo Classic · Nav2 · URDF/Xacro | MJX (CPU) · LeRobot 清洗流程 | Isaac Sim / Isaac Lab · Genesis · ManiSkill3 |
| **HPC & 并行** | C11 / C++17 · OpenMP · OpenMPI · CUDA（`axpy`/`reduce`/`softmax`/寄存器分块 GEMM/`cuBLAS` Tensor Core/`cuSPARSE`/混精度 RB-GS+FP64 IR）· `ncu` 硬件计数剖析 · 图着色 CSR | JAX (CPU) | Triton · NCCL · `nsys` · Ray |
| **实时与工具** | STM32 / 二进制帧 + CRC-16 · Python · pytest · Docker · GitHub Actions | Linux PREEMPT_RT 指标口径 · WSL2 | EtherCAT · HIL（dSPACE/Speedgoat） |
| **CAE** | COMSOL · ANSYS · Simulink | CarSim | Icepak |

**口头原则**：✅ 行能现场打开代码、跑出命令、讲出数字出处和负优化对照；🟡 行明确说"基线已通、GPU 或真机闭环还在补"；⬜ 行直接说"尚未实操"。
