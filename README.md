<p align="center">
  <img src="assets/jppd_architecture.png" alt="JPPD architecture" width="100%">
</p>

<h1 align="center">JPPD: Joint Prediction-Planning Diffusion</h1>

<h3 align="center">
  Differentiable safety-guided joint trajectory generation for dynamic obstacle avoidance in shared-space ITS.
</h3>

<p align="center">
  <img alt="Article" src="https://img.shields.io/badge/article-pending%20publication-0f766e?style=for-the-badge">
  <img alt="Method" src="https://img.shields.io/badge/method-joint%20diffusion-2563eb?style=for-the-badge">
  <img alt="Safety" src="https://img.shields.io/badge/guidance-DSPG-dc2626?style=for-the-badge">
  <img alt="Platform" src="https://img.shields.io/badge/platform-ROSOrin-7c3aed?style=for-the-badge">
</p>

<p align="center">
  <img alt="Python" src="https://img.shields.io/badge/Python-placeholder-3776AB?style=flat-square&logo=python&logoColor=white">
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-planned-EE4C2C?style=flat-square&logo=pytorch&logoColor=white">
  <img alt="ROS" src="https://img.shields.io/badge/ROS-ROSOrin%20deployment-22314E?style=flat-square&logo=ros&logoColor=white">
  <img alt="Isaac Sim" src="https://img.shields.io/badge/Isaac%20Sim-3D%20validation-76B900?style=flat-square&logo=nvidia&logoColor=white">
  <img alt="Status" src="https://img.shields.io/badge/code-release%20staging-F59E0B?style=flat-square">
</p>

<p align="center">
  <a href="#overview"><img alt="Overview" src="https://img.shields.io/badge/overview-111827?style=flat-square"></a>
  <a href="#visual-story"><img alt="Visual Story" src="https://img.shields.io/badge/visual%20story-4338ca?style=flat-square"></a>
  <a href="#why-blade"><img alt="BLADE" src="https://img.shields.io/badge/BLADE%20clarification-be123c?style=flat-square"></a>
  <a href="#method"><img alt="Method" src="https://img.shields.io/badge/method-0f766e?style=flat-square"></a>
  <a href="#results"><img alt="Results" src="https://img.shields.io/badge/results-15803d?style=flat-square"></a>
  <a href="#repository-status"><img alt="Repo Status" src="https://img.shields.io/badge/repository%20status-b45309?style=flat-square"></a>
</p>

---

<p align="center">
  <img alt="Success" src="https://img.shields.io/badge/2D%20success-99.2%25-16a34a?style=for-the-badge">
  <img alt="Latency" src="https://img.shields.io/badge/cycle%20time-32%20ms-2563eb?style=for-the-badge">
  <img alt="Shared space" src="https://img.shields.io/badge/shared--space%20collision-0.9%25-dc2626?style=for-the-badge">
  <img alt="ROSOrin" src="https://img.shields.io/badge/ROSOrin-191%2F200%20success-7c3aed?style=for-the-badge">
</p>

## Overview

Shared-space mobility is no longer a clean lane-following problem. Delivery robots, pedestrians, service carts, wheelchairs, micromobility users, and low-speed autonomous platforms increasingly negotiate the same sidewalks, corridors, plazas, campus spaces, and logistics zones.

Most navigation stacks still use a separated processing flow:

```text
observe the scene -> predict participant futures -> plan ego motion -> execute
```

That pipeline is practical, but it freezes participant futures before the robot plan is selected. The robot can react to predicted participants, yet the selected robot motion cannot reshape the predicted multi-agent evolution. JPPD removes this boundary by sampling the ego future and participant futures together from one coupled distribution.

## Visual Story

| Problem Setting | Joint Architecture |
| --- | --- |
| <img src="assets/jppd_problem_setting.png" alt="Separated prediction-planning versus JPPD" width="100%"> | <img src="assets/jppd_architecture.png" alt="JPPD system architecture" width="100%"> |

