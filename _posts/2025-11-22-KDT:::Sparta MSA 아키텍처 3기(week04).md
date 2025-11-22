---
title: KDT:::Sparta MSA 아키텍처 3기(week04) : Spring AI 활용하기
toc: true
toc_sticky: true
toc_label: "Contents of Page"
categories:
  - Study
tags:
  - MSA_architecture
---

<br><br><br>

# Spring AI 사용하기 Ollama와 Gemini에 대해

| 항목 | **Ollama** | **Gemini API (Google AI)** |
|------|------------|----------------------------|
| 실행 위치 | **내 컴퓨터(로컬)** | **구글 서버(클라우드)** |
| 설치 필요 | 있음 (Ollama 설치 + 모델 다운로드) | 없음 (HTTP API 호출만) |
| 모델 종류 | Llama, Mistral, Phi, Gemma 등 오픈소스 | Gemini 2.0 Flash / Pro / Experimental 등 |
| 비용 | 무료 (로컬 GPU/CPU 사용) | 유료(요금제) 또는 무료 크레딧 존재 |
| 속도 | GPU 성능에 따라 다름 | 클라우드 서버 성능 기반으로 빠름 |
| 보안 | 로컬이라 개인정보 처리에 유리 | 클라우드로 데이터 전송 필요 |
| 품질 | 모델 수준에 따라 다름 | 최신 대규모 모델로 품질 매우 높음 |
| 용도 | 로컬 테스트, 온프레미스 환경, 비용 절감 | 고성능 AI, 대규모 트래픽 처리, 기업 서비스 |

<br>

# Ollama 설치하고 구동하기
* 맥북 사용자 기준 명령어 3줄이면 된다.
* 로컬에 설치해서 쓰는 것이다보니 굉장히 느리며, 성능이 그닥...
  
```
brew install --cask ollama
ollama serve
ollama run llama3
```

<br>

# Gemini 연결
## 1. API 키 발급
* Google AI Studio 에서 api를 발급받는다.

## 2. 의존성 추가 및 설정
* build.gradle 의존성 추가
```Java
dependencies {
    implementation 'com.google.genai:google-genai:1.28.0'
}
```
* 발급받은 api키를 환경변수로 지정(터미널한정)
  - export GEMINI_API_KEY="your_real_key_here"
  - your_real_key_here : api 키로 변경
  - 위 명령어를 입력하면 현재 터미널 세션에서만 사용
* application.yml 파일 내용추가
```
gemini:
  api:
    key: ${GEMINI_API_KEY}
  model:
    name: "gemini-2.5-flash-lite-preview-06-17"
```

## 3. 패키지 경로 구성 및 소스코드

```
@Controller  →  @Service  →  Gemini Client
     ↓               ↓           ↓
HTTP 요청 → 비즈니스 로직 → Google Gemini 호출 → 텍스트 응답

/*------------------------------------*/
springAI_Project ## ✅  Google Gemini 연동에 필요한 모든 핵심 요소가 존재하는 패키지
└── src
    └── main
        ├── java
        │   └── com
        │       └── example
        │           └── springai_project
        │               ├── SpringAiProjectApplication.java ## ✅ 스프링 부트 메인 클래스
        │               │
        │               ├── gemini
        │               │   ├── GeminiConfig.java ## ✅ 시스템 프롬프트 & 모델 설정을 주입받아서 Bean 구성
        │               │   ├── GeminiModelProperties.java ## ✅ 모델명, temperature, maxOutputTokens 값 저장
        │               │   ├── GeminiPromptProperties.java ## ✅ 시스템 프롬프트 (assistant 역할 설명 등)
        │               │   ├── GeminiService.java ## ✅ 실제로 Gemini API 호출 담당
        │               │   ├── GeminiController.java ## ✅ REST API 제공. 서비스에서 받은 결과를 ChatResponse 에 담아 반환
        │               │   │
        │               │   └── dto
        │               │       ├── ChatRequest.java ## ✅ 요청
        │               │       └── ChatResponse.java ## ✅ 응답
        │               │
        │               └── (추가 기능 시 다른 패키지 만들기 가능)
        │
        └── resources ## ✅ 모든 설정 파일 존재
            ├── application.yml ## ✅ API Key 설정 ${GEMINI_API_KEY}
            └── (기타 설정 파일)
```
