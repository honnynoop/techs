# pandas pivot_table & melt 완전 정리

> ⭐ 빅데이터분석기사 / ADP / DAP 시험 단골 주제

---

## 1. 샘플 데이터

```python
import pandas as pd
import numpy as np

employees = pd.DataFrame({
    'id':     [1, 2, 3, 4, 5, 6, 7],
    'name':   ['Alice', 'Bob', 'Carol', 'Dave', 'Eve', 'Frank', 'Grace'],
    'dept':   ['IT', 'HR', 'IT', 'HR', 'IT', 'HR', 'FIN'],
    'gender': ['F', 'M', 'F', 'M', 'M', 'F', 'F'],
    'salary': [7800, 5200, 8500, 6100, 9200, 5700, 7400]
})
```

| id | name  | dept | gender | salary |
|----|-------|------|--------|--------|
| 1  | Alice | IT   | F      | 7800   |
| 2  | Bob   | HR   | M      | 5200   |
| 3  | Carol | IT   | F      | 8500   |
| 4  | Dave  | HR   | M      | 6100   |
| 5  | Eve   | IT   | M      | 9200   |
| 6  | Frank | HR   | F      | 5700   |
| 7  | Grace | FIN  | F      | 7400   |

---

## 2. pivot_table ⭐

### 개념

`index` 기준으로 **행**, `columns` 기준으로 **열**, 교차 셀에 `aggfunc` 를 적용하는 집계 교차표.

```
dept (index) × gender (columns) → mean(salary)
```

### 기본 문법

```python
pt = pd.pivot_table(
    df,
    values    = 'salary',      # 집계할 숫자 컬럼
    index     = 'dept',        # 행 레이블 (그룹 기준)
    columns   = 'gender',      # 열 레이블 (그룹 기준)
    aggfunc   = 'mean',        # 집계함수: mean / sum / count / min / max
    fill_value= 0,             # NaN → 0  (교차값 없는 셀 처리)
    margins   = True           # All 행/열 추가 (소계)
)
print(pt.round(1))
```

### 출력 결과

| dept \ gender | F      | M      | All    |
|---------------|--------|--------|--------|
| FIN           | 7400.0 | 0.0    | 7400.0 |
| HR            | 5700.0 | 5650.0 | 5666.7 |
| IT            | 8150.0 | 9200.0 | 8500.0 |
| **All**       | 7350.0 | 6983.3 | 7171.4 |

> `FIN / M = 0.0` → Grace(FIN, F)만 존재하여 교차값 없음 → `fill_value=0` 으로 대체  
> `All` 행/열 → `margins=True` 로 자동 추가되는 전체 소계

### 파라미터 상세

| 파라미터     | 설명                                      | 필수 여부 |
|------------|-------------------------------------------|---------|
| `values`   | 집계 대상 열 (리스트로 여러 개 가능)           | ✅ 필수  |
| `index`    | 행 그룹 기준 열                             | ✅ 필수  |
| `columns`  | 열 그룹 기준 열                             | 선택    |
| `aggfunc`  | `'mean'` · `'sum'` · `'count'` · 딕셔너리  | 선택    |
| `fill_value`| NaN 대체값                               | 선택    |
| `margins`  | `True` → All 소계 행/열 추가               | 선택    |
| `observed` | Categorical 열만 사용 시 `True` 권장        | 선택    |

### aggfunc 여러 개 동시 적용

```python
pt2 = pd.pivot_table(
    employees,
    values  = 'salary',
    index   = 'dept',
    aggfunc = {'salary': ['mean', 'max', 'count']}
)
print(pt2.round(1))
```

| dept | count | max  | mean   |
|------|-------|------|--------|
| FIN  | 1     | 7400 | 7400.0 |
| HR   | 3     | 6100 | 5666.7 |
| IT   | 3     | 9200 | 8500.0 |

---

## 3. melt — Wide → Long 변환

### 개념

여러 열(score1, score2 …)을 **하나의 value 열로 녹이는** 언피벗(unpivot) 연산.  
seaborn / ggplot 등 시각화 라이브러리는 **Long-form** 기본이므로 필수 전처리.

```
3행 × 4열 (Wide)  →  melt  →  6행 × 4열 (Long)
```

### 기본 문법

