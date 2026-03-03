# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

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
npm run dev       # 개발 서버 (Turbopack)
npm run build     # 프로덕션 빌드
npm run start     # 프로덕션 서버
npm run lint      # ESLint
```

### Pipeline (Python) — `pipeline/` 디렉토리에서 실행
```bash
source venv/bin/activate          # 가상환경 활성화
pip install -r requirements.txt   # 의존성 설치
python main.py                    # 파이프라인 실행
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

- `pipeline/config/channels.yaml` — 수집 대상 YouTube 채널 목록 (channel_id, name, enabled)
- `pipeline/src/` — 수집·요약·저장 모듈
- `web/src/app/` — Next.js App Router 페이지
- `web/src/lib/` — 유틸리티 및 Notion API 클라이언트
- `web/src/components/` — UI 컴포넌트 (shadcn/ui 기반)

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
