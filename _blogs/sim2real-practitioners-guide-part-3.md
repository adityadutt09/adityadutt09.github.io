---
title: "Part 3: Student Distillation and Finetuning"
date: 2026-04-17 12:10:00 +00:00
author: "Aditya Dutt"
series: "Sim2Real Practitioner's Guide"
part: 3
chapter_title: "Student Distillation and Finetuning"
draft: false
summary: "Distill privileged teacher behavior into an RGB student and improve robustness through fine-tuning."
subtitle: "This chapter covers DAgger/BC setup, visual encoder choices, and post-distillation reinforcement learning."
reading_time: "13 min read"
tags:
  - robotics
  - computer-vision
  - sim2real
toc:
  - id: distillation-objective
    title: Distillation objective
  - id: data-collection
    title: DAgger and behavior cloning
  - id: architecture
    title: Visual encoders and policy architecture
  - id: finetuning
    title: Post-distillation finetuning
  - id: isaaclab-placeholder
    title: IsaacLab implementation placeholder
---

<div id="distillation-objective"></div>
## Distillation objective

The student learns from RGB plus proprioception by imitating teacher actions on matched trajectories.
A typical objective minimizes action prediction error while preserving temporal consistency.

$$
\min_{\phi} \; \mathbb{E}_{(x_t, a_t^*)}
\left[\lVert \pi_{\phi}(x_t) - a_t^* \rVert_2^2 \right]
$$

Where:

- $x_t$ includes image observations and robot state,
- $a_t^*$ is the teacher target action,
- and optional sequence models improve long-horizon stability.

<div class="blog-callout note">
  <p><strong>Failure mode to watch.</strong> Offline imitation on narrow trajectories can create compounding errors. Use online data aggregation to reduce drift.</p>
</div>

<div id="data-collection"></div>
## DAgger and behavior cloning

A practical recipe:

1. collect teacher rollouts in diverse randomized simulation conditions,
2. train the student with behavior cloning,
3. run the student policy, query teacher labels on visited states (DAgger),
4. retrain with the expanded dataset.

This loop exposes the student to its own state distribution and improves recovery behavior.

<figure class="blog-figure">
  <img src="/images/doorman_pipeline.png" alt="DoorMan three-phase pipeline: teacher RL, student distillation, and student bootstrapping">
  <figcaption><strong>DoorMan three-phase training flow.</strong> Teacher PPO, DAgger distillation, and GRPO bootstrapping form a practical progression for vision-based students.</figcaption>
</figure>

<div id="architecture"></div>
## Visual encoders and policy architecture

For manipulation tasks, architecture choices usually matter as much as loss choice:

- **Encoder:** ResNet-style backbones are strong baselines; larger pretrained encoders can help when visual diversity is high.
- **Temporal model:** LSTM/GRU improves action stability under partial observability and occlusion.
- **Policy head:** keep output head simple and stable before adding auxiliary objectives.
- **Visual randomization:** vary textures, lighting, camera perturbations, and distractors to reduce overfitting.

<figure class="blog-figure">
  <img src="/images/visual_randomization.png" alt="Visual randomization examples including image, lighting, material, and camera perturbations">
  <figcaption><strong>Visual randomization stack.</strong> Image-level and renderer-level perturbations improve transfer by exposing the student to broad visual variation during training.</figcaption>
</figure>

<figure class="blog-figure">
  <img src="/images/doorman_visual_dr.png" alt="Procedurally generated door variations used in DoorMan training">
  <figcaption><strong>Task-specific procedural randomization.</strong> Door geometry and appearance variation helps the student generalize to previously unseen real-world instances.</figcaption>
</figure>

<div class="blog-callout success">
  <p><strong>Recommendation.</strong> Start with a small, reproducible student baseline and only scale encoder complexity after verifying data quality and label alignment.</p>
</div>

<div id="finetuning"></div>
## Post-distillation finetuning

After behavior cloning convergence, optional RL fine-tuning can improve task success and robustness.
Typical gains come from:

- better contact handling,
- improved precision near terminal states,
- and stronger recovery after perturbations.

When fine-tuning, keep safety and smoothness constraints aligned with deployment.

<div id="isaaclab-placeholder"></div>
## IsaacLab implementation placeholder

```python
# TODO(aditya): Student distillation + finetuning scaffold in IsaacLab

def collect_teacher_dataset(teacher_ckpt, env_cfg):
    # TODO: rollout teacher in randomized sim
    # TODO: save (rgb, proprio, teacher_action, done) tuples
    return "TODO/path/to/dataset"


def train_student_bc(dataset_path):
    # TODO: build encoder + temporal module + action head
    # TODO: behavior cloning training loop
    return "TODO/path/to/student_bc.ckpt"


def dagger_iteration(student_ckpt, teacher_ckpt):
    # TODO: run student policy
    # TODO: query teacher actions on visited states
    # TODO: append to dataset and retrain
    return "TODO/path/to/student_dagger.ckpt"


def finetune_student_with_rl(student_ckpt):
    # TODO: optional PPO/GRPO finetuning with safety constraints
    return "TODO/path/to/student_finetuned.ckpt"
```

<figure class="blog-figure">
  <video controls preload="metadata">
    <source src="/videos/student-distillation-placeholder.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
  <figcaption><strong>Student policy demo placeholder.</strong> Replace with side-by-side clips of teacher policy, distilled student, and post-finetuning behavior.</figcaption>
</figure>