```python
df_score = pd.DataFrame({
    'id':     [1, 2, 3],
    'name':   ['Alice', 'Bob', 'Carol'],
    'score1': [85, 92, 78],
    'score2': [90, 88, 95]
})

df_long = df_score.melt(
    id_vars    = ['id', 'name'],         # 유지할 식별자 열
    value_vars = ['score1', 'score2'],   # 녹일 열들
    var_name   = 'test',                 # 열 이름을 담을 새 열 이름
    value_name = 'score'                 # 값을 담을 새 열 이름
)
print(df_long)
```

### 변환 전후 비교

**변환 전 (Wide)**

| id | name  | score1 | score2 |
|----|-------|--------|--------|
| 1  | Alice | 85     | 90     |
| 2  | Bob   | 92     | 88     |
| 3  | Carol | 78     | 95     |

**변환 후 (Long)**

| id | name  | test   | score |
|----|-------|--------|-------|
| 1  | Alice | score1 | 85    |
| 2  | Bob   | score1 | 92    |
| 3  | Carol | score1 | 78    |
| 1  | Alice | score2 | 90    |
| 2  | Bob   | score2 | 88    |
| 3  | Carol | score2 | 95    |

> **행 수 변화**: 원본 3행 × 녹인 열 2개 = **6행**으로 증가

### 파라미터 상세

| 파라미터      | 설명                                          | 기본값       |
|-------------|-----------------------------------------------|------------|
| `id_vars`   | 유지할 식별자 열 (리스트)                         | 없음        |
| `value_vars`| 녹일 값 열 (생략 시 `id_vars` 외 전체)            | 나머지 전체  |
| `var_name`  | 녹인 열 이름을 담는 열 이름                       | `'variable'`|
| `value_name`| 값을 담는 열 이름                               | `'value'`  |

---

## 4. 역변환 관계

```
Wide  ──→  melt()         ──→  Long
Long  ──→  pivot_table()  ──→  Wide (복원)
```

```python
# melt 로 Long 변환 후 pivot_table 로 Wide 복원
df_wide_back = df_long.pivot_table(
    index   = ['id', 'name'],
    columns = 'test',
    values  = 'score'
).reset_index()
df_wide_back.columns.name = None
print(df_wide_back)
```

| id | name  | score1 | score2 |
|----|-------|--------|--------|
| 1  | Alice | 85     | 90     |
| 2  | Bob   | 92     | 88     |
| 3  | Carol | 78     | 95     |

---

## 5. 실전 시각화 예제

```python
import matplotlib.pyplot as plt
import seaborn as sns

fig, axes = plt.subplots(1, 2, figsize=(12, 4))

# ① pivot_table → 히트맵
pt_vis = pd.pivot_table(
    employees, values='salary',
    index='dept', columns='gender',
    aggfunc='mean', fill_value=0
)
sns.heatmap(pt_vis, ax=axes[0], annot=True, fmt='.0f', cmap='Blues')
axes[0].set_title('pivot_table → 히트맵')

# ② melt Long-form → 라인플롯
sns.lineplot(
    data=df_long, x='test', y='score',
    hue='name', marker='o', ax=axes[1]
)
axes[1].set_title('melt → Long-form 라인플롯')

plt.tight_layout()
plt.show()
```

---

## 6. 핵심 요약 & 시험 포인트

| 구분          | pivot_table                          | melt                              |
|-------------|--------------------------------------|-----------------------------------|
| **방향**      | Long → Wide (집계)                   | Wide → Long (언피벗)               |
| **행 수**     | 감소 (그룹 수만큼)                      | 증가 (열 수 × 원본 행 수)            |
| **주요 용도**  | 교차 집계표, 보고서                     | seaborn/그래프 전처리               |
| **NaN 처리**  | `fill_value=` 로 대체                 | 없음 (필요 시 `dropna()` 별도)      |
| **소계 추가** | `margins=True`                       | 해당 없음                          |

### ⚠️ 자주 틀리는 포인트

1. `fill_value` 생략 시 교차값 없는 셀은 **NaN** → 계산 오류 주의
2. `margins=True` 의 **All 행/열**은 `aggfunc` 그대로 적용
3. `melt` 에서 `id_vars` 생략 시 모든 열이 녹아버림 → **식별자 열은 반드시 명시**
4. `value_vars` 생략 시 `id_vars` 외 나머지 전체가 녹음
5. seaborn은 **Long-form** 기본 → `melt` 후 `hue=` 파라미터로 그룹핑

---

*참고: pandas 2.x 에서 Categorical index 사용 시 `observed=True` 명시 권장 (FutureWarning 방지)*
