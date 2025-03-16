---
title: 신발 탐지, 세그멘테이션 및 덧씌우기
lang: kr
layout: single
author_profile: true
permalink: /kr/projects/shoes-stitching/
classes: wide
github: https://github.com/JunhwanK/Shoes-For-Life
---

사용자의 구매결정을 돕기위해 사진 속 사용자의 신발 위에 쇼핑몰 신발을 자동으로 덧씌울 수 있는 파이프라인을 설계. {% if page.github %} <a href="{{ page.github }}">[GitHub 링크]</a> {% endif %}

## 주요 성과

<div class="side-by-side">
  <figure style="width: 40%" class="responsive-width">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/shoes-stitching/thumbnail2.png"
      alt="example shoes stitching result">
    <figcaption>예시 결과.</figcaption>
  </figure>
  <ul>
    <li>YOLOv3 사용하여 신발을 탐지. 담당자: <a href="https://github.com/mutichung">mutichung</a></li>
    <li>GrabCut과 SIFT feature를 사용하여 신발의 세그멘테이션 마스크를 추출. 담당자: <a href="https://github.com/tommykil123">tommykil123</a> & <a href="https://github.com/shuoqwang">shuoqwang</a></li>
    <li>신발 윤곽선을 기반으로 사용자 이미지 위에 쇼핑몰 신발을 덧씌우는 알고리즘을 독자적으로 고안 및 구현. 담당자: <a href="https://github.com/JunhwanK">JunhwanK</a> (본인)</li>
  </ul>
</div>

## 목적

사용자 사진에서 신발을 자동으로 탐지하고 세그멘테이션을 추출한 후, 이를 사용하여 사용자의 신발 위에 쇼핑몰 신발을 자동으로 덧씌운다.

<div class="side-by-side">
  <figure style="width: 50%" class="responsive-width">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/shoes-stitching/input-output.jpg"
      alt="Input and Output">
    <figcaption>쇼핑몰 신발(좌). 원본 사진(중). 결과(우).</figcaption>
  </figure>
</div>

## 파이프라인 설계

파이프라인은 총 3개의 단계로 이루어져 있다.

1. [신발 탐지](#신발-탐지)
2. [세그멘테이션 추출](#세그멘테이션-추출)
3. [덧씌우기](#덧씌우기)

### 신발 탐지

첫 단계에서는 YOLOv3를 사용하여 사용자 사진에서 신발의 바운딩 박스를 탐지한다 (<span data-figure-ref="detection1"></span>, <span data-figure-ref="detection2"></span>).

<div class="side-by-side">
  <figure style="width: 30%" class="responsive-width" id="detection1">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/shoes-stitching/detection1.png"
      alt="detection result">
    <figcaption>탐지결과 예시1.</figcaption>
  </figure>
  <figure style="width: 30%" class="responsive-width" id="detection2">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/shoes-stitching/detection2.png"
      alt="detection result">
    <figcaption>탐지결과 예시2.</figcaption>
  </figure>
</div>

### 세그멘테이션 추출

두번째 단계에서는 GrabCut 알고리즘을 사용하여 신발을 배경과 분리한다. 더욱 정확한 결과를 위해 SIFT 특징과 convex hull 알고리즘으로 신발이 있는 예상구역을 지정한다 (<span data-figure-ref="segmentation"></span>).

<div class="side-by-side">
  <figure style="width: 70%" class="responsive-width" id="segmentation">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/shoes-stitching/segmentation.png"
      alt="segmentation result">
    <figcaption>SIFT 특징(좌). SIFT 특징에 convex hull 알고리즘을 적용한 결과(중). 추출된 세그멘테이션(우).</figcaption>
  </figure>
</div>

### 덧씌우기

마지막 세번째 단계에서는, 먼저 세그멘테이션 마스크의 주성분 고유벡터(principal eigenvector)를 사용하여 쇼핑몰 신발과 사용자의 신발의 방향을 정렬한다 (<span data-figure-ref="orientation-matching"></span>).

<div class="side-by-side">
  <figure style="width: 70%" class="responsive-width" id="orientation-matching">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/shoes-stitching/orientation-matching.png"
      alt="orientation matching">
    <figcaption>각 신발의 주성분 고유벡터 (빨간 점선).</figcaption>
  </figure>
</div>

신발의 방향을 정렬시킨 뒤, 각 신발의 윤곽선을 수직과 수평 방향으로 잘라 샘플링한다 (<span data-figure-ref="contour-sampling"></span>).

<div class="side-by-side">
  <figure style="width: 50%" class="responsive-width" id="contour-sampling">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/shoes-stitching/contour-sampling.png"
      alt="contour sampling">
    <figcaption>윤곽선 샘플링. 수직 방향 샘플은 파란색, 수평 방향 샘플은 빨간색으로 표시.</figcaption>
  </figure>
</div>

샘플링된 점들을 기반으로 유사 변환(similarity transformation) 행렬을 계산한다.

단, 주성분 고유 벡터를 사용한 정렬만으로는 신발의 좌우 방향을 판별할 수 없기 때문에, 가능한 모든 좌우 조합을 시도한 후 가장 적절한 (가장 손실값이 작은) 매칭을 자동으로 선택한다 (<span data-figure-ref="contour-matching"></span>).

<div class="side-by-side">
  <figure style="width: 70%" class="responsive-width" id="contour-matching">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/shoes-stitching/contour-matching.png"
      alt="contour matching">
    <figcaption>모든 변환 조합.</figcaption>
  </figure>
</div>

마지막으로, 쇼핑몰 신발에 변환 행렬을 적용하여 사용자 신발 위에 덧씌운다 (<span data-figure-ref="stitching-result"></span>).

<div class="side-by-side">
  <figure style="width: 50%" class="responsive-width" id="stitching-result">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/shoes-stitching/stitching-result.png"
      alt="contour matching">
    <figcaption>덧씌운 결과.</figcaption>
  </figure>
</div>

## 결과

<div class="side-by-side">
  <figure style="width: 100%" class="align-center">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/shoes-stitching/results.jpg"
      alt="pipeline results">
    <figcaption>결과물.</figcaption>
  </figure>
</div>

## 제한점 및 개선방안

- 신발 탐지와 세그멘테이션 추출 단계를 나누지 않고, 인공신경망을 통해 세그멘테이션 마스크를 한번에 추출하는 것이 성능이 더 좋을거라 생각된다. 하지만 이를 위해서는 인공신경망을 학습시킬 신발 세그멘테이션 데이터가 필요하다. 한 가지 해결방법은 사용자에게서 신발의 좌표점을 요구하고 [Segment Anything Model](https://github.com/facebookresearch/sam2)을 사용하여 신발의 세그멘테이션을 추출하는 것이다.
- 이 파이프라인은 옷과 같이 윤곽/형체의 변형이 큰 물체는 다룰 수 없다. 형태의 변형이 큰 물체를 다룰려면 [OutfitAnyone](https://humanaigc.github.io/outfit-anyone/)과 비슷하게 조건부 확산 모델(conditional diffusion model)을 사용해야 한다.

<br><br>
<a href="{{ site.url }}{{ site.baseurl }}/kr/projects/">← 'Projects'로 돌아가기</a>
