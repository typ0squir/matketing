# 맡케팅

> 이제 맡겨주세요, 마케팅

SNS 마케팅이 낯선 50·60대 소상공인을 위한 AI 기반 인스타그램 콘텐츠 제작·게시 서비스입니다. 매장 정보 수집부터 콘텐츠 생성, 사용자 확인 후 게시, 성과 조회까지 하나의 흐름으로 연결했습니다.

| 구분 | 내용 |
|---|---|
| 개발 기간 | 2026.05 ~ 2026.06 |
| 개발 인원 | 6명 |
| 담당 역할 | Backend — POS·Naver Place 데이터 연동, Instagram 인증·성과 조회 |
| 프로젝트 상태 | SSAFY 팀 프로젝트 V1 완료 |

이 저장소는 팀 프로젝트의 구현 내용과 개인 기여를 정리한 문서용 저장소입니다. 소스 코드는 포함하지 않습니다.

## 주요 기능

- **매장 연결:** Toss POS를 연결해 메뉴·가격을 조회하고 Naver Place의 상세정보를 결합합니다.
- **콘텐츠 제작·게시:** 매장 정보와 사용자 입력을 바탕으로 AI가 이미지·캡션을 생성하고, 사용자가 확인한 콘텐츠를 Instagram에 게시합니다.
- **성과 확인:** Instagram의 `views`, `saves`, `shares` 지표를 조회합니다.

위 내용은 팀 전체의 구현 범위이며, 개인 담당 범위는 아래 세 항목입니다.

## 담당 역할과 구현

### 1. POS와 Naver Place의 메뉴 정보 병합

**배경** — POS의 메뉴·가격과 Naver Place의 상세 설명을 함께 활용해야 했지만, 서비스마다 같은 메뉴의 이름을 다르게 표기해 그대로 연결하기 어려웠습니다.

**구현** — Toss Catalog API와 Playwright 기반 Naver Place 크롤러를 연동했습니다. 메뉴명을 정규화한 값을 기준으로 데이터를 매핑하고, POS의 메뉴·가격에 Naver Place의 설명을 결합했습니다. Naver Place 상세정보에는 **Redis 캐시와 30분 TTL**을 적용했습니다.

**구현 결과** — 서로 다른 외부 데이터를 콘텐츠 생성에 활용할 수 있는 매장 정보로 구성했습니다. 캐시가 유효한 동안에는 저장된 상세정보를 재사용하는 경로를 마련했습니다.

### 2. Redis 기반 Toss POS 매장 연결

**배경** — Toss POS 플러그인에서 확인한 매장과 맡케팅 사용자 계정을 연결하는 절차가 필요했습니다.

**구현** — Merchant ID에 대응하는 일회용 PIN을 Redis에 저장하고 **180초 TTL**을 설정했습니다. 사용자가 입력한 PIN을 확인해 해당 POS 매장과 서비스 계정을 연결하는 온보딩 흐름을 구현했습니다.

**구현 결과** — 사용자가 Merchant ID를 직접 입력하지 않고 PIN으로 매장을 연결할 수 있도록 구성했습니다.

### 3. Instagram OAuth와 성과 지표 연동

**배경** — 사용자의 Instagram 계정에 접근하려면 인증 결과와 토큰, 외부 계정 정보를 서비스 사용자와 연결해 관리해야 했습니다.

**구현** — Meta OAuth 인증과 장기 액세스 토큰 관리 흐름을 구현하고, Instagram User Insights의 `views`, `saves`, `shares`를 조회해 내부 데이터로 변환했습니다.

**구현 결과** — 연결된 Instagram 계정의 성과 지표를 서비스에서 조회할 수 있도록 연동했습니다.

## Tech Stack

| 범위 | 주요 기술 |
|---|---|
| 개인 담당 Backend | Java 21, Spring Boot 3.5, JPA, PostgreSQL |
| 개인 담당 데이터·인증 연동 | Redis, Toss Catalog API, Naver Place 크롤러 연동, Meta OAuth, Instagram Graph API |
| 팀 Frontend | React, Vite, PWA |
| 팀 AI·Crawler | FastAPI, LangGraph, OpenCV, PyTorch, Playwright |
| 팀 Infrastructure | Docker Compose, Nginx, Jenkins, AWS S3, CloudFront |

## Related PoCs

- [Instagram Image PoC](https://github.com/typ0squir/images-to-instagramable): SDXL·ControlNet·RunPod를 활용한 이미지 변환 실험으로, V1의 실제 이미지 처리 경로와 구분됩니다.
- **Toss POS Integration PoC:** 매장 연결과 메뉴·결제 데이터 활용 가능성을 검토했습니다. 결제·매출 데이터 기반 피드백은 V1 구현 범위에 포함되지 않습니다.

## Retrospective

외부 서비스 연동에서는 API 호출뿐 아니라 서로 다른 식별자와 데이터 형식을 내부 모델에 연결하는 과정이 중요했습니다. Redis도 같은 저장소를 사용하지만 PIN의 유효 시간과 상세정보 캐시의 재사용 목적을 구분해 적용했습니다. 이후에는 외부 API 실패 상황과 데이터 변환 경계를 테스트로 검증하는 경험을 보완하고자 합니다.

## Team

<details>
<summary>팀 구성 펼쳐보기</summary>

| 이름 | 역할 |
|---|---|
| [윤지선](https://github.com/js-yunn) | Backend · Team Lead |
| [김희원](https://github.com/heewon916) | Backend · Infra |
| [명민주](https://github.com/typ0squir) | Backend — POS·Naver Place 연동, Instagram 인증·성과 조회 |
| [유주성](https://github.com/Juseong-Yu) | Frontend |
| [홍지운](https://github.com/qqjiwoon) | Frontend |
| 최다은 | AI |

</details>
