---
title: "Part 2: Teacher Training"
date: 2026-04-18 12:00:00 +00:00
author: "Aditya Dutt"
series: "Sim2Real Practitioner's Guide"
part: 2
chapter_title: "Teacher Training"
draft: true
summary: "Train a robust privileged teacher policy with the right algorithm, action space, and randomization stack."
subtitle: "This chapter covers teacher objective design, early RL algorithm choice, and practical training heuristics."
reading_time: "14 min read"
tags:
  - robotics
  - reinforcement-learning
  - sim2real
toc:
  - id: teacher-setup
    title: Teacher setup and objective
  - id: algorithm-choice
    title: RL algorithm choice (early decision)
  - id: action-space
    title: Action space design
  - id: training-tricks
    title: Resets, exploration, and randomization
  - id: isaaclab-placeholder
    title: IsaacLab implementation placeholder
---

<div id="teacher-setup"></div>
## Teacher setup and objective

Teacher training starts with privileged observations and an asymmetric actor-critic setup.
The actor receives noised observations to reduce over-reliance on perfect simulation state,
while the critic uses richer state to stabilize value estimation.

<div class="blog-equation">
$$
\pi_{\theta}(a \mid \tilde{o}), \quad
\tilde{o} = o + \epsilon, \quad
\epsilon \sim \mathcal{N}(0, \sigma^2 I)
$$
<span class="blog-equation-num">(1)</span>
</div>

This setup prepares the policy for the information gap that appears later during visual distillation.

<div class="blog-callout note">
  <p><strong>Why this matters.</strong> A teacher that is overly tuned to perfect state can become hard to distill and brittle under deployment noise.</p>
</div>

<div id="algorithm-choice"></div>
## RL algorithm choice (early decision)

Choose the RL algorithm before tuning other details. This decision controls data efficiency,
stability, and the kind of policy behavior your teacher converges to.

### PPO baseline

PPO remains a reliable default for many manipulation tasks, especially when compute is abundant and
you need predictable optimization behavior.

### SAC and stability trade-offs

SAC can be sample-efficient but may show instability in contact-rich settings if entropy and critic
tuning are not carefully managed.

### FlashSAC and improved off-policy training

FlashSAC-style improvements can make off-policy approaches more practical for sim2real,
particularly when you want to reuse replay data efficiently.

<div class="blog-callout success">
  <p><strong>Recommendation.</strong> Start with PPO for a dependable teacher baseline, then test SAC/FlashSAC only after you have stable reward design, reset logic, and action-space choices.</p>
</div>

<div id="action-space"></div>
## Action space design

Action space is a transfer-critical design choice. For many fixed-base manipulators:

- use delta joint targets for robust and simple control,
- use end-effector velocity/twist commands for precision-sensitive tasks,
- keep gripper control discrete unless the hardware benefits from continuous force control.

Action smoothing and rate limits are worth adding in simulation during teacher training because
they reduce distribution shift in later deployment.

<div id="training-tricks"></div>
## Resets, exploration, and randomization

After algorithm and action-space selection, focus on training robustness:

- **Diverse resets (OmniReset-style):** improve coverage of recovery states and reduce brittle behavior.
- **Exploration strategy:** keep enough stochasticity for long-horizon tasks with sparse success.
- **Physics randomization:** randomize friction, mass, damping, latency, and sensor noise within plausible ranges.
- **Compute scaling:** increase parallel environments to improve training throughput and sample diversity.
- **PPO details:** tune rollout horizon, minibatches, clip range, value loss weighting, and entropy schedule.

<div class="blog-callout">
  <p><strong>Practical sequence.</strong> Stabilize reward and reset coverage first, then randomization ranges, and only then aggressive hyperparameter sweeps.</p>
</div>

<div id="isaaclab-placeholder"></div>
## IsaacLab implementation placeholder

```python
# TODO(aditya): Teacher training scaffold in IsaacLab
# Replace these placeholders with actual config classes and runner APIs.

def build_teacher_env():
    # TODO: create privileged observation manager
    # TODO: add domain randomization manager
    # TODO: configure reset events (OmniReset style)
    pass


def train_teacher():
    algo = "PPO"  # TODO: compare against SAC / FlashSAC setup
    env = build_teacher_env()
    # TODO: instantiate policy + critic networks
    # TODO: run rollout collection and updates
    # TODO: checkpoint best teacher
    return {"checkpoint": "TODO/path/to/teacher.ckpt", "algo": algo}
```

<figure class="blog-figure">
  <video controls preload="metadata">
    <source src="/videos/teacher-training-placeholder.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
  <figcaption><strong>Teacher training demo placeholder.</strong> Replace with rollout and convergence clips comparing algorithm choices.</figcaption>
</figure>
