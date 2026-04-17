---
title: "Sim2Real Practitioner's Guide - Part 2"
date: 2026-04-18 12:00:00 +00:00
author: "Aditya Dutt"
series: "Sim2Real Practitioner's Guide"
part: 2
summary: "Distill the teacher into a visual student and deploy safely on hardware."
subtitle: "Part 2 covers student distillation, deployment, and runtime safety checks."
reading_time: "11 min read"
tags:
  - robotics
  - computer-vision
  - deployment
toc:
  - id: distillation
    title: Distillation target
  - id: deployment
    title: Deployment checklist
---

<div id="distillation"></div>
## Distillation target

Train a student policy from RGB + proprioception and regress to teacher actions:

$$
\\min_\\phi \\; \\mathbb{E}_{(x_t, a_t^*)}
\\left[\\lVert \\pi_\\phi(x_t) - a_t^* \\rVert_2^2\\right]
$$

For long-horizon manipulation, add recurrence to improve temporal consistency.

```python
with torch.no_grad():
    teacher_action = teacher(privileged_obs)

student_action = student(rgb_obs, proprio_obs, hidden_state)
loss = F.mse_loss(student_action, teacher_action)
loss.backward()
optimizer.step()
```

<div id="deployment"></div>
## Deployment checklist

- Validate camera calibration and latency before first rollout.
- Enforce action clipping and safe joint limits.
- Log failures and replay them in simulation for targeted retraining.

<div class="blog-callout">
  <p><strong>Safety first.</strong> Sim success does not imply hardware safety; enforce runtime guards independently of policy confidence.</p>
</div>

Native video files are also supported:

<figure class="blog-figure">
  <video controls preload="metadata">
    <source src="/videos/sample-demo.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
  <figcaption><strong>Figure 2.</strong> Put demo clips in `/videos/` and reference them with a relative URL.</figcaption>
</figure>
