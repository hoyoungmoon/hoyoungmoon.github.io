---
title: "React 자습"
date: 2022-03-01
categories: React
---

# React 매일 공부하기

## React를 사용하는 이유

현재 웹페이지는 주로 web server의 data를 동적으로 변경하는 방식이 대부분이다. 기존의 페이지 전체의 html 코드를 받아와 다시 로드하는 방식과 다르다. 이렇게 달라질 수 있었던 큰 요인 중 하나는 Ajax의 등장이다.

**Ajax (Asynchronous Javascript and XML)** 는 브라우저가 서버와 비동기적으로 통신하여 다양한 포멧(JSON, XML, HTML)들을 주고 받는 통신 기능이다. XMLHttpRequest라는 객체를 이용하여 주고 받으며 이는 서버로 보내는 HTTP request 기능을 제공하는 객체이다.

점차 웹사이트가 성능이 고도화 됨에 따라 관리해줘야하는 상태들이 많아졌다. 이를 동적으로 동작하게 하기 위해서는 수많은 DOM 요소들을 직접 관리해야한다. 좋은 코드 컨벤션이 있으면 물론 가능한 일이지만 효율적인 DOM조작이 되지 않는다면 성능 저하로 이어질 수 있다. 이러한 DOM관리와 상태값 업데이트에 드는 개발 비용을 최소화하고 기능, 사용자 인터페이스 구현에 집중할 수 있도록 하는 라이브러리이다.

## Virtual DOM

그렇다면 React는 DOM 조작을 어떤 방식으로 하고 있을까?

- DOM이란

## props, state

## 클래스형, 함수형 컴포넌트

## Pure component

## 생명 주기 메서드

## JSX

## ContextAPI

## useEffect 실행순서
