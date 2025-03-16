---
title: LSTM-based Classifier for Counter-Battery Radars
lang: en
layout: single
author_profile: true
permalink: /en/projects/ballistic-projectile/
classes: wide
github:
---

A neural network classifier to help counter-battery radars quickly distinguish between ballistic projectiles and other clutters (birds, airplanes, false detections, etc.).

## Key Highlights

- Achieved 94.3% accuracy and 0.85 F1-score on real-world data collected by counter-battery radars.
- Patent granted (10-2641022-0000) in Korea as the property of the Republic of Korea Army.

## Objective

The goal for this model is to be used as a light-weight first-layer filter for counter-battery radars. The model must prioritize high recall and short runtime.

Available input: coordinates of the trajectory, radar cross section (RCS), signal-to-noise ratio, radial velocity, etc.

Output: a value in [0,1] range (0 for clutter, 1 for projectile).

## Existing Study

An existing study by Koh et al. [[1]](#references) used a two-layer LSTM model with dropout layers, and experimented with the following four cases of input features.

Given the target’s location (east, north, up) at time _$$t$$_ is denoted as _$$e_t$$, $$n_t$$, $$u_t$$_

- Case 1: _$$e_t$$, $$n_t$$, $$u_t$$, radial velocity, RCS_
- Case 2: _$$\Delta e_t$$, $$\Delta n_t$$, $$\Delta u_t$$, radial velocity, RCS_
- Case 3: _$$e_t / n_t$$, radial velocity, RCS_
- Case 4: _$$\Delta (e_t / n_t)$$, radial velocity, RCS_

Upon trying to replicate the study's results, we found that these models perform well (about 95% accuracy and 0.88 f1-score) when tested on unaugmented data (<span data-figure-ref="trajectories"></span>).

However, we found that these models do not generalize well to data with random augmentations in the trajectory direction (<span data-figure-ref="augmented-trajectories"></span>). The model performances dropped to about 60% accuracy when being trained/ tested on the augmented dataset.

<div class="side-by-side">
  <figure style="width: 33%" class="responsive-width" id="trajectories">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/ballistic-projectile/trajectories.png"
      alt="trajectory data">
    <figcaption>Trajectory data.</figcaption>
  </figure>
  <figure style="width: 34%" class="responsive-width" id="augmented-trajectories">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/ballistic-projectile/augmented-trajectories.png"
      alt="augmented trajectory data">
    <figcaption>Augmentation trajectory data.</figcaption>
  </figure>
  <figure style="width: 33%" class="responsive-width">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/ballistic-projectile/clutter.png"
      alt="clutter data">
    <figcaption>Clutter data.</figcaption>
  </figure>
</div>

## Methodology

To keep the input feature invariant to trajectory direction, the following features are instead chosen for the model, $$ \Delta u $$, and $$ a_t $$.

Change in trajectory angle ($$a_t$$) and change in altitude ($$\Delta u_t$$) can be computed using the following formula.

Given that the position of the target (east, north up) at times $$t-1$$, $$t$$, $$t+1$$ are denoted as $$(e_{t-1}, n_{t-1}, u_{t-1})$$, $$(e_t, n_t, u_t)$$, $$(e_{t+1}, n_{t+1}, u_{t+1})$$ respectively,

$$ v_t = [ e_t - e_{t-1} , n_t - n_{t-1} , u_t - u_{t-1} ] $$

$$ v_{t+1} = [ e_{t+1} - e_t , n_{t+1} - n_t , u_{t+1} - u_t ] $$

$$ a_t = \arccos(\frac{v_t \cdot v_{t+1}}{|v_t| |v_{t+1}|}) $$

$$ \Delta u_t = u_t - u_{t-1} $$

The model structure is kept the same as [[1]](#references), a two-layer LSTM with a dropout layer in between.

To reduce computation time, only the first 13 data points in the sequence are used as input to the model.

## Results

- 94.3% accuracy with 0.85 F1-score.
- 0.623 precision @ 0.98 recall.
- 0.813 precision @ 0.96 recall.

## Limitations and Future Work
 
- The network needs to be robust against missing data points in the trajectory. Different interpolation methods and their effect on the input features are worth exploring. Incorporating $$ dt $$ or positional embedding into the inputs may be an alternative to requiring interpolation for missing data points.
- The model can be extended to multi-class classification, classifying the different types of ballistic projectiles or the different types of clutters (e.g. drones).
- A transformer-based model with a similar number of parameters performed slightly worse (93.69% accuracy with 0.7899 F1-score) than the LSTM-based model. It is worth exploring how transformer-based models perform in comparison to LSTM-based models under varying circumstances (e.g. longer input sequences, greater noise, etc.).

## Patent

Based on this study, we got a patent granted in Korea as the property of Republic of Korea Army.

- Patent name: Artificial Intelligence-Based Target Classification Method and System For Counter-Battery Radar (인공지능 기반 대포병탐지레이더 표적분류 방법 및 시스템)
- Korean Patent Number: 10-2641022-0000

## References {#references}

[1] I.-S. Koh, H. Kim, S.-H. Chun, and M.-K. Chong, 'Efficient Recurrent Neural Network for Classifying Target and Clutter: Feasibility Simulation of Its Real-Time Clutter Filter for a Weapon Location Radar', _Journal of Electromagnetic Engineering and Science_, vol. 22, no. 1, pp. 48–55, 2022.

<br><br>
<a href="{{ site.url }}{{ site.baseurl }}/en/projects/">← Back to 'Projects'</a>
