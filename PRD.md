# PRD: YouTube 경제 뉴스 요약 서비스

> 작성일: 2026-03-03
> 상태: 초안

---

## 프로젝트 개요

- **프로젝트명**: EconDigest (가칭)
- **목적**: YouTube 경제/금융 채널 영상을 자동으로 수집·요약하고, Notion에 저장된 콘텐츠를 웹에서 열람
- **CMS 선택 이유**: Notion API를 활용하면 비개발자도 콘텐츠 수정·큐레이션이 가능하고, 별도 DB 관리 없이 운영 가능

---

## 주요 기능

1. **자동 수집 파이프라인**: 지정된 YouTube 채널의 최신 영상을 주기적으로 수집하고 AI로 요약해 Notion에 저장
2. **웹 뷰어**: Notion에 저장된 요약 콘텐츠를 카드 형태로 조회 (채널별·날짜별 필터)
3. **상세 페이지**: 영상별 AI 요약, 키워드, 원본 링크 제공

---

## 기술 스택

| 영역 | 기술 |
|------|------|
| Frontend | Next.js 15, TypeScript |
| CMS | Notion API |
| Styling | Tailwind CSS, shadcn/ui |
| Icons | Lucide React |
| Pipeline | Python (백그라운드 배치) |
| AI | Claude API (요약), Whisper (전사) |

---

## Notion 데이터베이스 구조

| 필드명 | 타입 | 설명 |
|--------|------|------|
| 이름 | title | 페이지 제목 (채널명 + 날짜) |
| 날짜 | date | 영상 처리 날짜 |
| 채널 | select | YouTube 채널명 |
| 영상제목 | rich_text | 원본 YouTube 영상 제목 |
| 영상URL | url | YouTube 링크 |
| 요약 | rich_text | AI 핵심 요약 |
| 키워드 | multi_select | 주요 키워드 태그 |
| 공개여부 | checkbox | 웹 노출 여부 (Notion에서 직접 관리) |

---

## 화면 구성

- **홈(목록) 화면**: 최신 요약 카드 그리드, 채널 필터, 날짜 정렬
- **상세 화면**: 요약 전문, 키워드, 원본 영상 임베드(또는 링크)

---

## MVP 범위

- Python 파이프라인: 채널 설정 → YouTube RSS 수집 → AI 요약 → Notion 저장
- Next.js 웹: Notion DB 읽기 → 목록 페이지 → 상세 페이지
- Notion `공개여부` 체크박스로 콘텐츠 노출 제어

> MVP에서 제외: 검색, 사용자 인증, 댓글, 알림 기능

---

## 구현 단계

1. **파이프라인 재구성**: 기존 코드를 정리하고 채널 설정·AI 요약·Notion 저장 흐름 안정화
2. **Next.js 프로젝트 세팅**: Notion API 연동, 목록·상세 페이지 구현
3. **배포 & 스케줄링**: Vercel(웹) + cron/GitHub Actions(파이프라인) 연결

---

## 미결 사항 (추후 결정)

- 파이프라인 실행 주기 (1회/일 vs 수시)
- 웹 호스팅 방식 (Vercel / 자체 서버)
- 요약 스타일 및 분량 기준
