---
title: 'Machine Learning Engineering on AWS'
date: 2026-07-06 00:00:00 +0900
categories: [학습, aws]
tags: [aws, 머신러닝, sagemaker, swm, 전처리, 인코딩, eda]
description: 'SWM AI Infra 교육 "Machine Learning Engineering on AWS" 강의 정리 노트. AWS에서의 ML 개념과 SageMaker AI 파이프라인을 다룬다.'
---
SWM AI Infra 교육 "Machine Learning Engineering on AWS" 강의 정리 노트. AWS에서의 ML 개념과 SageMaker AI 파이프라인을 다룬다.

---

## AWS의 ML 소개

기존 인공지능이 인간이 규칙을 넣어주는 규칙 기반 시스템(지능형 시스템)이었다면, ML(기계 학습)은 기계가 데이터를 통해 스스로 규칙을 찾아내는 시스템이다.

### AWS SageMaker AI

파이프라인은 다음 순서로 이어진다.

> Data Wrangler → Feature Store → Notebooks → Canvas Python SDK → Automatic Model Training → Endpoints → Model Monitor
>
> (데이터 준비 → 특성 저장 → Notebooks로 구축 → 모델 훈련 → 하이퍼파라미터 튜닝 → 프로덕션 배포 → 관리 및 모니터링)

훈련은 EC2 기반 Docker 컨테이너를 사용해 진행하며, 특성과 (레이블이 포함된) 훈련 데이터 세트를 입력으로 사용한다.

알고리즘은 크게 3가지 옵션을 제공한다.

- SageMaker 내장 알고리즘을 포함한 빌트인 Docker 이미지 — 지도학습 알고리즘(서포트 벡터 머신 등)
- TensorFlow, PyTorch 등의 프레임워크를 포함한 빌트인 Docker 이미지
- 커스텀 프레임워크 지원

### Responsible AI/ML

Responsible AI/ML이 필요한 이유는 다음과 같다.

1. 가짜뉴스를 만들어내는 데 인공지능이 사용되고 있기 때문
2. ChatGPT를 이용한 보안 공격이 존재하기 때문

---

## Module 2: ML 도전 과제 분석

머신러닝은 기계가 학습하여 패턴을 인식하고 예측을 수행하는 것으로, 앞으로 3일 동안 다룰 주제다. 기존 프로그램으로 해결 가능하다면 그 방법을 쓰는 편이 낫다. ML 모델을 만드는 것은 예상보다 많은 비용과 시간이 든다.

---

## Module 3: ML에 사용할 데이터 처리

데이터를 어디에서 가져오고, 어떻게 처리하고, 어떤 스토리지를 사용할 것인가에 대한 내용이다.

기계 학습용 데이터 소스는 다음과 같다.
- 데이터 레이크: 모든 형식의 데이터를 원본 그대로 저장
- 데이터 웨어하우스: 정형 데이터와 반정형 데이터를 정제·변환하여 저장
- 데이터베이스: 다양한 형식으로 애플리케이션이 이용하는 데이터를 저장

데이터 레이크와 데이터 웨어하우스는 애플리케이션이 직접 이용하는 저장소가 아니라 분석과 활용을 위한 데이터 저장소라는 점에서 데이터베이스와 다르다. 데이터 레이크는 호수처럼 온갖 형식의 데이터가 그대로 들어갈 수 있고, 데이터 웨어하우스는 csv, excel, json, xml 등 정제·변환된 데이터가 저장된다. 둘은 비슷해 보이지만 담기는 데이터의 속성이 다르다.

데이터 훈련 전에는 막대그래프, 히스토그램, 밀도 플롯, 상자 플롯, [상관관계 행렬](/posts/correlation-matrix/), [히트맵](/posts/heatmap/) 등으로 시각화해 데이터가 정상인지 확인할 수 있다. 연관관계가 너무 밀접하면 오버피팅·쏠림 위험이 있으므로 둘 중 하나는 특성에서 제외한다.

#### SageMaker AI 모델 훈련 시 S3 사용 방법
- File mode: 훈련하기 전에 데이터셋을 연결된 Amazon EBS 또는 NVMe SSD 볼륨에 다운로드
- Fast File mode: 훈련 인스턴스의 파일 시스템 형태로 S3 스토리지를 연결해, 작성한 훈련 스크립트가 이를 읽어서 스트리밍
- Amazon EBS: EC2 인스턴스의 파일 시스템
- Amazon EFS: 여러 컴퓨팅 인스턴스가 공유 데이터셋, 훈련 코드, 사용자 지정 패키지, 모델 아티팩트에 동시 액세스해야 하는 다중 인스턴스 분산 훈련용 스토리지 (NAS와 유사)
- Amazon FSx for Lustre: Amazon S3 버킷과 연결해 import/export가 가능한 병렬 파일 시스템. 초당 수백만 건의 입출력 작업과 1밀리초 미만의 지연 시간을 제공

