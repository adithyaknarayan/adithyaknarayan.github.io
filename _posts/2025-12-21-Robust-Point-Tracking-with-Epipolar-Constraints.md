---
layout: post
title: "Robust Point Tracking with Epipolar Constraints"
date: 2025-12-21
description: Reducing long-horizon drift in point trackers by enforcing epipolar geometry via model-agnostic post-processing and weakly-supervised finetuning.
tags: [computer-vision, geometry, tracking]
categories: [research]
---

Point tracking is a core primitive for video understanding, editing, and reconstruction. Modern deep trackers like CoTracker can produce dense tracks over long sequences, but **geometric drift** still accumulates—especially when there’s significant camera motion—leading to correspondences that violate basic multi-view constraints.

This project explores a simple question:
**Can we reduce long-horizon drift by explicitly enforcing epipolar geometry?**

Links:
- Code: `https://github.com/adithyaknarayan/EpiTrack`
- Full report (PDF): `/assets/pdf/Geom_Final_report-2.pdf`
- Project page: `/projects/RobustPointTrackingEpipolar/`

## Key idea: epipolar constraints as “guardrails”
For rigid/static parts of the scene, correspondences between two frames should satisfy:
\[
x_t^\top F x_0 = 0
\]
where \(F\) is the fundamental matrix between the two views.

When tracks drift, this constraint is violated. We use that signal in two ways.

## Approach 1: Model-agnostic post-processing (iterative epipolar refinement)
Given tracks from any point tracker:

- Estimate \(F\) with **RANSAC** using mutually visible correspondences.
- Identify outliers via epipolar error (point-to-epipolar-line distance).
- Correct outliers using a **local SIFT search along the epipolar line** (fallback: snap to closest point on the line).
- Re-estimate \(F\) with a tighter threshold and iterate until convergence.

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/projects/RobustPointTrackingEpipolar/iterative_refinement_1.png" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Post-processing pipeline overview: estimate fundamental matrices and iteratively correct outliers to enforce epipolar consistency.
</div>

## Approach 2: Weakly-supervised finetuning (soft epipolar loss)
We also finetune CoTracker with a **soft epipolar loss** on rigid/static regions, using a teacher–student setup to encourage:

- stronger geometric consistency on static points
- minimal disruption to non-rigid/dynamic motion

## Results: geometric consistency improves (sometimes at a cost)
Across long sequences, both approaches reduce geometric drift and lower epipolar error. In some settings, enforcing geometry can slightly reduce standard tracking metrics (especially in dynamic regions), but it yields **significantly better multi-view consistency** over long horizons.

On TAP‑Vid DAVIS (reported in our writeup), we see:

| Method | δ_visavg (%) | OA (%) | AJ (%) | Mean Error (px) | Median Error (px) |
| --- | ---:| ---:| ---:| ---:| ---:|
| PIPs | 64.0 | 77.0 | 63.5 | 4.87 | 3.42 |
| TAPIR | 62.9 | 88.0 | 73.3 | 3.95 | 2.76 |
| CoTracker3 (baseline) | 77.3 | 91.8 | 64.5 | 3.24 | 2.18 |
| CoTracker3 + Finetune (ours) | 75.1 | 90.5 | 62.7 | 2.41 | 1.58 |
| Ours + Post-processing | 75.8 | 90.3 | 63.1 | **1.31** | **0.89** |

## Project page
Full summary + figures live here:
- `/projects/RobustPointTrackingEpipolar/`


