---
title: "Part 1: Overview"
date: 2026-04-17 12:00:00 +00:00
author: "Aditya Dutt"
series: "Sim2Real Practitioner's Guide"
part: 1
chapter_title: "Overview"
draft: true
summary: "A multi-page roadmap for deploying robust sim2real manipulation policies end to end."
subtitle: "This chapter frames the full pipeline and links to each implementation-focused sub-page."
reading_time: "8 min read"
tags:
  - robotics
  - reinforcement-learning
  - sim2real
toc:
  - id: why-this-guide
    title: Why this guide
  - id: pipeline-map
    title: End-to-end pipeline map
  - id: chapter-roadmap
    title: Chapter roadmap
  - id: implementation-assets
    title: Code and demo assets
---

<div id="why-this-guide"></div>
## Why this guide

This guide turns a dense sim2real draft into a practical, implementation-first series for robotics policy deployment.
The structure follows the path most teams actually execute:

- train a privileged teacher in simulation,
- distill and fine-tune a deployable student,
- calibrate simulation with system identification,
- and close the sim-to-real gap in hardware rollout.

Each chapter includes implementation scaffolds, TODO placeholders for IsaacLab code, and media placeholders for videos of each stage.

<div class="blog-callout note">
  <p><strong>Scope.</strong> This series focuses on policy training and deployment for manipulation tasks using the teacher-student paradigm. We prioritize practical deployment decisions over exhaustive literature coverage.</p>
</div>

<div id="pipeline-map"></div>
## End-to-end pipeline map

```text
Phase A: Teacher training in simulation
  -> Privileged observations
  -> RL algorithm choice
  -> Action space + resets + randomization

Phase B: Student distillation and finetuning
  -> RGB + proprioception student
  -> Distillation objective
  -> Optional RL finetuning

Phase C: SysID and sim2real deployment
  -> URDF and actuator calibration
  -> Visual alignment
  -> Runtime action and safety checks
```

<div class="blog-callout note">
  <p><strong>How to use this guide.</strong> Read chapter-by-chapter if you are building a new pipeline, or jump directly to the chapter that matches your current bottleneck.</p>
</div>

