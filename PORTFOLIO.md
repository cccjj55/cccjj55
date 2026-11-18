# Portfolio — by category

> 每条只写三件事：**技术栈 / 证据在哪 / 能不能复现**。
> 状态标记：**✅ 有代码有数据** · **🟡 跑通过官方 demo** · **⬜ 在读或未开始**（不写"熟悉/精通"）

---

## A. 高性能仿真 / Performance

| 项目 | 技术栈 | 证据 | 状态 |
| --- | --- | --- | --- |
| 仿真吞吐基准 `sim-throughput-benchmark` | Python · MuJoCo · MJX · JAX · CUDA | [repo](https://github.com/cccjj55/sim-throughput-benchmark) · `results/latest.md` · CI | ✅ 四后端框架 + MuJoCo CPU 基线 + MJX GPU 后端，CI 无 GPU 也绿 |
| 吞吐–精度 Pareto `bench/accuracy.py` | 数值积分 · dt 扫描 · RMSE | 脚本 + CI（dt 1e-3~1e-2 vs 参考解 dt=1e-4） | ✅ 出表；数字待本机采 |
| 瓶颈定位方法 | `nsys` / `ncu` · CUDA 基础 | — | ⬜ 在读 |

**能说的话**：现在能讲"`env-steps/s` 与 `FPS` 的区别、spread 大于 20% 不算测量"；**还不能**讲"我把某个 kernel 做到理论带宽的 X%"。

---

| 文献值对照表 | 论文/仓库声明 + 硬件 + 版本 + 证据标签 | [robot-sim-benchmarks](https://github.com/cccjj55/robot-sim-benchmarks)（`UPSTREAM` / `3P` / `MEASURED` 三级） | ✅ 维护中 |

## B. 机器人仿真栈 / Simulation

| 引擎 | 用途 | 证据 | 状态 |
| --- | --- | --- | --- |
| **Gazebo Classic + Nav2** | 仓储 AMR 全栈（定位·规划·控制） | [ros2-warehouse-amr](https://github.com/cccjj55/ros2-warehouse-amr)，一条命令起，CI 校验 launch | ✅ |
| **MuJoCo** | 刚体动力学、系统辨识 | `backends/mujoco_cpu.py`（内联 cart-pole，CI 每次跑） | ✅ |
| **MuJoCo MJX** | GPU 批量仿真 | `backends/mjx.py`（vmap + jit + fori_loop，含 CPU 回落） | 🟡 代码就绪，待 GPU 采数 |
| **Isaac Sim / Isaac Lab** | PhysX 5 并行环境、合成数据 | — | ⬜ 计划 12 月第 3 周 |
| Genesis / ManiSkill3 | 备选后端（同一 harness 插拔） | — | ⬜ |

**为什么按引擎分**：同一个任务在不同引擎上的**物理保真度**和**吞吐**是两件事。我的 harness 设计成可插拔（`Backend.step(n)`），换引擎只加一个文件。

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
| 结构/热仿真 | ANSYS Mechanical / Thermal | 转子发动机、AGV 车体多场景力学 | ✅ |
| 代理模型（AI4CAE） | Python · MLP / GP · 数据来自 COMSOL 扫描 | — | ⬜ 计划 |

---

## 技术栈总表（三级，不许模糊）

| 类别 | ✅ 有代码/有数据 | 🟡 跑通过 demo | ⬜ 在读/未开始 |
| --- | --- | --- | --- |
| **仿真引擎** | Gazebo Classic | MuJoCo · MJX | Isaac Sim / Isaac Lab · Genesis · ManiSkill3 |
| **AI Infra** | Docker · GitHub Actions · pytest/CI 门禁 | JAX | CUDA · Triton · Ray · NCCL · K8s · vLLM |
| **实时系统** | STM32 · PID · 二进制协议 | PREEMPT_RT · WSL/Linux | EtherCAT · HIL（Speedgoat/dSPACE） |
| **数据** | pandas · JSONL/CSV · Matplotlib | LeRobot 格式 | Parquet · webdataset · RLDS |
| **CAE** | COMSOL · ANSYS（结构/热） · Simulink | CarSim | Icepak/Flotherm · Lumerical（硅光） |
| **语言/工程** | Python · C++（读写） · MATLAB · Git · Linux | Shell 脚本 · Dockerfile | CMake · 多线程/无锁 |

**用法**：面试时只说 ✅ 那一列；🟡 说"跑通过 demo，还没自己写"；⬜ 一律说"在学"。反过来做一次，就够被人记住。

---

## 复现清单 / Reproductions（"借别人成果"的正确姿势）

想把别人的东西放到自己主页，**只有一种合法做法：自己跑一遍，署名上游，报自己的数**。
这也是招聘方最认的"借力"方式——它证明你能复现，而不是能截图。

| 上游 | 我要复现什么 | 署名格式 | 状态 |
| --- | --- | --- | --- |
| [MuJoCo Playground](https://github.com/google-deepmind/mujoco_playground) / MJX 示例 | 同任务吞吐（env 数 × device） | `Reproduced from mujoco_playground@<sha>, measured on RTX 5070 Ti Laptop 12 GB: <N> env-steps/s` | ⬜ |
| [Isaac Lab](https://github.com/isaac-sim/IsaacLab) 基准任务 | 同任务吞吐与显存 | 同上格式 | ⬜ |
| [ManiSkill3](https://github.com/haosulab/ManiSkill) | 同任务吞吐 | 同上 | ⬜ |
| [LeRobot](https://github.com/huggingface/lerobot) | 用其数据集格式 + ACT/DP 训练 | `Trained with lerobot@<sha> on <my dataset>` | ⬜ |
| [Nav2](https://github.com/ros-navigation/navigation2) bringup | 已复现并二次封装 | 已在 README 与 `NOTICE.md` 标注 | ✅ |

**三条底线**（踩了就是不可逆的减分）：
1. 别人的 GIF/截图 **不当自己的 demo** 发，只能作为"上游效果"引用并注明。
2. 数字必须是自己机器上测的，写清硬件、版本、commit。
3. 有上游代码参与的地方，`NOTICE.md` 里逐文件写清（`ros2-warehouse-amr` 已经这么做了）。

---

## 复现命令（照抄即可，WSL 里跑）

```bash
# MuJoCo / MJX
python3 -m venv ~/mjx && source ~/mjx/bin/activate
pip install -U pip && pip install mujoco mujoco-mjx "jax[cuda12]" flax
python -c "import jax; print(jax.devices())"

# 吞吐扫描（先 dummy 验管道，再换 mjx 后端）
python bench/harness.py --backend dummy --envs 64,256,1024 --steps 200 --repeat 3
```
