---
description: 소스(meeting·workboard·갤럽 선택)로 조직 보고서 + 파트원별 보고서를 frontend-scoreboard/ HTML로 일괄 생성
---

# 강점 분석 보고서 일괄 생성

`meeting/` · `workboard/`(+ 있으면 갤럽)을 읽어 **조직 보고서 1개 + 파트원별 보고서 N개**를 `frontend-scoreboard/`에 HTML로 생성한다.

## 절차

1. **소스 점검** — `meeting/*-meeting.md`와 `workboard/*.csv`가 채워졌는지 확인.
   - 면담 파일이 하나도 없으면 멈추고 "소스를 먼저 채우라"고 안내한다.
   - 갤럽 텍스트/PDF가 인자나 작업폴더에 있으면 3소스, 없으면 2소스로 진행하고 각 인물 보고서에 **"갤럽 없음 → 잠정 신뢰도"**를 `.note`로 표기한다.

2. **분석** — [`guide.md`](../../guide.md)의 절차를 따른다:
   - 인물별로 `갤럽 테마(있으면) ↔ 업무 증거 ↔ 면담 발언`을 교차(triangulation).
   - 2~3소스가 같은 곳을 가리키면 강점 확정, 충돌하면 플래그.
   - 강점 → 직무 → 측정 지표(KR) 사슬을 잇고, 성과마다 **단독 오너**를 둔다.

3. **생성** — [`report-spec.md`](../../report-spec.md)의 출력 계약(공통 `<style>`, 섹션 골격, 데이터 무결성 규칙)을 그대로 따라:
   - `frontend-scoreboard/index.html` — 조직 보고서
   - 면담이 있는 인원마다 `frontend-scoreboard/member-<id>.html` (`<id>` = 영문 슬러그)
   - index ↔ member 양방향 링크.

## 철칙

- **소스에 없는 사실·수치는 지어내지 않는다.** 모르는 수치는 `<span class="ph">__</span>`.
- 소스 충돌은 덮지 말고 `.note`로 남긴다. 활동 "건수"로 우열 금지.
- 결과물은 실명·개인정보를 포함하므로 `frontend-scoreboard/`(로컬 전용)에만 쓴다. 커밋하지 않는다.

## 마무리

생성한 파일 목록과, 각 인물의 한 줄 배치 요약 + 남긴 플래그(충돌/재평가)를 보고한다.
