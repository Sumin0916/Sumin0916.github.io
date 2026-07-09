---
layout: post
title: Orbit Wars 회고록
date: 2026-06-26 05:30:00
description: Kaggle Orbit Wars에서 구현한 과정과 회고
tags: RL
categories: studies
thumbnail: assets/img/Orbit-Wars/thumb.png
giscus_comments: false
---

<div class="text-center mt-3 mt-md-0">
  <img
    src="{{ '/assets/img/Orbit-Wars/thumb.gif' | relative_url }}"
    class="rounded z-depth-1"
    alt="Orbit Wars gameplay gif"
    style="width: 50%; height: auto;"
  >
</div>

## Orbit Wars 소개

Kaggle Orbit Wars는 참가자들이 제출한 agent들이 행성을 점령하고 함대를 보내며 맵 장악을 겨루는 competition이다.  

처음 이 대회에 참여한 목적은 단순히 높은 순위를 얻는 것보다, 강화학습을 실제 게임 환경에 적용할 때 어디서 어려움이 생기는지 직접 경험해보는 것이었다.  

최종 결과는 **4,729팀 중 321위, Bronze Medal​이다. (Top 6.7%)**
초반에는 6월 26일 기준 440위, 6월 30일 기준 206위까지 올라가기도 했지만, 최종 스코어 수렴 이후에는 321위로 마무리되었다.  

제한된 시간과 GPU 환경에서 behavior cloning, heuristic planning, PPO self-play를 직접 구현하고 실험해볼 수 있었다는 점에서 의미 있는 프로젝트였다.  

---

## 왜 시작했나

이전까지는 강화학습을 간단한 예제로만 접하는 경우가 많았다.
하지만 실제 경쟁 환경에서는 단순히 PPO 코드를 작성하는 것만으로는 부족했다.

Orbit Wars에서는 매 턴마다 어느 행성에서, 어느 행성으로, 몇 척을, 어떤 각도로 보낼지 결정해야 한다.
또 행성은 계속 움직이고, 함대는 속도와 경로에 따라 도착 시간이 달라지며, 상대도 동시에 행동한다.

이런 환경에서 강화학습이 어려워지는 이유를 직접 보고 싶었다.
특히 action space 설계, sparse reward, self-play 불안정성, rollout 처리량, 그리고 강한 휴리스틱 에이전트와 학습 기반 정책 사이의 차이를 경험하는 것이 목표였다.

---

## 처음 접근: 휴리스틱 플래너