<figure class="blog-figure">
  <svg width="100%" viewBox="0 0 680 440" xmlns="http://www.w3.org/2000/svg">
    <defs><marker id="pipeline-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M2 1L8 5L2 9" fill="none" stroke="#888" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></marker></defs>
    <text x="340" y="22" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="14" font-weight="600" fill="#1c1c1a">End-to-end sim2real pipeline</text>
    <rect x="55" y="38" width="570" height="36" rx="6" fill="#f3f1ea" stroke="#c8c5bb" stroke-width="0.5"/>
    <text x="340" y="60" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="12" font-weight="500" fill="#6b6b66">Phase 0 · System identification — URDF, controller, camera [Part 4]</text>
    <line x1="340" y1="74" x2="340" y2="92" stroke="#aaa" stroke-width="1" marker-end="url(#pipeline-arrow)"/>
    <rect x="55" y="92" width="570" height="96" rx="10" fill="#eeedfe" stroke="#afa9ec" stroke-width="0.5"/>
    <text x="340" y="114" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="13" font-weight="600" fill="#3c3489">Phase 1 · Privileged RL teacher [Part 2]</text>
    <rect x="75" y="124" width="145" height="48" rx="5" fill="#dddaf8" stroke="#afa9ec" stroke-width="0.5"/>
    <text x="148" y="145" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#3c3489">Asymmetric AC</text>
    <text x="148" y="161" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="10" fill="#534ab7">Noisy actor + GT critic</text>
    <rect x="235" y="124" width="145" height="48" rx="5" fill="#d2f0e3" stroke="#5dcaa5" stroke-width="0.5"/>
    <text x="308" y="145" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#085041">OmniReset</text>
    <text x="308" y="161" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="10" fill="#0f6e56">Event-based resets</text>
    <rect x="395" y="124" width="145" height="48" rx="5" fill="#faeeda" stroke="#fac775" stroke-width="0.5"/>
    <text x="468" y="145" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#633806">Physics DR</text>
    <text x="468" y="161" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="10" fill="#854f0b">ADR / manual ranges</text>
    <line x1="340" y1="188" x2="340" y2="210" stroke="#aaa" stroke-width="1" marker-end="url(#pipeline-arrow)"/>
    <text x="355" y="204" font-family="DM Sans,sans-serif" font-size="10" fill="#888">DAgger + BC</text>
    <rect x="55" y="210" width="570" height="96" rx="10" fill="#faece7" stroke="#f0997b" stroke-width="0.5"/>
    <text x="340" y="232" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="13" font-weight="600" fill="#712b13">Phase 2 · Student distillation [Part 3]</text>
    <rect x="75" y="242" width="115" height="48" rx="5" fill="#f5c4b3" stroke="#f0997b" stroke-width="0.5"/>
    <text x="133" y="263" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#712b13">Encoder</text>
    <text x="133" y="279" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="10" fill="#993c1d">ResNet / DINOv2</text>
    <rect x="205" y="242" width="100" height="48" rx="5" fill="#f5c4b3" stroke="#f0997b" stroke-width="0.5"/>
    <text x="255" y="263" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#712b13">LSTM</text>
    <text x="255" y="279" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="10" fill="#993c1d">Temporal ctx</text>
    <rect x="320" y="242" width="100" height="48" rx="5" fill="#f5c4b3" stroke="#f0997b" stroke-width="0.5"/>
    <text x="370" y="263" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#712b13">Action head</text>
    <text x="370" y="279" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="10" fill="#993c1d">Gaussian MLP</text>
    <rect x="435" y="242" width="170" height="48" rx="5" fill="#faeeda" stroke="#fac775" stroke-width="0.5"/>
    <text x="520" y="263" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#633806">Visual DR</text>
    <text x="520" y="279" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="10" fill="#854f0b">Light · texture · camera</text>
    <line x1="340" y1="306" x2="340" y2="328" stroke="#aaa" stroke-width="1" marker-end="url(#pipeline-arrow)"/>
    <text x="355" y="322" font-family="DM Sans,sans-serif" font-size="10" fill="#888">Optional RL finetuning</text>
    <rect x="120" y="328" width="440" height="44" rx="8" fill="#e6f1fb" stroke="#85b7eb" stroke-width="0.5"/>
    <text x="340" y="348" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="13" font-weight="600" fill="#0c447c">Phase 3 · Zero-shot deployment [Part 4]</text>
    <text x="340" y="364" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="10" fill="#185fa5">RGB + proprioception → velocity actions · binary gripper</text>
  </svg>
  <figcaption><strong>Pipeline overview.</strong> Four phases from teacher training to deployment across this guide.</figcaption>
</figure>

<div id="chapter-roadmap"></div>
## Chapter roadmap

### Part 2: Teacher training

Build a robust privileged teacher policy with early algorithm selection, then refine with action-space design, reset strategy, exploration choices, and randomization.

- Link: [Part 2 - Teacher Training](/blogs/sim2real-practitioners-guide-part-2/)
- Focus: RL algorithm selection, training stability, and policy robustness in sim.

### Part 3: Student distillation and finetuning

Convert the teacher into an RGB student, choose model architecture, and optionally fine-tune for task-specific robustness.

- Link: [Part 3 - Student Distillation and Finetuning](/blogs/sim2real-practitioners-guide-part-3/)
- Focus: DAgger/BC setup, visual encoders, and post-distillation improvements.

### Part 4: SysID and sim2real deployment

Calibrate simulation to hardware and deploy safely with execution smoothing, validation, and runtime guardrails.

- Link: [Part 4 - SysID and Sim2Real Deployment](/blogs/sim2real-practitioners-guide-part-4/)
- Focus: system identification, transfer reliability, and deployment playbook.

<div class="blog-callout success">
  <p><strong>Recommended order.</strong> Start with Part 2 unless you already have a strong teacher. If a teacher is already available, jump to Part 3 and then Part 4.</p>
</div>

<div id="implementation-assets"></div>
## Code and demo assets

Below is the shared scaffold we will expand across Parts 2-4.

```python
# TODO(aditya): Shared experiment config scaffold for this series.
# Replace with your IsaacLab experiment registry and task definitions.

EXPERIMENTS = {
    "teacher_training": "TODO: IsaacLab teacher config path",
    "student_distillation": "TODO: IsaacLab student config path",
    "sysid_and_deployment": "TODO: IsaacLab deployment config path",
}
```

Demo index placeholder:

<figure class="blog-figure">
  <video controls preload="metadata">
    <source src="/videos/sim2real-overview-placeholder.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
  <figcaption><strong>Overview demo placeholder.</strong> Replace with a montage clip covering teacher training, distillation, and real-world rollout.</figcaption>
</figure>
