---
title: LSTM 기반 대포병탐지레이더 표적분류
lang: kr
layout: single
author_profile: true
permalink: /kr/projects/ballistic-projectile/
classes: wide
github:
---

# LSTM 기반 대포병탐지레이더 표적분류

대포병탐지레이더의 빠른 표적/비표적 분류를 돕기 위한 인공신경망.

표적: 포탄.

비표적: 새, 비행기, 허상표적, 등 포탄 외 모든 것.

## 주요 성과

- 대포병탐지레이더가 수집한 실제 데이터에서 94.3% 정확도와 0.85 F1 점수를 달성.
- 특허 등록 번호: 10-2641022 (대한민국 육군 소유).

## 목적

이 연구의 목적은 대포병탐지레이더에 1차 표적/비표적 분류 필터로 사용될 경량 모델을 개발하는 것이다. 1차 필터로 사용될 모델이기 때문데 높은 재현율(recall)과 짧은 연산시간을 우선시 해야한다.

사용 가능한 데이터: 궤적의 좌표값, 레이더 반사 면적 (RCS), 신호 대 잡음비 (SNR), 시선속도, 등.

모델 출력값: [0,1]의 값 (0에 가까울수록 비표적, 1에 가까울수록 표적).

## 선행 연구

한 선행 연구에서 두 개의 LSTM 레이어 사이에 dropout 레이어를 추가한 모델을 사용하여 다음 네 가지 특징 조합 입력값으로 실험했다 [[1]](#references).

시간 $$t$$에 탐지된 물체의 좌표값(east, north, up)을 각 $$e_t$$, $$n_t$$, $$u_t$$이라고 할때,

- Case 1: $$e_t$$, $$n_t$$, $$u_t$$, _시선속도, RCS_
- Case 2: $$\Delta e_t$$, $$\Delta n_t$$, $$\Delta u_t$$, _시선속도, RCS_
- Case 3: $$e_t / n_t$$, _시선속도, RCS_
- Case 4: $$\Delta (e_t / n_t)$$, _시선속도, RCS_

위 연구를 재현한 결과, 해당 모델들은 증강되지 않은 데이터(<span data-figure-ref="trajectories"></span>)에서는 약 95%의 정확도와 0.88의 F1 점수를 기록하지만, 궤적 방향을 무작위로 변형한 증강 데이터(<span data-figure-ref="augmented-trajectories"></span>)에서는 정확도가 60% 정도로 감소하는 것을 발견했다.

<div class="side-by-side">
  <figure style="width: 33%" class="responsive-width" id="trajectories">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/ballistic-projectile/trajectories.png"
      alt="trajectory data">
    <figcaption>표적 데이터 (포탄 궤적).</figcaption>
  </figure>
  <figure style="width: 34%" class="responsive-width" id="augmented-trajectories">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/ballistic-projectile/augmented-trajectories.png"
      alt="augmented trajectory data">
    <figcaption>증강된 표적 데이터.</figcaption>
  </figure>
  <figure style="width: 33%" class="responsive-width">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/ballistic-projectile/clutter.png"
      alt="clutter data">
    <figcaption>비표적 데이터.</figcaption>
  </figure>
</div>

## 개선 방법

기존 모델의 문제점을 해결하기 위해, 궤적 방향에 영향 받지 않는 입력 특징 $$ a_t $$와 $$ \Delta u $$를 고안했다.

궤적 각도 변화량 $$a_t$$와 고도 변화량 $$\Delta u_t$$는 아래 공식을 사용하여 계산한다.

시간 $$t-1$$, $$t$$, $$t+1$$에 탐지된 물체의 좌표값(east, north, up)을 각각 $$(e_{t-1}, n_{t-1}, u_{t-1})$$, $$(e_t, n_t, u_t)$$, $$(e_{t+1}, n_{t+1}, u_{t+1})$$이라고 할때,

$$ v_t = [ e_t - e_{t-1} , n_t - n_{t-1} , u_t - u_{t-1} ] $$

$$ v_{t+1} = [ e_{t+1} - e_t , n_{t+1} - n_t , u_{t+1} - u_t ] $$

$$ a_t = \arccos(\frac{v_t \cdot v_{t+1}}{|v_t| |v_{t+1}|}) $$

$$ \Delta u_t = u_t - u_{t-1} $$

모델의 구조는 기존 연구와[[1]](#references) 같게 2개의 LSTM 레이어 사이에 dropout 레이어를 추가한다.

연산시간을 단축하기 위해 궤적의 첫 13개의 데이터 포인트만 모델의 입력값으로 사용한다.

## 결과

- 94.3% 정확도와 0.85 F1 점수.
- 0.623 precision @ 0.98 recall.
- 0.813 precision @ 0.96 recall.

## 제한점 및 향후 연구방향

- 궤적 내 결측치에 견고한 모델을 만들어야 하기 때문에 다양한 보간법(interpolation)에 대한 실험과 결측치가 특성값에 미치는 영향에 대한 연구가 필요하다. 또한, 보간법 사용하는 대신 $$ dt $$ 또는 위치 임베딩(positional embedding)을 모델 입력값에 포함하는 방안도 있다.
- 멀티클래스 분류가 가능하도록 모델을 확장할 수 있다. 이를 통해 다양한 종류의 포탄이나 비표적(드론, 등)을 분류를 시도할 수 있다.
- 비슷한 파라미터 개수를 가진 트랜스포머 기반 모델은 LSTM 기반 모델보다 조금 낮은 성능(93.69% 정확도와 0.79 F1 점수)를 보였다. 입력 시퀀스 길이의 변화, 노이즈 강도의 변화, 결측치 비율의 변화, 등 다양한 상황에서 LSTM과 트랜스포머 모델의 성능을 비교해 볼 가치가 있다.

## 특허

본 연구를 토대로, 대한민국 육군 소유의 특허를 등록했다.

- 특허 이름: 인공지능 기반 대포병탐지레이더 표적분류 방법 및 시스템 (Artificial Intelligence-Based Target Classification Method and System For Counter-Battery Radar)
- 특허 등록번호: 10-2641022

## 참고문헌 {#references}

[1] I.-S. Koh, H. Kim, S.-H. Chun, and M.-K. Chong, 'Efficient Recurrent Neural Network for Classifying Target and Clutter: Feasibility Simulation of Its Real-Time Clutter Filter for a Weapon Location Radar', _Journal of Electromagnetic Engineering and Science_, vol. 22, no. 1, pp. 48–55, 2022.

<br><br>
<a href="{{ site.url }}{{ site.baseurl }}/kr/projects/">← 'Projects'로 돌아가기</a>
