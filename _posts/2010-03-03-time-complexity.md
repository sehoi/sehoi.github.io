---
layout: single
classes: wide
title: "시간 복잡도(Time-Complexity) 정리"
date: 2014-03-03
description: "시간 복잡도(Time-Complexity)의 분석(analysis) 및 함수(function) 종류에 대해서 정리한 글입니다."
categories:
  - Etc
tags: [etc, time-complexity, complexity]
comments: true
share: true
---

시간 복잡도(Time-Complexity)의 분석(analysis) 및 함수(function) 종류에 대해서 정리한 글입니다.

#### 시간 복잡도 분석
- 입력크기에 따라서 단위연산이 몇 번 수행되는지 결정하는 것
<br /><br />

#### 시간 복잡도 분석 종류
1. Every-case analysis: T(n)
  - 입력크기(input size)에만 종속
  - 입력값과는 무관하게 결과값은 항상 일정
<br />

2. Worst-case analysis: W(n)
  - 입력크기와 입력값 모두에 종속
  - 단위연산이 수행되는 횟수가 최대인 경우 선택
<br />

3. Average-case analysis: A(n)
  - 입력크기와 입력값 모두에 종속
  - 모든 입력에 대해서 단위연산이 수행되는 기대치(평균)
  - 각 입력에 대해서 확률 할당 가능
<br />

4. Best-case analysis: B(n)
  - 입력크기와 입력값 모두에 종속
  - 단위연산이 수행되는 횟수가 최소인 경우 선택
<br /><br />

#### 시간 복잡도 함수 종류
1. 시간 복잡도 함수의 차수
  - O(lgn) < O(n) < O(nlgn) < O(n²) < O(n³) <  O(2ⁿ) <  O(n!)
<br />

2. Big O 표기법
  - 정의: 점근적 상한(Asymptotic Upper Bound)
  - g(n) ∈ O(f(n))
    - g(n)은 아무리 나빠도 f(n)보다 같거나 좋은 시간 복잡도를 가짐
    - => 즉, O(f(n))은 g(n)의 worst-case임
<br />

3. Ω 표기법
  - 정의: 점근적 하한(Asymptotic Lower Bound)
  - g(n) ∈ Ω(f(n))
    - g(n)은 아무리 빨라도 f(n)밖에 되지 않음
    - => 즉,  Ω(f(n))은 g(n)의 best-case임
<br />

4. Θ 표기법
  - 정의: 점근적 하한(Asymptotic Tight Bound)
  - Θ(f(n)) = O(f(n)) ∩ Ω(f(n))
  - g(n) ∈ Θ(f(n))
    - g(n)은 f(n)의 차수(order)라고 함
    - => 즉, Θ(f(n))은 g(n)의 average-case임
<br />

5. Small o 표기법
  - g(n) ∈ o(f(n))
    - g(n)이 궁극적으로 f(n)보다 훨씬 좋다는 의미
  - if g(n) ∈ o(f(n)) then, g(n) ∈ O(f(n)) - Ω(f(n))
    - => 즉, g(n)은 O(f(n))에 속하지만 Ω(f(n))에는 속하지 않음
