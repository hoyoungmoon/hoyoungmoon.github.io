---
title: "[공익인간] 첫 사이드 프로젝트 리뷰"
date: 2020-05-23 22:31:28 -0400
categories: side-project android java
---

## 시작 동기

학기중에 공부하기 귀찮았던 c언어 공부를 하며 프로그래밍에 관심을 가지게 되었다.
몇 개월간 책으로만 코딩테스트로만 공부하는게 지쳐 간단한 거라도 만들어 보면서 배우기로 결심했다.

프로젝트의 목표는 대략

- 일상생활에서 겪는 문제를 해결해보자
- c언어만이 아닌 객체지향언어에 대해 경험해보자
- 필요에 의해서 찾아 배우는 공부를 해보자

통합해본 결과 간단한 안드로이드 어플리케이션을 만들기로 했다.
언어는 객체지향의 대표언어라고 할 수 있는 자바를 선택했고, android studio에서 개발환경을 세팅하였다.

당시 일상생활에서 겪는 문제하면 가장 먼저 생각났던 것이 **사회복무요원 2020년 월급 인상**이였다.
기존에 완성도 높은 군인을 위한 어플리케이션은 많았다. 하지만 사회복무요원의 특성상 군인과는 달리

- 출퇴근을 하며, 휴가(1년차, 2년차, 병가)를 사용할 수 있다.
- 휴가 사용에 따라 식비, 교통비 유뮤가 결정된다.

이러한 특징과 더불어 2020년이 새해가 되면 월급 인상과 복무일 단축에 의해 진급일 또한 바뀌게 되면서 내가 언제 얼마를 받아야하는지 굉장히 헷갈리는 기간이 다가오고 있었다. 이를 해결할 수 있는 앱을 개발할 수 있도록 기획을 시작하였다.

## 개발 과정

#### 기획 단계

초기에 기획을 할 땐 fragment를 3개 정도 만들어 설정창, 휴가 정보, 복무일 들을 나누어서 넣기로 하였다.
그러나 계속해서 구성을 해본 결과 내가 만들고자 하는 앱이 그렇게 많은 기능을 담고 있지 않다는 것을 깨달았다.
사회복무라는 것은 원해서 자발적으로 하고 있는 사람은 단 한명도 없기 때문에 복무 관리를 해주는 **앱이 복잡하게 여러번 클릭해야하면
사용하지 않을 것 같다**는 생각이 들었다. 그래서 수정한 컨셉은

> 한 화면에 모든 정보가 보이도록 하되, 클릭을 했을 때 자세한 정보를 알 수 있도록 하자

이렇게 수정 한 결과 한 화면에 정보를 압축해서 담을 수 있게 되었고 컨셉에 맞게 개발을 진행할 수 있었다.

## 수정 과정

#### 객체지향 프로그래밍

## 처음 사이드 프로젝트를 하며

솔직히 공부를 하고 있는 상황에 사이드 프로젝트라고 말하기엔 적합하지 않은 것 같다.
누군가 쓸 수 있는 프로그램을 하나 끝까지 만들어 보면서 예상하지 못한 문제들과 마주쳤을 때 해결할 수 있는 공부를 한 것 같다.

## 참고한 도서

#### 이것이 안드로이드다

#### 오브젝트