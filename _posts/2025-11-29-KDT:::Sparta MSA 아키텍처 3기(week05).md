---
title: KDT:::Sparta MSA 아키텍처 3기(week05)
toc: true
toc_sticky: true
toc_label: "Contents of Page"
categories:
  - Study
tags:
  - MSA_architecture
---

<br><br><br>

# 바이브 코딩과 MCP
## RAG(Retrieval-Augmented Generation)
* RAG는 생성형 AI가 외부 지식 저장소(예: 데이터베이스, 문서, 내부 자료 등)에서 정보를 검색(Retrieval)하여 답변 생성(Generation)에 반영하는 기술
* 모델이 원래 알고 있던 지식 + 최신/전문 정보”를 합쳐 더 정확한 답변을 생성하는 방식
* ✨ 왜 RAG를 쓰는가?
  1) 할루시네이션(허위 정보 생성) 감소
     - 모델이 모르는 내용을 지어내지 않고, 검색된 실제 문서를 근거로 답변합니다.
  2) 최신 정보 반영 가능
     - 모델이 2024년까지 배웠어도, RAG를 쓰면 2025년 자료도 검색해 답변할 수 있어요.
  3) 도메인 특화 지식 활용
     - 의료, 금융, 은행 내 문서 등 특정 조직의 사내 자료도 연결 가능 →
     - 은행(예: 카카오뱅크) 백엔드나 CRM에 RAG 도입 시 매우 유용
## Vector DB와 임베딩
* 일반 db는 like검색 또는 = 일치검색
* vector db는 의미기반.유사한 키워드의 정보 검색
* **PostgreSQL + pgvector** 조합을 추천
  - 기존 PostgreSQL에 extension으로 추가
  - 별도 인프라 불필요
  - ACID 트랜잭션 지원
  - 익숙한 SQL 사용
  - 무료 오픈소스

## Function Calling
* LLM(대형 언어 모델)이 외부 함수나 API를 자동으로 호출해 결과를 활용할 수 있게 하는 기술
* 왜 사용할까?
  - LLM은 자연어 생성에는 뛰어나지만, 정확한 구조화 데이터(JSON 등) 생성. 특정 기능 실행(검색, 계산, DB 조회 등) 에는 한계가 있습니다.
* function calling을 사용하면:
  - 모델이 사용자의 요청을 분석해 적절한 함수 이름과 파라미터를 JSON 형식으로 만들어 외부 시스템이 그 함수를 실제로 실행하고 모델이 결과를 다시 사용자에게 전달합니다.
* 장점
  - ✅ 1. 정확한 데이터 전달. LLM이 문장 내 숫자나 형식을 실수로 바꾸는 문제를 줄임.
  - ✅ 2. 외부 시스템과 연결 가능. DB 조회, 계산, API 호출 등 다양한 기능 확장.
  - ✅ 3. 자연어 → 구조화 데이터 변환. 자연어 입력을 자동으로 JSON 요청으로 변환 가능.
  - ✅ 4. 높은 확장성. 챗봇, 자동화 서비스, 에이전트 개발에 매우 적합.

## 프로젝트 해보기

```
mcpProject/                         # Spring Boot + Docker 기반 프로젝트 루트
├─ .gradle/                         # Gradle 캐시 및 내부 빌드 데이터
├─ .idea/                           # IntelliJ IDEA 프로젝트 설정
├─ build/                           # 빌드 결과물(JAR 파일 등) 출력 디렉토리
├─ gradle/                          # Gradle wrapper 실행 관련 파일
├─ src/
│  └─ main/
│     ├─ java/
│     │  └─ com/example/mcpproject/
│     │     ├─ McpProjectApplication.java   # Spring Boot 메인 실행 클래스
│     │     ├─ controller/
│     │     │  ├─ ChatController.java       # /v1/chat/completions API (LLM 호출용)
│     │     │  └─ ModelController.java      # /v1/models API (모델 목록 제공)
│     │     └─ service/
│     │        └─ OllamaChatService.java    # Ollama 서버와 HTTP 통신 담당 서비스
│     └─ resources/
│        ├─ static/                         # 정적 파일(css, js) 위치
│        ├─ templates/                      # 템플릿 엔진 사용 시 HTML 위치
│        └─ application.properties          # Spring Boot 환경 설정 파일
├─ src/test/java/...                        # JUnit 테스트 코드
├─ Dockerfile                               # Spring API 컨테이너 이미지 빌드 설정
├─ docker-compose.yml                       # 전체 스택(Postgres, Ollama, Spring, WebUI) 구성
├─ build.gradle                             # 프로젝트 빌드 설정 (의존성, 플러그인 등)
├─ settings.gradle                          # Gradle 프로젝트 이름/설정
├─ gradlew                                  # Gradle wrapper 실행 스크립트 (Mac/Linux)
├─ gradlew.bat                              # Gradle wrapper 실행 스크립트 (Windows)
├─ .gitignore                               # Git에서 제외할 파일 목록
└─ .gitattributes                           # Git 파일 속성 설정
```
