# JPPD Method Brief

JPPD treats dynamic obstacle avoidance as a joint generative decision problem.
Instead of predicting participant futures first and planning afterward, it samples
the ego robot trajectory and all participant trajectories as one coupled future
tensor.

## Core Objects

```text
context = ego state + goal + observed participant history + local occupancy + agent mask
Y       = [ego future, participant future 1, ..., participant future N]
```

The model learns `p(Y | context)` directly. At runtime, multiple joint futures are
sampled, scored by safety and goal-progress terms, and the first ego control
segment is executed before replanning.

## Components

| Component | Function |
| --- | --- |
| CDiT | Cross-trajectory causal diffusion Transformer. |
| DSPG | Differentiable Safety Potential Guidance. |
| Flow matching | Low-step continuous sampler for embedded replanning. |
| Risk-aware selection | Chooses the executable ego segment from complete joint futures. |

## BLADE Clarification

In this project, BLADE denotes a broad decoupled composition pattern:

```text
predict participants -> freeze predicted futures -> plan ego trajectory
```

The term is used to identify the separated prediction-planning baseline family.
It does not imply a single unique implementation. JPPD is contrasted against that
family by removing the frozen boundary between prediction and planning.
