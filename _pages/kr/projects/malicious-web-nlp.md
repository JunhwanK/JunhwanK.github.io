---
title: NLP를 활용한 악성 웹요청 분류
lang: kr
layout: single
author_profile: true
permalink: /kr/projects/malicious-web-nlp/
classes: wide
github:
---

# NLP를 활용한 악성 웹요청 분류

악성 웹요청을 9개의 클래스 중 하나로 분류하는 인공신경망 모델. {% if page.github %} <a href="{{ page.github }}">[GitHub 링크]</a> {% endif %}

## 주요 성과

- BERT 모델을 구현하여 4.5만 개의 악성 웹요청 로그에 학습.
- 학습한 BERT 임베딩에 랜덤 포레스트 분류기를 적용하여 85.6%의 정확도와 0.90 매크로 F1 점수를 달성.

## 목적

악성 웹요청 로그(web application firewall log)를 9개의 악성 요청 종류 중 하나로 분류한다.

악성 요청 종류: HOST_Scan, SQL_Injection, Path_Disclosure, Vulnerability_Scan, Leakage_Through_NW, Directory_Indexing, System_Cmd_Execution, Cross_Site_Scripting, and Automatically_Searching_Infor.

**데이터셋**: 4.5만 개 악성 웹요청 로그 (70% 학습, 20% 검증, 10% 테스트)

**입력값 예시**:

- **HOST_Scan**: "GET /boaform/admin/formLogin?username=user&psd=user HTTP/1.0\r\n\r\n"
- **Cross_Site_Scripting**: "GET /board/board_view?code=%3Cscript%3Eprompt(document.cookie)%3C/script%3E HTTP/1.1\r\nHost: <www.college.school\r\nAccept-Encoding>: identity\r\nCookie: ID=1094200543; designart_site=lbtbqr99b9n4vr0e2en2p5eoh83idq5i\r\nUser-Agent: python-urllib3/1.26.9\r\n\r\n"

## 방법

먼저 로그 입력값의 전처리하고 토큰화 한다. 그다음, 토큰화된 로그를 분류하는 BERT 모델을 학습시킨다.
BERT 모델이 학습한 임베딩을 가지고, 최종적으로 랜덤 포레스트 분류기를 학습시키고 추론에 사용한다.

### 전처리

전처리 과정은 다음 같다.

- 문자열을 소문자 변환 (casefold).
- "\r\n"를 공백으로 변환.
- url 인코딩을 해독, 예시. '%40'을 다시 '@'으로 변환.
- 모든 기호와 특수문자 앞뒤로 공백 추가.

**전처리 전**: "GET /board/board_view?code=%3Cscript%3Eprompt(document.cookie)%3C/script%3E HTTP/1.1\r\n"

**전처리 후**: "get  / board / board _ view ? code =  < script > prompt ( document . cookie )  <  / script >  http / 1 . 1 "

### 토큰화

공백을 사용해 전처리된 입력값을 토큰화한다.

**토큰화 전**: "get  / board / board _ view ? code =  < script > prompt ( document . cookie )  <  / script >  http / 1 . 1 "

**토큰화 후**: ["get", "/", "board", "/", "board", "_", "view", "?", "code", "=", "<", "script", ">", "prompt", "(", "document", ".", "cookie", ")", "<", "/", "script", ">", "http", "/", "1", ".", "1"]

### BERT 모델

토큰화된 로그를 분류하는 BERT 모델을 학습시킨다.

<!-- <figure style="width: 1000px" class="align-center">
  <img
    src="{{ site.url }}{{ site.baseurl }}/assets/images/malicious-web-nlp/model.jpg"
    alt="model architecture">
  <figcaption>Model architecture. FIXME</figcaption>
</figure> -->

### 랜덤 포레스트 분류기

BERT 모델이 학습한 임베딩을 가지고, 최종적으로 랜덤 포레스트 분류기를 학습시키고 추론에 사용한다.

## 결과

- 85.6% 정확도
- 0.90 매크로 F1 점수

<!-- #### t-SNE를 사용한 BERT 임베딩 시각화

<figure style="width: 1000px" class="align-center">
  <img
    src="{{ site.url }}{{ site.baseurl }}/assets/images/malicious-web-nlp/embedding-visualization.jpg"
    alt="embedding-visualization">
  <figcaption>t-SNE visualization of BERT embedding.</figcaption>
</figure> -->

## 제한점 및 개선방안

- 가장 큰 문제는 모델이 아무런 도메인 지식을 활용하지 않는다는 것이다. 예를 들어, `<script>` 태그를 포함하는 웹요청은 cross-site scripting 공격일 확률이 높다.
- 도메인 지식을 통해 추출한 룰 기반 피쳐값을 BERT 임베딩과 함께 사용한다면 성능이 향상될 수도 있다.
- 코드에 알맞는 불용어(stopwords) 제거나 토큰화 기법을 사용한다면 모델 성능이 향상될 수도 있다.

<br><br>
<a href="{{ site.url }}{{ site.baseurl }}/en/projects/">← Back to 'Projects'</a>
