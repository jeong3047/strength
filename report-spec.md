# 보고서 생성 스펙 (report-spec)

> **목적**: `meeting/` · `workboard/` (+ 갤럽 선택) 소스를 읽어 **조직 보고서 1개 + 파트원별 보고서 N개**를 한 번에 HTML로 생성하기 위한 출력 계약.
> 분석 원리·절차는 [`guide.md`](guide.md)를 따른다. 이 문서는 **"무엇을, 어떤 모양으로 출력하나"**만 정의한다.
>
> 실행 방법: Claude Code면 `/strength-report` (→ `.claude/commands/strength-report.md`), 그 외 LLM이면 이 문서 + `guide.md`를 그대로 프롬프트로 넣는다.

---

## 1. 입력 (소스)

| 소스 | 위치 | 필수 | 용도 |
| --- | --- | --- | --- |
| 면담 기록 | `meeting/*-meeting.md` | ✅ | 동기·몰입·회피·자기인식 |
| 업무 이력 | `workboard/*.csv` | ✅ | 활동·도메인 친숙도 |
| 갤럽 34 | (외부 PDF/텍스트) | ⬜ 선택 | 재능 1차 신호 — 없으면 2소스로 진행 + "신뢰도 낮음(잠정)" 명시 |
| 실적 지표(KR) | 있으면 | ⬜ 선택 | 드러난 성과. 미측정이면 placeholder(`__`) |

**데이터 무결성 (반드시 준수)**
- 소스에 없는 사실을 **지어내지 않는다.** 모르는 수치는 placeholder(`<span class="ph">__</span>`)로 둔다.
- 소스가 충돌하면 더 강한 신호로 덮지 말고 **`.note`로 플래그**를 남긴다.
- 활동 "건수"로 우열을 매기지 않는다(친숙도 신호일 뿐).

---

## 2. 출력 (산출물)

`frontend-scoreboard/` 에 다음을 쓴다 (이 폴더는 로컬 전용 — 결과물엔 실명·개인정보 포함):

| 파일 | 종류 | 내용 |
| --- | --- | --- |
| `index.html` | **조직 보고서** | 팀 전략·노스스타·KR 롤업·인물별 월간 트래킹 |
| `member-<id>.html` | **파트원 보고서** | 1인당 1개. 강점 카드·현황·담당 KR. `<id>`는 영문 슬러그 |

- 모두 **self-contained 단일 HTML** (인라인 `<style>`, 외부 JS 없음).
- index → 각 member 링크, member → `index.html` 백링크 필수.

---

## 3. 공통 HTML 골격

`<head>`는 두 종류 페이지 공통:

```html
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>{{페이지 제목}}</title>
<link rel="preconnect" href="https://fonts.googleapis.com"><link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Gowun+Batang:wght@400;700&family=IBM+Plex+Sans+KR:wght@300;400;500;600;700&display=swap" rel="stylesheet">
<style>
  :root{--ink:#1a1714;--ink-soft:#52483f;--paper:#f4efe6;--paper-2:#ece4d6;--line:#d8cdb9;--accent:#9a3412;--accent-2:#3f6212;--gold:#a16207;--shadow:rgba(40,28,16,.12)}
  *{box-sizing:border-box;margin:0;padding:0}
  body{font-family:'IBM Plex Sans KR',sans-serif;background:radial-gradient(circle at 12% 8%,#f9f5ec,transparent 45%),radial-gradient(circle at 88% 92%,#efe7d8,transparent 40%),var(--paper);color:var(--ink);line-height:1.7;-webkit-font-smoothing:antialiased}
  .wrap{max-width:860px;margin:0 auto;padding:0 28px}
  .back{display:inline-block;margin:28px 0 0;font-size:.82rem;color:var(--accent);text-decoration:none;font-weight:600}
  .back:hover{text-decoration:underline}
  header.mast{padding:14px 0 38px;border-bottom:2px solid var(--ink)}
  .kicker{font-size:.7rem;letter-spacing:.36em;text-transform:uppercase;color:var(--accent);font-weight:600;margin-bottom:18px}
  h1{font-family:'Gowun Batang',serif;font-weight:700;font-size:clamp(2.1rem,5vw,3.1rem);line-height:1.1;margin-bottom:12px}
  h1 span{color:var(--accent);font-size:.5em;font-family:'IBM Plex Sans KR';font-weight:500;display:inline-block;margin-left:10px;vertical-align:middle}
  .dek{font-size:1.02rem;color:var(--ink-soft);max-width:620px;font-weight:300}
  .dek b{color:var(--ink);font-weight:600}
  section{padding:42px 0;border-bottom:1px solid var(--line)}
  .sec-no{font-family:'Gowun Batang',serif;font-size:.85rem;color:var(--accent);letter-spacing:.28em;margin-bottom:10px;display:block}
  h2{font-family:'Gowun Batang',serif;font-weight:700;font-size:clamp(1.4rem,3.2vw,1.9rem);margin-bottom:20px}
  .grid{display:grid;grid-template-columns:1fr 1fr;gap:16px}
  .card{background:linear-gradient(160deg,#fbf8f1,#f0e8d8);border:1px solid var(--line);border-radius:5px;padding:20px 22px}
  .card .tag{font-size:.64rem;letter-spacing:.14em;text-transform:uppercase;color:var(--accent-2);font-weight:600;margin-bottom:8px;display:block}
  .card h3{font-size:1.04rem;font-weight:600;margin-bottom:6px}
  .card p{font-size:.85rem;color:var(--ink-soft)}
  .two{display:grid;grid-template-columns:1fr 1fr;gap:18px}
  .box{border:1px solid var(--line);border-radius:5px;padding:20px 22px;background:#fbf8f1}
  .box .l{font-size:.64rem;letter-spacing:.14em;text-transform:uppercase;color:var(--gold);font-weight:600;margin-bottom:10px}
  .box ul{list-style:none}
  .box li{font-size:.88rem;color:var(--ink-soft);padding-left:14px;position:relative;margin-bottom:6px}
  .box li::before{content:"›";position:absolute;left:0;color:var(--accent)}
  .box li b{color:var(--ink)}
  .track{width:100%;border-collapse:collapse;font-size:.86rem}
  .track th,.track td{border:1px solid var(--line);padding:10px 12px;text-align:left}
  .track th{background:var(--paper-2);font-size:.64rem;letter-spacing:.1em;text-transform:uppercase;color:var(--ink-soft);font-weight:600}
  .ph{color:var(--ink-soft);background:#fff;border:1px dashed var(--line);border-radius:5px;padding:1px 8px;font-size:.8rem}
  .note{font-size:.88rem;color:var(--accent);background:#fff3ec;border:1px solid #e8cdbd;border-radius:6px;padding:14px 18px;margin-top:20px}
  .note b{color:var(--ink)}
  footer{padding:34px 0 70px;text-align:center}
  @media(max-width:640px){.grid,.two{grid-template-columns:1fr}}
</style>
</head>
```