처음에는 [The Producer](https://www.kaggle.com/code/slawekbiel/the-producer-agent)를 바탕으로 한 휴리스틱 에이전트에서 출발했다. 시험 기간과 겹쳐 절대적인 시간이 부족했기 때문에, 행성 이동 예측, 발사 각도 계산, 도착 시간 추정 등 기본적인 코드를 최대한 빠르게 활용하고 싶었다.

이 플래너는 observation을 받은 뒤, 미래 행성 위치와 함대 도착을 예측하고, 물리적으로 발사 가능한 source-target 후보를 만든다.
각 후보에 대해 ETA, 도착 가능성, capture threshold, safe drain, reinforcement risk 등을 계산한 뒤, flow-based score를 이용해 좋은 후보를 선택한다.

이 방식은 안정적이었다.
각도 계산이나 도착 가능성 같은 물리적인 부분은 deterministic하게 처리할 수 있었고, 잘못된 action을 줄이기에도 좋았다.

이후에는 상위권 replay를 이용해 behavior cloning을 시도했다.
목표는 수작업으로 설계한 score만으로는 부족한 후보 선택 감각을 모델이 보완하게 만드는 것이었다.

---

## PPO를 어디에 넣었나

처음 PPO를 적용할 때는 매우 보수적으로 접근했다.
PPO 학습도 처음이었고, 사용할 수 있는 GPU도 RTX 3060 Ti 한 장이었기 때문에, 완전히 end-to-end policy를 학습시키는 것은 어렵다고 판단했다.

그래서 기존 플래너 구조는 유지하고, 물리적으로 가능한 launch candidate가 만들어진 뒤에 BC/PPO 모델을 넣었다. PPO 학습은 PyTorch 기반으로 구현했고, behavior cloning으로 얻은 checkpoint를 초기 policy로 사용한 뒤 self-play rollout을 통해 policy를 업데이트하는 방식으로 구성했다.

{% include figure.liquid path="assets/img/Orbit-Wars/Pipeline.png" class="img-fluid rounded z-depth-1" caption="Producer 파이프라인에 BC/PPO candidate scoring 단계를 추가한 구조." %}

전체 흐름은 대략 다음과 같았다.

1. observation 입력
2. 미래 행성 상태와 함대 도착 예측
3. 발사 가능한 source-target 후보 생성
4. BC/PPO 모델이 후보 점수 계산
5. 높은 점수의 후보를 greedy하게 선택
6. 남은 함선은 regroup에 사용
7. 최종 action 출력

즉, PPO 모델은 전체 action을 직접 만드는 것이 아니라, 이미 생성된 후보군을 scoring하거나 reranking하는 역할에 가까웠다.

이 구조는 테스트하기 쉽고 안정적이었다.
하지만 시간이 지나면서 한계도 분명해졌다.

---

## 막혔던 지점

대회 후반으로 갈수록 단순히 휴리스틱 score를 더 정교하게 만들거나, BC 모델을 조금 더 학습시키는 것만으로는 순위가 크게 오르지 않았다.

이유를 돌아보면, PPO 모델이 실제로 배울 수 있는 범위가 너무 좁았다.
플래너가 이미 후보군을 제한했기 때문에, 모델은 새로운 전략을 발견하기보다는 기존 휴리스틱이 제안한 행동들 사이에서 순서를 바꾸는 역할에 머물렀다.

처음에는 이 정도만으로도 성능 향상을 기대했다.
하지만 대회 종료 후 생각해보니, 이런 방식은 강화학습의 장점 중 하나인 handcrafted rule을 넘어서는 행동을 학습하는 능력을 충분히 살리기 어려웠다.

특히 많은 참가자가 사용하던 Producer-style agent를 큰 차이로 이기기 위해서는, 단순히 기존 파이프라인 끝에 PPO scorer를 붙이는 것만으로는 부족했을 것 같다.

---

## 상위 RL 솔루션 복기

대회가 끝난 뒤, 나와 비슷하게 제한된 GPU 환경에서 PPO를 시도한 상위 랭커들의 글을 읽었다.

가장 인상 깊었던 차이는 모델 크기나 학습 시간보다, “학습이 파이프라인의 어디에 놓였는가”였다.

그들은 복잡한 action space를 그대로 두지 않았다.
오히려 각 행성이 target으로 full-send를 할지, 아니면 no-op을 할지처럼 action space를 단순화했다. 표현력은 줄어들지만, PPO가 실제로 학습하고 디버깅하기 쉬운 형태를 먼저 만든 것이다.

또한 내가 휴리스틱 내부에서 사용하던 미래 projection 정보를 모델 입력으로 직접 제공했다.
예를 들어 어떤 행성이 몇 턴 뒤에 적에게 넘어가는지, 도착 시점에 병력이 얼마나 되는지 같은 정보를 policy가 직접 볼 수 있게 했다.

나는 이런 정보를 주로 score 계산에 사용했지만, 상위 솔루션들은 모델이 timing-sensitive decision을 직접 학습할 수 있도록 feature로 넣었다.

이러한 차이가 크게 느껴졌다.

---

## 배운 점

중요했던 것은 다음과 같았다.

* PPO가 학습하기 쉬운 action representation을 설계하는 것
* 미래 상태 예측을 휴리스틱 score에만 쓰지 않고 policy input으로 제공하는 것
* rollout 처리량을 높이는 것
* 어떤 부분은 deterministic하게 계산하고, 어떤 부분은 policy가 배우게 할지 나누는 것

처음에는 강한 휴리스틱 플래너 위에 모델을 얹으면 자연스럽게 성능이 오를 것이라고 생각했다.
하지만 실제로는, policy가 충분한 결정권을 가지지 못하면 PPO가 새로운 전략을 배우기 어렵다는 것을 알게 되었다.

---

## 마무리

“좋은 휴리스틱 에이전트”와 “학습 가능한 정책” 사이에는 생각보다 큰 차이가 있다는 것을 배웠다.
휴리스틱은 안정적이고 해석 가능하지만, 그 구조가 너무 강하면 모델이 배울 수 있는 공간을 제한할 수 있다.

다시 이 프로젝트를 한다면, 물리 계산과 미래 상태 시뮬레이션은 deterministic하게 유지하되, source-target 의사결정 자체는 policy가 더 직접적으로 다루도록 설계해보고 싶다.
그리고 PPO를 단순한 후보 scorer가 아니라, 더 명확한 action space 위에서 전략을 학습하는 정책으로 만들고 싶다.
