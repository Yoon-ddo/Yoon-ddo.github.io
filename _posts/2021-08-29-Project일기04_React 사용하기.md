---
title: Project일기04_React 사용하기
toc: true
toc_sticky: true
toc_label: "Contents of Page"
categories:
  - Project
tags:
  - Web_Project
---

<br><br>

# React란?
* [참고블로그](https://velog.io/@jini_eun/React-React.js%EB%9E%80-%EA%B0%84%EB%8B%A8-%EC%A0%95%EB%A6%AC)
* 웹 프레임워크
* 자바스크립트 라이브러리의 하나로서 사용자 인터페이스를 만들기 위해 사용된다.
* facebook에서 제공하는 프론트엔드 라이브러리
* 현재 많이 활용되는 웹,앱의 View를 개발할 수 있도록 하는 요즘 핫!한 라이브러리이다.

## 장점
* React를 사용하지 않아도 프론트를 구성할 수 있다.
* React를 이용하여 사용자와 상요작용가능한 동적 UI를 쉽게 만들 수 있기 때문에 많이 이용됨

## 특징
* 단방향 Data Flow
  - 대규모 애플리케이션일수록 데이터 흐름을 추적하기 어렵다.
  - 데이터 흐름에서 일어나는 변화를 예측할 수 있도록 단방향 흐름을 가지도록 함.
* Component 기반 구조
  - 소프트웨어를 독립적인 하나의 부품으로 만드는 방법.
  - View를 여러 컴포턴트를 쪼개서 만든다.
  - 여러 부분을 독립된 컴포넌트로 만들고, 이 컴포넌트를 조립해 화면을 구성한다.
  - 애플리케이션이 복잡해지더라도 코드의 유지보수, 관리가 용이해진다.
* Virtual DOM
  - 가상의 Document Object Model
  - html, xml, css를 트리 구조로 인식하고 데이터를 객체로 간주
  - 변경이 필요한 최소한의 변경사항만 실제 DOM에 반영해 효율성과 속도를 개선
* Props and State
  - Props
    + 부모컴포넌트에서 자식 컴포넌트로 전달해 주는 데이터.
    + 읽기 전용 데이터
    + 변경 불가능. 최상위 부모 컴포넌트만 변경가능
  - State
    + 컴포넌트 내부에서 선언
    + 내부에서 값 변경 가능
    + 동적 데이터를 다룰때 사용.
    + 클래스형 컴포넌트에서만 사용할 수 있고, 각각의 State는 독립적 
* JSX
  - JSX의 사용은 필수는 아니지만 React에서 사용됨
  - JavaScript의 확장문법

<br><br>

# 환경세팅하기
## 1. Node.js 설치
1. [Node.js](https://nodejs.org/en/) 사이트에 접속하여 `LTS`버전 다운로드
2. 계속 `Next`, 체크해야하는거 모두 `체크`, 권한부여 `예`
3. 설치완료 후 cmd 창 새로 열어서 `node -v` 입력했을 때 버전이 나오면 설치 완료

## 2. NPM 설치
1. `npm -v`로 npm 버전 확인

## 3. 최신 버전의 Node.js 설치
1. `npm install npx` 입력하여 설치
2. `npx -v` 입력하여 버전 확인

## 4. Visual Studio Code 설치
1. [Visual Studio Code 다운로드](https://code.visualstudio.com/docs/?dv=win)
2. 다음 누르고 PATH에 추가는 체크하고 계속 다음 누르기

## 5. React 참고문서
* [Reactjs](https://ko.reactjs.org/docs/getting-started.html)








<br><br><br><br>
