# 📈 InvesTube

> **투자 영상과 기업 재무 정보를 한곳에서 살펴보세요.**

InvesTube는 **투자 관련 유튜브 영상·커뮤니티·기업 재무 분석을 함께 제공하는 웹 플랫폼**입니다. 관심 영상과 종목을 관리하고, 투자 성향에 따른 재무 점수와 AI 설명을 확인하며, 챗봇으로 종목 데이터에 관한 질문을 할 수 있습니다.

| 항목 | 내용 |
| --- | --- |
| 개발 기간 | 2025.11.21 ~ 2025.12.26 |
| 개발 인원 | 2명 |
| 플랫폼 | Web |
| 기관 | 삼성 청년 SW·AI 아카데미 14기 |

## 📋 목차

- [🧩 주요 기능](#-주요-기능)
- [🧰 기술 스택](#-기술-스택)
- [🧱 시스템 아키텍처](#-시스템-아키텍처)
- [🔧 핵심 구현](#-핵심-구현)
- [🤝 팀 구성 및 역할](#-팀-구성-및-역할)
- [🚀 시작하기](#-시작하기)
- [📚 관련 문서](#-관련-문서)

## 🧩 주요 기능

| 기능 | 설명 |
| --- | --- |
| 투자 영상 탐색 및 등록 | 영상을 검색·정렬·분류하고 찜과 리뷰를 관리합니다. 등록 시 AI가 메타데이터를 바탕으로 투자 관련성과 카테고리를 판단합니다. |
| 커뮤니티 | 게시글·댓글 작성과 수정, 이미지 업로드, 조회수 및 댓글 수 표시를 지원합니다. |
| 종목 정보 및 관심 종목 | 종목별 가격 이력·차트·관련 뉴스를 확인하고 관심 종목을 관리합니다. |
| 투자 성향 및 재무 분석 | 설문으로 투자 성향을 설정하고, 재무 지표 점수·동종업계 비교·AI 분석 요약을 조회합니다. |
| AI 챗봇 | 선택한 종목의 재무 데이터와 투자 성향을 문맥에 포함해 질문에 답합니다. |
| 프로필 및 알림 | 프로필·팔로우·내 활동을 관리하고, 댓글·리뷰·팔로우·영상 찜 알림과 수신 설정을 제공합니다. |

## 🧰 기술 스택

| 분야 | 주요 기술 |
| --- | --- |
| Frontend | JavaScript, Vue 3, Vite, Vue Router, Tailwind CSS |
| 상태 / 통신 / 차트 | Pinia, Axios, Chart.js |
| Backend | Java 17, Spring Boot, MyBatis, JWT |
| 데이터베이스 | MySQL |
| AI | GMS 연동 GPT-4.1-mini, Gemini 2.5 Flash |
| 외부 데이터 | YouTube Data API, DART, KRX, 네이버 뉴스 검색 API |
| 개발 도구 | Maven, Swagger / OpenAPI, ESLint, Prettier |

세부 의존성과 버전 범위는 [Frontend package.json](./Front/package.json), [Backend pom.xml](./Back/pom.xml)을 참고합니다.

## 🧱 시스템 아키텍처

Vue 프론트엔드가 Spring Boot API를 호출하고, 백엔드에서 인증·콘텐츠·재무 데이터·외부 API를 처리합니다. MyBatis로 MySQL 데이터를 조회·저장하며, 주가 이력과 재무 데이터는 파일 캐시도 활용합니다.

```text
[사용자]
Vue / Vue Router / Pinia
  └─ Axios + JWT → Spring Boot REST API
                       ├─ MyBatis → MySQL
                       │   사용자·영상·게시글·종목·투자 성향·알림
                       ├─ YouTube API → 영상 메타데이터
                       ├─ DART / KRX / 파일 캐시 → 재무·종목 데이터
                       ├─ 네이버 API → 관련 뉴스
                       └─ GMS
                            ├─ GPT → 영상 분류·챗봇 응답
                            └─ Gemini → 재무 분석 보정·설명
```

```text
InvesTube/
├── Front/
│   └── src/
│       ├── api/         # 도메인별 API 호출 및 공통 HTTP 처리
│       ├── views/       # 영상·커뮤니티·종목·마이페이지 화면
│       ├── components/  # 헤더·알림·챗봇 등 UI 컴포넌트
│       └── stores/      # 인증·사용자·챗봇 상태
└── Back/
    ├── src/main/java/   # REST API, 서비스, 데이터 접근, 외부 API 연동
    ├── src/main/resources/
    │   ├── mappers/    # MyBatis SQL 매퍼
    │   └── schema.sql  # DB 스키마 및 초기 데이터
    ├── data/           # 주가 JSON 및 재무·업종 CSV 데이터
    └── uploads/        # 업로드 파일
```

## 🔧 핵심 구현

### 🎬 메타데이터 기반 영상 검토와 분류

영상 등록 시 YouTube의 제목·설명·채널·태그와 사용자 입력을 바탕으로 AI 검토를 요청합니다. 응답을 JSON으로 파싱하고, 등록 허용 여부·판단 이유·추천 카테고리를 정규화합니다.

영상 자체의 내용이나 투자 주장의 사실성을 검증하는 기능과는 구분되며, 메타데이터를 이용한 주제 분류에 초점을 둡니다.

### 📊 규칙 기반 재무 점수와 AI 보정

재무 지표와 투자 성향별 가중치로 기본 점수를 계산하고, 업종 및 동종업계 통계를 포함한 문맥을 Gemini에 전달합니다. AI 보정 점수는 -10~+10, 최종 점수는 0~100 범위로 제한합니다.

AI 호출 또는 응답 처리에 실패하면 기본 점수 기반 결과를 반환하도록 구성했습니다. 종목·재무 데이터는 API와 저장된 JSON·CSV를 함께 사용하므로 조회 시점의 실시간 시세를 보장하는 구조는 아닙니다.

### 🤖 종목 문맥을 반영한 챗봇

현재 종목의 기본 정보·재무 지표·투자 성향을 질문과 함께 전달합니다. 입력 길이가 길어지면 주요 지표 중심으로 문맥을 축약하며, 프롬프트에서 데이터 해석을 중심으로 답하고 단정적인 매수·매도 권유를 피하도록 지시합니다.

프론트엔드는 챗봇 상태를 공유하고, 응답에 포함된 감정 태그를 마스코트 표현과 연결합니다.

### 🔐 인증 상태와 사용자 활동 연결

공통 Axios 클라이언트에서 JWT를 요청에 첨부하고, 401 응답 시 저장된 인증 정보를 정리한 뒤 로그인 화면으로 이동합니다.

마이페이지는 영상·게시글·댓글·관심 종목 등 사용자 활동을 모아 보여줍니다. 댓글·리뷰·팔로우·찜 이벤트는 알림 조회·읽음 처리 및 사용자별 수신 설정과 연결합니다.

## 🤝 팀 구성 및 역할

저장소의 구현 및 작성자별 커밋 기록을 기준으로 주요 기여를 요약했습니다.

| 이름 | 담당 | 주요 기여 |
| --- | --- | --- |
| 김건남 | Full Stack | 영상·리뷰 및 커뮤니티 기능, 재무 분석·투자 성향, AI 챗봇 |
| 김태영 | Full Stack | 인증·마이페이지 및 알림 설정, 영상 AI 검토·분류, 종목 데이터·뉴스 연동 |

## 🚀 시작하기

### 개발 환경

- Java 17, MySQL
- Node.js `^20.19.0` 또는 `>=22.12.0` (Frontend 설정 기준)
- 사용하는 기능에 맞는 YouTube·DART·KRX·네이버·GMS API 설정

아래 명령은 각각 **저장소 루트에서 시작하는 별도 터미널** 기준입니다.

### 데이터베이스

[스키마 파일](./Back/src/main/resources/schema.sql)로 로컬 개발 DB와 초기 데이터를 구성합니다. **이 파일에는 기존 `investube` 데이터베이스를 삭제하고 재생성하는 구문이 있으므로, 보존할 데이터가 있는 환경에서는 그대로 실행하지 않습니다.**

DB 연결 주소·사용자·비밀번호는 `Back/src/main/resources/application.properties`의 `spring.datasource.*` 설정을 로컬 환경에 맞춥니다.

### Backend

`Back/.env.example`을 `Back/.env`로 복사하고 아래 값을 설정합니다. `application.properties`에서 해당 파일을 읽도록 구성되어 있습니다.

| 환경 변수 | 용도 |
| --- | --- |
| `YOUTUBE_API_KEY` | 영상 메타데이터 조회 |
| `DART_API_KEY` | 기업 재무 데이터 조회 |
| `KRX_API_KEY` | 종목 데이터 연동 |
| `NAVER_CLIENT_ID`, `NAVER_CLIENT_SECRET` | 관련 뉴스 검색 |
| `GMS_ENDPOINT_URL`, `GMS_API_KEY` | 영상 검토·챗봇·재무 분석 |

```bash
cd Back
./mvnw spring-boot:run
```

Windows PowerShell에서는 `./mvnw` 대신 `.\mvnw.cmd`를 사용합니다. 기본 API 주소는 `http://localhost:8080`입니다.

애플리케이션 시작 시 종목 및 재무 데이터를 로드합니다. 파일 캐시의 기준 경로가 `Back/data/`이므로 `Back/`에서 실행하며, 데이터 로딩과 외부 API 연결 상태는 서버 로그에서 확인할 수 있습니다.

### Frontend

기본 API 주소는 `http://localhost:8080`입니다. 다른 주소를 사용한다면 `Front/.env.local`에 다음과 같이 설정합니다.

```dotenv
VITE_API_BASE_URL=http://localhost:8080
```

```bash
cd Front
npm install
npm run dev
```

터미널에 표시된 Vite 개발 서버 주소로 접속합니다. 배포용 빌드는 `npm run build`로 생성합니다.

## 📚 관련 문서

| 문서 | 내용 |
| --- | --- |
| [Frontend](./Front/README.md) | Vue·Vite 개발 및 빌드 명령어 |
| [Backend](./Back/README.md) | 기존 백엔드 개요 및 설계 자료 |
| [DB 스키마](./Back/src/main/resources/schema.sql) | 테이블 구조 및 초기 데이터 |
| [환경 변수 예시](./Back/.env.example) | 외부 API 연결 설정 |
