---
title: "Part 4: SysID and Sim2Real Deployment"
date: 2026-04-17 12:15:00 +00:00
author: "Aditya Dutt"
series: "Sim2Real Practitioner's Guide"
part: 4
chapter_title: "SysID and Sim2Real Deployment"
draft: false
summary: "Calibrate simulation to hardware and deploy policies with stable execution and validation loops."
subtitle: "This chapter covers system identification, runtime controls, and deployment checklists for reliable transfer."
reading_time: "12 min read"
tags:
  - robotics
  - deployment
  - sim2real
toc:
  - id: deployment-runtime
    title: Deployment runtime controls
  - id: sysid-stages
    title: System identification stages
  - id: validation
    title: Validation and iteration loop
  - id: isaaclab-placeholder
    title: IsaacLab implementation placeholder
---

<div id="deployment-runtime"></div>
## Deployment runtime controls

Even with a strong student policy, deployment quality is often limited by runtime interfaces.
Use conservative execution defaults:

- smooth policy outputs before sending commands to hardware,
- enforce action clipping and rate limits,
- use explicit gripper command logic and state guards,
- track latency and dropped-frame behavior in perception pipelines.

<div class="blog-callout">
  <p><strong>Safety principle.</strong> Treat policy confidence as advisory. Hard safety checks should be independent and always active during deployment.</p>
</div>

<div id="sysid-stages"></div>
## System identification stages

A practical SysID flow can be staged in three levels:

<figure class="blog-figure">
  <svg width="100%" viewBox="0 0 680 440" xmlns="http://www.w3.org/2000/svg">
    <defs><marker id="sysid-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path d="M2 1L8 5L2 9" fill="none" stroke="#888" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></marker></defs>
    <rect x="40" y="10" width="600" height="100" rx="10" fill="#e6f1fb" stroke="#85b7eb" stroke-width="0.5"/>
    <text x="60" y="34" font-family="DM Sans,sans-serif" font-size="13" font-weight="600" fill="#0c447c">Stage 1 · URDF and kinematic calibration</text>
    <rect x="60" y="46" width="150" height="48" rx="5" fill="#b5d4f4" stroke="#85b7eb" stroke-width="0.5"/>
    <text x="135" y="67" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#0c447c">Joint offsets</text>
    <text x="135" y="83" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="10" fill="#185fa5">Encoder zeros</text>
    <rect x="225" y="46" width="150" height="48" rx="5" fill="#b5d4f4" stroke="#85b7eb" stroke-width="0.5"/>
    <text x="300" y="67" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#0c447c">Link geometry</text>
    <text x="300" y="83" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="10" fill="#185fa5">Lengths, mass, CoM</text>
    <rect x="390" y="46" width="230" height="48" rx="5" fill="#b5d4f4" stroke="#85b7eb" stroke-width="0.5"/>
    <text x="505" y="67" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#0c447c">Camera extrinsics</text>
    <text x="505" y="83" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="10" fill="#185fa5">ChArUco board + PnP</text>
    <line x1="340" y1="110" x2="340" y2="130" stroke="#aaa" stroke-width="1" marker-end="url(#sysid-arrow)"/>
    <rect x="40" y="130" width="600" height="100" rx="10" fill="#d2f0e3" stroke="#5dcaa5" stroke-width="0.5"/>
    <text x="60" y="154" font-family="DM Sans,sans-serif" font-size="13" font-weight="600" fill="#085041">Stage 2 · Actuator dynamics (PACE)</text>
    <rect x="60" y="166" width="110" height="48" rx="5" fill="#9fe1cb" stroke="#5dcaa5" stroke-width="0.5"/>
    <text x="115" y="187" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#085041">Armature</text>
    <text x="115" y="203" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="10" fill="#0f6e56">Per-joint</text>
    <rect x="185" y="166" width="120" height="48" rx="5" fill="#9fe1cb" stroke="#5dcaa5" stroke-width="0.5"/>
    <text x="245" y="187" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#085041">Damping</text>
    <text x="245" y="203" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="10" fill="#0f6e56">Viscous + Coulomb</text>
    <rect x="320" y="166" width="110" height="48" rx="5" fill="#9fe1cb" stroke="#5dcaa5" stroke-width="0.5"/>
    <text x="375" y="187" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#085041">Latency</text>
    <text x="375" y="203" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="10" fill="#0f6e56">~7.5 ms global</text>
    <rect x="445" y="166" width="175" height="48" rx="5" fill="#9fe1cb" stroke="#5dcaa5" stroke-width="0.5"/>
    <text x="533" y="187" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#085041">CMA-ES optimizer</text>
    <text x="533" y="203" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="10" fill="#0f6e56">~20s encoder data</text>
    <line x1="340" y1="230" x2="340" y2="250" stroke="#aaa" stroke-width="1" marker-end="url(#sysid-arrow)"/>
    <rect x="40" y="250" width="600" height="100" rx="10" fill="#faece7" stroke="#f0997b" stroke-width="0.5"/>
    <text x="60" y="274" font-family="DM Sans,sans-serif" font-size="13" font-weight="600" fill="#712b13">Stage 3 · Visual and camera alignment</text>
    <rect x="60" y="286" width="155" height="48" rx="5" fill="#f5c4b3" stroke="#f0997b" stroke-width="0.5"/>
    <text x="138" y="307" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#712b13">Intrinsics</text>
    <text x="138" y="323" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="10" fill="#993c1d">f_x, f_y, c_x, c_y, dist</text>
    <rect x="230" y="286" width="155" height="48" rx="5" fill="#f5c4b3" stroke="#f0997b" stroke-width="0.5"/>
    <text x="308" y="307" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#712b13">Color calibration</text>
    <text x="308" y="323" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="10" fill="#993c1d">WB, gamma, saturation</text>
    <rect x="400" y="286" width="220" height="48" rx="5" fill="#f5c4b3" stroke="#f0997b" stroke-width="0.5"/>
    <text x="510" y="307" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#712b13">Render fidelity</text>
    <text x="510" y="323" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="10" fill="#993c1d">PBR, tiled ray-tracing</text>
    <line x1="340" y1="350" x2="340" y2="374" stroke="#aaa" stroke-width="1" marker-end="url(#sysid-arrow)"/>
    <rect x="140" y="374" width="400" height="32" rx="6" fill="#f3f1ea" stroke="#c8c5bb" stroke-width="0.5"/>
    <text x="340" y="394" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="11" font-weight="500" fill="#6b6b66">Validate: open-loop replay error &lt; 5 mm RMS</text>
  </svg>
  <figcaption><strong>Three-stage SysID pipeline.</strong> Calibration progresses from kinematics to actuator dynamics to visual alignment.</figcaption>
