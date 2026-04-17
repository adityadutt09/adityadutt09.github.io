---
title: "Sim2Real Practitioner's Guide - Part 1"
date: 2026-04-17 12:00:00 +00:00
author: "Aditya Dutt"
series: "Sim2Real Practitioner's Guide"
part: 1
summary: "How to structure teacher-policy training for robust sim-to-real transfer."
subtitle: "Part 1 focuses on teacher policy training and robust action-space design."
reading_time: "10 min read"
tags:
  - robotics
  - reinforcement-learning
  - sim2real
toc:
  - id: setup
    title: Setup and assumptions
  - id: teacher
    title: Teacher policy objective
  - id: implementation
    title: Training loop and implementation
---

<div id="setup"></div>
## Setup and assumptions

This starter post demonstrates the new blog stack: custom theme, chapter navigation, callouts, figures, inline math like $J(\\theta)$, display math, and code snippets.

<figure class="blog-figure">
  <img src="/images/404.jpg" alt="Placeholder pipeline figure">
  <figcaption><strong>Figure 1.</strong> Replace this placeholder with your own figures in `/images/`.</figcaption>
</figure>

<div class="blog-callout note">
  <p><strong>Tip.</strong> Keep each chapter scoped to one major idea and use `series` + `part` for continuity.</p>
</div>

<div id="teacher"></div>
## Teacher policy objective

Use a privileged teacher with asymmetric actor-critic:

<div class="blog-equation">
$$
\\pi_\\theta(a \\mid \\tilde{o}), \\quad
\\tilde{o} = o + \\epsilon, \\quad
\\epsilon \\sim \\mathcal{N}(0, \\sigma^2 I)
$$
<span class="blog-equation-num">(1)</span>
</div>

The critic sees privileged state while the actor is intentionally noised.

<div class="blog-callout success">
  <p><strong>Recommendation.</strong> Start with moderate observation noise to reduce the sim-to-real information gap before distillation.</p>
</div>

<div id="implementation"></div>
## Training loop and implementation

```python
for step in range(num_steps):
    noisy_obs = add_noise(obs, sigma=0.1)
    action = actor(noisy_obs)
    next_obs, reward, done, info = env.step(action)
    buffer.add(obs, action, reward, next_obs, done)

update_actor_critic(buffer)
```

Embed a YouTube explanation or demo:

<div class="blog-video-embed">
  <iframe
    src="https://www.youtube.com/embed/dQw4w9WgXcQ"
    title="Demo video"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    referrerpolicy="strict-origin-when-cross-origin"
    allowfullscreen>
  </iframe>
</div>
