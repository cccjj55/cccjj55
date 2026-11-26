Hi, I'm **Zeyi Chen (陈泽一)** — M.Sc. Mechanical Engineering @ Zhejiang University (B.Eng. @ CSU).

My path started in **multiphysics PEM electrolyzer simulation** (COMSOL / Simulink, 3 Q1 papers including 1st-author in *IJHE*, 2 granted patents) and Formula-style vehicle integration. That pulled me into a habit I couldn't drop when moving into robotics and GPU computing: **whenever a simulator or solver claims "10× faster", I pin the hardware, check numerical equivalence against a serial reference (`max|diff|` or exact checksum), and measure where time actually goes** — keeping negative results (`0.05×` GIL contention, `0.44×` CUDA host sync, intra-node `MPI_Isend` overhead) right next to the wins.

[![CI](https://github.com/cccjj55/nav2-mission-fsm/actions/workflows/ci.yml/badge.svg)](https://github.com/cccjj55/nav2-mission-fsm/actions)

### Embodied Simulation & Robotics Stack

- [mujoco-omp-rollout](https://github.com/cccjj55/mujoco-omp-rollout) — MuJoCo `mj_step` across Python threads (`0.05×`), persistent multiprocessing (`5.62×`), `mujoco.rollout`, and a reusable C++17 OpenMP `RolloutPool` (`11.11×` / `10.75M env-steps/s`, `atol=1e-12`) ![v](https://img.shields.io/github/v/release/cccjj55/mujoco-omp-rollout?display_name=tag&style=flat&color=0f766e)
- [sim-dt-pareto](https://github.com/cccjj55/sim-dt-pareto) — paired throughput (`env-steps/s`) vs integrator step `dt` RMSE frontier (`MuJoCo` CPU flat at `212k` vs `MJX` CPU scaling to `522k`) ![v](https://img.shields.io/github/v/release/cccjj55/sim-dt-pareto?display_name=tag&style=flat&color=0f766e)
- [sim-claims-ledger](https://github.com/cccjj55/sim-claims-ledger) — reproducible claims ledger tracking hardware, version, and `UPSTREAM / 3P / MEASURED` evidence pills (`docs/claims_overview.svg`) ![v](https://img.shields.io/github/v/release/cccjj55/sim-claims-ledger?display_name=tag&style=flat&color=0f766e)
- [nav2-mission-fsm](https://github.com/cccjj55/nav2-mission-fsm) — ROS 2 Humble warehouse AMR: Nav2 bringup wrapper + ROS-free mission FSM (24 unit tests in 0.2 s) ![v](https://img.shields.io/github/v/release/cccjj55/nav2-mission-fsm?display_name=tag&style=flat&color=0f766e)
- [servo-frame-probe](https://github.com/cccjj55/servo-frame-probe) — 1 kHz binary servo frame protocol (`CRC-16/CCITT-FALSE` over `seq..payload`) and P50/P95/P99 jitter analyzer ![v](https://img.shields.io/github/v/release/cccjj55/servo-frame-probe?display_name=tag&style=flat&color=0f766e)
- [lerobot-qc-pipeline](https://github.com/cccjj55/lerobot-qc-pipeline) — trajectory resampling, `k×MAD` outlier filtering, and dataset card checks prior to BC/ACT training ![v](https://img.shields.io/github/v/release/cccjj55/lerobot-qc-pipeline?display_name=tag&style=flat&color=0f766e)

### HPC & Multiphysics Solvers

- [fem-diffusion-cuda](https://github.com/cccjj55/fem-diffusion-cuda) — 2D P1 FEM assembly (8-color lock-free CSR) + CG on CPU & CUDA: device loop lifts `159k` DOF from `0.44×` to `1.15×` (`2.91×` at `638k`, `392.9 GB/s` SpMV); Jacobi PCG cuts graded-diffusivity iterations `2.96×` (`2.70×` wall speedup) ![v](https://img.shields.io/github/v/release/cccjj55/fem-diffusion-cuda?display_name=tag&style=flat&color=0f766e)
- [cuda-omp-roofline](https://github.com/cccjj55/cuda-omp-roofline) — `axpy` / `reduce` / `softmax` / `matmul` across C11, OpenMP, and CUDA (`sm_120`): CPU-verified (`max|diff|=1.19e-18`), grid-stride `softmax` `32.3×`, plus generated log-log roofline plot ![v](https://img.shields.io/github/v/release/cccjj55/cuda-omp-roofline?display_name=tag&style=flat&color=0f766e)
- [jacobi-halo-mpi](https://github.com/cccjj55/jacobi-halo-mpi) — 2D Jacobi domain decomposition (`MPI_Sendrecv` vs non-blocking `MPI_Isend/Irecv` interior overlap), rank-invariant checksum gates, and L3-threshold (`6.43×` @ `1024²` vs `3.25×` @ `2048²`) + intra-node overlap analysis ![v](https://img.shields.io/github/v/release/cccjj55/jacobi-halo-mpi?display_name=tag&style=flat&color=0f766e)

### Reproducibility Tooling

- [dsh-plugin-bench](https://github.com/cccjj55/dsh-plugin-bench) — command runner with dual noise gates (`spread` + `cv`), baseline delta tracking, and hardware fingerprinting (31 tests across Linux/Windows/macOS) ![v](https://img.shields.io/github/v/release/cccjj55/dsh-plugin-bench?display_name=tag&style=flat&color=0f766e)
- [dsh-plugin-wsl](https://github.com/cccjj55/dsh-plugin-wsl) — run Linux toolchain builds cleanly inside WSL2 from Windows workflows ![v](https://img.shields.io/github/v/release/cccjj55/dsh-plugin-wsl?display_name=tag&style=flat&color=0f766e)

> More notes: [Portfolio evidence matrix](https://github.com/cccjj55/cccjj55/blob/main/PORTFOLIO.md) · [Architecture & negative results](https://github.com/cccjj55/cccjj55/blob/main/PROJECTS.md)
