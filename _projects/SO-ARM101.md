---
layout: page
title: SO-ARM101 Project
description: ACT-based pick-and-place experiment with a low-cost robot arm
img: assets/img/so-arm101/thumb.png
importance: 1
category: ongoing
related_publications: false
---

## Overview

This was a semester project conducted as part of ROBOIN, a robotics club at Yonsei University, from April to June 2026.

The goal was to build a complete real-world robot learning pipeline using the open-source **SO-ARM101** robot arm.
Starting from hardware setup and teleoperation, we collected real-world demonstrations, trained an imitation learning policy, and tested whether the robot could pick up predefined fruit-shaped objects.

The detailed hardware setup and teleoperation process is documented separately: [SO-ARM101 Hardware Setup & Teleoperation](/blog/2026/SOARM101-HW-Teleop/) (in Korean)

<div class="row">
  <div class="col-sm mt-3 mt-md-0 text-center">
    <video
      src="{{ 'assets/video/so-arm101/Banana_success_2.mp4' | relative_url }}"
      class="rounded z-depth-1"
      style="width:50%; height:auto; display:block; margin:0 auto;"
      autoplay
      muted
      loop
      playsinline>
    </video>
  </div>
</div>

<div class="caption">
  Banana pick-and-place result, 2x speed
</div>

---

## Goal

The main objective was to connect the full workflow required for real-world robot imitation learning:

1.      set up and verify leader-follower teleoperation,

2.      collect pick-and-place demonstrations,

3.      train an ACT-based imitation learning policy,

4.      deploy the trained policy on the real robot and verify whether imitation learning worked in practice.

The target task was a simple **fruit-object pick-and-place** task, designed as a first step toward real-world manipulation.

---

### Data Collection

<div class="row">
  <div class="col-sm mt-3 mt-md-0 text-center">
    <video
      src="{{ 'assets/video/so-arm101/Frist_pick_place_data_collection.mp4' | relative_url }}"
      class="rounded z-depth-1"
      style="width:70%; height:auto; display:block; margin:0 auto;"
      autoplay
      muted
      loop
      playsinline>
    </video>
  </div>
</div>

<div class="caption">
  Demonstration collection using the leader arm
</div>

The first task was **Pick-and-Place** with fruit-shaped objects.  
We used both a **top-view camera** and a **wrist camera** to record the workspace and the robot’s local view.

To make the dataset more diverse, we varied the bowl position and object orientation, creating approximately 38 demonstration episodes.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/so-arm101/Dataset_config.png" title="Bowl and object placement configurations used for data collection" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Bowl and object placement configurations used for data collection
</div>

### Policy Training

We trained an **ACT (Action Chunking with Transformers)** policy using the collected demonstrations.

ACT predicts a sequence of future actions instead of a single action at each timestep, which helps generate smoother robot trajectories during deployment.

---

## Experiment

<div class="row">
  <div class="col-sm mt-3 mt-md-0 text-center">
    <video
      src="{{ 'assets/video/so-arm101/Green_bean_fail.mp4' | relative_url }}"
      class="rounded z-depth-1"
      style="width:50%; height:auto; display:block; margin:0 auto;"
      autoplay
      muted
      loop
      playsinline>
    </video>
  </div>
</div>

<div class="caption">
  Failed green bean grasping trial, 2x speed
</div>

From the first deployment, we identified several likely issues:  
-	insufficient visual separation between the robot arm and the dark background,  
-	slow CPU-based inference during real-time control,  
-	action chunk size being too long for precise grasping behavior.  

To improve the setup, we changed the workspace floor from a black mat to a white material that did not interfere with the grasping motion. This made the robot arm and objects more visually distinguishable in the camera images.  

We also reduced the action chunk size from 100 to 32 to make the policy more responsive during deployment. This value was chosen after analyzing the collected trajectories and estimating that a shorter chunk length would better match the time scale of the grasping motion.  

After the revised setup, the robot showed more stable behavior on the banana pick-and-place task, although performance was still sensitive to object position and orientation.

---

## Resources  

The dataset we collected and the model used for inference are publicly available on Hugging Face.  
- [Pick-and-place Dataset](https://huggingface.co/datasets/ddduk/Pick_and_place_merged) — Demonstration dataset collected with the SO-ARM101 leader-follower setup.
- [ACT Model](https://huggingface.co/ddduk/ACT_SoArm_pick_place_box_bean) — ACT policy trained on the revised pick-and-place dataset.

---

## Lessons Learned

This project taught us that successful robot learning depends not only on model performance but also on the overall system design. Factors such as dataset quality, camera configuration, workspace design, inference speed, and control frequency significantly influence real-world deployment results.  

One of the biggest challenges was the high cost of iteration. Even a small model update required retraining, redeployment, and physical testing on the robot. This experience highlighted the importance of simulation-based evaluation before conducting experiments on real hardware.  

---

## Future Work

The next step is to build a simulation environment where policies can be tested before running them on the real robot.

We also plan to define quantitative evaluation metrics, such as:  

- pick-and-place success rate,  

- grasp success rate,  

- target placement accuracy,  

- task completion time, 

- failure type classification.  


Beyond ACT, we would like to test **Diffusion Policy** and VLA-based models such as **π0** and **GR00T**.  
A future direction is to fine-tune these models with LoRA and compare their generalization ability on new objects and environments.

---

## Summary

This project built a complete real-world imitation learning pipeline using SO-ARM101.  
Through hardware setup, teleoperation, data collection, ACT training, deployment, and dataset iteration, we explored the practical challenges of applying robot learning models to real hardware.

The project will be extended toward simulation-based testing, quantitative evaluation, and comparison with diffusion-based and VLA-based robot policies.
