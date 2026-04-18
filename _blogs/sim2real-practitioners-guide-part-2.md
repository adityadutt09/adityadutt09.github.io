---
title: "Part 2: Teacher Training"
date: 2026-04-17 12:05:00 +00:00
author: "Aditya Dutt"
series: "Sim2Real Practitioner's Guide"
part: 2
chapter_title: "Teacher Training"
draft: false
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

<figure class="blog-figure">
  <svg width="100%" viewBox="0 0 680 300" xmlns="http://www.w3.org/2000/svg">
    <defs><marker id="teacher-ac-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M2 1L8 5L2 9" fill="none" stroke="#888" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></marker></defs>
    <rect x="235" y="12" width="210" height="36" rx="6" fill="#f3f1ea" stroke="#c8c5bb" stroke-width="0.5"/>
    <text x="340" y="34" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="12" font-weight="500" fill="#444">Simulation environment</text>
    <line x1="270" y1="48" x2="160" y2="76" stroke="#aaa" stroke-width="1" marker-end="url(#teacher-ac-arrow)"/>
    <line x1="410" y1="48" x2="520" y2="76" stroke="#aaa" stroke-width="1" marker-end="url(#teacher-ac-arrow)"/>
    <text x="195" y="65" font-family="DM Sans,sans-serif" font-size="9" fill="#888">+ noise ε</text>
    <text x="445" y="65" font-family="DM Sans,sans-serif" font-size="9" fill="#888">full GT</text>
    <rect x="40" y="76" width="240" height="42" rx="6" fill="#d2f0e3" stroke="#5dcaa5" stroke-width="0.5"/>
    <text x="160" y="94" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#085041">Actor obs: proprio + noisy obj + ε</text>
    <text x="160" y="110" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="9" fill="#0f6e56">Transferred to student</text>
    <rect x="400" y="76" width="240" height="42" rx="6" fill="#faeeda" stroke="#fac775" stroke-width="0.5"/>
    <text x="520" y="94" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#633806">Critic obs: full privileged state</text>
    <text x="520" y="110" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="9" fill="#854f0b">Discarded after training</text>
    <line x1="160" y1="118" x2="160" y2="142" stroke="#aaa" stroke-width="1" marker-end="url(#teacher-ac-arrow)"/>
    <line x1="520" y1="118" x2="520" y2="142" stroke="#aaa" stroke-width="1" marker-end="url(#teacher-ac-arrow)"/>
    <rect x="60" y="142" width="200" height="42" rx="6" fill="#eeedfe" stroke="#afa9ec" stroke-width="0.5"/>
    <text x="160" y="160" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#3c3489">π(a | õ) — LSTM + MLP</text>
    <text x="160" y="176" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="9" fill="#534ab7">Dense skip connections</text>
    <rect x="420" y="142" width="200" height="42" rx="6" fill="#faeeda" stroke="#fac775" stroke-width="0.5"/>
    <text x="520" y="160" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#633806">V(o_priv) — MLP</text>
    <text x="520" y="176" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="9" fill="#854f0b">GAE advantage estimation</text>
    <line x1="160" y1="184" x2="160" y2="208" stroke="#aaa" stroke-width="1" marker-end="url(#teacher-ac-arrow)"/>
    <line x1="520" y1="184" x2="520" y2="208" stroke="#aaa" stroke-width="1" marker-end="url(#teacher-ac-arrow)"/>
    <rect x="90" y="208" width="140" height="32" rx="6" fill="#eeedfe" stroke="#afa9ec" stroke-width="0.5"/>
    <text x="160" y="228" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#3c3489">Actions a_t</text>
    <rect x="450" y="208" width="140" height="32" rx="6" fill="#faeeda" stroke="#fac775" stroke-width="0.5"/>
    <text x="520" y="228" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#633806">Advantage Â_t</text>
    <rect x="100" y="260" width="480" height="24" rx="4" fill="none" stroke="#ccc" stroke-width="0.5" stroke-dasharray="4 3"/>
    <text x="340" y="276" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="10" fill="#999">PPO objective: max E[min(r_t Â_t, clip(r_t, 1±ε) Â_t)]</text>
  </svg>
  <figcaption><strong>Asymmetric actor-critic.</strong> The actor is trained on noised observations while the critic uses privileged state.</figcaption>
</figure>

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

<figure class="blog-figure">
  <img src="/images/omnireset_pipeline.png" alt="OmniReset staged reset pipeline with collected reset states and zero-shot transfer">
  <figcaption><strong>OmniReset pipeline.</strong> Collecting diverse reset states broadens coverage and supports robust teacher RL before perception distillation.</figcaption>
</figure>

<figure class="blog-figure">
  <img src="/images/doorman_reset_scheme.png" alt="DoorMan staged reset buffer across task stages with cached snapshots">
  <figcaption><strong>DoorMan staged resets.</strong> Cached snapshots from later stages are reused to accelerate long-horizon exploration and reduce sparse-reward failure modes.</figcaption>
</figure>

<figure class="blog-figure">
  <img src="/images/reference_state_initialization.png" alt="Reference state initialization examples across simulated manipulation scenes">
  <figcaption><strong>Reference state initialization.</strong> Initializing from diverse demonstration states helps the teacher practice recovery and downstream student robustness.</figcaption>
</figure>

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