</figure>

### Stage 1: URDF and kinematic calibration

Align link dimensions, inertial parameters, and coordinate frames so simulated geometry matches hardware.

### Stage 2: Actuator and control dynamics

Fit actuator behavior (latency, damping, gains, saturation) using replay traces from hardware trajectories.

### Stage 3: Visual alignment

Match camera intrinsics/extrinsics, image timing, and observation preprocessing between simulation and real setup.

<figure class="blog-figure">
  <img src="/images/visual_fov_alignment.png" alt="Comparison of real camera FOV and simulation FOV before and after alignment">
  <figcaption><strong>FOV alignment sanity check.</strong> Matching simulation and real camera framing is a high-leverage visual calibration step before deployment validation.</figcaption>
</figure>

<div class="blog-callout success">
  <p><strong>Recommendation.</strong> Iterate SysID in short loops with task-relevant trajectories rather than attempting one global calibration pass.</p>
</div>

<div id="validation"></div>
## Validation and iteration loop

Use a staged validation protocol before full deployment:

1. dry-run kinematic checks with low-speed commands,
2. evaluate short-horizon task slices,
3. log failures and replay in simulation,
4. adjust randomization and calibration ranges,
5. redeploy and compare metrics.

Keep a compact dashboard of:

- success rate by task phase,
- recovery rate after perturbation,
- command smoothness and saturation frequency,
- perception timing and dropped frame counts.

<div id="isaaclab-placeholder"></div>
## IsaacLab implementation placeholder

```python
# TODO(aditya): SysID + deployment scaffold in IsaacLab

def calibrate_urdf_and_frames():
    # TODO: load hardware measurements
    # TODO: update robot model parameters
    return {"urdf_status": "TODO"}


def fit_actuator_dynamics(log_paths):
    # TODO: identify actuator delay/gain/damping from hardware traces
    return {"actuator_model": "TODO/path/to/model.json"}


def align_visual_pipeline():
    # TODO: camera intrinsics/extrinsics and preprocessing parity checks
    return {"camera_alignment": "TODO"}


def deploy_student_policy(student_ckpt):
    # TODO: runtime action smoothing + clipping + safety guards
    # TODO: telemetry logging for sim replay
    return {"deployment_run_id": "TODO"}
```

<figure class="blog-figure">
  <video controls preload="metadata">
    <source src="/videos/sysid-deployment-placeholder.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
  <figcaption><strong>Deployment demo placeholder.</strong> Replace with calibration clips, first hardware rollouts, and failure-recovery examples.</figcaption>
</figure>
