<p align="center">
  <img src="assets/jppd_architecture.png" alt="JPPD architecture" width="96%">
</p>

<h1 align="center">JPPD: Joint Prediction-Planning Diffusion</h1>

<p align="center">
  <b>Differentiable safety-guided joint trajectory generation for dynamic obstacle avoidance in shared-space intelligent transportation systems.</b>
</p>

<p align="center">
  <a href="#overview"><img alt="Project" src="https://img.shields.io/badge/project-JPPD-111827"></a>
  <a href="#method"><img alt="Method" src="https://img.shields.io/badge/method-joint%20diffusion-2563eb"></a>
  <a href="#results"><img alt="Results" src="https://img.shields.io/badge/focus-tail%20safety-16a34a"></a>
  <a href="#repository-status"><img alt="Status" src="https://img.shields.io/badge/code-placeholder%20release-f59e0b"></a>
</p>

---

## Overview

Shared-space mobility is no longer a clean lane-following problem. Delivery robots, pedestrians, service carts, wheelchairs, micromobility users, and low-speed autonomous platforms increasingly negotiate the same sidewalks, corridors, plazas, and campus logistics zones.

Most navigation stacks still split the problem into two steps:

1. predict how nearby participants will move;
2. plan the ego robot trajectory against those predicted futures.

That separation creates a one-way information flow. The robot plan can react to predicted participants, but the selected robot plan cannot reshape the predicted multi-agent evolution. JPPD removes that boundary by sampling the ego future and participant futures together from one coupled distribution.

<p align="center">
  <img src="assets/jppd_problem_setting.png" alt="Separated prediction-planning versus JPPD" width="96%">
</p>

## What JPPD Changes

| Old separated stack | JPPD joint stack |
| --- | --- |
| Predict obstacle futures first, then freeze them. | Sample ego and participant futures as one joint state. |
| Planner reacts to predictions, but cannot influence them. | Cross-trajectory attention lets ego and participant hypotheses co-evolve. |
| Safety is often added as a post-hoc repulsive field. | DSPG injects a differentiable safety gradient during sampling. |
| Sequential prediction and planning diffusion increases latency. | Conditional flow matching uses one compact sampler for fast replanning. |

The central object is a joint future tensor:

```text
Y = [ego trajectory, participant trajectory 1, ..., participant trajectory N]
```

JPPD learns and samples `p(Y | context)` directly, then executes the first ego segment in a receding-horizon loop.

## Important Note On "BLADE"

In this repository and the associated manuscript discussion, **BLADE is used as a broad shorthand for a decoupled prediction-then-planning composition**. It refers to the architectural pattern where an upper component predicts obstacle or participant futures and a lower component plans the ego trajectory against those fixed futures.

We call this separated flow **BLADE** because it comes from our earlier bi-layer obstacle-avoidance design: one layer generates or predicts the surrounding dynamic scene, and another layer plans the ego motion using that already-generated scene. In other words, BLADE names a layered, decoupled decision structure rather than a single fixed codebase.

It should not be read as a claim that every BLADE mention corresponds to one unique, monolithic, mandatory implementation. The key contrast is conceptual:

```text
BLADE-style composition: prediction module + planning module, connected sequentially.
JPPD: one joint generative sampler for prediction and planning.
```

This clarification matters because the paper's novelty is not a cosmetic renaming of an earlier stack. The contribution is the move from a decoupled composition to a bidirectionally coupled joint prediction-planning diffusion process.

## Method

JPPD contains four pieces:

| Component | Role |
| --- | --- |
| Joint Prediction-Planning Diffusion | Samples ego and participant futures from one conditional distribution. |
| CDiT | A causal diffusion Transformer with agent-time tokens and cross-trajectory attention. |
| DSPG | Differentiable Safety Potential Guidance, a learned time-varying occupancy potential whose gradient guides the sampler. |
| Risk-aware selector | Scores complete joint futures and executes the first safe ego segment before replanning. |

The sampler uses conditional flow matching so that embedded deployment can run with fewer inference steps than a long DDPM chain. Safety guidance enters the vector field itself:

```text
guided_vector_field = learned_joint_vector_field - safety_weight * grad(safety_potential)
```

This makes safety a sampling-time bias over the joint prediction-planning posterior, not a post-processing patch after a trajectory has already been generated.

## Results

The evaluation emphasizes operational shared-space metrics: near misses, blockage time, induced participant deviation, hard-braking events, collision rate, and embedded latency.

<p align="center">
  <img src="assets/jppd_navigation_results.png" alt="JPPD navigation results" width="96%">
</p>

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

<p align="center">
  <img src="assets/rosorin_deployment.png" alt="ROSOrin deployment visualization" width="96%">
</p>

Physical ROSOrin trials reported 191/200 successful runs across open, corridor, cluttered, and dynamic scenarios. The observed failures were dominated by LiDAR track merging, sudden participant reversals, and localization drift, which are explicitly outside the claim of formal safety certification.

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

- CDiT joint sampler implementation.
- DSPG occupancy-potential model and guidance hooks.
- Synthetic shared-space scenario generator.
- ETH/UCY replay conversion scripts.
- Isaac Sim ROSOrin validation configuration.
- ROSOrin deployment interface.
- Evaluation scripts for success, collision rate, latency, ADE/FDE, near misses, blockage time, induced deviation, and hard-braking events.
- Trained weights and trial logs where redistribution is permitted.

## Citation

If you discuss or build on this work, please cite the manuscript:

```bibtex
@article{wu2026jppd,
  title  = {JPPD: Joint Prediction--Planning Diffusion with Differentiable Safety Guidance for Dynamic Obstacle Avoidance in Intelligent Transportation Systems},
  author = {Wu, Jiahao and Yu, Shengwen},
  journal = {Manuscript under review},
  year   = {2026}
}
```

## Contact

Jiahao Wu<br>
Department of Engineering, Mechanical Engineering<br>
The University of Hong Kong<br>
Email: 13620926353@163.com
