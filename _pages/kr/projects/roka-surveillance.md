---
title: 실시간 객체탐지 경계 시스템
lang: kr
layout: single
author_profile: true
permalink: /kr/projects/roka-surveillance/
classes: wide
github:
---

# 실시간 객체탐지 경계 시스템

육군 분석평가단 근무 중에 개발한 실시간 객체탐지 기능을 갖춘 경계 시스템.

## 시연 영상

전체 화면에 1080p 화질로 시청을 권장합니다.
{% include youtube_video.html id="loa6Bc2EMkc" %}

## 주요 성과

- 객체탐지 모델에 군용 데이터를 학습시키고 UI와 경계기능을 추가. 6개의 후방부대에 시험적용.
- 멀티스레딩 및 멀티프로세싱을 활용하여 초당 프레임 처리량을 60% 증가.

## 주요 기능

사용자 피로도 감소와 향후 모델 파인튜닝을 위한 데이터 수집을 위해 다음과 같은 기능을 추가했다.

### 예외구역

<div class="side-by-side">
  <figure style="width: 35%" class="responsive-width" id="ignore-region1">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/roka-surveillance/ignore-region1.png"
      alt="inside ignore-region">
    <figcaption>예외구역 내 사람.</figcaption>
  </figure>
  <figure style="width: 35%" class="responsive-width">
    <img
      src="{{ site.url }}{{ site.baseurl }}/assets/images/roka-surveillance/ignore-region2.jpg"
      alt="outside ignore-region">
    <figcaption>예외구역 밖 사람.</figcaption>
  </figure>
</div>

**기능:** 탐지된 객체의 바운딩 박스가 예외구역에 n% 이상 포함될 경우, 해당 탐지결과를 화면에 표시하지 않음.

**용도:** 지속적인 오탐을 유발하는 객체나 중요하지 않은 배경 영역을 감시구역에서 제외.

**사용 방법:** 카메라 화면을 좌클릭 후 드래그하여 생성. 제거하고 싶은 예외영역은 우클릭하여 삭제.

### 이벤트 발생과 팝업창

<figure style="width: 500px" class="align-left">
  <img
    src="{{ site.url }}{{ site.baseurl }}/assets/images/roka-surveillance/pop-up.png"
    alt="pop-up window">
  <figcaption>팝업창.</figcaption>
</figure>

**기능:** 새로운 '이벤트' 발생 시 팝업창을 통해 사용자에게 알림. 각 카메라의 이벤트는 개별 탭에 표시. 팝업창의 초기 화면은 이벤트가 시작된 순간의 캡처 이미지를 표시.

이벤트는 카메라가 일정 시간 동안 지속적으로 객체를 감지할 때 시작되며, 일정 시간 동안 객체가 감지되지 않아야 종료된다.

**용도:** 감지된 활동을 사용자에게 알림. 객체가 이미 카메라 화면에서 움직여 사라진 후에도 어떤 객체가 이벤트를 발생시켰는지 확인 가능.

**사용 방법:** 사이드바에서 팝업 활성화 여부 및 이벤트 시작/종료 임계값 조정. 팝업의 빨간색 버튼을 클릭하면 해당 카메라의 실시간 영상을 확인 가능.

### 화면 캡쳐

<figure style="width: 500px" class="align-left">
  <img
    src="{{ site.url }}{{ site.baseurl }}/assets/images/roka-surveillance/screen-capture.png"
    alt="screen capture">
  <figcaption>화면 캡처가 저장되면 파란색 테두리가 깜박인다.</figcaption>
</figure>

**기능:** 바운딩 박스가 그려진 이미지를 저장. 원본 이미지와 바운딩 박스 레이블이 포함된 .txt 파일도 함께 저장.

**용도:** 기록/ 보관 및 향후 모델 파인튜닝을 위한 데이터 수집.

**사용 방법:** 카메라 화면을 더블 우클릭하거나 팝업의 파란색 '캡처 저장' 버튼을 클릭. 화면 캡처가 저장되면 파란색 테두리가 깜박인다.

<br><br><br><br>
<a href="{{ site.url }}{{ site.baseurl }}/kr/projects/">← 'Projects'로 돌아가기</a>
