# 바르셀로나 Airbnb 예약 전환율 분석

숙소·호스트 특성이 예약(점유)에 어떤 영향을 주는지 분석하고, 분석 도중 발견한 지표 오류를 직접 검증·수정한 프로젝트입니다.

---

## 문제정의 & 성과

**문제정의**
> Airbnb는 검색·조회는 많지만 실제 예약으로 이어지는 비율이 낮다는 공통 과제를 안고 있다. 어떤 숙소 특성, 어떤 호스트 특성이 예약 성사(점유)로 이어지는가?

**성과 (요약)**
- 숙소 타입·지역·가격·리뷰·Superhost·최소숙박일 등 8개 질문에 대해 정량 분석 완료
- 자체 설계한 예약 프록시 지표(`occupancy_rate`)를 공식 벤치마크 지표(`estimated_occupancy_l365d`)와 교차검증 → **상관계수 -0.04로 사실상 무관함을 발견**
- 지표 오류로 인해 뒤바뀐 결론을 재검증 → **Private room이 "1위"에서 "최하위"로 정정** 등 핵심 결과 전량 재확인
- 데이터에 없는 지표(즉시예약 여부 등)는 억지로 추정하지 않고 한계로 명시 → 신뢰할 수 있는 결론만 제시

| 항목 | 내용 |
|---|---|
| 최종 인사이트 | Private room 카테고리 재검토 필요, Nou Barris 지역 지원책 필요, 가격은 예약 성사의 결정 요인 아님 |
| 핵심 역량 증명 | 지표 설계 → 교차검증 → 결함 발견 → 재분석까지 전체 파이프라인 직접 수행 |

---

## 데이터 출처

- [Inside Airbnb](https://insideairbnb.com/get-the-data/) — Barcelona, 2026-06-24 스크래핑본
- `listings.csv.gz` (상세 리스팅, 15,293건)
- `calendar.csv.gz` (날짜별 예약가능여부, 약 560만 행)
- `neighbourhoods.geojson` (지역 지도용)

용량 문제로 원본 데이터는 리포지토리에 포함하지 않았습니다. 위 링크에서 직접 받아 `data/raw/`에 넣고 실행하시면 됩니다.

---

## 폴더 구조

```
barcelona-airbnb-analysis/
├── data/
│   ├── raw/                  # 원본 데이터 (직접 다운로드 필요)
│   │   ├── listings.csv.gz
│   │   ├── calendar.csv.gz
│   │   └── neighbourhoods.geojson
│   └── processed/            # 정제 결과물 (노트북 실행 시 자동 생성)
│       ├── occupancy.pkl
│       └── master.pkl
├── notebooks/
│   ├── 01_occupancy.ipynb    # calendar → 리스팅별 점유율 계산
│   ├── 02_clean_merge.ipynb  # listings 정제 + occupancy 병합
│   └── 03_eda_questions.ipynb# 분석질문 8개 + 지표 검증 + 재분석
├── outputs/                  # 노트북 실행 시 생성되는 차트(png)
├── report/
│   └── final_insight_report.md
└── README.md
```

---

## 실행 방법

```bash
pip install pandas scipy matplotlib

# 순서대로 실행 
jupyter notebook notebooks/01_occupancy.ipynb
jupyter notebook notebooks/02_clean_merge.ipynb
jupyter notebook notebooks/03_eda_questions.ipynb
```

---

## 분석 질문 및 결과 요약

| # | 질문 | 결과 |
|---|---|---|
| Q1 | 숙소 타입별 예약 특징은? | 검증 지표 기준 Hotel room·Entire home이 우수, Private room 최하위 |
| Q2 | 가격이 예약률에 영향을 주는가? | 상관관계 거의 없음 (r=-0.06) |
| Q3 | 리뷰 수·평점이 영향을 주는가? | 리뷰 수는 반직관적 결과, 평점은 양의 상관 |
| Q4 | Superhost 여부 영향은? | 유의미하게 높음 (p<0.0001) |
| Q5 | 지역별 예약률 차이는? | Sants-Montjuïc·Gràcia 상위, Nou Barris 최하위 |
| Q6 | 최소숙박일이 예약을 방해하는가? | 단순 방해 아님, 4~14박 구간이 오히려 유리 |
| Q7 | 즉시예약 가능 여부 영향은? | 데이터 결측(100%)으로 분석 불가 |
| Q8 | 라이선스·호스트 세그먼트 영향은? | 유의미한 차이 확인, 단 검증 지표로 재확인 필요 |

자세한 과정과 해석은 [`report/final_insight_report.md`](./report/final_insight_report.md)에 정리되어 있습니다.

---

## 사용 기술

`Python` `pandas` `scipy` `matplotlib` · 통계적 가설검정(t-test), 상관분석, 지표 교차검증

---

## 데이터 한계 및 주의사항

- 실제 "예약 완료" 이벤트 로그가 없어, 캘린더 기반 점유율을 예약 프록시로 사용함
- 이 프록시 지표는 자체 검증 결과 신뢰도가 낮아, 최종 결론은 공식 지표(`estimated_occupancy_l365d`) 기준으로 재확인함
- `instant_bookable`, `host_response_rate`, `host_acceptance_rate`는 스크래핑 시점 기준 100% 결측으로 분석 제외