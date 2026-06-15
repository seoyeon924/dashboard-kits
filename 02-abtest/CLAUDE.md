# CLAUDE.md — A/B 테스트 실험 결과 대시보드

## 이 프로젝트의 목표
`data/`의 실험 CSV를 받아, **이 폴더의 디자인을 그대로 따르는** A/B 테스트 의사결정 대시보드 HTML을 `output/`에 만든다.
"예쁜 화면"이 아니라 "3초 안에 어떤 변형(Variant)을 배포할지 보이는 화면"을 만든다.

---

## 가장 중요한 규칙 (반드시 먼저 읽을 것)
**처음부터 새로 디자인하지 않는다.** 항상 `templates/reference.html`을 먼저 Read 하고,
그 **구조·레이아웃·색상·차트 종류·카드 패턴을 그대로 재사용**한 뒤, **숫자와 인사이트만 새 데이터로 교체**한다.
- 레퍼런스는 "품질 기준선"이다. 결과물이 레퍼런스보다 단순하거나 다른 톤이면 실패다.
- 단, **복붙이 아니다.** 데이터는 반드시 새로 분석해서 실제 계산값을 넣는다 (아래 KPI 정의대로).
- 레퍼런스의 JS 로직·차트 초기화 코드는 **건드리지 않는다.** 교체하는 것은 데이터 배열(`const`)뿐이다.
- 데이터 배열의 **필드명·형식을 레퍼런스와 100% 동일하게** 유지해야 차트가 깨지지 않는다.

---

## 디자인 토큰 (reference.html의 :root에서 추출 — 동일하게 고정)
```css
--bg:#F0F2F7;        /* 컨텐츠 배경 */
--card:#FFFFFF;      /* 카드 배경 */
--border:#E2E8F0;    /* 카드 테두리 */
--sidebar:#2E3B55;   /* 사이드바 (네이비) */
--sidebar-active:rgba(255,255,255,.1);
--t1:#0F172A;        /* KPI 수치 — 검정 고정, 빨강/초록 금지 */
--t2:#334155; --t3:#64748B; --t4:#94A3B8;  /* 보조 텍스트 단계 */
--grid:#E2E8F0;      /* 차트 그리드 */
--green:#10B981;     /* 좋음/상승/Ship */
--red:#EF4444;       /* 나쁨/하락/Stop·위험 */
--amber:#2E3B55;     /* 경고(레퍼런스에선 네이비로 처리) */
--purple:#4A7AB5;
--green-bg:#ECFDF5; --red-bg:#FEF2F2; --amber-bg:#EEF2F8; --purple-bg:#EEF2F8;
/* 효과크기 팔레트 (진→연, uplift 순) */
--c1:#4A7AB5; --c2:#5B6AB0; --c3:#7B8FCC; --c4:#A8B8E0; --c5:#D4DCF0;
--r:10px;
```
- 폰트: **Pretendard** (reference와 동일, CDN)
- 차트: reference가 쓰는 방식 그대로 — `chart.js`(CDN) + `<canvas>` / Forest Plot은 인라인 `<svg>`, 아이콘은 인라인 `<svg>`
- KPI 수치 색은 **검정(`--t1`) 고정**. 빨강/초록은 증감 화살표·뱃지·Decision 색에만.
- 임의 색 사용 금지. 위 토큰 밖의 색을 새로 만들지 않는다.

---

## 핵심 KPI 정의 (계산식 고정)
| 지표 | 계산식 | 표기 |
|---|---|---|
| 전환율(CVR) | conversions / users × 100 | `%` |
| 절대 차이(Absolute Δ) | CVR(B) − CVR(A) | `%p` |
| 상대 Uplift | (CVR(B) − CVR(A)) / CVR(A) × 100 | `%` |
| 표준오차(SE) | √( p_a(1−p_a)/n_a + p_b(1−p_b)/n_b ) | 비율 |
| z-통계량 | (p_b − p_a) / SE | 수치 |
| p-value | 양측검정, 2·(1 − Φ(\|z\|)) | `<0.001` / `0.0xx` |
| 95% 신뢰구간(CI) | Δ ± 1.96·SE (상대 uplift 환산) | `[+x%, +y%]` |
| 유의성(sig) | p < 0.05 → true | bool |
| 증분 매출 | revenue(B) − revenue(A) | `₩` (추정) |

