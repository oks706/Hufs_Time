# HUFS 수강 시간표 빌더

한국외국어대학교 수강신청을 앞두고 강의를 찾아 시간표를 미리 짜 볼 수 있는 웹 서비스입니다. 학사정보시스템(WIS)의 강의 데이터를 모아 한 화면에서 검색하고, 시간이 겹치는지 바로 확인하면서 시간표를 만들 수 있습니다.

## 주요 기능

- **강의 데이터 수집**: 서울·글로벌캠퍼스의 전공·교양·기초 강의를 WIS에서 수집합니다. 2026년 3월 수집 기준 강의 3,720개, 학과·영역 192개입니다.
- **3탭 필터**: 전공/부전공 · 교양 · 기초 탭으로 나누고, 캠퍼스·학년·요일·키워드(과목명, 교수명)로 좁혀 볼 수 있습니다.
- **시간표 편집**: 월~금 09:00~21:00 그리드에 강의를 넣으면 시간 충돌을 막고, 과목마다 색을 자동으로 붙입니다.
- **수강유의 교과목 표시**: 내 학과를 설정하면 전공필수·전공교류·수강금지 과목을 뱃지로 보여 주고, 수강금지 과목은 추가하지 못하게 합니다.
- **저장과 내보내기**: 시간표 여러 개를 브라우저에 저장하고, PNG 이미지로 내보낼 수 있습니다.

## 기술 스택

| 영역 | 기술 |
|---|---|
| 프론트엔드 | Vue 3, Vite, Pinia, Vue Router, Tailwind CSS, html2canvas |
| 백엔드 | Node.js 20, Express, Axios, Helmet |
| 데이터베이스 | PostgreSQL (`pg`) |
| 배포 | Docker (nginx, Node 20), GitHub Actions → GHCR |

## 폴더 구조

```
backend/
  src/routes, services, models   API (라우터 → 서비스 → 모델)
  src/scraper/                   WIS 강의 데이터 수집기
  src/db/                        스키마(schema.sql)와 시드 스크립트
  data/                          시드 강의 데이터, 수강유의 교과목 JSON
  scripts/parse-xls.js           수강유의 교과목 XLS → JSON 변환
  tests/                         단위 테스트
frontend/
  src/views, components          시간표 편집, 강의 검색, 저장 목록 화면
  src/stores/                    Pinia 스토어 (강의, 시간표, 사용자 설정)
docs/project-report.md           프로젝트 보고서
```

## 로컬에서 실행하기

Node.js 20 이상과 PostgreSQL이 필요합니다.

### 1. 데이터베이스

PostgreSQL이 없다면 Docker로 띄울 수 있습니다.

```bash
docker run -d --name hufs-time-db -e POSTGRES_PASSWORD=postgres -p 5432:5432 postgres:16
```

### 2. 백엔드

```bash
cd backend
npm install
export DATABASE_URL=postgresql://postgres:postgres@localhost:5432/postgres
npm run seed            # 테이블을 만들고 data/seed-lectures.json으로 강의 데이터를 채움
npm run seed-metadata   # 전공필수·전공교류·수강금지 데이터를 채움
npm run dev             # http://localhost:8080
```

PowerShell에서는 `$env:DATABASE_URL = "postgresql://..."`로 설정합니다. 서버는 시작할 때 `src/db/schema.sql`로 테이블을 만들기 때문에, 시드를 넣지 않아도 빈 DB로 실행됩니다.

### 3. 프론트엔드

```bash
cd frontend
npm install
npm run dev             # http://localhost:5173/hufs-timetable/
```

API 주소는 `frontend/.env.development`의 `VITE_API_URL`(기본값 `http://localhost:8080`)입니다.

### 테스트

```bash
cd backend
npm test
```

## 강의 데이터 새로 받기

백엔드가 켜진 상태에서 스크래퍼를 실행하면 WIS에서 강의 데이터를 새로 받아 DB를 교체합니다. 수집은 백그라운드에서 진행되고, 학교 서버에 부담을 주지 않도록 요청 사이에 간격을 두기 때문에 8~10분쯤 걸립니다.

```bash
curl -X POST http://localhost:8080/api/scraper/run -H "Content-Type: application/json" -d '{"mode":"live"}'
curl http://localhost:8080/api/scraper/status    # 최근 수집 기록과 진행 상태
```

`{"mode":"seed"}`로 실행하면 WIS 대신 `data/seed-lectures.json`을 불러옵니다.

수강유의 교과목은 학교에서 배포하는 XLS 파일(`2026-1학기 글로벌캠퍼스 수강유의교과목.xls`)을 저장소 루트에 두고 아래처럼 갱신합니다.

```bash
cd backend
npm run parse-xls       # data/ 아래 JSON 3개를 새로 만듦
npm run seed-metadata
```

## API

| 메서드 | 경로 | 설명 |
|---|---|---|
| GET | `/api/lectures` | 강의 목록 (gubun, department, year, category, day, keyword, page, limit) |
| GET | `/api/lectures/categories` | 카테고리 목록 |
| GET | `/api/lectures/search` | 키워드 검색 (q, page, limit) |
| GET | `/api/lectures/:id` | 강의 상세 |
| GET | `/api/departments` | 학과 목록 (campus) |
| GET | `/api/metadata/cross-major` | 전공교류 강의 (department) |
| POST | `/api/scraper/run` | 강의 데이터 수집 (mode: live, seed) |
| GET | `/api/scraper/status` | 최근 수집 기록 |
| GET | `/api/health` | 서버 상태 확인 |

## 문서

설계와 화면 구성은 [프로젝트 보고서](docs/project-report.md)에 자세히 있습니다. 보고서는 첫 버전 기준이라 DB를 SQLite로 적고 있고, 지금 코드는 PostgreSQL을 씁니다.