#### 데이터 포맷
- 행 기반 데이터 저장: data ingestion, writing, ETL, 파이프라인에 유리. 전체 레코드를 빠르게 읽을 수 있지만 압축이 쉽지 않음 (csv, json, apache avro)
- 열 기반 데이터 저장: 모델 훈련, 분석, 특성 엔지니어링에 유리. 특정 데이터 유형의 압축이 빠름 (apache parquet, apache ORC, pandas)

---

## Module 4: 데이터 변환 및 특성 추출

전처리에 대한 내용이다. 데이터의 이상치, 형식 불일치, 중복값을 처리해야 할 뿐 아니라, 컴퓨터는 데이터 처리 시 문자열이 아닌 숫자로 처리하므로 이를 위한 인코딩이 필요하다. 또한 누락된 값은 대체(imputation)가 필요하다.

[Feature Engineering](/posts/feature-engineering/)은 모델의 성능을 향상시키기 위한 작업이다.

#### 범주형 변수와 인코딩
- 명목형: 각 범주 간 순위가 없는 변수 (성별, 국적, 혈액형)
- 순서형: 각 범주 간 순위가 있는 변수. 다만 범주 간 간격이 동일하다는 보장은 없음 (학력, 크기, 성적)

범주 종류에 따라 인코딩 방식이 달라진다.

#### 인코딩 방법
- 레이블 인코딩: 하나의 범주 값들을 0부터 연속적인 수치 데이터로 변환 (크기를 의미하지 않음). 단순히 1, 2, 3에 매핑하는 방식으로, 임의로 부여된 숫자가 크기나 순서로 비춰져 잘못된 학습을 유발할 위험이 있다.
- 원핫 인코딩: n개의 특성값을 n개의 비트 벡터로 표현하는 방식. 범주가 너무 많으면 새로운 열이 과도하게 생성되어 [차원의 저주](/posts/curse-of-dimensionality/)가 발생한다. 숫자 간 크기 차이가 모델 학습에 영향을 미치는 신경 회귀, 신경망 같은 알고리즘에 적합하다.
- [순서형 인코딩](/posts/ordinal-encoding/): n개의 특성값을 0~n의 연속적인 숫자 데이터로 표현
- 바이너리 인코딩: 범주형 변수를 이진 변수로 변환

#### 스케일링
- 정규화: 데이터 범위를 특정 구간(0~1)으로 제한. 이상치에 영향을 많이 받음
- 표준화: 데이터의 평균을 0, 표준편차를 1로 조정하여 표준정규분포를 따르게 함. 데이터가 정규분포를 따를 때 유용

SageMaker Data Wrangler는 데이터 전처리를 코드 없이 시각적 UI로 할 수 있게 해준다 (코드로 처리하는 것도 가능).

코드로 전처리할 때 쓰는 서비스는 다음과 같다.
1. SageMaker Processing: 관리형 서비스. 스크립트만 있으면 됨
2. Amazon EMR: 대규모 ETL, 빅데이터 분석용. 클러스터 형태로 리소스를 [프로비저닝](/posts/provisioning/)해서 제공

---

## Module 5: 모델링 방식 선택

기본 제공 알고리즘 중 하나인 XGBoost는, 하나의 모델을 훈련하면 오차가 발생하는데 그 오차에 대한 훈련을 다시 한 번 하는 알고리즘이다. 오차를 줄이기 위해 그 범위 안에서 훈련을 계속 추가한다.

## 관련 노트

- [사전강의 - AWS Cloud Practitioner Essentials](/posts/aws-cloud-practitioner-essentials/) — 같은 SWM AI Infra 교육에서 먼저 다룬 AWS 클라우드 기본 개념 강의
- [Feature-Engineering](/posts/feature-engineering/) — 이 노트의 특성 추출·가공 작업을 설명하는 용어 노트
- [차원의-저주](/posts/curse-of-dimensionality/) — 원핫 인코딩에서 범주가 많을 때 발생하는 문제
- [서수 인코딩](/posts/ordinal-encoding/) — 이 노트의 순서형 인코딩과 같은 개념을 다루는 용어 노트
- [상관관계행렬](/posts/correlation-matrix/) — 데이터 훈련 전 시각화 목록에 나오는 상관관계 행렬
- [열지도](/posts/heatmap/) — 같은 시각화 목록에 나오는 히트맵
- [프로비저닝](/posts/provisioning/) — Amazon EMR이 클러스터 리소스를 준비하는 방식
