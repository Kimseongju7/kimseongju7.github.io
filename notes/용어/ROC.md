---
title: ROC
date: 2026-07-06 15:29:36 +0900
categories: [학습, 용어]
tags: [머신러닝, 분류, 평가지표, sagemaker]
publish: true
---
## 한 줄 정의

ROC(Receiver Operating Characteristic) 곡선은 이진 분류 모델이 판단 기준(임계값)을 바꿔가며 얼마나 잘 구별해내는지를, 참 양성 비율과 거짓 양성 비율의 관계로 그린 그래프다.

## 자세히

이진 분류 모델은 대개 "양성일 확률 0.73" 같은 점수를 출력하고, 이 점수가 특정 임계값(보통 0.5)을 넘으면 양성으로 판정한다. 이 임계값을 0부터 1까지 바꿔가면서 매번 두 가지 비율을 계산할 수 있다 — 실제 양성 중 맞게 양성으로 예측한 비율인 **TPR**(True Positive Rate, 재현율/민감도)과, 실제 음성 중 잘못 양성으로 예측한 비율인 **FPR**(False Positive Rate). x축을 FPR, y축을 TPR로 두고 임계값을 바꿔가며 찍은 점들을 이은 것이 ROC 곡선이다.

곡선이 왼쪽 위 모서리(FPR 0, TPR 1)에 바짝 붙을수록 좋은 모델이다. 대각선(y=x)은 동전 던지기와 다를 바 없는 무작위 모델을 뜻한다. 곡선 하나하나를 눈으로 비교하기보다, 곡선 아래 면적인 **AUC**(Area Under the Curve, 0~1 사이 값)로 요약해서 모델 성능을 숫자 하나로 비교하는 것이 실무에서 훨씬 흔하다. AUC 0.5는 무작위 수준, 1.0은 완벽한 분류를 의미하며 보통 0.8 이상이면 꽤 쓸만한 모델로 본다.

ROC/AUC는 특히 **클래스 불균형**이 있을 때(예: 사기 탐지처럼 양성이 1%뿐인 경우) 정확도(accuracy)보다 신뢰할 수 있는 지표로 꼽힌다. 정확도는 전부 "음성"이라고만 찍어도 99%가 나올 수 있지만, ROC/AUC는 임계값 전 구간에서의 판별력을 보기 때문에 이런 함정에 잘 속지 않는다. Amazon SageMaker에서는 XGBoost·Linear Learner 같은 내장 이진분류 알고리즘의 훈련 로그에 `validation:auc` 지표로 등장하고, SageMaker Autopilot의 모델 후보 리더보드나 Model Monitor의 모델 품질(Model Quality) 모니터링 리포트에서도 ROC 곡선과 AUC 값을 확인할 수 있다.

## 예시

같은 모델이라도 임계값에 따라 TPR/FPR이 달라진다.

| 임계값 | TPR (재현율) | FPR |
|---|---|---|
| 0.9 (엄격) | 0.40 | 0.02 |
| 0.5 (기본) | 0.75 | 0.10 |
| 0.2 (느슨) | 0.95 | 0.35 |

임계값을 낮출수록 양성을 더 잘 잡아내지만(TPR↑) 오탐도 늘어난다(FPR↑). 이 표의 모든 임계값에서의 (FPR, TPR) 점을 이으면 ROC 곡선이 되고, 그 아래 면적이 AUC다. scikit-learn에서는 이렇게 구한다.

```python
from sklearn.metrics import roc_curve, roc_auc_score

fpr, tpr, thresholds = roc_curve(y_true, y_score)
auc = roc_auc_score(y_true, y_score)  # 예: 0.87
```

## 관련

- [[Machine Learning Engineering on AWS]] — 이번에 실습 중인 SageMaker 강의 정리 노트
- [[오류 임계값]] — 둘 다 "임계값(threshold)"을 조절하는 개념이지만, 이쪽은 분류 결정 기준을 가리키고 저쪽은 SageMaker Data Wrangler의 데이터 분할 비율 오차 허용치를 가리키는 별개 개념
- [[혼돈행렬]] — ROC가 임계값 전 구간을 훑는 것과 달리, 한 임계값에서의 TP/TN/FP/FN 스냅샷을 보여주는 표
- [[Ground-Truth|Ground Truth]] — ROC의 TPR·FPR을 계산하는 기준이 되는 정답 데이터