| Navigation Effects | ROSOrin Deployment |
| --- | --- |
| <img src="assets/jppd_navigation_results.png" alt="JPPD navigation results" width="100%"> | <img src="assets/rosorin_deployment.png" alt="ROSOrin deployment visualization" width="100%"> |

## What JPPD Changes

| <img alt="Old" src="https://img.shields.io/badge/Separated%20Stack-BLADE--style-991b1b?style=for-the-badge"> | <img alt="New" src="https://img.shields.io/badge/Joint%20Stack-JPPD-166534?style=for-the-badge"> |
| --- | --- |
| Predict participant futures first, then freeze them. | Sample ego and participant futures as one joint state. |
| The planner reacts to predictions, but cannot influence them. | Cross-trajectory attention lets interaction hypotheses co-evolve. |
| Safety is often added as a post-hoc repulsive field. | DSPG injects a differentiable safety gradient during sampling. |
| Sequential prediction and planning diffusion increases latency. | Conditional flow matching uses one compact sampler for fast replanning. |

The central object is a joint future tensor:

```text
Y = [ego trajectory, participant trajectory 1, ..., participant trajectory N]
```

JPPD learns and samples `p(Y | context)` directly, then executes the first ego segment in a receding-horizon loop.

## Why BLADE

In this repository and the associated manuscript discussion, **BLADE is used as a broad shorthand for a decoupled prediction-then-planning composition**. It refers to the architectural pattern where an upper component predicts obstacle or participant futures and a lower component plans the ego trajectory against those fixed futures.

We call this separated flow **BLADE** because it comes from our earlier bi-layer obstacle-avoidance design: one layer generates or predicts the surrounding dynamic scene, and another layer plans the ego motion using that already-generated scene. In other words, BLADE names a layered, decoupled decision structure rather than a single fixed codebase.

```text
BLADE-style composition: prediction module + planning module, connected sequentially.
JPPD: one joint generative sampler for prediction and planning.
```

The key contrast is conceptual. BLADE-style systems keep prediction and planning as two coupled-but-separate pieces. JPPD treats low-speed dynamic obstacle avoidance as a single joint generative decision problem.

## Method

| Component | Visual Tag | Role |
| --- | --- | --- |
| Joint Prediction-Planning Diffusion | <img alt="Joint" src="https://img.shields.io/badge/joint%20state-Y-2563eb"> | Samples ego and participant futures from one conditional distribution. |
| CDiT | <img alt="CDiT" src="https://img.shields.io/badge/Transformer-CDiT-7c3aed"> | Uses agent-time tokens and cross-trajectory attention. |
| DSPG | <img alt="DSPG" src="https://img.shields.io/badge/Safety-DSPG-dc2626"> | Guides the sampler with a learned time-varying occupancy potential. |
| Flow Matching | <img alt="Flow Matching" src="https://img.shields.io/badge/Sampler-flow%20matching-0f766e"> | Reduces inference steps for embedded replanning. |
| Risk-aware Selection | <img alt="Selector" src="https://img.shields.io/badge/Control-receding%20horizon-f59e0b"> | Scores complete joint futures and executes the first safe ego segment. |

Safety guidance enters the vector field itself:

```text
guided_vector_field = learned_joint_vector_field - safety_weight * grad(safety_potential)
```

This makes safety a sampling-time bias over the joint prediction-planning posterior, not a post-processing patch after a trajectory has already been generated.

## Results

The evaluation emphasizes operational shared-space metrics: near misses, blockage time, induced participant deviation, hard-braking events, collision rate, and embedded latency.

### Scenario-Grounded Shared-Space Simulation

| Method | Success | Collision | Near-miss / 100 | Blockage | Hard braking / 100 |
| --- | ---: | ---: | ---: | ---: | ---: |
| BLADE-style separated stack | 95.9% | 2.4% | 14.8 | 2.31 s/episode | 6.8 |
| JPPD-Uni | 96.7% | 1.9% | 12.6 | 2.04 s/episode | 5.3 |
| JPPD-FixedRep | 97.3% | 1.5% | 10.1 | 1.86 s/episode | 4.7 |
| **JPPD** | **98.4%** | **0.9%** | **7.2** | **1.24 s/episode** | **2.6** |

