# Projects — 详细版

> README 只放一屏，细节放这里（Yanjie Ze 的做法：README 短表 + `PROJECTS.md` 长表）。
> 面试官点进主页看到短表，想深挖时会点 "All projects"，这时候才需要细节。

## 1. 机器人系统 / 仿真

### [ros2-warehouse-amr](https://github.com/cccjj55/ros2-warehouse-amr) ✅ 可跑

仓储 AMR 的 ROS 2 软件栈。**我写的是任务层与度量层**，导航本体是官方的 Nav2 `nav2_bringup`。

| 模块 | 文件 | 说明 |
| --- | --- | --- |
| 任务状态机 | `warehouse_amr_mission/mission_fsm.py` | 显式迁移表 + 每段重试预算 + 任务级看门狗，零 ROS 依赖 |
| 执行器 | `mission_server.py` | Nav2 `NavigateToPose` 异步链，不阻塞 executor；JSON 状态发布 |
| 基准 | `bench_goals.py` + `metrics.py` | 随机目标压测 → JSONL → P50/P95/失败原因 → 自动回写 README |
| 修复 | `launch/*.launch.py` | 修掉 3 个导致 Humble 上起不来的缺陷（`spawner.py`、Foxy 节点列表、AMCL 初始位姿落在墙里） |

- 测试：24 个 pytest，CI 里 0.2 s 跑完
- CI：colcon build + xacro 展开 + YAML 解析 + `ros2 launch --show-args` 解析校验
- 证据：`CHANGELOG.md` 里每个 bug 都有根因

### [rt-joint-control](https://github.com/cccjj55/rt-joint-control) 🚧

1 kHz 关节伺服的实时性证据链。

- 二进制帧协议（`0xAA55` + seq + cmd + len + payload + CRC-16/CCITT-FALSE），接收端可重同步
- `tools/jitter_report.py`：从原始 CSV 复算 jitter P50/P90/P99，README 不手填
- 单测 9 个（编解码往返、CRC 失败、垃圾前缀重同步、截断尾帧）
- 待补：STM32 固件、逻辑分析仪测点、三配置实测对比、端到端时延四段分解

### [sim-throughput-benchmark](https://github.com/cccjj55/sim-throughput-benchmark) 🚧

仿真吞吐基准，统一 backend 接口（`step(n)` 一行契约）。

- `bench/harness.py`：扫 env 数 → `env_steps_per_s` / `steps_per_s` / `spread_pct` → CSV + Markdown
- `bench/backends/dummy.py`：自测后端 + Python 侧开销下限（用来验证 harness 本身）
- 待补：Gazebo（CPU 基线）、Isaac Lab、MuJoCo MJX 三个 adapter；4090 实测曲线与拐点分析

### [embodied-data-pipeline](https://github.com/cccjj55/embodied-data-pipeline) 🚧

示教数据从"一堆文件"到"能训练的数据集"。

- `profile.py`：episode 数 / 时长 / 频率 / 动作分布 / 丢帧（`dt > 2.5×P50`）
- `quality.py`：重采样到目标频率 + 基于 **MAD** 的异常帧剔除（尺度自适应，不用固定阈值）
- `dataset_card.md`：模板含"已知缺陷"必填栏
- 实测（合成数据）：`3 episodes / 600 frames / 30.1 Hz / 剔除 2.12%`
- 待补：真实示教数据 20 条、LeRobot 导出、BC baseline 成功率

## 2. 规划 / 调度（已有仓库，补描述即可）

### [rrt-dwa-ppo-planner](https://github.com/cccjj55/Autonomous-driving-algorithm-RRT-_DWA-based-on-PPO-regulation-final) ✅

RRT 做全局规划、DWA 做局部避障，用 PPO 调 DWA 的权重。

- 价值点：**"学习调参"替代"手工调参"** 的最小验证
- 面试可讲：为什么用 PPO 而不是网格搜索（连续权重空间、在线适应）
- 建议：仓库名太长，改名为 `rrt-dwa-ppo-planner`（GitHub 会自动重定向）

### [fleet-sim](https://github.com/cccjj55/fleet_sim_multiairport_v9) ✅

多机场机队仿真。

- 价值点：**多智能体调度 + 仿真**，和 fab AMHS / 多机 AMR 调度是同一类问题
- 建议：加 README 说清"仿真了什么指标"（makespan / 延误 / 吞吐），并改名 `fleet-sim`

### [quant-research](https://github.com/cccjj55/quant-research) —

量化研究实验记录（FinSight update_ETF）。与机器人无关，**建议设为 private 或加一行说明**，
避免主页上出现一个"没有描述"的仓库。

## 3. 论文与专利

见 [README](../README.md) 底部 Publications 段。要点：3 篇 Q1（1 篇一作）+ 2 项实用新型专利，
方向是多物理场仿真与实验对标，**不是具身智能**——写清楚反而更可信。

## 4. 一条主线

所有项目都在回答同一个问题：**"你说的这个数字，我能不能自己复现？"**

- 机器人：导航成功率 / P50 / P95 —— 由 `bench_goals` 产出
- 实时性：jitter P99 / 端到端时延 —— 由 `jitter_report.py` 从 CSV 复算
- 仿真：`env-steps/s per GPU` —— 由 harness 扫描产出
- 数据：剔除率 / 频率分布 —— 由 `profile.py`/`quality.py` 产出
