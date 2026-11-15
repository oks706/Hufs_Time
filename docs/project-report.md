---
title: false
stylesheet: false
pdf_options:
  format: A4
  margin: 30mm 25mm 30mm 25mm
  printBackground: true
---

<style>
  body {
    font-family: 'Malgun Gothic', 'Apple SD Gothic Neo', sans-serif;
    font-size: 11pt;
    line-height: 1.8;
    color: #1a1a1a;
  }
  h1 { font-size: 22pt; text-align: center; margin-top: 60px; margin-bottom: 5px; color: #002843; }
  h2 { font-size: 15pt; color: #0a5e8a; border-bottom: 2px solid #0a5e8a; padding-bottom: 5px; margin-top: 35px; }
  h3 { font-size: 12pt; color: #002843; margin-top: 20px; }
  table { border-collapse: collapse; width: 100%; margin: 10px 0; font-size: 10pt; }
  th { background-color: #002843; color: white; padding: 8px 12px; text-align: left; }
  td { border: 1px solid #ddd; padding: 7px 12px; }
  tr:nth-child(even) { background-color: #f8fbfd; }
  code { background: #f0f7fb; padding: 2px 6px; border-radius: 3px; font-size: 9.5pt; }
  .center { text-align: center; }
  .subtitle { text-align: center; color: #555; font-size: 12pt; margin-bottom: 40px; }
  .info-box { background: #f0f7fb; border-left: 4px solid #0a5e8a; padding: 12px 16px; margin: 15px 0; border-radius: 0 8px 8px 0; }
  .stat-grid { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 12px; margin: 15px 0; }
  .stat-card { background: #f8fbfd; border: 1px solid #e0e0e0; border-radius: 8px; padding: 15px; text-align: center; }
  .stat-card .num { font-size: 24pt; font-weight: bold; color: #0a5e8a; }
  .stat-card .label { font-size: 9pt; color: #666; margin-top: 3px; }
  .page-break { page-break-before: always; }
  h2 { page-break-before: always; }
  h2:first-of-type { page-break-before: avoid; }
  table, .info-box, .stat-grid, pre { page-break-inside: avoid; }
  h3 { page-break-after: avoid; }
</style>

# HUFS 수강 시간표 빌더

<div class="subtitle">
프로젝트 완성 보고서<br>
2026년 3월 31일<br>
한국외국어대학교 정보통신공학과
</div>

---

## 1. 프로젝트 개요

### 1.1 목적

한국외국어대학교 학생들이 매 학기 수강신청 전 시간표를 효율적으로 구성할 수 있도록 돕는 **웹 기반 시간표 빌더**를 개발하였다. 학교 수강신청 시스템(WIS)에서 강의 데이터를 자동으로 수집하고, 직관적인 UI를 통해 시간표를 편집, 저장, 내보내기할 수 있다.

### 1.2 개발 동기

기존 수강신청 시스템은 강의 검색과 시간표 작성이 분리되어 있어, 학생들이 여러 창을 오가며 수동으로 시간 충돌을 확인해야 하는 불편함이 있었다. 본 프로젝트는 이를 하나의 화면에서 해결할 수 있도록 설계하였다.

### 1.3 핵심 수치

<div class="stat-grid">
  <div class="stat-card">
    <div class="num">3,720</div>
    <div class="label">수집된 강의 수</div>
  </div>
  <div class="stat-card">
    <div class="num">192</div>
    <div class="label">학과 및 교양 영역</div>
  </div>
  <div class="stat-card">
    <div class="num">51</div>
    <div class="label">카테고리 종류</div>
  </div>
</div>

| 구분 | 강의 수 | 비율 |
|------|---------|------|
| 전공/부전공 | 2,499개 | 67.2% |
| 교양 | 1,143개 | 30.7% |
| 기초 | 78개 | 2.1% |
| **합계** | **3,720개** | **100%** |

---

## 2. 기술 아키텍처

### 2.1 기술 스택

| 영역 | 기술 | 선정 사유 |
|------|------|-----------|
| Frontend | Vue 3 (Composition API) + Vite | 반응형 SPA, 빠른 HMR 개발 환경 |
| Backend | Express.js (Node.js) | 프론트와 동일 JS 생태계, 경량 REST API |
| Database | SQLite (better-sqlite3) | 서버리스, 동기 API로 높은 성능 |
| Styling | Tailwind CSS | 유틸리티 퍼스트, 빠른 UI 개발 |
| 데이터 수집 | Axios | HUFS WIS API와 HTTP 통신 |
| 배포 | Docker + GitHub Actions + ArgoCD | ICE GitOps 클러스터 자동 배포 |

### 2.2 시스템 구조

```
┌─────────────────────────────────────────────────┐
│                  사용자 (브라우저)                 │
│          Vue 3 SPA + Tailwind CSS               │
│    시간표 그리드 / 강의 검색 / 저장·내보내기       │
└──────────────────────┬──────────────────────────┘
                       │ HTTP (REST API)
┌──────────────────────▼──────────────────────────┐
│               Express.js 백엔드                  │
│   /api/lectures  /api/departments  /api/scraper  │
│   Route → Service → Model 레이어 분리            │
└──────────────────────┬──────────────────────────┘
                       │ SQL (better-sqlite3)
┌──────────────────────▼──────────────────────────┐
│                  SQLite Database                  │
│   lectures / departments / lecture_times          │
│   scrape_logs / 메타데이터 테이블 3개             │
└──────────────────────────────────────────────────┘
                       ▲
                       │ HTTP POST (스크래핑)
┌──────────────────────┴──────────────────────────┐
│            HUFS WIS API (wis.hufs.ac.kr)         │
│   전공(gubun=1) / 교양(gubun=2) / 기초(gubun=3)  │
└──────────────────────────────────────────────────┘
```

### 2.3 데이터베이스 스키마

| 테이블 | 설명 | 주요 컬럼 |
|--------|------|-----------|
| `departments` | 학과/영역 정보 | name, campus(H1/H2) |
| `lectures` | 강의 정보 | course_code, course_name, professor, credit, category, gubun |
| `lecture_times` | 강의 시간 슬롯 | day_of_week, start_time, end_time, room |
| `scrape_logs` | 스크래핑 실행 로그 | status, total_count, error_message |
| `required_courses` | 전공필수 교과목 | department, course_code |
| `cross_major_courses` | 전공교류 교과목 | receiving_department, course_name |
| `banned_courses` | 수강금지 교과목 | department, course_name |

## 3. 주요 기능 소개

### 3.1 강의 데이터 자동 수집 (스크래퍼)

HUFS 학사정보시스템(WIS)의 API를 분석하여, 전체 강의 데이터를 자동으로 수집하는 스크래퍼를 구현하였다.

<div class="info-box">
<strong>수집 대상</strong>: 서울캠퍼스(H1) + 글로벌캠퍼스(H2)의 전공, 교양, 기초 전 영역<br>
<strong>수집 방식</strong>: HUFS WIS API에 POST 요청 → JSON 응답 파싱 → SQLite 저장<br>
<strong>소요 시간</strong>: 약 8~10분 (레이트 제한 준수)
</div>

**수집 프로세스**:

1. **학과 코드 수집**: WIS API(`process3_1a`)로 캠퍼스별 학과 목록을 동적으로 가져옴. API 실패 시 내장된 폴백 리스트(151개 학과) 사용
2. **교양/기초 영역 코드 수집**: `process4_1a`(교양), `process4_1b`/`process4_1c`(기초)로 영역 코드 동적 수집
3. **강의 데이터 요청**: 각 학과/영역별로 `getDataLssnLista` API 호출
4. **시간 정보 파싱**: `"월 1 금 1 2 (5213)"` 형식의 교시 정보를 `09:00~09:50` 형식으로 변환
5. **DB 저장**: 트랜잭션으로 일괄 저장 (기존 데이터 교체)

**교시-시간 매핑**:

| 교시 | 시간 | 교시 | 시간 |
|------|------|------|------|
| 1교시 | 09:00~09:50 | 7교시 | 15:00~15:50 |
| 2교시 | 10:00~10:50 | 8교시 | 16:00~16:50 |
| 3교시 | 11:00~11:50 | 9교시 | 17:00~17:50 |
| 4교시 | 12:00~12:50 | 10교시 | 18:00~18:50 |
| 5교시 | 13:00~13:50 | 11교시 | 19:00~19:50 |
| 6교시 | 14:00~14:50 | 12교시 | 20:00~20:50 |

---

### 3.2 3탭 필터링 시스템

강의 데이터를 효율적으로 탐색할 수 있도록 **전공/부전공**, **교양**, **기초** 3개 탭으로 구분한 필터 시스템을 설계하였다.

**전공/부전공 탭**:
- 내 학과 설정 기능 (전공교류, 필수, 수강금지 메타데이터 활성화)
- 학과 선택 드롭다운
- 학년 필터 (1~4학년)
- 세부 카테고리 필터 (전공, 이중(부)전공, 이중(제2), 교직)
- 전공교류 필터 (내 학과 설정 시 활성화)

**교양 탭**:
- 교양 영역별 필터 (대학외국어, 미네르바인문, AI·SW·데이터, 사회와문화, 역사와철학, 문학과예술, 과학과기술, 생활과스포츠 등 16개 영역)

**기초 탭**:
- 기초 영역별 필터 (영어대학, 공과대학, 사회과학대학 등 단과대학별 기초 교과)

**공통 필터**:
- 캠퍼스 선택 (서울 / 글로벌)
- 요일 필터 (월~금 칩 버튼)
- 키워드 검색 (과목명, 교수명)

---

### 3.3 시간표 그리드 편집

시간표를 시각적으로 편집할 수 있는 인터랙티브 그리드를 구현하였다.

- **그리드 구조**: 월~금 5일 × 09:00~21:00 (30분 단위 24슬롯)
- **강의 추가**: 검색 결과에서 "추가" 버튼 클릭 시 그리드에 즉시 반영
- **시간 충돌 감지**: 기존 강의와 시간이 겹치면 경고 메시지 표시, 추가 차단
- **자동 색상 배정**: 과목코드 해시 기반 16가지 파스텔 색상 자동 할당
- **강의 상세 보기**: 그리드 내 강의 블록 클릭 시 상세 정보 팝업
- **강의 제거**: 팝업에서 제거 버튼 클릭

---

### 3.4 수강유의 교과목 메타데이터

학교에서 배포하는 수강유의교과목 데이터(XLS)를 파싱하여, 강의 검색 시 시각적 뱃지로 표시한다.

| 메타데이터 | 뱃지 색상 | 설명 |
|-----------|----------|------|
| 전공필수 | 🟠 주황 | 내 학과 전공필수 교과목 |
| 전공교류 | 🟣 보라 | 다른 학과 강의지만 내 전공으로 인정 |
| 수강금지 | 🔴 빨강 | 내 학과에서 수강 불가능한 교과목 (추가 차단) |

<div class="info-box">
<strong>전공교류 표시 로직</strong>: 내 학과를 설정하면, 다른 학과의 강의 중 전공으로 인정되는 과목이 보라색 뱃지와 함께 표시된다. "전체 구분" 검색 시에도 교류 과목이 자동으로 병합되어 표시되므로, 수강 가능한 전공 과목을 놓치지 않을 수 있다.
</div>

---

### 3.5 시간표 저장 및 내보내기

- **다중 시간표 관리**: 여러 시간표를 이름을 지정하여 저장 (LocalStorage 기반)
- **총 학점 실시간 표시**: 현재 시간표의 총 학점을 자동 계산
- **PNG 이미지 내보내기**: html2canvas를 활용하여 시간표를 고해상도(2x) PNG로 저장
- **저장된 시간표 목록**: 별도 페이지에서 저장된 시간표를 미니 프리뷰로 확인, 불러오기, 삭제

## 4. API 설계

### 4.1 강의 API

| 메서드 | 경로 | 설명 | 주요 파라미터 |
|--------|------|------|--------------|
| GET | `/api/lectures` | 강의 목록 조회 | gubun, department, year, category, day, keyword, page, limit |
| GET | `/api/lectures/categories` | 카테고리 목록 | gubun (gubun별 카테고리 필터) |
| GET | `/api/lectures/search` | 키워드 통합 검색 | q, page, limit |
| GET | `/api/lectures/:id` | 강의 상세 조회 | my_department (메타데이터용) |

### 4.2 학과 API

| 메서드 | 경로 | 설명 |
|--------|------|------|
| GET | `/api/departments` | 학과 목록 (campus 필터 지원) |

### 4.3 스크래퍼 API

| 메서드 | 경로 | 설명 |
|--------|------|------|
| POST | `/api/scraper/run` | 스크래핑 실행 (mode: live/seed) |
| GET | `/api/scraper/status` | 최근 스크래핑 로그 조회 |

### 4.4 메타데이터 API

| 메서드 | 경로 | 설명 |
|--------|------|------|
| GET | `/api/metadata/cross-major` | 전공교류 강의 조회 (department 파라미터) |

---

## 5. 프론트엔드 구조

### 5.1 페이지 구성

| 경로 | 화면 | 설명 |
|------|------|------|
| `/` | HomeView | 메인 시간표 편집 (검색 패널 + 시간표 그리드) |
| `/search` | SearchView | 강의 검색 전용 화면 (3컬럼 레이아웃) |
| `/saved` | SavedView | 저장된 시간표 목록 관리 |

### 5.2 핵심 컴포넌트

| 컴포넌트 | 역할 |
|----------|------|
| `TimetableGrid.vue` | 시간표 그리드 렌더링 (월~금, 09:00~21:00) |
| `TimetableCell.vue` | 개별 강의 블록 (색상, 팝업, 삭제) |
| `LectureSearchPanel.vue` | 강의 검색 결과 패널 (스켈레톤 로딩, 무한 스크롤) |
| `LectureCard.vue` | 강의 카드 (학점, 카테고리, 메타데이터 뱃지) |
| `FilterBar.vue` | 3탭 필터 (전공/교양/기초, 캠퍼스, 요일) |
| `ExportButton.vue` | PNG 내보내기 버튼 |
| `TimetableMini.vue` | 저장된 시간표 미니 프리뷰 |

### 5.3 상태 관리 (Pinia)

| 스토어 | 역할 |
|--------|------|
| `lectures.js` | 강의 목록, 필터, 페이지네이션, 교류 과목 병합 |
| `timetable.js` | 현재 시간표 강의 목록, 충돌 검사, LocalStorage 저장 |
| `user-settings.js` | 내 학과, 내 캠퍼스 설정 |

### 5.4 반응형 디자인

| 화면 크기 | 레이아웃 |
|-----------|----------|
| 1024px 이상 (데스크탑) | 2컬럼: 검색 패널(38%) + 시간표(62%) |
| 768~1023px (태블릿) | 2컬럼 축소 + 가로 스크롤 |
| 767px 이하 (모바일) | 탭 전환: 검색 / 시간표 |

---

## 6. 배포 환경

### 6.1 Docker 구성

| 구성 요소 | 베이스 이미지 | 설명 |
|-----------|-------------|------|
| Frontend | `nginx:alpine` | Vite 빌드 결과물 정적 서빙 |
| Backend | `node:20-alpine` | Express 서버 + SQLite |

### 6.2 배포 URL

| 구분 | URL |
|------|-----|
| Frontend | `https://iceweb.hufs.ac.kr/hufs-timetable/` |
| Backend | `https://iceweb.hufs.ac.kr/hufs-timetable-back/` |

### 6.3 CI/CD 파이프라인

```
GitHub Push → GitHub Actions → Docker Build → GHCR Push → ArgoCD → K8s 배포
```

- GitHub Actions로 Docker 이미지 자동 빌드
- GitHub Container Registry(GHCR)에 이미지 푸시
- ICE GitOps(ArgoCD)가 변경 감지 후 자동 배포

---

## 7. 디자인

### 7.1 컬러 시스템

HUFS 브랜드 컬러를 기반으로 6단계 블루 팔레트를 설계하였다.

| 이름 | 색상 코드 | 용도 |
|------|----------|------|
| Navy | `#002843` | 주요 텍스트, 네비게이션 |
| Dark | `#001a2e` | 강조 배경 |
| Blue | `#0a5e8a` | 주요 액센트, 링크 |
| Sky | `#1a8bc5` | 보조 액센트, 호버 |
| Light | `#e8f4fa` | 카드 배경, 뱃지 |
| Pale | `#f0f7fb` | 페이지 배경 |

### 7.2 UI 특징

- 둥근 모서리(`rounded-2xl`) + 카드 그림자로 모던한 느낌
- 파스텔 톤 강의 색상 (16가지 자동 배정)
- 페이지 전환 트랜지션 애니메이션
- 스켈레톤 로딩 UI (검색 중 로딩 상태)
- 토스트 메시지 (충돌 경고, 추가 완료 등)

## 8. 프로젝트 구조

```
hufs-timetable/
├── frontend/                    # Vue 3 SPA
│   ├── src/
│   │   ├── components/          # UI 컴포넌트 7개
│   │   ├── views/               # 페이지 뷰 3개
│   │   ├── stores/              # Pinia 스토어 3개
│   │   ├── composables/         # 재사용 로직
│   │   ├── utils/               # API 인스턴스, 색상 유틸
│   │   └── router/              # Vue Router 설정
│   ├── Dockerfile               # nginx 기반 배포
│   └── nginx.conf               # SPA 라우팅 설정
│
├── backend/                     # Express.js API 서버
│   ├── src/
│   │   ├── routes/              # API 라우터 4개
│   │   ├── services/            # 비즈니스 로직 3개
│   │   ├── models/              # DB 쿼리 함수 4개
│   │   ├── scraper/             # HUFS API 크롤러
│   │   ├── config/              # DB 연결 + 마이그레이션
│   │   ├── middleware/          # 에러 핸들러
│   │   └── db/                  # 스키마, 시드 스크립트
│   ├── data/                    # 시드 데이터, 메타데이터 JSON
│   ├── tests/                   # 단위 테스트
│   └── Dockerfile               # Node.js 기반 배포
│
├── .github/workflows/ci.yml    # GitHub Actions CI/CD
└── .gitignore
```

**코드 규모**: 64개 파일, 15,784줄

---

## 9. 향후 계획

| 항목 | 설명 | 우선순위 |
|------|------|----------|
| ICE GitOps 실배포 | GITOPS_TOKEN 설정 후 쿠버네티스 배포 | 높음 |
| 다크 모드 | 사용자 선호 테마 전환 | 중간 |
| 과목명 퍼지 매칭 | 메타데이터 과목명 유사도 매칭 개선 | 중간 |
| 학기 선택 기능 | 여름/겨울 학기 등 다학기 지원 | 낮음 |
| 친구 시간표 비교 | URL 공유로 시간표 겹침 확인 | 낮음 |

---

## 10. 결론

본 프로젝트는 한국외국어대학교 학생들의 수강신청 편의를 위해, 학교 API에서 실시간으로 강의 데이터를 수집하고 직관적인 시간표 편집 환경을 제공하는 웹 애플리케이션을 구현하였다.

**주요 성과**:
- 서울·글로벌 양 캠퍼스 **3,720개 강의**를 전공/교양/기초 구분으로 수집
- **3탭 필터링**으로 원하는 구분의 강의만 빠르게 탐색
- **시간 충돌 자동 감지**로 안전한 시간표 구성
- **수강유의교과목 메타데이터** 연동으로 필수/교류/금지 과목 시각적 안내
- Docker + CI/CD 파이프라인 구축으로 **자동 배포** 환경 완성

---

<div class="center" style="margin-top: 40px; color: #888; font-size: 9pt;">
GitHub Repository: https://github.com/oks706/Hufs_Time (Private)<br>
기술 스택: Vue 3 · Express.js · SQLite · Tailwind CSS · Docker
</div>
