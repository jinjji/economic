# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Project Context

- PRD 문서: @docs/PRD.md
- 개발 로드맵: @docs/ROADMAP.md

---

## 프로젝트 개요

EconDigest — YouTube 경제/금융 채널 영상을 자동 수집·요약하여 Notion에 저장하고, 웹에서 열람하는 서비스.
Monorepo 구조로 `pipeline/`(Python 백엔드)과 `web/`(Next.js 프론트엔드) 두 모듈로 구성.

## 아키텍처

```
YouTube RSS → [pipeline] 자막 추출 → Claude API 요약 → Notion DB 저장
                                                            ↓
                                          [web] Notion API 읽기 → 카드 목록 / 상세 페이지
```

- **CMS**: Notion API를 CMS로 활용. 별도 DB 없이 Notion 데이터베이스가 유일한 데이터 저장소
- **공개여부 필드**(checkbox)로 웹 노출 제어 — Notion에서 직접 토글

## 명령어

### Web (Next.js) — `web/` 디렉토리에서 실행
```bash
npm install                  # 의존성 설치
npm run dev                  # 개발 서버 실행 (localhost:3000)
npm run build                # 프로덕션 빌드
npm run start                # 빌드된 프로덕션 서버 실행
npm run lint                 # ESLint 실행
npx shadcn-ui add <name>     # shadcn/ui 컴포넌트 추가
```

### Pipeline (Python) — `pipeline/` 디렉토리에서 실행
```bash
source venv/bin/activate          # 가상환경 활성화
pip install -r requirements.txt   # 의존성 설치
python main.py                    # 파이프라인 실행
```

### Monorepo 초기 설정
```bash
# 1. 루트에서 web 의존성 설치
cd web && npm install

# 2. pipeline 가상환경 설정
cd ../pipeline
python3.13 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# 3. 환경변수 설정
# web/.env.local 및 pipeline/.env 에 API 키 추가
```

## 기술 스택

| 영역 | 기술 |
|------|------|
| Frontend | Next.js 15 (App Router, RSC), React 19, TypeScript 5 |
| Styling | Tailwind CSS v4, shadcn/ui (New York style), Lucide React |
| CMS/DB | Notion API (`@notionhq/client`) |
| Pipeline | Python 3.13, feedparser, youtube-transcript-api |
| AI | Anthropic Claude API (`anthropic` SDK) |

## 프로젝트 구조

### Web (Next.js)
- `web/src/app/` — App Router 페이지 (`layout.tsx`, `page.tsx` 등)
- `web/src/components/` — UI 컴포넌트 (shadcn/ui 기반, `ui/` 서브디렉토리)
- `web/src/lib/` — 유틸리티 및 Notion API 클라이언트 래퍼
  - `notion.ts` — Notion API 클라이언트 초기화 및 타입 정의
  - `utils.ts` — cn() 등 공통 유틸 함수
- `web/public/` — 정적 에셋
- `web/components.json` — shadcn/ui 설정 (path alias, style 등)
- `web/.env.local` — 환경변수 (`.env.local.example` 참조)

### Pipeline (Python)
- `pipeline/config/channels.yaml` — 수집 대상 YouTube 채널 목록 (channel_id, name, enabled)
- `pipeline/src/` — 수집·요약·저장 모듈
  - `services/` — 핵심 로직 (youtube_service.py, summarizer.py, notion_service.py 등)
  - `main.py` — 파이프라인 메인 엔트리포인트
- `pipeline/.env` — 환경변수 (`.env.example` 참조)

## Notion 데이터베이스 스키마

이름(title), 날짜(date), 채널(select), 영상제목(rich_text), 영상URL(url), 요약(rich_text), 키워드(multi_select), 공개여부(checkbox)

## 코딩 규칙

- 모든 주석, 커밋 메시지, 문서는 **한국어**로 작성
- 변수명/함수명은 영어 camelCase
- TypeScript path alias: `@/*` → `./src/*`
- 들여쓰기 2칸 스페이스, 함수 30줄 이하 유지
- 매직넘버 금지 — 상수로 정의

## 환경변수