> 미측정 수치는 `<span class="ph">__</span>`, 충돌·재평가 플래그는 `<div class="note">…</div>`.

---

## 4. 조직 보고서 (`index.html`) 섹션

순서대로 `<section>`으로 구성. 각 섹션 머리에 `<span class="sec-no">01 — …</span>`.

1. **히어로** — 팀 전략을 한 문장으로 (`header.mast` + `h1` + `.dek`). 무엇이 플라이휠/엔진인지.
2. **전략 흐름** — 입력 → 단계 → 성과로 이어지는 흐름 설명.
3. **노스스타 정의** — 왜 이 지표가 팀의 단일 입력인가 (프론트가 통제 가능한지 근거).
4. **결과 지표** — 노스스타를 움직이는 하위 KR 목록.
5. **각자의 KR과 롤업** — 누가 어떤 KR의 단독 오너인지 매핑 (성과는 N분의 1 금지).
6. **인물별 월간 트래킹** — `.track` 테이블: 인물 × 지표 × 월 목표/성과 (미측정은 `.ph`).
7. **핵심 실행** — "이게 정해져야 숫자가 돈다" — 가장 먼저 깔아야 할 계측/선결 과제.

각 인물명은 `member-<id>.html`로 링크.

---

## 5. 파트원 보고서 (`member-<id>.html`) 섹션

상단에 `<a class="back" href="index.html">← 조직 보고서</a>`, 그 뒤 `header.mast`(이름 + 역할 `h1 span` + `.dek`).

1. **강점** (`.grid` + `.card`) — 면담·업무이력(·갤럽) 교차로 도출. 카드 태그 규칙:
   - `Core` 핵심 재능/동기 · `Craft` 숙련·실행 강점 · `Meta` 자기인식·작동 방식 · `Caution` **회피·약점 영역(직무 주축에서 제외)**
2. **현황 & 담당** (`.two` + `.box`) — 좌: 현황, 우: 레버(강점이 성과로 가는 통로).
3. **담당 KR · 월간** (`.track`) — 본인 오너 KR의 월 목표/성과 (미측정 `.ph`).
4. (선택) **`.note`** — 흥미 정렬 근거 또는 소스 충돌·재평가 플래그.

> 약점(`Caution`)은 사람 탓이 아니라 **직무에서 빼거나 구조(페어·체크리스트·오너 분리)로 덮는다**는 점을 현황/레버에 반영한다.

---

## 6. 완료 기준

- [ ] `index.html` 1개 + 면담이 있는 인원 수만큼 `member-*.html` 생성
- [ ] index ↔ member 링크가 양방향으로 연결됨
- [ ] 지어낸 수치 없음 (모르는 건 `.ph`)
- [ ] 강점 → 직무 → KR(측정 지표) 사슬이 각 인물에 이어짐
- [ ] 성과마다 단독 오너 / 충돌은 `.note` 플래그 / 재평가 기한 명시
