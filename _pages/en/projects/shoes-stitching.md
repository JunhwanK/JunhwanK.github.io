---
title: Shoe Detection, Segmentation, and Stitching
lang: en
layout: single
author_profile: true
permalink: /en/projects/shoes-stitching/
classes: wide
github: https://github.com/JunhwanK/Shoes-For-Life
---

# Shoe Detection, Segmentation, and Stitching

A pipeline to automatically overlay retailer shoes onto user photos, helps users visualize how they would look in different retailer shoes. {% if page.github %} <a href="{{ page.github }}">View on GitHub</a> {% endif %}

## Key Highlights

<div class="side-by-side">
  <figure style="width: 40%" class="responsive-width">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/shoes-stitching/thumbnail2.png"
      alt="example shoes stitching result">
    <figcaption>Example result.</figcaption>
  </figure>
  <ul>
    <li>Used YOLOv3 for shoe detection.</li>
    <li>Combined GrabCut with SIFT feature convex hull for segmentation.</li>
    <li>Stitched retailer shoes onto user images by applying similarity transformations based on shoe contours.</li>
  </ul>
</div>

## Objective

The goal of the pipeline is to automatically detect and segment shoes from user photos and then to overlay/stitch them with retailer shoes.

<div class="side-by-side">
  <figure style="width: 50%" class="responsive-width">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/shoes-stitching/input-output.jpg"
      alt="Input and Output">
    <figcaption>Retailer shoes (left). User input (middle). Expected Output (right).</figcaption>
  </figure>
</div>

## Methodology

The pipeline consists of three steps.

1. Detection
2. Segmentation
3. Stitching

### Detection

In the detection step, the bounding boxes of shoes in the user-provided photo are detected using YOLOv3 (<span data-figure-ref="detection1"></span> and <span data-figure-ref="detection2"></span>).

<div class="side-by-side">
  <figure style="width: 30%" class="responsive-width" id="detection1">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/shoes-stitching/detection1.png"
      alt="detection result">
    <figcaption>Example detection result.</figcaption>
  </figure>
  <figure style="width: 30%" class="responsive-width" id="detection2">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/shoes-stitching/detection2.png"
      alt="detection result">
    <figcaption>Example detection result.</figcaption>
  </figure>
</div>

### Segmentation

In the segmentation step, the GrabCut algorithm segments the shoe from the background. A convex hull of the SIFT features is provided to GrabCut as the probable foreground region (<span data-figure-ref="segmentation"></span>).

<div class="side-by-side">
  <figure style="width: 70%" class="responsive-width" id="segmentation">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/shoes-stitching/segmentation.png"
      alt="segmentation result">
    <figcaption>SIFT features (left). Convex hull of SIFT features (middle). Segmentation result (right).</figcaption>
  </figure>
</div>

### Stitching

First, align the retailer shoes and the user's shoes by using the segmentation masks' principal eigenvector (<span data-figure-ref="orientation-matching"></span>).

<div class="side-by-side">
  <figure style="width: 70%" class="responsive-width" id="orientation-matching">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/shoes-stitching/orientation-matching.png"
      alt="orientation matching">
    <figcaption>Orientation matching using the principal eigenvector (shown in red).</figcaption>
  </figure>
</div>

After aligning the shoes by rotating, points from the shoes' contour are sampled by slicing the shoes vertically and horizontally (<span data-figure-ref="contour-sampling"></span>).

<div class="side-by-side">
  <figure style="width: 50%" class="responsive-width" id="contour-sampling">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/shoes-stitching/contour-sampling.png"
      alt="contour sampling">
    <figcaption>Contour sampling. Samples from horizontal slices shown in red. Samples from vertical slices shown in blue.</figcaption>
  </figure>
</div>

Using the sampled points from the shoes' contour, a similarity transformation matrix is computed.

However, aligning the shoes using the principal eigenvector still doesn't tell the direction in which the shoes are facing. Hence, the pipeline tries all possible combinations using left/right-facing shoes, and then it automatically chooses the best matching transformation (<span data-figure-ref="contour-matching"></span>).

<div class="side-by-side">
  <figure style="width: 70%" class="responsive-width" id="contour-matching">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/shoes-stitching/contour-matching.png"
      alt="contour matching">
    <figcaption>Combinations for stitching.</figcaption>
  </figure>
</div>

Finally, the best matching transformation is applied to the retailer shoe, which is then overlayed on the user's shoe (<span data-figure-ref="stitching-result"></span>).

<div class="side-by-side">
  <figure style="width: 50%" class="responsive-width" id="stitching-result">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/shoes-stitching/stitching-result.png"
      alt="contour matching">
    <figcaption>Stitching result.</figcaption>
  </figure>
</div>

## Results

<div class="side-by-side">
  <figure style="width: 100%" class="align-center">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/shoes-stitching/results.jpg"
      alt="pipeline results">
    <figcaption>Successful results.</figcaption>
  </figure>
</div>

## Limitations and Future Work

- Combining the detection and segmentation stages into one, using a neural network to directly perform segmentation would improve performance and simplify the pipeline. Also, while YOLO is fast, it is not the best model to use when aiming for detection/segmentation accuracy.

- The pipeline is limited to side-views of shoes, and the pipeline cannot handle soft-body objects that are more deformable, such as clothes. To handle novel-view synthesis for deformable objects, conditional diffusion models will need to be used (similar to [OutfitAnyone](https://humanaigc.github.io/outfit-anyone/)).

<br><br>
<a href="{{ site.url }}{{ site.baseurl }}/en/projects/">← Back to 'Projects'</a>