- `pipeline/.env`: `NOTION_API_KEY`, `NOTION_DATABASE_ID`, `ANTHROPIC_API_KEY`
- `web/.env.local`: `NOTION_API_KEY`, `NOTION_DATABASE_ID`
- `.env.example` / `.env.local.example` 파일 참조

---

## Web 개발 가이드

### shadcn/ui 컴포넌트 추가
```bash
cd web
npx shadcn-ui add button    # 버튼 컴포넌트 추가
npx shadcn-ui add card      # 카드 컴포넌트 추가
# 추가된 컴포넌트는 src/components/ui/ 에 저장됨
```

### Notion API 래퍼 사용
- `src/lib/notion.ts` 에서 `notionClient` 인스턴스를 export
- 페이지/컴포넌트에서 `notionClient.databases.query()` 호출
- 타입 안전성을 위해 `PageProperties` 등의 타입 정의 필요

### TypeScript Path Alias
- `@/*` → `./src/*` 로 설정됨 (`tsconfig.json` 참조)
- 예: `import { Button } from '@/components/ui/button'`

### 스타일링
- Tailwind CSS v4로 스타일 작성 (`src/app/globals.css`)
- shadcn/ui는 CSS variables 기반이므로 테마 커스터마이징 가능
- `cn()` 유틸 함수로 클래스명 병합 (조건부 스타일 등)

---

## Pipeline 개발 가이드

### 구조 설계 (MVP 기준)
파이프라인은 다음 단계를 따름:
1. **YouTube RSS 수집** → feedparser로 RSS 파싱
2. **자막 추출** → youtube-transcript-api 사용
3. **AI 요약** → Anthropic Claude API 호출
4. **Notion 저장** → Notion API로 데이터베이스에 저장

각 단계는 `services/` 디렉토리의 모듈로 분리 권장:
- `youtube_service.py` — RSS 수집 및 자막 추출
- `summarizer.py` — Claude API 요약
- `notion_service.py` — Notion DB 읽기/쓰기
- `config_loader.py` — channels.yaml 파싱

### 환경변수 로딩
```python
from dotenv import load_dotenv
load_dotenv()
api_key = os.getenv('ANTHROPIC_API_KEY')
```

### 에러 처리
- API 호출 실패 시 재시도 로직 구현 권장
- 로깅은 `logging` 표준 라이브러리 사용
- 부분 실패 처리: 한 영상 요약 실패 시 다음 진행

### Notion 클라이언트 래퍼
- `services/notion_service.py` 에서 재사용 가능한 함수로 구현
- 데이터베이스 쿼리, 생성, 업데이트를 메서드로 분리
- 웹 모듈과 같은 스키마 사용

---

## 로컬 개발 환경 체크리스트

### 초기 설정 (처음 개발 시)
- [ ] Python 3.13 설치 확인 (`python --version`)
- [ ] Node.js 18+ 설치 확인 (`node --version`)
- [ ] Notion API 키 발급 및 데이터베이스 생성
- [ ] Claude API 키 발급 (Anthropic 콘솔)
- [ ] YouTube 채널 URL 수집
- [ ] `web/.env.local` 및 `pipeline/.env` 파일 작성

### 개발 시작
```bash
# 터미널 1: Web 개발 서버
cd web && npm run dev

# 터미널 2: Pipeline 테스트
cd pipeline
source venv/bin/activate
python src/main.py
```

---

## 주의사항

### API 비용 및 할당량
- **Claude API**: 토큰 기반 과금 (요약 길이, 영상 개수 모니터링 필수)
- **Notion API**: 월 1,000 요청 무료 (캐싱 권장)
- **YouTube API**: 불필요 (RSS 피드 사용으로 API 할당량 회피)

### Notion 스키마 변경 시
- 파이프라인 및 웹의 타입 정의를 모두 업데이트
- 데이터베이스 필드 추가/삭제는 Notion UI에서 직접 관리

### 공개여부 필드 활용
- Notion에서 checkbox로 표시된 "공개여부" 필드로 노출 제어
- 웹에서 조회 시 `공개여부 == true` 인 항목만 필터링
