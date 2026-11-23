# Portfolio — by category

> 每条只写三件事：**技术栈 / 证据在哪 / 能不能复现**。
> 状态标记：**✅ 有代码有数据** · **🟡 跑通过官方 demo** · **⬜ 在读或未开始**（不写"熟悉/精通"）

---

## A. 高性能仿真 & HPC / Performance

| 项目 | 技术栈 | 证据 | 状态 |
| --- | --- | --- | --- |
| 环境并行与 C++ 向量化 `parallel-envs` | C++17 · OpenMP · Python · MuJoCo 4.0.1 | [repo](https://github.com/cccjj55/parallel-envs) · `docs/parallel_scaling.svg` · CI (`v0.2.0`)：线程 `0.05×`、进程复用 `5.62×`、C++ OpenMP `11.11×` (`10.75M steps/s`) | ✅ 本机 + CI |
| CAE 并行求解器 `cae-parallel-solver` | C11 · OpenMP · 8-color CSR 装配 · CG | [repo](https://github.com/cccjj55/cae-parallel-solver) · `docs/fem_breakdown.svg` · CI：4.00× `O(h²)` 收敛；`nx=400` `5.92×` (`157.8 GB/s`)，`nx=800` 撞上 `61.9 GB/s` DRAM 墙 | ✅ 本机 + CI |
| MPI 域分解 `mpi-heat` | C11 · OpenMPI · 2D halo 交换 | [repo](https://github.com/cccjj55/mpi-heat) · `docs/scaling.svg` · CI (`v0.1.0`)：checksum 与 rank 无关；`1024²` `6.43×` vs `2048²` `3.25×` | ✅ 本机 + CI |
| HPC 基线 kernel `hpc-kernels` | C11 · OpenMP · 缓存分块 | [repo](https://github.com/cccjj55/hpc-kernels) · `docs/kernels.svg` · CI (`v0.1.0`)：正确性先于计时，matmul 分块+线程 `70.62 GFLOP/s`，`axpy` 测得 DRAM 上限 `58.35 GB/s` | ✅ CPU 跑通，CUDA 待补 |
| 仿真吞吐基准 `sim-throughput-benchmark` | Python · MuJoCo · MJX · JAX | [repo](https://github.com/cccjj55/sim-throughput-benchmark) · `docs/throughput.svg` · `results/latest.md` · CI (`v0.1.0`) | ✅ CPU 基线 + Pareto；GPU 待采 |
| 文献值对照表 `robot-sim-benchmarks` | 论文/仓库声明 + 硬件 + 版本 + 证据标签 | [repo](https://github.com/cccjj55/robot-sim-benchmarks) · `data/claims.yaml` (`UPSTREAM` / `3P` / `MEASURED` 三级，含 EnvPool 对照) | ✅ 维护中 |
| 可复现跑分工具 `dsh-plugin-bench` | Node.js · 双噪声闸门 (`spread` + `cv`) · 指纹 | [repo](https://github.com/cccjj55/dsh-plugin-bench) · 31 tests · 三平台 CI (`v0.1.0`) | ✅ 跑通 |
| GPU 瓶颈定位 | `nsys` / `ncu` · CUDA kernel | — | ⬜ 在读（等 WSL CUDA Toolkit） |

**能说的话**：现在能用自己实测的数据讲清三件事——① 为什么 Python 多线程跑 MuJoCo 会跌到 `0.05×`、C++ OpenMP 如何把 `1body` 推到 `10.75M steps/s`（`11.11×`）以及 `6c/12t` 物理核边界；② 为什么 FEM 里装配只占 `1.1%`、SpMV 占 `77.9%` 且工作集越过 L3 后准确撞上 `axpy` 测出的 `~58–62 GB/s` DRAM 带宽墙；③ 为什么分母带噪声的加速比不可引用。**还不能**讲"我把某个 CUDA kernel 做到理论带宽的 X%"。

---

| GPU 稀疏求解 | CUDA · CSR SpMV · 带宽墙 | [cae-parallel-solver](https://github.com/cccjj55/cae-parallel-solver) v0.2.0：同一矩阵 CPU 12 线程 93.1 GB/s vs GPU 392.9 GB/s（= 实测 copy 带宽，证明带宽受限）；warp-per-row 因每行仅 7 nnz 反而慢 4.6× | ✅ 本机实测 |

## B. 机器人仿真栈 / Simulation

| 引擎 | 用途 | 证据 | 状态 |
| --- | --- | --- | --- |
| **Gazebo Classic + Nav2** | 仓储 AMR 全栈（定位·规划·控制） | [ros2-warehouse-amr](https://github.com/cccjj55/ros2-warehouse-amr)，一条命令起，CI 校验 launch (`v0.2.0`) | ✅ |
| **MuJoCo (Python & C API)** | 刚体动力学、多线程/多进程/OpenMP rollout | [parallel-envs](https://github.com/cccjj55/parallel-envs) (`mujoco 4.0.1` C API + `mujoco.rollout` + `mj_step`) & `backends/mujoco_cpu.py` | ✅ |
| **MuJoCo MJX** | 向量化仿真 | `backends/mjx.py`（vmap + jit + fori_loop，CPU `522k steps/s` 已测） | 🟡 CPU 跑通，待 GPU 采数 |
| **Isaac Sim / Isaac Lab** | PhysX 5 并行环境、合成数据 | — | ⬜ 计划中 |
| Genesis / ManiSkill3 | 备选后端（同一 harness 插拔） | — | ⬜ |

**为什么按引擎分**：同一个任务在不同引擎上的**物理保真度**和**吞吐**是两件事。`sim-throughput-benchmark` 设计成可插拔（`Backend.step(n)`），而 `parallel-envs` 专门切开调度与语言边界（Python loop / GIL / IPC / C++ OpenMP）。

---

## C. 实时控制 / Real-time

| 项目 | 技术栈 | 证据 | 状态 |
| --- | --- | --- | --- |
| 关节伺服通信与度量 `rt-joint-control` | STM32 · 二进制帧 + CRC-16 · Python 工具链 | [repo](https://github.com/cccjj55/rt-joint-control) · 9 个单测 · `jitter_report.py` | ✅ 协议与工具 |
| 1 kHz 回路抖动（三配置） | CFS / `SCHED_FIFO` / PREEMPT_RT · 逻辑分析仪 | 表结构已定 | ⬜ 待上板实测 |
| 端到端时延四段分解 | 采样→总线→计算→PWM | 同上 | ⬜ |

---

## D. 具身数据 / Embodied data

| 项目 | 技术栈 | 证据 | 状态 |
| --- | --- | --- | --- |
| 示教数据管线 `embodied-data-pipeline` | Python · JSONL · MAD 剔除 · 重采样 · LeRobot 格式 | [repo](https://github.com/cccjj55/embodied-data-pipeline) · `reports/` · 数据集卡片模板 | 🟡 管线跑通（**合成数据**） |
| 真机示教 20 条 + BC baseline | LeRobot · ACT / Diffusion Policy | — | ⬜ |
| 数据卡（dataset card） | 数据画像 / 已知缺陷 / 基线 | `dataset_card.md` 模板 | ✅ |

---

## E. 规划与调度 / Planning & scheduling

| 项目 | 技术栈 | 证据 | 状态 |
| --- | --- | --- | --- |
| RRT + DWA + PPO 规划器 | Python · RRT · DWA · PPO | [repo](https://github.com/cccjj55/rrt-dwa-ppo-planner) | ✅ |
| 多机场机队仿真 `fleet-sim` | 多智能体 · 调度 · 离散事件 | [repo](https://github.com/cccjj55/fleet-sim-multiairport) | ✅ |
| 多机 AMR 任务分配 | 拍卖分配 · Nav2 多命名空间 | — | ⬜ 与 fab AMHS 同一类问题 |

---

## F. CAE / 多物理场

| 项目 | 技术栈 | 证据 | 状态 |
| --- | --- | --- | --- |
| PEM 电解水仿真与实验对标 | COMSOL 多物理场 · Simulink · 参数扫描 | **第一作者 IJHE 2025**（Q1, IF 8.1）+ 合作 2 篇 + 2 项专利 | ✅ |
| 有限元并行计算 | C11 · OpenMP · 8-color CSR 装配 · CG | [cae-parallel-solver](https://github.com/cccjj55/cae-parallel-solver)（~64 万自由度实测与带宽墙分析） | ✅ |
| 结构/热仿真 | ANSYS Mechanical / Thermal | 转子发动机、AGV 车体多场景力学 | ✅ |
| 代理模型（AI4CAE） | Python · MLP / GP · 数据来自 COMSOL 扫描 | — | ⬜ 计划 |

---

## 技术栈总表（三级，不许模糊）

| 类别 | ✅ 有代码/有数据 | 🟡 跑通过 demo | ⬜ 在读/未开始 |
| --- | --- | --- | --- |
| **仿真引擎** | Gazebo Classic · MuJoCo (Python & C API) | MJX (CPU) | Isaac Sim / Isaac Lab · Genesis · ManiSkill3 |
| **HPC / 并行** | C11 / C++17 · OpenMP · OpenMPI · 缓存分块 · 图着色 CSR 装配 | JAX | CUDA · Triton · Ray · NCCL · `ncu` / `nsys` · K8s · vLLM |
| **实时系统** | STM32 · PID · 二进制协议 | PREEMPT_RT · WSL/Linux | EtherCAT · HIL（Speedgoat/dSPACE） |
| **数据与工程** | Python · Git · Docker · GitHub Actions · pytest/CI 门禁 · pandas/JSONL | Shell 脚本 · LeRobot 格式 | Parquet · webdataset · CMake · 无锁队列 |
| **CAE** | COMSOL · ANSYS（结构/热） · Simulink | CarSim | Icepak/Flotherm |

**用法**：面试时只说 ✅ 那一列；🟡 说"跑通过 demo，还没自己做深度工程"；⬜ 一律说"在学"。

---

## 复现清单 / Reproductions（"借别人成果"的正确姿势）

想把别人的东西放到自己主页，**只有一种合法做法：自己跑一遍，署名上游，报自己的数**。

| 上游 | 我要复现/借鉴什么 | 署名格式 / 落点 | 状态 |
| --- | --- | --- | --- |
| [mujoco_mpc](https://github.com/google-deepmind/mujoco_mpc) / [EnvPool](https://github.com/sail-sg/envpool) | 线程私有 `mjData` + C++ OpenMP 批量 rollout 架构 | 已实现于 [parallel-envs](https://github.com/cccjj55/parallel-envs) 并附 `NOTICE.md`，12 线程实测 `10.75M env-steps/s` | ✅ |
| [Nav2](https://github.com/ros-navigation/navigation2) bringup | 已复现并二次封装 | 已在 README 与 `NOTICE.md` 标注 | ✅ |
| [MuJoCo Playground](https://github.com/google-deepmind/mujoco_playground) / MJX | GPU 端任务吞吐（env 数 × device） | `Reproduced from mujoco_playground@<sha>, measured on RTX 5070 Ti Laptop 12 GB` | ⬜ |
| [Isaac Lab](https://github.com/isaac-sim/IsaacLab) 基准任务 | 同任务吞吐与显存 | 同上格式 | ⬜ |
| [LeRobot](https://github.com/huggingface/lerobot) | 数据集格式 + ACT/DP 训练 | `Trained with lerobot@<sha> on <my dataset>` | ⬜ |

**三条底线**：
1. 别人的 GIF/截图 **不当自己的 demo** 发，只能作为"上游效果"引用并注明。
2. 数字必须是自己机器上测的，写清硬件、版本、commit。
3. 有上游思路/代码参考的地方，在 `NOTICE.md` 里写清。
