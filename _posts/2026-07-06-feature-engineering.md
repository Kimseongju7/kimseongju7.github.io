---
title: 'Feature Engineering'
date: 2026-07-06 14:27:51 +0900
categories: [학습, 용어]
tags: [머신러닝, 데이터분석, eda, 전처리]
description: '피처 엔지니어링(feature engineering)은 원시 데이터를 머신러닝 모델이 학습하기 좋은 형태의 입력 변수(피처)로 가공·선택·생성하는 작업이다.'
---
## 한 줄 정의

피처 엔지니어링(feature engineering)은 원시 데이터를 머신러닝 모델이 학습하기 좋은 형태의 입력 변수(피처)로 가공·선택·생성하는 작업이다.

## 자세히

머신러닝 모델의 성능은 알고리즘 자체보다 "모델에 무엇을 먹이느냐"에 좌우되는 경우가 많다. 원시 데이터는 그대로 쓰기 어렵다 — 단위가 제각각이고, 문자열이 섞여 있고, 정작 중요한 정보가 여러 컬럼에 흩어져 있기 때문이다. 피처 엔지니어링은 도메인 지식과 데이터 탐색(EDA) 결과를 바탕으로 이 원시 데이터를 모델이 패턴을 잡아내기 쉬운 형태로 바꾸는 과정 전체를 가리킨다. "같은 모델이라도 좋은 피처를 주면 훨씬 잘 배운다"는 것이 핵심 전제다.

대표적인 작업 유형은 크게 네 가지다. **변환(transformation)** — 수치형 피처의 스케일링·정규화, 치우친 분포에 로그 변환, 범주형 문자열의 원-핫/레이블 인코딩. **생성(creation)** — 기존 컬럼을 조합해 새 피처를 만드는 것으로, 날짜에서 요일·월·공휴일 여부를 뽑거나, `총가격 ÷ 면적 = 평당가격` 같은 파생 변수를 만드는 식이다. **선택(selection)** — 타깃과 상관이 높은 피처를 고르고, 서로 중복 정보를 담은 피처(다중공선성)는 덜어내는 것. 이때 [상관관계행렬](/posts/correlation-matrix/)을 [열지도](/posts/heatmap/)로 시각화해 판단하는 것이 흔한 방법이다. **결측·이상치 처리** — 빈 값을 평균/중앙값으로 채우거나 결측 여부 자체를 새 피처로 만드는 것도 포함된다.

주의할 점은 **데이터 누수(data leakage)** 다. 스케일링 기준이나 평균값 같은 통계를 전체 데이터로 계산한 뒤 훈련/테스트에 나눠 쓰면, 테스트 데이터의 정보가 훈련에 새어 들어가 성능이 부풀려진다. 반드시 훈련 데이터로만 변환 규칙을 학습(fit)하고 테스트 데이터에는 적용(transform)만 해야 한다.

딥러닝은 신경망이 표현(representation)을 스스로 학습하기 때문에 이미지·텍스트 영역에서는 수작업 피처 엔지니어링의 비중이 크게 줄었다. 하지만 표 형태(tabular) 데이터에서는 여전히 성능을 가르는 핵심 단계이고, AWS SageMaker의 Data Wrangler(시각적 변환 도구)나 Feature Store(만든 피처를 저장·재사용하는 저장소)처럼 이 과정을 지원하는 관리형 서비스도 있다.

## 예시

주택 가격 예측 데이터라면:

```python
import pandas as pd
from sklearn.preprocessing import StandardScaler

# 생성: 날짜에서 파생 피처 뽑기
df["계약월"] = pd.to_datetime(df["계약일"]).dt.month
df["평당가격"] = df["가격"] / df["면적"]

# 변환: 범주형 인코딩 + 수치형 스케일링
df = pd.get_dummies(df, columns=["지역구"])          # 원-핫 인코딩
scaler = StandardScaler().fit(train[["면적", "층수"]]) # 훈련 데이터로만 fit
train_scaled = scaler.transform(train[["면적", "층수"]])
test_scaled = scaler.transform(test[["면적", "층수"]])  # 테스트엔 적용만
```

- 선택: 상관관계행렬을 그려 `방 개수`와 `면적`의 상관이 0.95라면 하나만 남긴다.
- 결측 처리: `리모델링연도`가 비어 있으면 "리모델링 안 함"이라는 정보일 수 있으니, 채우는 대신 `리모델링여부` 피처를 새로 만든다.

## 관련

- [상관관계행렬](/posts/correlation-matrix/) — 피처 선택(상관 높은 피처 고르기, 중복 피처 제거)에 쓰는 도구
- [열지도](/posts/heatmap/) — 상관관계행렬 등 피처 탐색 결과를 시각화하는 기법
- [Machine Learning Engineering on AWS](/posts/machine-learning-engineering-on-aws/) — 훈련 전 데이터 준비 단계의 하나로 등장
- [차원의-저주](/posts/curse-of-dimensionality/) — 피처 선택·차원 축소가 대응하는 문제 상황
- [서수 인코딩](/posts/ordinal-encoding/) — 변환(transformation) 단계에 속하는 범주형 인코딩 기법의 하나
