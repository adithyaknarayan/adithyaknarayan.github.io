---
layout: distill
title: "Camera Control for Long Video Generation in Diffusion Models"
date: 2018-09-27
categories: [Computer Vision, Signals Processing]
tags: [diffusion models, cfg, dmd distillation]
math: true
img_path: /docs/assets/shannon_energy
---
# Introduction
Video diffusion models do a simple thing: given a random noisy vector we generate a temporally consistent video. Similar to image diffusion models, in this generation process it is often important to be able to generate videos in a consistent and controllable way. This is where conditioninal models come into play. Video diffusion models are no different. We look to introduce conditioning signals as a way to control the content that's being generated. One such example is camera motion.

Conceptually, this form of conditioning is simple. Can we specify a camera trajectory and have the video model generate a plausible video following that same trajectory. For instance, if we gave the model a horizontal camera motion, can we generate a video mimicing this leftward motion?

## Why is this challenging?
While conceptually easy to understand - the problem is nuanced. For instance, the first question we ask is this: What is a good conditioning signal for camera trajectories? One plausible answer is camera extrinsics - (R,T) that define the global rotation and position in the world coordinates. In this way, we feed the model the explicit pose of the camera for every frame generated and teach the model to follow it.

While this is a reasonable

This is naive. Largely because it is difficult for models to understand the scale of the scene. What does 1 unit of camera translation mean? It's easy to understand for each individual datapoint. In a room, 1 unit perhaps translates to 1 meter. How about if it's outdoor or a drone shot? 1 unit could translate to 100s of meters. If that's a bit confusing, let's put it another way. Imagine a person and a drone. Both sample the camera pose at the start of a video sequence and after exactly 1s. In this 1 second, the human might have travelled around 1m but in the same second the drone may have moved around 10-15m. So for the same time frame you can end up with significantly different magnitudes of motion. Now this wouldn't be a problem if camera extrinsics was "metric" in nature. For instance, if the models used to predict camera pose output directly in meters what the motion is we'd have no issues. Unfortunately, most models are scaled to some unknown scale factor.

Another interesting challenge is the actual pose itself. For the human, relative to some arbitrary world coordinate system he could be at (1,1,1). So for a 1m displacement along the z-axis you end up at (1,1,2). A drone however, in the same coordinate system might start at (100,100,100) and translate to (100,100,101). Obviously, we need some form of normalization to address this.

P