---
layout: page
title: "Robust Point Tracking with Epipolar Constraints"
description: Geometry-driven point tracking that reduces long-horizon drift by enforcing epipolar consistency via post-processing refinement and weakly-supervised finetuning.
img: assets/img/projects/RobustPointTrackingEpipolar/splash_p1.png
importance: 2
category: projects
selected: true
github: https://github.com/adithyaknarayan/EpiTrack
pdf: /assets/pdf/Geom_Final_report-2.pdf
authors:
  - name: "Adithya Narayan"
    link: "https://adithyaknarayan.github.io/"
  - name: "Tanisha Gupta"
  - name: "Lamia Alsalloom"
---

## Overview
We present a geometry-driven, robust point-tracking framework that improves tracking accuracy in videos by enforcing **epipolar constraints**.
While modern trackers (e.g., CoTracker) are strong, they can accumulate **geometric drift** over long sequences, producing correspondences that violate multi-view geometry.

We address this in two complementary ways:

- **Post-processing (model-agnostic)**: An iterative refinement module that corrects correspondences frame-by-frame to satisfy epipolar constraints.
- **Finetuning**: A CoTracker finetuning strategy that adds a **soft epipolar loss** on rigid/static regions.

If you want the full report, see: **[PDF]({{ page.pdf | relative_url }})**. Code: **[GitHub]({{ page.github }})**.

## Contributions
- **Iterative epipolar refinement** that post-processes tracks from any point tracker using RANSAC-based fundamental matrix estimation + correspondence correction.
- **Teacher–student weak supervision** with a soft epipolar constraint that encourages geometric consistency while learning to separate static/dynamic motion.

## Motivation: geometric drift grows over time
For rigid/static scene points, correspondences between two frames must satisfy the epipolar constraint:
\[
x_t^\top F x_0 = 0
\]
where \(F\) is the fundamental matrix. In practice, long-horizon trackers can drift and increasingly violate this constraint.

## Figures

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/projects/RobustPointTrackingEpipolar/splash_p1.png" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/projects/RobustPointTrackingEpipolar/splash_p2.png" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Point tracking can cover both foreground and background points and handle both rigid and non-rigid motion.
</div>

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/RobustPointTrackingEpipolar/geometric_drift_plot_motivation.png" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Epipolar drift increases over time for long videos, motivating geometric constraints.
</div>

## Method 1 — Post-processing: iterative epipolar refinement (model-agnostic)
Given tracks from any point tracker:

- Estimate a fundamental matrix \(F_{0t}\) between frame 0 and each later frame \(t\) using **RANSAC** on mutually-visible correspondences.
- Compute epipolar error (point-to-epipolar-line distance):
\[
d_{\text{epi}}(x_0, x_t)=\frac{|x_t^\top F x_0|}{\sqrt{(Fx_0)_1^2 + (Fx_0)_2^2}}.
\]
- Identify outliers and **correct correspondences**:
  - **Primary**: SIFT-based local search in a 20px window around the epipolar line to recover a plausible match.
  - **Fallback**: project the outlier point onto the epipolar line.
- Re-estimate \(F\) with tighter thresholds and iterate until convergence.

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/RobustPointTrackingEpipolar/iterative_refinement_1.png" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Post-processing pipeline: estimate fundamental matrices and iteratively correct outliers to improve multi-view consistency.
</div>

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/RobustPointTrackingEpipolar/iterative_refinement_2.png" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/RobustPointTrackingEpipolar/iterative_refinement_3.png" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Left: SIFT-based search along the epipolar line to correct outliers. Right: naive “snap to closest point on line” can destroy correspondence and destabilize \(F\) estimation.
</div>

## Method 2 — Finetuning: soft epipolar loss with teacher–student supervision
We also finetune CoTracker with a weakly-supervised epipolar objective on (approximately) static regions, inspired by ROMO-style motion masks.

High-level idea:

- A **teacher** produces tracks.
- A **student** is trained to stay close to the teacher on dynamic regions, while receiving an epipolar-consistency gradient on static regions.
- At inference time, we drop masks and hope the student internalizes the geometry prior.

## Evaluation setup & metrics
We evaluate on long sequences (80+ frames) with both rigid and non-rigid motion. In addition to standard TAP‑Vid DAVIS metrics, we report:

- **Mean / Median epipolar error (px)**: lower means more multi-view consistent tracks.

## Results
Across our experiments, enforcing geometry often improves epipolar consistency (sometimes at a small cost to standard tracking metrics).

| Method | δ_visavg (%) | OA (%) | AJ (%) | Mean Error (px) | Median Error (px) |
| --- | ---:| ---:| ---:| ---:| ---:|
| PIPs | 64.0 | 77.0 | 63.5 | 4.87 | 3.42 |
| TAPIR | 62.9 | 88.0 | 73.3 | 3.95 | 2.76 |
| CoTracker3 (baseline) | 77.3 | 91.8 | 64.5 | 3.24 | 2.18 |
| CoTracker3 + Finetune (ours) | 75.1 | 90.5 | 62.7 | 2.41 | 1.58 |
| Ours + Post-processing | 75.8 | 90.3 | 63.1 | **1.31** | **0.89** |

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/RobustPointTrackingEpipolar/drift_reduce.png" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Geometric error over time on a larger TAP‑Vid subset, demonstrating reduced drift after finetuning.
</div>

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/RobustPointTrackingEpipolar/finetune_metrics_over_time.png" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Training curves (1000 finetuning steps): standard tracking metrics vs. geometric (epipolar) error.
</div>

## Qualitative results & failure cases
We generally observe geometrically consistent corrections in static regions, but those corrections can sometimes leak into dynamic regions and distort motion.

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/RobustPointTrackingEpipolar/finetune_qual_1.png" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/RobustPointTrackingEpipolar/finetune_qual_2_bad.png" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Left: static-region corrections can be beneficial. Right: in low-motion scenes, weak supervision can destabilize correspondences.
</div>

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/RobustPointTrackingEpipolar/epi_opt_failure.png" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="assets/img/projects/RobustPointTrackingEpipolar/epi_opt_corrections.png" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
<div class="caption">
  Post-processing can improve early-frame consistency, but can fail late when many tracks drift and \(F\) becomes mis-estimated.
</div>

## Links
- **Code**: `https://github.com/adithyaknarayan/EpiTrack`
- **Report (PDF)**: `{{ page.pdf | relative_url }}`