### 표기 규칙 (절대/상대 혼용 금지)
- **비율 그 자체**는 `%` (예: CVR 6.8%).
- **두 비율의 차이**는 `%p` (예: +1.6%p). `%`와 `%p`를 절대 섞지 않는다.
- **상대 uplift**(`+30.8%`)와 **절대 delta**(`+1.6%p`)는 항상 따로 표기한다. 한 칸에 섞지 않는다.
- 증분 매출에는 "추정" 맥락을 유지한다(관측 revenue 차이 기반).

### 의사결정 규칙 (decision 필드)
| decision | 조건 | 색(decisionColors) |
|---|---|---|
| `ship` | sig=true AND uplift 상위 + guardrail 통과 | `#10B981` |
| `hold` | sig=true 이지만 효과 약함(작은 uplift) | `#4A7AB5` |
| `retest` | guardrail=SRM 등 실험 오염 | `#4A5F80` |
| `monitor` | sig=false 또는 guardrail 악화(AOV↓) | `#6B7280` |
| `stop` | sig=false AND uplift 미미 | `#EF4444` |
- Forest Plot·테이블·인사이트의 decision 값은 위 규칙으로 **일관되게** 산출한다.

---

## 대시보드 구성 (reference의 섹션 순서 유지 — 바꾸지 않는다)
1. **상단 KPI 6개 (row1)**: Active Experiments · Decision-Ready · Guardrail Pass Rate · Risk Alerts · 검증 완료율 · Incremental Revenue
   - 라벨은 reference 그대로. 값만 CSV 계산값으로 교체.
2. **누적 전환율 차이 + 95% CI (row2 좌, `cumulChart`)** — 라인 + CI 밴드. 대표 실험 2개의 일별 누적 Δ.
3. **실험 품질 경고 패널 (row2 우)** — SRM · Guardrail 악화 · 표본 부족 · 다중비교 보정.
4. **Forest Plot (row3 좌, `forestPlot` SVG)** — 실험별 효과크기(uplift)와 95% CI, 귀무가설(0) 세로선. `forestData`.
5. **세그먼트 스캐터 (row3 우, `segChart` bubble)** — X=상대 Uplift, Y=통계적 신뢰도, 원크기=표본. `segData`.
6. **AI 판정 인사이트 (row4 좌, `aiList`)** — 상위 실험 근거·제약·리스크·액션. `aiItems`.
7. **실험 의사결정 테이블 (row4 우, `decTable`)** — Control/Variant/Absolute Δ/Relative Uplift/CI/p/N/기간/Guardrail/Decision. `tableData`.

---

## 교체 대상 데이터 배열 (이것만 바꾼다)
| 배열 | 위치(레퍼런스 기준) | 필드 | 무엇을 채우나 |
|---|---|---|---|
| `diff1`,`diff2` | 누적차이 라인 | 28일 누적 Δ(소수) | 대표 실험 2개의 일별 누적 전환율 차이 |
| `forestData` | Forest Plot | name,effect,ci[lo,hi],n,sig,decision | 실험별 상대 uplift(%)+95%CI+표본+유의성+판정 |
| `segData` | 세그먼트 스캐터 | x,y,r,label,status | 세그먼트별 uplift(x)·신뢰도(y)·표본(r)·status |
| `aiItems` | AI 인사이트 | idx,name,decision,metric{...},constraint,risk,action | 상위 3개 실험 상세 |
| `tableData` | 의사결정 테이블 | exp,metric,ctrl,variant,abs,rel,ci,pval,n,runtime,guardrail,decision | 전체 실험 요약 한 행씩 |
- `decisionColors`,`statusColors`,JS 로직, 차트 옵션은 **건드리지 않는다.**

---

## 금지
- reference를 무시하고 처음부터 새 레이아웃을 만드는 것
- KPI 수치에 색 입히기 / 한쪽 변만 있는 border(border-left 등 단독)
- 차트 종류를 이유 없이 늘리기 / 장식용 그라디언트
- `%`와 `%p` 혼용 / 상대 uplift와 절대 delta를 한 칸에 섞기
- 데이터 분석 없이 레퍼런스 숫자를 그대로 남겨두는 것
- 데이터 배열의 필드명·형식을 바꿔 차트를 깨뜨리는 것

## 작업이 끝나면
- `output/abtest_dashboard.html`로 저장
- 브라우저로 열 수 있는 절대경로 링크를 안내
- 마지막에 "배포 권고 1줄 요약"을 텍스트로도 출력 (어느 Variant를 배포할지)
