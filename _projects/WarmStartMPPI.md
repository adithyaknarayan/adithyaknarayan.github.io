---
layout: page
title: "Warm Start and Knot Point Interpolation for Whole Body MPPI"
description: We enhance whole-body control for quadrupedal robots by introducing warm-starting, spline interpolation adjustments, and terrain-specific testing, demonstrating improved performance in complex tasks like stair-climbing and uneven terrain traversal.
abstract: We build upon the results from “Real-Time Whole-Body Control of Legged Robots with Model-Predictive Path Integral Control,” by Alvarez-Padilla, et al., which introduces whole-body sampling-based control for locomotion and manipulation on quadrupedal robots. Our contributions include the implementation of warm-starting to speed up the control trajectory optimization process, an ablation of spine interpolation order and knotpoint density, and tests of these improvements on newly designed terrains. We see that warm-starting and changes to spline order can help in some cases, such as stair-climbing and uneven terrain traversal, whereas the nominal implementation of MPPI is best for more basic tasks such as trotting in place.
img: assets/img/projects/MPPI/push_box_linear_k_3.gif
importance: 2
category: projects
selected: true
slides: https://docs.google.com/presentation/d/1aIJi8TTQ_2apgSVLX4fu_1wvldGSHf7OTNuZTS-WaT4/edit#slide=id.p
pdf: https://drive.google.com/file/d/1oq_rm9oHCNvl-zMzQ6gcCnyWImXM0DPZ/view?usp=sharing
authors:
  - name: "Anoushka Alavilli"
  - name: "Adithya Narayan"
    link: "https://adithyaknarayan.github.io/"
  - name: "Alyn Kirsch Tornell"
  - name: "Colleen Que"
  - name: "Lamia Alsalloom"

---