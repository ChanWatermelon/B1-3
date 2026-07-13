# 미국장 마감 후 텔레그램 다이제스트 자동화

개인 투자자를 위해 **매일 미국장 마감 후, 포트폴리오 성과와 시장 상태를 텔레그램으로 자동 발송**하는 프로젝트입니다.
KIS OpenAPI·yfinance·Finviz·CNN Fear&Greed·Google Gemini 등 여러 데이터 소스를 하나의 파이프라인으로 통합하고, Raspberry Pi 4에서 24시간 무인 운영합니다.

- **환경**: Raspberry Pi 4 (Ubuntu, ARM64)
- **실행 주기**: 매일 07:00 KST (NYSE 거래일에만)
- **제출일**: 2026-07-13

---

## 프로젝트 구성

| 문서 | 내용 |
|------|------|
| [프로젝트 1 — 자동화 도구 비교 구현](프로젝트1_자동화도구_비교구현.md) | Python + APScheduler(자체 구현) vs Make.com(No-code) 비교 분석 |
| [프로젝트 2 — 자유 주제 자동화 설계 및 구현](프로젝트2_자유주제_자동화.md) | 다이제스트 자동화의 워크플로우 설계·구현·실행 결과 |

---

## 핵심 워크플로우

```
Trigger (매일 07:00 KST)
      │
      ▼
Filter #1  ─ NYSE 거래일인가?  ─── 아니오 → skip + 로그
      │ 예
      ▼
Action 1  ─ Finviz 트리맵 캡처 (Playwright)
      ▼
Action 2  ─ 포트폴리오 데이터 수집 (KIS 4계좌 + yfinance + 환율)
      ▼
Filter #2  ─ 자산 유형별 라우팅 (국내/해외/외화예수금/외화 RP)
      ▼
Action 3  ─ 이미지 생성 (Plotly 트리맵 / 환율 차트, Kaleido PNG)
      ▼
Action 4  ─ Fear & Greed Index 조회 (CNN dataviz API)
      ▼
Action 5  ─ 텔레그램 순차 전송 (사진 5장 + 텍스트 2건)
```

- **Trigger**: APScheduler `cron(hour=7, minute=0)` (Asia/Seoul), `misfire_grace_time=3600`으로 누락 잡 복구
- **Filter #1**: `pandas_market_calendars`로 NYSE 거래일 판정 → 주말·미국 공휴일 자동 skip
- **Filter #2**: 자산 유형별 KIS API TR ID·엔드포인트 라우팅
- **Actions**: 각 단계는 `try/except`로 격리 실행 → 한 단계 실패가 전체 실패로 이어지지 않음

---

## 사용 기술

| 영역 | 도구 |
|------|------|
| 스케줄링 | APScheduler (`BackgroundScheduler`) |
| 거래일 판정 | pandas_market_calendars (NYSE) |
| 잔고 조회 | KIS OpenAPI (국내/해외 주식, 외화 예수금, 외화 RP) |
| 시세 | yfinance, pykrx |
| 웹 캡처 | Playwright + Chromium (headless) |
| 이미지 생성 | Plotly + Kaleido (PNG) |
| 시장 심리 | CNN Fear & Greed Index API |
| AI 분석 | Google Gemini (`gemini-2.5-flash`) |
| 전송 | Telegram Bot API (`sendPhoto` / `sendMessage`) |
| 대시보드 | Streamlit (8개 탭) |

---

## 도구 비교 요약 (프로젝트 1)

| 항목 | Python + APScheduler (자체) | Make.com (No-code) |
|------|------------------------------|--------------------|
| 설정 난이도 | Python 기본기 필요 | 계정 생성만으로 시작 |
| 무료 범위 | **완전 무료** (전기값만) | 1,000 ops/월 |
| 데이터 프라이버시 | 모든 데이터 로컬 유지 | API 키가 SaaS 서버 저장 |
| 복잡 로직·이미지·AI | 자유도 압도적 | 노드 폭증, 관리 부담 |
| 외부 의존성 | 라즈베리파이만 있으면 자체 실행 | 서비스 가용성에 종속 |

**결론**: 개인 금융 데이터·복합 파이프라인·무료 무제한 실행이 요구되어 **Python 자체 구현**을 선택. 팀 협업·시제품·단순 알림에는 Make.com이 유리.

---

## 실행 결과

### 텔레그램 전송 화면 (매일 07:00 KST 자동 발송)

![텔레그램 전송 화면](텔레그램_전송.jpg)

### Streamlit 대시보드 홈

![Streamlit 대시보드](streamlit_00_홈_초기로드.png)

### 자동 실행 이력 (nohup.out 발췌)

```
[2026-05-26 07:00:01] KST — NYSE trading day (Tue) → digest 실행
  ✓ Finviz 캡처 4/4, 트리맵·차트·섹터맵 전송, F&G·P/L 메시지 전송 (총 47.2초)
[2026-05-25 07:00:01] KST — Memorial Day (미국 휴장) → skip
[2026-05-24 07:00:01] KST — Saturday (주말 휴장) → skip
```

---

## 보너스 구현

- **AI 연동**: Streamlit "🤖 AI 분석" 탭 — 포트폴리오 분석, 개별 종목 기술적 분석(RSI/MACD/볼린저 등), 뉴스 톤 분석. 모델 혼잡 시 자동 fallback.
- **실패 대응**: `misfire_grace_time`으로 1시간 내 복구, Action별 격리 실행, KIS 토큰 캐시 자동 갱신.

---

## 리스크 관리

- **민감 정보**: `data/secrets.json`, `data/accounts.json`, `data/tokens/` → `.gitignore` 등록. 스크린샷은 계좌번호·토큰 마스킹.
- **과금**: KIS·yfinance·Gemini(무료 티어)·Telegram·Cloudflare Tunnel 모두 무료 → 전기값(월 약 500원)을 제외하면 **완전 무료** 운영.

