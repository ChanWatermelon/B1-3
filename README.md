# 자동화 도구 실습 과제 — 제출물

> 작성일: 2026-07-14 · 계정(마스킹): `de****@gmail.com`
> 워크플로우 주제: **"워크숍 참가 신청 자동 심사·기록·안내"**
> 사용 도구: **Make** · **Zapier** · Google Forms/Sheets · Discord · Google Gemini(AI)

---

## 📦 제출 문서

| 파일 | 내용 | 비고 |
|------|------|------|
| **[04_프로젝트1_비교분석보고서.md](04_프로젝트1_비교분석보고서.md)** | 프로젝트 1 — Make vs Zapier 비교 분석 | ✅ 제출물 |
| **[05_프로젝트2_자유주제.md](05_프로젝트2_자유주제.md)** | 프로젝트 2 — 자유주제 자동화 설계·구현(화면 포함) | ✅ 제출물 |

### 🖼 첨부 이미지 (실행 증빙)
| 파일 | 내용 |
|------|------|
| `make.png` | Make 워크플로우 전체 구성 (Forms→Gemini→Router→Sheets→Discord) |
| `googlesheet.png` | 트리거 입력 (Google Forms 제출: 홍길동 / 92점) |
| `gemini.png` | (보너스1) Gemini AI 맞춤 안내 메시지 생성 결과 |
| `zaps.png` | Zapier 실행 로그 (전 단계 성공) |
| `discord.jpg` | Discord 채널 알림 도착 (Make/Zapier 공통) |

---

## 🔁 구현한 워크플로우

**"워크숍 참가 신청 자동 심사·기록·안내"**

```
[Trigger] Google Forms 새 응답
   → [AI] Gemini 맞춤 안내 메시지 생성 (보너스1)
   → [조건 분기] 사전 퀴즈 점수 80점 기준
        ├─ ≥80 (승인) → Sheets '승인자' 기록 → Discord 승인 알림
        └─ <80 (대기) → Sheets '대기자' 기록 → Discord 보완 안내
   → (보너스2) 오류 시 관리자 알림 + '오류로그' 시트 적재
```

- **프로젝트 1**: 위 워크플로우를 **Make와 Zapier 두 도구**로 동일 구현 → 비교 분석
- **프로젝트 2**: 위 워크플로우를 **Make 단독 도구**로 선정한 자유주제 자동화로 구현

---

## ✅ 요구사항 충족 표

| 요구사항 | 충족 내용 |
|----------|-----------|
| Trigger 1개 이상 | Google Forms 새 응답 감지 |
| Action 2개 이상 | Gemini 생성 + Sheets 기록 + Discord 알림 (3+) |
| 조건 분기 1개 이상 | 점수 80점 기준 (Make=Router / Zapier=Filter) |
| 각 분기 1회 이상 실행 | 승인(92점)·대기(65점) 각각 제출·실행 |
| 프로젝트 1: 도구 2개 이상 | Make + Zapier |
| 프로젝트 1: 비교 항목 5개 이상 | **9개** 항목 비교 |
| 프로젝트 2: 도구 1개 선정 + 이유 | Make 선정 (분기·에러·AI 무료 통합) |
| 프로젝트 2: 자동 실행 | 폼 제출 시 자동 실행 (Scheduling ON) |
| 보너스1: AI 연동 Action | Google Gemini 맞춤 메시지 생성 |
| 보너스2: 실패 알림·대체 경로 | Error handler 알림 + 임시 시트 적재 |

---

## 💰 비용 / 보안

- **전부 무료 플랜으로 완수** (Make Free, Zapier Free, Gemini 무료 API, Discord 무료).
- 유료 기능 대안: Zapier Paths(유료) → Filter+Zap 2개 / Webhooks(유료) → Forms 네이티브 트리거.
- 모든 스크린샷의 **API Key·토큰·웹훅 URL은 `***` 마스킹**, 계정 이메일은 일부 가림 처리.

---

## 📄 제출 방법
1. `04`, `05` 문서를 VS Code 미리보기(Ctrl+Shift+V)로 이미지 확인
2. "Markdown PDF" 확장 등으로 **PDF 변환** 후 제출
3. 이미지 파일(`*.png`, `*.jpg`)은 문서와 같은 폴더에 유지
