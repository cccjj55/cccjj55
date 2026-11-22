**Robot simulation · Parallel HPC · Multiphysics CAE.** Measuring where physics loops and numerical solvers hit CPU, memory-bus, and GPU walls.

[![AMR CI](https://github.com/cccjj55/ros2-warehouse-amr/actions/workflows/ci.yml/badge.svg)](https://github.com/cccjj55/ros2-warehouse-amr/actions)
[![parallel-envs CI](https://github.com/cccjj55/parallel-envs/actions/workflows/ci.yml/badge.svg)](https://github.com/cccjj55/parallel-envs/actions)
[![cae-parallel-solver CI](https://github.com/cccjj55/cae-parallel-solver/actions/workflows/ci.yml/badge.svg)](https://github.com/cccjj55/cae-parallel-solver/actions)

### Embodied Simulation & Control
- [parallel-envs](https://github.com/cccjj55/parallel-envs) — MuJoCo 12-thread CPU scaling: GIL collapse (`0.05×`) → worker reuse (`5.62×`) → C++17 OpenMP (`11.11×`, `10.75M steps/s`)
- [sim-throughput-benchmark](https://github.com/cccjj55/sim-throughput-benchmark) — env-steps/s & accuracy–speed Pareto (MuJoCo · MJX · Isaac Lab)
- [robot-sim-benchmarks](https://github.com/cccjj55/robot-sim-benchmarks) — published sim claims with hardware, version, and `UPSTREAM`/`3P`/`MEASURED` labels
- [ros2-warehouse-amr](https://github.com/cccjj55/ros2-warehouse-amr) — ROS 2 Humble AMR: Nav2 bringup + mission FSM, 24 tests, CI
- [rt-joint-control](https://github.com/cccjj55/rt-joint-control) — 1 kHz joint servo framing (CRC16), jitter, and latency decomposition
- [embodied-data-pipeline](https://github.com/cccjj55/embodied-data-pipeline) — demonstration profile → resample / MAD outlier filter → LeRobot dataset

### HPC & Numerical Simulation
- [cae-parallel-solver](https://github.com/cccjj55/cae-parallel-solver) — 2D P1 FEM 8-color CSR assembly + CG solver: L3 (`157.8 GB/s`) vs DRAM wall (`61.9 GB/s`)
- [mpi-heat](https://github.com/cccjj55/mpi-heat) — 2D Jacobi MPI halo exchange: `6.43×` at `1024²` vs `3.25×` at `2048²` (bandwidth-bound), checksum invariant
- [hpc-kernels](https://github.com/cccjj55/hpc-kernels) — axpy/softmax/matmul in ref, cache-blocked, and OpenMP variants (`70.6 GFLOP/s`), CI + local

### Reproducibility Tooling
- [dsh-plugin-bench](https://github.com/cccjj55/dsh-plugin-bench) & [dsh-plugin-wsl](https://github.com/cccjj55/dsh-plugin-wsl) — warmup + dual-gate spread/CV benchmark with machine fingerprint & WSL bridge

M.Sc. @ Zhejiang University · 3 Q1 papers (1 first-author) · 2 granted patents

[Portfolio by category](https://github.com/cccjj55/cccjj55/blob/main/PORTFOLIO.md) · [Project details](https://github.com/cccjj55/cccjj55/blob/main/PROJECTS.md)
