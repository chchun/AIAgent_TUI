# SAFETYSAAS Agent — AI 안전관리 에이전트

> **Next.js 14 + TypeScript** 기반 AI 에이전트 프론트엔드  
> **A2UI v0.9** 생성형 UI 프로토콜 구현 포함  
> Phase 1 (Mock 스트리밍) ✅ 완료 · Phase 2 (FastAPI 백엔드 연동) 예정

---

## 목차

1. [프로젝트 개요](#1-프로젝트-개요)
2. [스크린샷](#2-스크린샷)
3. [기술 스택](#3-기술-스택)
4. [파일 구조](#4-파일-구조)
5. [로컬 실행](#5-로컬-실행)
6. [구현 기능](#6-구현-기능)
7. [A2UI v0.9 — 생성형 UI 프로토콜](#7-a2ui-v09--생성형-ui-프로토콜)
8. [A2UI 테스트 방법](#8-a2ui-테스트-방법)
9. [Mock 응답 시나리오 전체 목록](#9-mock-응답-시나리오-전체-목록)
10. [Phase 2 로드맵](#10-phase-2-로드맵)
11. [트러블슈팅 기록](#11-트러블슈팅-기록)

---

## 1. 프로젝트 개요

SAFETYSAAS Agent는 산업 안전관리 도메인에 특화된 AI 에이전트 채팅 인터페이스입니다.

**핵심 특징:**

- **스트리밍 응답**: 상태 메시지 → 추론 과정(CoT) → 텍스트 답변 순차 표시
- **RAG 인용**: 답변에 포함된 `[N]` 출처 번호 클릭 시 우측 슬라이드 패널로 원문 확인
- **A2UI 생성형 UI**: 텍스트 답변 대신 인터랙티브 UI 컴포넌트(폼, 슬라이더, 테이블 등)를 에이전트가 직접 생성
- **세션 관리**: 복수 대화 세션을 localStorage에 영속 저장, 라이트/다크 테마

Phase 1은 **setTimeout 기반 Mock 스트리밍**으로 동작하며, 백엔드(FastAPI) 없이 완전히 프론트엔드만으로 실행됩니다.

---

## 2. 스크린샷

| 텍스트 스트리밍 + RAG 인용 | A2UI — 인시던트 보고 폼 |
|:---:|:---:|
| CoT 추론 과정 접기/펼치기, 코드블록 하이라이팅, 출처 패널 | ChoicePicker · TextField · CheckBox · Button |

| A2UI — 카나리 배포 승인 | A2UI — 위험성평가 DataTable |
|:---:|:---:|
| Slider 실시간 바인딩, 복합 유효성 검사 | 공종 선택 → 위험요인 행 자동 생성 |

---

## 3. 기술 스택

| 항목 | 선택 | 버전 |
|------|------|------|
| 프레임워크 | Next.js (App Router) | 14.2.x |
| 언어 | TypeScript (strict) | 5.x |
| 스타일 | 전역 CSS + CSS 변수 (디자인 토큰) | — |
| 아이콘 | lucide-react | 최신 |
| 마크다운 렌더링 | marked | 14.x (CJS 호환) |
| 코드 하이라이팅 | highlight.js | — |
| HTML Sanitize | DOMPurify | — |
| 폰트 | Pretendard Variable + JetBrains Mono | CDN |
| 상태 관리 | React useState / useCallback | — |
| 영속성 | localStorage | — |

> **`react-markdown` 대신 `marked`를 사용하는 이유**  
> `react-markdown` v8/v10은 ESM 전용 모듈로 Next.js webpack 환경에서 무한 컴파일 대기 현상이 발생합니다.  
> CJS 호환인 `marked` v14로 교체 후 정상 동작합니다.

---

## 4. 파일 구조

```
safetysaas/
├── docs/
│   ├── phase1-implementation.md   Phase 1 구현 수행 내역
│   └── a2ui-spec.md               A2UI v0.9 인터페이스 스펙 (시나리오 4개)
├── src/
│   ├── app/
│   │   ├── layout.tsx             루트 레이아웃, 폰트 CDN 로드
│   │   ├── page.tsx               앱 셸 조립 (상태/이벤트 연결, A2UI 콜백 배선)
│   │   └── globals.css            전체 CSS (@import tokens + a2ui)
│   ├── components/
│   │   ├── A2UISurface.tsx        A2UI 렌더러 — 15개 컴포넌트 타입 구현 (~570줄)
│   │   ├── BotMessage.tsx         AI 응답 (StatusLine → CoT → Prose → A2UISurface)
│   │   ├── UserMessage.tsx        사용자 말풍선 (파일 첨부 칩 포함)
│   │   ├── StatusLine.tsx         스피너 + shimmer 텍스트 애니메이션
│   │   ├── CoT.tsx                추론 과정 접기/펼치기 (grid-rows 트랜지션)
│   │   ├── Prose.tsx              marked 기반 마크다운 렌더러 (코드블록 복사, 인용 칩)
│   │   ├── MsgActions.tsx         복사 / 좋아요 / 싫어요 / 다시 생성
│   │   ├── Composer.tsx           입력창 (자동 리사이즈, 파일 첨부, 마이크, Enter 전송)
│   │   ├── Sidebar.tsx            대화 히스토리 사이드바 (오늘/지난7일/이전 그룹)
│   │   ├── Topbar.tsx             모델 칩, 테마 토글, 사이드바 접기
│   │   ├── EmptyState.tsx         초기 화면 (추천 카드 6개 + 기능 칩)
│   │   └── SourcePanel.tsx        우측 슬라이드 출처 패널 (flash 스크롤 효과)
│   ├── hooks/
│   │   ├── useChat.ts             스트리밍 엔진 + A2UI 봉투 적용 + 세션 관리 + localStorage
│   │   └── useTheme.ts            라이트/다크 테마 토글 (data-theme 방식)
│   ├── lib/
│   │   ├── a2ui.ts                A2UI 순수 로직 (JSON Pointer, 봉투 적용, 바인딩, 검증)
│   │   ├── a2ui-data.ts           4개 A2UI 시나리오 + 위험성평가 빌더 + 액션 응답 팩토리
│   │   ├── data.ts                텍스트 RAG 4개 시나리오 + pickAnswer()
│   │   └── utils.ts               uid, genTitle, sleep, groupSessions, guessFileIcon
│   ├── styles/
│   │   ├── tokens.css             라이트/다크 CSS 변수 (디자인 토큰)
│   │   └── a2ui.css               A2UI 컴포넌트 전용 스타일
│   └── types/
│       └── index.ts               Session, Message, A2UISurfaceState, A2UIComponent 등
├── next.config.mjs
├── tsconfig.json
└── package.json
```

---

## 5. 로컬 실행

### 사전 요구사항

- Node.js 18+ / npm 9+

### 설치 및 실행

```bash
# 저장소 클론
git clone https://github.com/chchun/AIAgent_TUI.git
cd AIAgent_TUI

# 의존성 설치
npm install

# 개발 서버 실행
npm run dev
# → http://localhost:3000
```

### 프로덕션 빌드

```bash
npm run build
npm start
```

### 타입 검사

```bash
npx tsc --noEmit
```

---

## 6. 구현 기능

### 레이아웃 & 반응형

- `280px` 사이드바 + `1fr` 메인 컬럼 그리드
- 데스크톱: 사이드바 접기/펼치기 (width + opacity 트랜지션)
- 모바일(≤860px): 사이드바 오버레이 드로어 + 스크림
- Topbar 스크롤 시 `backdrop-filter blur(8px)` + 테두리 노출

### 스트리밍 엔진 (`useChat.ts`)

```
status 페이즈    → "응답을 생성하고 있어요…" (shimmer 애니메이션)
reasoning 페이즈 → CoT 추론 단계 순차 공개 (접기/펼치기)
answer 페이즈    → 마크다운 텍스트 토큰 스트리밍 (타이핑 캐럿)
a2ui 페이즈      → A2UI 봉투 순차 적용 (skeleton → 구조 → 데이터)
done             → MsgActions 노출
```

- 응답 중단 (`stop`) — 스트리밍 즉시 종료, 부분 텍스트 보존
- 다시 생성 (`regenerate`) — 이전 사용자 메시지로 재실행
- 페이지 리로드 시 중단된 스트리밍 자동 복구
- 자동 스크롤 (autoStick — 하단 120px 이내일 때 유지)

### Composer (입력창)

- textarea 1줄~220px 자동 리사이즈
- `Enter` → 전송, `Shift+Enter` → 줄바꿈
- 파일 첨부 (드래그&드롭 + 버튼, 최대 5개, 아이콘+이름+크기)
- 마이크 버튼 (음성 입력 시뮬레이션, recPulse 애니메이션)
- 전송 버튼 ↔ 중단 버튼 전환

### 마크다운 렌더링 (Prose.tsx)

- 코드블록: 언어 라벨 + 복사 버튼 + highlight.js 하이라이팅
- `[N]` 인용 → 클릭 시 출처 패널 오픈 + 해당 항목 flash 스크롤
- DOMPurify HTML sanitize
- 스트리밍 중 타이핑 캐럿 표시

### 세션 관리

- `localStorage` 키 `safetysaas_agent_v1` 에 전체 세션 영속 저장
- 첫 메시지로 세션 제목 자동 생성 (38자 제한)
- 히스토리 그룹화: 오늘 / 지난 7일 / 이전

---

## 7. A2UI v0.9 — 생성형 UI 프로토콜

A2UI(Agent-to-UI)는 에이전트가 텍스트 대신 **인터랙티브 UI 컴포넌트**를 직접 생성하는 프로토콜입니다.

### 봉투(Envelope) 스트리밍 흐름

```
createSurface      → 서피스 메타데이터 선언 (skeleton 표시)
  ↓ 300~500ms
updateComponents   → 컴포넌트 트리 삽입 (레이아웃 구조 확정)
  ↓ 300~500ms
updateDataModel    → 초기 데이터 바인딩 → 완성된 UI 표시
```

### 지원 컴포넌트 (15종)

| 카테고리 | 컴포넌트 |
|---------|---------|
| 레이아웃 | `Card` · `Column` · `Row` · `Divider` · `Tabs` · `Modal` |
| 표시 | `Text` (formatString 보간) · `Icon` |
| 입력 | `Button` · `TextField` · `CheckBox` · `ChoicePicker` · `Slider` · `Switch` · `Select` · `TagInput` |
| 데이터 | `DataTable` (편집 가능, 상태 사이클, CRUD) |

### 데이터 바인딩 방식

```jsonc
// JSON Pointer로 dataModel 경로 참조
{ "value": { "path": "/incident/severity" } }

// formatString 동적 보간
{ "text": { "call": "formatString", "args": { "value": "트래픽 ${/deploy/canary}%" } } }

// 유효성 검사 (버튼 비활성화 조건)
{ "checks": [{ "call": "required", "args": { "value": { "path": "/incident/summary" } }, "message": "필수입니다." }] }
```

### 액션 Round-trip

```
버튼 클릭
  → onA2UIAction(payload) 발송
  → 새 bot 메시지 생성
  → 텍스트 응답 스트리밍 (또는 새 A2UI 서피스)
```

자세한 인터페이스 스펙은 **[docs/a2ui-spec.md](./docs/a2ui-spec.md)** 참조

---

## 8. A2UI 테스트 방법

개발 서버를 실행하고 아래 키워드를 채팅창에 입력하면 각 시나리오가 트리거됩니다.

### 시나리오 1 — 인시던트 보고 폼

**트리거 키워드** (아무거나 입력)
```
인시던트
장애 보고
incident
보고서 폼
인시던트 생성
```

**기대 동작:**
1. "인시던트 스키마를 불러오고 있어요…" 상태 표시
2. CoT 추론 단계 표시
3. 마크다운 안내 텍스트 스트리밍
4. 빨간 테마 `Incident Bot` 폼 UI 생성
   - `ChoicePicker`: P1 / P2 / P3 심각도 선택
   - `TextField`: 영향 서비스, 증상 요약 (필수)
   - `CheckBox`: 고객 영향 여부
5. **"인시던트 생성" 버튼** — 증상 요약 미입력 시 비활성
6. 버튼 클릭 → `INC-XXXX` 생성 완료 메시지 스트리밍

---

### 시나리오 2 — 카나리 배포 승인

**트리거 키워드**
```
배포
deploy
카나리
canary
롤아웃
배포 승인
payment-service 카나리
```

**기대 동작:**
1. 초록 테마 `Deploy Bot` 패널 UI 생성
   - `Slider`: 카나리 트래픽 비중 (0~100%, step 5)
   - `Text`: Slider 값 실시간 반영 — `"신버전으로 **20%**의 트래픽을 전환합니다."`
   - `CheckBox`: 스테이징 헬스체크 확인
2. **"배포 시작" 버튼** — 헬스체크 체크 AND 카나리 ≥ 1% 둘 다 만족해야 활성화
3. 버튼 클릭 → 배포 시작 완료 메시지

> **테스트 포인트**: Slider를 0%로 두거나 CheckBox 미체크 상태에서 버튼이 비활성인지 확인

---

### 시나리오 3 — 서비스 상태 카드

**트리거 키워드**
```
상태 카드
서비스 상태
status card
상태 대시보드
헬스 카드
payment 상태
```

**기대 동작:**
1. 파란 테마 `Observability Bot` 상태 카드 UI 생성
   - `ChoicePicker`: 기간 선택 (15분 / 1시간 / 24시간)
   - 메트릭 3열 표시: **p99 지연** / **에러율** / **RPS**
     - 각 값은 `formatString` 보간으로 dataModel과 실시간 연결
2. **"새로고침" 버튼** 클릭 → 현재 선택된 기간 포함해서 응답 메시지 생성

> **테스트 포인트**: ChoicePicker 기간 변경 후 새로고침 버튼을 눌러 context에 변경된 range가 포함되는지 확인

---

### 시나리오 4 — 위험성평가 자동작성 (2단계 플로우)

**트리거 키워드**
```
위험성평가
공종
위험성 평가
자동작성
안전 평가
위험 요인
```

**기대 동작 — 1단계 (입력 폼):**
1. 청록 테마 `Safety Bot` 입력 폼 UI 생성
   - `Select`: 공종 선택 (배관 / 용접 / 비계 / 전기 / 굴착 / 도장)
   - `TagInput`: 작업환경·장비 태그 입력
     - 제안 칩: `저온` `고온` `밀폐` `화기` `분진` `소음` `진동` `고소` `양중` `크레인` `지게차` `용접기` `그라인더`
2. **"위험성평가 생성" 버튼** — 공종 미선택 시 비활성

**기대 동작 — 2단계 (결과 DataTable):**
3. 버튼 클릭 후 새 메시지에 **편집 가능한 DataTable** 생성
   - 공종별 기본 위험요인 행 (배관 3행, 용접 3행, 그 외 2행)
   - 선택한 태그 1개당 +1행 추가
   - 컬럼: 작업단계 · 위험요인 · 재해시나리오 · 가능성 · 중대성 · **위험도** · 개선대책 · 개선후 위험도 · 보호구 · 관련법령 · 상태
4. DataTable 행 편집 테스트:
   - **상태** 열 버튼 클릭 → `초안 → 검토 → 승인 → 초안` 사이클
   - **가능성 / 중대성** Select 드롭다운 변경
   - 우측 액션 버튼: 행 **복제** / **제외** (취소선) / **삭제**

> **위험도 계산**: 가능성 × 중대성 (상=3, 중=2, 하=1) → 6이상: 상(빨강), 3이상: 중(노랑), 미만: 하(초록)

---

### A2UI 공통 테스트 포인트

| 항목 | 확인 내용 |
|------|---------|
| Progressive rendering | createSurface → updateComponents → updateDataModel 순서로 UI가 단계적으로 나타나는지 (skeleton → 구조 → 데이터) |
| 양방향 바인딩 | Slider 드래그 시 formatString Text가 즉시 업데이트되는지 |
| 버튼 disabled 상태 | checks 조건 미충족 시 버튼 비활성 + 안내 메시지 표시 |
| 세션 영속 | 페이지 새로고침 후 A2UI 서피스 상태(입력값, 선택값)가 복원되는지 |
| 액션 round-trip | 버튼 클릭 → 새 bot 메시지 스트리밍 → (위험성평가) 새 DataTable A2UI 생성 |
| 다크 테마 | 우상단 테마 토글 클릭 시 A2UI 컴포넌트도 다크 테마 적용 |

---

## 9. Mock 응답 시나리오 전체 목록

### 텍스트 RAG 응답 (4개)

| 트리거 키워드 | 시나리오 | 특징 |
|------------|---------|------|
| `타임아웃` `timeout` `간헐적` | order→payment 타임아웃 원인 분석 | 코드블록 + 비교표 + RAG 인용 [1][2][3] |
| `결제 승인` `명세` `페이로드` | POST /api/v1/payments/approve 요청·응답 명세 | JSON 코드블록, 필드 설명 표 |
| `서킷브레이커` `Resilience4j` | application.yml 서킷브레이커 설정 | YAML 코드블록, 파라미터 설명 |
| `로그` `에러 로그` `order-service` | 로그 분석 결과 요약 | 로그 패턴, 원인 추정 |

### A2UI 생성형 UI 응답 (4개)

| 트리거 키워드 | 시나리오 | 테마 색상 |
|------------|---------|---------|
| `인시던트` `incident` `장애 보고` | 인시던트 보고 폼 | `#E5484D` 빨강 |
| `배포` `deploy` `카나리` `canary` | 카나리 배포 승인 패널 | `#1F8A5B` 초록 |
| `상태 카드` `서비스 상태` `status card` | payment-service 상태 카드 | `#2A6FDB` 파랑 |
| `위험성평가` `공종` `위험 요인` | 위험성평가 자동작성 (2단계) | `#0E7C66` 청록 |

> **어떤 키워드도 해당되지 않으면** 기본 텍스트 응답(타임아웃 분석)이 반환됩니다.

---

## 10. Phase 2 로드맵

| 항목 | Phase 1 현황 | Phase 2 계획 |
|------|-------------|-------------|
| 백엔드 | 없음 (setTimeout 시뮬레이션) | FastAPI + SSE 실시간 스트리밍 |
| RAG | Mock 고정 데이터 | 실제 문서 인덱싱 + 벡터 검색 (pgvector / Qdrant) |
| A2UI 봉투 소스 | 클라이언트 사전 정의 배열 | 서버가 SSE로 봉투 스트리밍 |
| A2UI 액션 응답 | 클라이언트 Mock | 서버가 새 봉투 스트림 반환 |
| 위험도 재계산 | 수동 Select | 서버 연산 후 updateDataModel 발송 |
| DataTable 저장 | 없음 | 서버 POST + 문서 생성 연동 |
| 인증 | 없음 | JWT (NextAuth.js) |

---

## 11. 트러블슈팅 기록

### react-markdown ESM 충돌

- **증상**: `npm run dev` 후 브라우저에 아무것도 표시되지 않음 — `○ Compiling /...` 무한 대기
- **원인**: `react-markdown` v8/v10이 ESM 전용 — Next.js webpack CJS 컨텍스트에서 로드 실패
- **해결**: `react-markdown` + `remark-gfm` 제거 → `marked` v14 (CJS 호환)로 교체

### marked v14 Renderer API 변경

- **증상**: `renderer.code` 타입 오류 — `(code: string, lang?: string)` 시그니처 불일치
- **원인**: marked v14에서 renderer 메서드가 문자열 대신 객체(`{ text, lang }`)를 수신하도록 변경
- **해결**: `(renderer as any).code = ({ text, lang }) => ...` 형태로 수정

### DOMPurify Config 타입 오류

- **증상**: `DOMPurify.Config` 타입 명시 시 `PARSER_MEDIA_TYPE` 호환성 오류
- **원인**: `@types/dompurify`와 `dompurify` 내장 타입 간 `DOMParserSupportedType` 정의 불일치
- **해결**: 타입 명시 제거 — TypeScript 자동 추론으로 처리

### A2UI — `unknown` 타입 JSX 렌더 오류

- **증상**: `Type 'unknown' is not assignable to type 'ReactNode'` 빌드 오류 6건
- **원인**: `A2UIComponent`의 인덱스 시그니처 `[key: string]: unknown` 때문에 `comp.label` 등이 모두 `unknown` 타입으로 추론됨. TypeScript에서 `unknown && <JSX>` 표현식의 반환 타입은 `unknown`
- **해결**: 모든 `{comp.X && <...>}` 패턴을 `{comp.X != null && <...>}` 로 변경 — `!= null` guard는 `boolean`으로 평가되어 결과 타입이 `ReactNode`로 좁혀짐

### A2UI — `onA2UIAction` 타입 불일치

- **증상**: `(payload: object) => void`와 `(action: A2UIActionPayload) => void` 타입 충돌
- **원인**: 함수 파라미터 타입은 반공변(contravariant) — `object`가 더 넓은 타입이므로 좁은 타입(A2UIActionPayload)을 받는 함수에 할당 불가
- **해결**: `BotMessage.tsx`와 `page.tsx`에 `A2UIActionPayload` import 추가, `(p) => chat.onA2UIAction(p as A2UIActionPayload)` 캐스트 적용

---

## 관련 문서

- [Phase 1 구현 수행 내역](./docs/phase1-implementation.md)
- [A2UI v0.9 인터페이스 스펙](./docs/a2ui-spec.md)

---

<p align="center">
  Built with Next.js 14 · TypeScript · A2UI v0.9
</p>
