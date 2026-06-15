# Dashboard Kits — 한 줄로 만드는 대시보드

CSV 하나만 있으면 **명령어 한 줄**로 강사와 동일한 퀄리티의 대시보드가 만들어집니다.
도메인별로 `CLAUDE.md`(디자인 규칙) + `Skill`(작업 절차) + `reference.html`(품질 기준)을 묶어둔 스타터킷입니다.

## 5개 킷

| 킷 | 주제 | 만들어지는 대시보드 |
|---|---|---|
| [01-marketing](01-marketing/) | 캠페인 성과 | 채널별 ROAS·시계열 예측·클러스터·예산 시뮬레이터 |
| [02-abtest](02-abtest/) | A/B 테스트 | 효과크기 forest plot·세그먼트·유의성·의사결정표 |
| [03-pm](03-pm/) | 프로덕트 지표 | Metric Hierarchy·원인 분석·RFM 세그먼트·Action |
| [04-retention](04-retention/) | 리텐션 | 코호트 매트릭스·MRR·LTV·What-if 시뮬레이터 |
| [05-hr](05-hr/) | 채용 퍼널 | 단계별 전환·time-to-hire·부서별·이탈 리스크 |

미리보기(GitHub Pages): 각 대시보드가 어떻게 나오는지 먼저 보고 시작하세요 → **[라이브 갤러리](https://seoyeon924.github.io/dashboard-kits/)**

## 사용법 (3단계)

```bash
# 1. 원하는 킷 폴더에서 Claude Code 실행
cd 01-marketing
claude

# 2. 명령어 한 줄
/build-dashboard

# 3. 끝. output/ 에 대시보드가 생성됩니다.
```

## 내 데이터로 바꾸기
각 킷의 `data/` 안 CSV를 본인 데이터로 교체하고 다시 `/build-dashboard`. 컬럼 형식은 킷별 README 참고.

## 한 킷의 구조

```
01-marketing/
├── CLAUDE.md                       # 디자인 헌법 (색·KPI·구성 규칙)
├── .claude/skills/build-dashboard/ # 작업 절차 = 그 한 줄 명령어
├── templates/reference.html        # 품질 기준 = 강사의 완성 대시보드
├── data/<sample>.csv               # 샘플 데이터 (여기를 교체)
└── output/                         # 결과물이 여기 생성됨
```

## 왜 매번 같은 퀄로 나오나?
- **CLAUDE.md** = 디자인·KPI 규칙을 고정 (헌법)
- **Skill** = 분석 → 렌더 절차를 고정 (작업 매뉴얼)
- **reference.html** = 품질 기준선 (이걸 보고 같은 수준으로 만듦)

세 개가 묶여 있어서, 데이터만 바뀌어도 결과 퀄리티는 일정합니다. 복붙이 아니라 "구조 재사용 + 실제 분석값 주입"입니다.

---
패스트캠퍼스 「세계 3등에게 배우는 AI 데이터 시각화」 강의 실습 자료 · 전서연
