# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Python data analysis project examining **Cookie Cats** (F2P mobile puzzle game) A/B test data.

**핵심 비즈니스 질문:** Gate를 레벨 30에서 레벨 40으로 옮기면 유저 리텐션이 증가할까, 감소할까?

Dataset (`data/cookie_cats.csv`, 90,189 rows):

| 컬럼 | 타입 | 설명 |
|---|---|---|
| `userid` | int | 고유 유저 ID |
| `version` | str | A/B 그룹 — `gate_30` (대조군) / `gate_40` (실험군) |
| `sum_gamerounds` | int | 설치 후 14일간 플레이한 총 라운드 수 |
| `retention_1` | bool | 설치 다음날 접속 여부 (1-day retention) |
| `retention_7` | bool | 설치 7일 후 접속 여부 (7-day retention) |

## Package Manager

이 프로젝트는 [uv](https://docs.astral.sh/uv/)를 사용한다.

```bash
uv add <package>             # 패키지 추가
uv sync                      # lock file 기준 의존성 설치
uv run jupyter notebook      # 프로젝트 venv 안에서 Jupyter 실행
```

- Python 3.12 (`.python-version`으로 고정)
- Jupyter 커널 이름: **`cookie-cats`** (`Python (cookie-cats)`)

## Dependencies

현재 `pyproject.toml` 등록 패키지:

```
numpy, pandas, matplotlib, seaborn
```

> `ipykernel`, `jupyter`는 별도로 설치 필요:
> ```bash
> uv add ipykernel jupyter
> ```

## Notebook Structure

노트북은 `notebooks/`에 위치. 데이터 경로는 `../data/` 상대 경로 사용. 주석·설명은 한국어.

---

### `01_EDA.ipynb` — Exploratory Data Analysis

#### 섹션 구성

| # | 제목 | 주요 내용 |
|---|---|---|
| 1 | 데이터 불러오기 | pandas/matplotlib/seaborn import, AppleGothic 한글 폰트, CSV 로드 |
| 2 | 기본 데이터 파악 | `info()`, 결측치, 중복, `describe()` |
| 3 | A/B 그룹 분포 | 그룹별 유저 수·비율, 막대 차트 |
| 4 | sum_gamerounds 분포 | 분위수, 0라운드 유저, 히스토그램 + 그룹별 boxplot (99th pct 기준) |
| 5 | Retention 비율 비교 | 그룹별 1-day / 7-day retention 막대 차트 |
| 6 | 라운드 구간별 패턴 | 라운드 구간 × retention 꺾은선 그래프 |
| 7 | EDA 핵심 요약 | 주요 수치 정리, 다음 단계 안내 |

#### EDA 주요 결과 (원본 데이터 기준)

| 항목 | 결과 |
|---|---|
| A/B 비율 | gate_30 49.9% / gate_40 50.1% — 균형 잡힌 실험 |
| 결측치·중복 | 없음 |
| `sum_gamerounds` | 중앙값 ≈ 17, 최대 49,854 (심한 우측 편중) |
| 0라운드 유저 | 약 3,994명 (설치 후 미실행) |
| 1-day retention | gate_30 ≈ 44.8% / gate_40 ≈ 44.2% |
| 7-day retention | gate_30 ≈ 19.0% / gate_40 ≈ 18.2% |

> **초기 관찰:** gate_40(실험군)에서 1-day·7-day retention 모두 소폭 하락.
> 통계적 유의성은 다음 단계 가설 검정에서 확인 필요.

#### 시각화에서 사용한 이상치 처리 기준

```python
# 히스토그램·boxplot 시각화 시 99th percentile 이하만 표시 (약 1% 극단치 제외)
p99 = df['sum_gamerounds'].quantile(0.99)
df_plot = df[df['sum_gamerounds'] <= p99]
```

---

---

### `02_가설검정.ipynb` — 가설 검정

**분석 대상:** D7 리텐션 (7-day retention)
**전처리:** 0라운드 유저(3,994명) + 극단 이상치(sum_gamerounds > 30,000, 1명) 제거 → 최종 86,194명

| 가설 | 내용 |
|---|---|
| H₀ | 게이트 위치는 D7 리텐션에 영향을 미치지 않는다 (π₃₀ = π₄₀) |
| H₁ | 게이트 위치는 D7 리텐션에 영향을 미친다 (π₃₀ ≠ π₄₀) |
| α | 0.05 |

#### 검정 결과 종합

| 검정 방법 | 통계량 | p-value | H0 기각 | 95% CI |
|---|---|---|---|---|
| 카이제곱 | chi2 = 8.9849 | 0.002722 | 기각 | — |
| Two-proportion Z-test | Z = 3.0061 | 0.002646 | 기각 | [+0.2820%p, +1.3387%p] |
| Bootstrap (10,000회) | diff = +0.8103%p | 0.0033 | 기각 | [+0.2686%p, +1.3507%p] |
| 순열 검정 (10,000회) | diff = +0.8103%p | ~0.003 | 기각 | — |

> **결론:** 네 검정 모두 H0 기각. gate_30 D7 리텐션(19.84%)이 gate_40(19.03%)보다 통계적으로 유의미하게 높다.
> Gate를 레벨 40으로 옮기면 D7 리텐션이 감소한다 → **gate_30 유지 권장.**

---

### `03_이상치_민감도.ipynb` — 이상치 기준별 민감도 분석

**목적:** 이상치 제거 기준이 달라져도 결론이 동일한지(robust) 검증

| 기준 | 상한 | 분석 n |
|---|---|---|
| 통계적 P99 | 499 라운드 | 85,335명 |
| 통계적 P99.5 | 634 라운드 | 85,764명 |
| 도메인 기반 | 1,120 라운드 (14일×4시간×180초/라운드) | 86,120명 |

#### 민감도 결과 종합

| 기준 | gate_30 D7 | gate_40 D7 | 차이 | 결론 |
|---|---|---|---|---|
| P99 (~499) | 19.0859% | 18.2558% | +0.8300%p | H0 기각 |
| P99.5 (~634) | 19.4840% | 18.6201% | +0.8639%p | H0 기각 |
| 도메인 (1120) | 19.7767% | 18.9616% | +0.8151%p | H0 기각 |

> **결론:** 3기준 × 3검정 = **총 9개 검정 모두 H0 기각.** 결과가 이상치 기준에 무관하게 안정적(robust)이다.
> 차이 범위: +0.8151%p ~ +0.8639%p

---

## 진행 현황 (Progress)

| 노트북 | 상태 | 내용 |
|---|---|---|
| `01_EDA.ipynb` | ✅ 완료 | 탐색적 데이터 분석 |
| `02_가설검정.ipynb` | ✅ 완료 | 카이제곱 / Z-test / Bootstrap 가설 검정 |
| `03_이상치_민감도.ipynb` | ✅ 완료 | 이상치 기준별 민감도 분석 (결론의 robust성 검증) |
| `04_conclusion.ipynb` | ✅ 변경 | report.md 파일 참고 |

## 최종 비즈니스 결론

**Gate를 레벨 30으로 유지한다.**

- gate_30 D7 리텐션이 gate_40보다 약 0.8%p 높으며, 이 차이는 통계적으로 유의미함 (p < 0.003)
- 결론은 검정 방법(3종) 및 이상치 제거 기준(3종) 모두에서 일관되게 재현됨

## 한글 폰트

macOS 기본 `AppleGothic` 사용. 다른 OS에서는 아래 우선순위로 자동 탐지:

```
AppleGothic → Malgun Gothic → NanumGothic → Noto Sans CJK KR → Arial Unicode MS → DejaVu Sans(fallback)
```
