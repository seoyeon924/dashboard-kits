---
name: build-dashboard
description: data/의 채용 CSV를 분석해, templates/reference.html과 동일한 퀄리티의 HR 채용 퍼널·이탈 리스크 대시보드를 output/에 생성하는 스킬. "대시보드 만들어줘", "/build-dashboard" 시 사용.
argument-hint: "[CSV — 생략 시 data/recruiting.csv(채용) + data/employees.csv(직원)]"
disable-model-invocation: false
allowed-tools: Read, Glob, Grep, Write, Bash
---

# build-dashboard — HR 채용 퍼널 대시보드 자동 생성

수강생이 이 명령 하나만 치면, 강사 퀄리티의 채용 퍼널 대시보드가 자기 데이터로 나온다.
**반드시 아래 5단계를 순서대로 수행한다. 단계를 건너뛰지 않는다.**

## STEP 1 — 데이터 읽기 (EDA)
- 두 CSV를 python으로 집계 로드: `data/recruiting.csv`(채용 퍼널)·`data/employees.csv`(직원 단위). 채용 퍼널·time-to-hire·오퍼 수락률은 recruiting에서, 재직인원·자발적 이직률·성과 등급·이탈 위험은 employees에서 — CLAUDE.md 매핑대로. 행을 다 읽지 말고 집계.
- 컬럼·행 수·부서 목록·단계 목록을 한 줄로 요약해 사용자에게 보여준다.
- 필수 컬럼 확인: `department, stage, stage_order, candidates, passed, source, avg_days, offer_sent, offer_accepted`.
  - 컬럼명이 다르면 가장 가까운 것에 매핑하고, 매핑 결과를 알려준다.
- 퍼널 단계 순서는 `지원 → 서류 → 1차면접 → 2차면접 → 오퍼 → 입사`로 stage_order에 따라 정렬.

## STEP 2 — KPI 계산 (실제 숫자)
- `CLAUDE.md`의 계산식대로 계산한다:
  - **단계별 통과율** = passed/candidates × 100 (단계마다)
  - **퍼널 전환율(전체)** = 입사 candidates / 지원 candidates × 100
  - **단계 이탈률** = (1 − 통과율) × 100 → **가장 큰 이탈 구간 = 병목**으로 식별
  - **time-to-hire** = 부서별 avg_days 누적의 평균
  - **offer acceptance rate** = offer_accepted/offer_sent × 100
  - **부서별 집계** = 부서별 지원→입사 전환율, 입사 인원, 자발적 이직률
- 부서를 위험도(이직률/전환율) 순으로 정렬. 병목 단계와 최저 전환 부서를 식별.
- 계산은 Bash(python)로 실제 수행하고, 결과 수치를 표로 한 번 보여준다. **추정·반올림 임의값 금지.**

## STEP 3 — 레퍼런스 구조 흡수
- `templates/reference.html`을 **반드시 Read** 한다 (제목 "HR People Analytics OS", 778줄).
- HTML 골격(사이드바, STEP 라벨, KPI 카드, 차트 wrap, 퍼널 바, 테이블), CSS 토큰, chart.js 초기화 패턴을 파악한다.
- 교체 대상 `const` 배열을 모두 찾는다: `headcountSpark data`, `months`, `depts`, `perfChart data`, `aiItems`, `riskData`, 그리고 퍼널/부서 막대 HTML 수치.
- 이 구조를 그대로 재사용한다 — 새로 디자인하지 않는다. **배열 필드명/형식은 100% 동일하게 유지한다.**

## STEP 4 — 데이터 주입해서 렌더
- 레퍼런스 구조 위에 STEP 2의 **실제 계산값**을 채워 `output/hr_dashboard.html` 생성.
- 정규식으로 `const` 배열만 교체, JS 로직·차트 옵션은 불변. 제목·날짜 갱신.
- 퍼널 단계별 전환율 바, KPI 카드 숫자, 부서별 막대(위험도순 색), 이탈 위험 테이블, AI 알림을 모두 새 값으로.
- 카드 제목은 결론/질문형으로. KPI 수치는 검정 고정.
- `CLAUDE.md`의 금지 항목(한쪽 border, 수치 색, 장식, 필드명 변경)을 위반하지 않았는지 자체 점검.

## STEP 5 — 전달
- 저장 경로의 **절대경로 file:// 링크**를 안내한다.
- "이번 데이터 기준 채용 개선 권고"를 3줄 이내로 요약 출력 (어느 단계가 병목인지 / 어느 부서가 최저 전환인지 / 누구를 먼저 붙잡을지).
- 끝에 한 줄: "data/의 CSV를 본인 데이터로 바꾸고 다시 `/build-dashboard` 하면 같은 퀄로 재생성됩니다."

## 핵심 원칙
- reference = 품질 기준선. 결과가 그보다 단순/다른 톤이면 다시 만든다.
- 복붙이 아니라 "구조 재사용 + 데이터 교체". 숫자는 항상 실제 분석값.
- 배열 필드명/형식 100% 동일 유지 — 안 그러면 차트가 깨진다.
