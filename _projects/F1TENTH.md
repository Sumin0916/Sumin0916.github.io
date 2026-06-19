---
layout: page
title: F1TENTH Project
description: Applying a 2D LiDAR-based end-to-end driving model to F1TENTH Gym with ROS 2
img: assets/img/f1tenth/thumb.png
importance: 2
category: archived
related_publications: true
---

## Overview

This project applied **TinyLidarNet**, a 2D LiDAR-based end-to-end driving model proposed in the TinyLidarNet paper, to the **F1TENTH Gym Simulator** with ROS2.  {% cite zarrar2024tinylidarnet %}.

The original TinyLidarNet implementation was released as a ROS1-based codebase: [CSL-KU/TinyLidarNet](https://github.com/CSL-KU/TinyLidarNet). In this project, I adapted the model and implementation flow to a ROS2 Humble-based F1TENTH simulation environment.  
 
<div class="row">
  <div class="col-sm mt-3 mt-md-0 text-center">
    <video
      src="{{ 'assets/video/f1tenth/overview.mp4' | relative_url }}"
      class="rounded z-depth-1"
      style="width:65%; height:auto; display:block; margin:0 auto;"
      autoplay
      muted
      loop
      playsinline>
    </video>
  </div>
</div>

<div class="caption">
  TinyLidarNet driving in ROS 2 simulation
</div>

---

## Goal

The goal of this project was to adapt TinyLidarNet to a ROS 2-based F1TENTH Gym environment and evaluate its driving behavior in simulation.

I also explored whether a lightweight end-to-end model could simplify the conventional autonomous driving pipeline by directly predicting steering and speed commands from LiDAR scan data.

The project followed this workflow:  

1.	Analyze the TinyLidarNet paper and original ROS1 implementation.
2.	Convert the ROS1-based implementation to a ROS2 Humble environment.
3.	Build a ROS2 node that subscribes to LiDAR scan data and publishes Ackermann drive commands.
4.	Test the original model in F1TENTH Gym and identify failure cases.
5.	Improve driving stability through data augmentation and retraining.


---

## Model

TinyLidarNet takes 1D LiDAR range data as input and outputs the vehicle’s **steering angle** and **speed**.  

The model uses 1D CNN layers to capture spatial patterns in LiDAR scans, such as walls, corners, and track boundaries.

One practical issue was the output range.  
Since the final activation is tanh, the speed output is also constrained between -1 and 1.  
Therefore, before publishing the command to the simulator, the predicted speed had to be mapped to a usable driving speed range.  

<div class="row">
    <div class="col-sm mt-3 mt-md-0 d-flex justify-content-center">
        <div style="width:35%;">
            {% include figure.liquid loading="eager" path="assets/img/f1tenth/model_archi.png" title="TinyLidarNet Architecture" class="img-fluid rounded z-depth-1" %}
        </div>
    </div>
</div>
<div class="caption">
    TinyLidarNet Architecture
</div>

## ROS2 Application

The original TinyLidarNet implementation was written for ROS1 Noetic, so it could not be used directly in my ROS2 Humble environment.  

I first built a new ROS2 package with a simple inference pipeline:  

1.	Subscribe to the /scan topic.
2.	Preprocess the LiDAR scan data.
3.	Run inference with the trained TinyLidarNet model.
4.	Convert the model output into steering and speed commands.
5.	Publish an Ackermann drive command to the simulator.  

During this process, several compatibility issues had to be handled.  

- The first issue was the ROS version difference.  

  The original ROS1 node structure had to be rewritten using the ROS2 rclpy interface.  

- The second issue was the training and inference environment.  

  The original implementation was designed around an older TensorFlow/TFLite setup and a real vehicle environment.  
  I modified the training and inference code to make it easier to run on a desktop-based Keras environment.  


<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <div style="width:60%; margin:0 auto;">
            {% include figure.liquid loading="eager" path="assets/img/f1tenth/ROS2_node.png" title="ROS 2 node diagram" class="img-fluid rounded z-depth-1" %}
        </div>
    </div>
</div>
<div class="caption">
    ROS2 node diagram
</div>

## Initial Experiment

After porting TinyLidarNet to ROS2, I tested the original model in the F1TENTH Gym Simulator. However, it failed to complete some maps reliably.

Since the ROS2 implementation did not show any major issues, I suspected that the main problem came from the domain gap between the real vehicle and the simulator.  

This led me to focus on improving the model through additional data collection and augmentation.

<div class="row">
  <div class="col-sm mt-3 mt-md-0 text-center">
    <video
      src="{{ 'assets/video/f1tenth/2. Austin_ocilation_Problem.mp4' | relative_url }}"
      class="rounded z-depth-1"
      style="width:65%; height:auto; display:block; margin:0 auto;"
      autoplay
      muted
      loop
      playsinline>
    </video>
  </div>
</div>

<div class="caption">
  Oscillation problem on the Austin map
</div>

## Data Augmentation

The augmentation process included the following steps:  

1.	Collect additional driving data on two small simulation tracks using MPC and Pure Pursuit.
2.	Flip LiDAR scan data and steering angles to reduce steering distribution bias.
3.	Add Gaussian noise to LiDAR scans to improve robustness.
4.	Apply dropout by replacing some scan values with the maximum range value, making the data closer to the simulator scan pattern.  

Through this process, I constructed a training dataset that was approximately 5x larger than the original dataset.


## Result

After retraining the model with the augmented dataset, the driving behavior became more stable than the original model.  

The retrained model showed reduced oscillation on straight sections and handled sharp corners more reliably. It also performed better on F1TENTH Gym maps that were not included in the training data.  

This improvement was also visible in the steering angle time-series plot.  

Before augmentation, the model produced sudden steering changes in some sections.  
After augmentation, these abrupt changes were reduced, and the overall steering variance became lower.  

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/f1tenth/result_series.png" title="ROS" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Steering angle comparison between the original model and the augmented model
</div>

---

## Limitations

TinyLidarNet is lightweight and suitable for real-time inference, but several limitations remained.  

•	The speed output still needs to be mapped manually to a usable driving speed range.  
•	The current model does not naturally support reverse driving.  
•	I did not implement an online retraining pipeline that continuously improves the model from newly collected driving data.  
•	If the vehicle platform changes, new data may need to be collected to match the new vehicle dynamics.  
•	Since the model is end-to-end, it cannot directly explain the cause of driving failures.  

## Summary

This project showed that applying a research model to a different environment requires more than code conversion. Differences in ROS version, LiDAR configuration, simulator dynamics, and training data distribution all affected driving performance.

By augmenting the dataset with simulation-based driving data, the retrained model showed more stable behavior than the original model. Through this project, I learned both the practicality and limitations of lightweight end-to-end driving models in a ROS 2-based autonomous driving pipeline.  