### Aggregate 2D Navigation

| Method | Success | Planning-cycle time | Path efficiency |
| --- | ---: | ---: | ---: |
| BLADE-style separated stack | 98.7% | 47 ms | 0.96 |
| JPPD-Uni | 97.8% | 31 ms | 0.96 |
| JPPD without DSPG | 96.6% | 28 ms | 0.95 |
| **JPPD** | **99.2%** | **32 ms** | **0.97** |

The measured success-rate gain should be read as a tail-risk reduction. At high baseline success rates, the remaining failures are difficult, safety-critical cases such as near-simultaneous crossings, short-horizon reversals, or crowded local minima.

## ROSOrin Deployment

JPPD was validated in simulation, naturalistic pedestrian replay, Isaac Sim, and physical ROSOrin trials. The deployment interface exposes joint ego-participant futures, DSPG risk regions, selected ego trajectory, and the first receding-horizon control segment.

| Scenario | Obstacles | Success | Collision | Mean Time | Control Rate |
| --- | ---: | ---: | ---: | ---: | ---: |
| Open | 2S + 0D | 50/50 | 0 | 9.1 s | 13.6 Hz |
| Corridor | 3S + 0D | 49/50 | 0 | 11.7 s | 13.1 Hz |
| Cluttered | 2S + 2D | 47/50 | 1 | 13.9 s | 12.4 Hz |
| Dynamic | 1S + 4D | 45/50 | 2 | 16.1 s | 11.9 Hz |
| **Overall** | **mixed** | **191/200** | **3** | **12.7 s** | **12.8 Hz** |

The observed failures were dominated by LiDAR track merging, sudden participant reversals, and localization drift, which are explicitly outside the claim of formal safety certification.

## Repository Status

This repository is currently maintained as the public project page for the JPPD manuscript.

| Directory | Purpose |
| --- | --- |
| `assets/` | Figures used by this README. |
| `docs/` | Method notes, release plan, and citation material. |
| `src/` | Placeholder for the future JPPD implementation. |
| `experiments/` | Placeholder for benchmark and reproduction scripts. |
| `checkpoints/` | Placeholder for trained CDiT and DSPG weights. |
| `data/` | Placeholder for dataset preparation scripts and metadata. |
| `logs/` | Placeholder for simulation, Isaac Sim, and ROSOrin trial logs. |

The full code, trained models, simulation configurations, and real-robot trial logs are planned for public release after the paper review and release process are complete. The placeholders are intentional: they keep the public repository stable while avoiding the accidental publication of incomplete or non-reproducible research artifacts.

## Planned Release

<p>
  <img alt="CDiT" src="https://img.shields.io/badge/CDiT-joint%20sampler-2563eb?style=flat-square">
  <img alt="DSPG" src="https://img.shields.io/badge/DSPG-safety%20potential-dc2626?style=flat-square">
  <img alt="Replay" src="https://img.shields.io/badge/ETH%2FUCY-replay-7c3aed?style=flat-square">
  <img alt="Isaac" src="https://img.shields.io/badge/Isaac%20Sim-validation-76B900?style=flat-square">
  <img alt="ROSOrin" src="https://img.shields.io/badge/ROSOrin-deployment-22314E?style=flat-square">
</p>

- CDiT joint sampler implementation.
- DSPG occupancy-potential model and guidance hooks.
- Synthetic shared-space scenario generator.
- ETH/UCY replay conversion scripts.
- Isaac Sim ROSOrin validation configuration.
- ROSOrin deployment interface.
- Evaluation scripts for success, collision rate, latency, ADE/FDE, near misses, blockage time, induced deviation, and hard-braking events.
- Trained weights and trial logs where redistribution is permitted.

## Citation

Article pending publication.

## Contact

Jiahao Wu<br>
Department of Engineering, Mechanical Engineering<br>
The University of Hong Kong<br>
Email: 13620926353@163.com
