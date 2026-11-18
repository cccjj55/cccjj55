# 陈泽一 · Zeyi Chen

**浙江大学** 机械工程 硕士在读（2026–）｜**中南大学** 交通设备与控制工程 本科（专业排名 1/29）

杭州 · 13567058401@163.com · [github.com/cccjj55](https://github.com/cccjj55)

> 求职：机器人仿真 / 具身数据 / 实时控制方向实习　·　2026 年 12 月中旬可到岗，全职 3 个月（可延长）

---

## 项目经历

### 仿真吞吐与精度-速度前沿　`2026.11 – 至今`　[repo](https://github.com/cccjj55/sim-throughput-benchmark)

自建仿真基准框架，同一套 cart-pole 模型换后端跑，回答"多快、多准"，而不是只报一个吞吐数。

- 实测（RTX 5070 Ti Laptop 12 GB / WSL2 / 24 核）：MuJoCo CPU 基线 **212k env-steps/s**，
  MJX（CPU）**522k env-steps/s**；env 从 8 加到 64，前者几乎不变、后者涨 65% —— 说明"上 GPU 才快"是未经检验的假设。
- 精度侧：以 `dt=1e-4` 为参考解，`dt=0.01` 时轨迹 RMSE **2.30 mm**、加速 **58×**；
  输出"误差 1 mm 以内的最大吞吐"，用于时间步选型。
- 框架按 `Backend.step(n)` 契约可插拔（dummy / MuJoCo / MJX 已接，Gazebo / Isaac Lab 待接）；
  CI 在无 GPU 环境也能全绿，README 数字全部由脚本生成。

### 仓储 AMR 的 ROS 2 软件栈　`2026.06 – 至今`　[repo](https://github.com/cccjj55/ros2-warehouse-amr)

导航本体用官方 Nav2，自己写的是任务层与度量层。

- 任务状态机与 ROS 解耦（零 ROS 依赖）：空闲→取货→放货→完成，每段航点有重试上限，整单有超时保护；
  非法事件直接抛错。**24 个 pytest 覆盖全分支，CI 0.2 s 跑完**。
- 修掉三个"能看不能跑"的缺陷：`spawner.py`→`spawner`（控制器根本没加载）、Foxy 时代导航节点列表、
  AMCL 初始位姿落在地图原点（原点是墙，粒子云发散）。`CHANGELOG.md` 逐条记根因。
- 度量层：随机 25 个目标压测 → JSONL → 成功率 / P50 / P95 / 失败原因（超时·取消·被拒）→ 脚本自动回写 README。

### 关节伺服通信与实时性度量　`2026.11 – 至今`　[repo](https://github.com/cccjj55/rt-joint-control)

- 二进制帧协议：`0xAA55` + seq + cmd + len + payload + **CRC-16/CCITT-FALSE**；CRC 覆盖 seq，
  可识别重复/乱序帧；接收端逐字节重同步，一帧错不废整条流。**9 个单测**（往返、CRC 失败、垃圾前缀、截断尾帧）。
- jitter 报告工具从原始 CSV 复算 P50/P90/P99 与周期标准差，README 数字不手填。
- 待补：STM32 1 kHz 双环固件、逻辑分析仪测点、CFS / `SCHED_FIFO` / PREEMPT_RT 三配置实测、端到端时延四段分解。

### 具身示教数据管线　`2026.11 – 至今`　[repo](https://github.com/cccjj55/embodied-data-pipeline)

- `profile`（episode 数 / 时长 / 频率 / 动作分布 / 丢帧）→ 重采样到目标频率 → **基于 MAD 的动作差分剔除** →
  导出 `dataset_card.md`（含"已知缺陷"必填栏）。
- 合成数据验证：3 episodes / 600 frames / 30.1 Hz，剔除率 2.12%；CI 每次跑完整管线。
- 待补：真实示教 20 条、LeRobot 格式导出、BC baseline 成功率。

### 其他项目（简）

- **无人物流小车 AGV**（2024.09–2025.03）：PCB 设计、SolidWorks 建模 + ANSYS 力学仿真、3D 打印、Keil 固件、
  OpenCV 二维码/货物识别与巡线，整机装配联调。
- **方程式赛车队 电气与无人驾驶**（2024.09–）：Simulink 融合油门/转角/轮速，CarSim 闭环后集成主控；
  低压电路与故障检测；Jetson Xavier NX + ROS + OpenCV 视觉；基于 MPC 的路径规划。
- **RRT + DWA + PPO 规划器**：[repo](https://github.com/cccjj55/Autonomous-driving-algorithm-RRT-_DWA-based-on-PPO-regulation-final)，
  用 PPO 调 DWA 权重，验证"学习调参"替代手工调参。
- **多机场机队仿真**：[repo](https://github.com/cccjj55/fleet_sim_multiairport_v9)，多智能体调度与吞吐/延误指标。

---

## 论文与专利

- **第一作者** — *Reducing precious metal concentration in PEMWE anodes catalyst layer…*,
  **Int. J. Hydrogen Energy**, 2025, 164: 150838（JCR Q1, IF 8.1）
- 合作作者 — *Int. J. Hydrogen Energy*, 2024, 67: 390-405；*Renewable Energy*, 2024, 235: 121290（Q1, IF 9.0）
- 实用新型专利 2 项（已授权）：转子发动机组合型偏心轴（ZL 202420052228.4）、可调式防护结构（ZL 202420306991.5）

> 方向是多物理场仿真与实验对标，不是具身智能。放在这里是因为方法一致：建模 → 扫参数 → 实验验证 → 量化偏差。

---

## 技能

| 类别 | 熟练使用 | 用过（跑通过 demo / 小规模） | 学习中 |
| --- | --- | --- | --- |
| 仿真 | Gazebo Classic + Nav2、MuJoCo | MJX（JAX 批量）、CarSim | Isaac Sim / Isaac Lab、Genesis |
| 实时与控制 | STM32、PID、二进制协议 | PREEMPT_RT、逻辑分析仪时序测量 | EtherCAT、HIL |
| 数据 | Python（pandas / numpy / pytest）、JSONL/CSV | LeRobot 数据格式 | Parquet、webdataset |
| CAE | COMSOL 多物理场、ANSYS（结构/热）、Simulink | — | Icepak / Flotherm |
| 工程 | Linux、Git、Docker、GitHub Actions | Shell 自动化 | K8s、CUDA/Triton |

---

## 教育

- **浙江大学** · 机械工程 · 硕士 · 2026 – 至今
- **中南大学** · 交通设备与控制工程 · 本科 · 2022 – 2026｜加权 92.49（1/29）｜国家奖学金

## 荣誉

美国大学生数学建模竞赛 H 奖 · 全国大学生节能减排竞赛国家三等奖 · 全国大学生英语竞赛国家三等奖 ·
中南大学升华杯科技作品一等奖 · 国家奖学金 / 校一等奖学金 / 优秀学生
