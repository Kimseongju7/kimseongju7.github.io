---
title: "일리야 수츠케버가 존 카맥에게 건넨 딥러닝 읽기 목록"
date: 2026-07-14 14:09:48 +0900
categories: [학습, 아티클]
tags: [deep-learning, papers, reading-list, transformer, ai, machine-learning]
slug: ilya-sutskever-reading-list
publish: true
---
일리야 수츠케버(Ilya Sutskever)가 존 카맥(John Carmack)에게 건넨 것으로 알려진 딥러닝 읽기 목록을 정리한 사이트 30papers.com의 목록. CNN 기초부터 트랜스포머, 스케일링 법칙, 복잡도 이론까지 딥러닝의 핵심 흐름을 관통하는 논문·강의·에세이 모음이다.

## 기초와 CNN

- **CS231n: Convolutional Neural Networks for Visual Recognition** — 선형 분류기부터 이미지용 심층 아키텍처까지, 합성곱 신경망을 제1원리부터 가르치는 강의 노트.
- **ImageNet Classification with Deep Convolutional Neural Networks** — AlexNet. ImageNet에서 큰 격차로 우승하며 현대 딥러닝 시대를 연 합성곱 신경망.
- **Deep Residual Learning for Image Recognition** — ResNet. 전체 변환 대신 입력에 대한 변화량을 학습하는 잔차 연결(residual connection)로 네트워크를 수백 층까지 키울 수 있게 했다.
- **Multi-Scale Context Aggregation by Dilated Convolutions** — 해상도를 잃지 않고 수용 영역(receptive field)을 넓히는 팽창 합성곱(dilated convolution). 세그멘테이션 같은 밀집 예측 작업을 정교하게 만들었다.
- **Identity Mappings in Deep Residual Networks** — ResNet 후속 연구. 항등 지름길(identity shortcut)이 왜 잘 작동하는지 분석하고 더 깔끔한 pre-activation 잔차 블록을 제안.

## RNN과 시퀀스 모델

- **The Unreasonable Effectiveness of Recurrent Neural Networks** — 문자 단위 RNN으로 텍스트를 생성하며 RNN이 얼마나 많은 구조를 포착하는지 생생한 예시로 보여주는 실습형 블로그 글.
- **Understanding LSTM Networks** — LSTM 게이트가 긴 시퀀스에서 정보를 어떻게 운반하는지에 대한 가장 명쾌한 시각적 설명. 첫 입문 자료로 널리 쓰인다.
- **Recurrent Neural Network Regularization** — LSTM에 드롭아웃을 올바르게(비순환 연결에만) 적용해 대형 순환 모델의 과적합을 잡는 방법.
- **Deep Speech 2** — CTC(connectionist temporal classification)로 학습한 엔드투엔드 음성 인식 시스템. 영어와 중국어라는 매우 다른 두 언어에서 모두 작동했다.
- **Order Matters: Sequence to Sequence for Sets** — 입력·출력 순서가 seq2seq 모델에 미치는 영향과, 본질적으로 집합(set)인 데이터를 다루는 법을 탐구.

## 어텐션과 트랜스포머

- **Neural Machine Translation by Jointly Learning to Align and Translate** — 어텐션 메커니즘의 시초. 번역 모델이 고정된 요약 하나 대신 관련 원문 단어를 되돌아볼 수 있게 했다.
- **Pointer Networks** — 출력이 입력의 위치를 가리키는 시퀀스 모델. 답이 입력의 선택이나 순서 매기기인 문제에 적합하다.
- **Attention Is All You Need** — 트랜스포머. 순환 구조를 셀프 어텐션으로 완전히 대체했으며, 거의 모든 현대 대형 언어 모델의 기반 아키텍처다.
- **The Annotated Transformer** — 트랜스포머 원 논문을 실행 가능한 읽기 좋은 코드로 한 줄씩 재구현한 자료.

## 메모리와 관계 추론

- **Neural Turing Machines** — 신경망에 미분 가능한 어텐션으로 읽고 쓸 수 있는 외부 메모리를 결합해, 예시로부터 간단한 알고리즘을 학습한다.
- **A Simple Neural Network Module for Relational Reasoning** — 객체 쌍 사이의 관계를 추론하게 해주는 작은 플러그인 모듈, 관계 네트워크(relation network) 소개.
- **Relational Recurrent Neural Networks** — 순환 신경망에 셀프 어텐션 기반 메모리를 추가해 저장된 기억끼리 상호작용하게 함으로써, 시간에 걸친 관계 추론이 필요한 작업을 개선.
- **Neural Message Passing for Quantum Chemistry** — 다양한 그래프 신경망을 메시지 패싱 프레임워크 하나로 통합하고 분자 특성 예측에 적용.

## 스케일링과 대규모 학습

- **Scaling Laws for Neural Language Models** — 언어 모델 손실이 모델 크기·데이터·컴퓨트에 대한 매끄러운 거듭제곱 법칙으로 감소함을 측정. 더 큰 모델을 만드는 흐름의 실증적 토대.
- **GPipe** — 거대한 모델을 여러 디바이스에 쪼개면서도 디바이스를 놀리지 않는 파이프라인 병렬화 라이브러리. 초대형 네트워크 학습을 실용화했다.

## 압축, 복잡도, 지능 이론

- **Keeping Neural Networks Simple by Minimizing the Description Length of the Weights** — 좋은 네트워크란 가중치를 적은 비트로 기술할 수 있는 네트워크라는 초기 정보이론적 논증. 일반화를 압축과 연결한다.
- **A Tutorial Introduction to the Minimum Description Length Principle** — 데이터를 얼마나 잘 압축하는지로 모델을 고르는, 학습을 가장 짧은 기술 찾기로 보는 MDL 원리의 읽기 쉬운 입문.
- **The First Law of Complexodynamics** — 닫힌 계의 복잡도가 단순히 엔트로피를 따르지 않고 왜 오르고, 정점을 찍고, 내려가는지를 설명할 형식적 법칙을 묻는 블로그 에세이.
- **Quantifying the Rise and Fall of Complexity in Closed Systems: The Coffee Automaton** — 커피와 크림이 섞이는 단순한 셀룰러 오토마톤 모델로, 계가 평형으로 갈수록 복잡도가 왜 상승했다가 하락하는지를 탐구.
- **Kolmogorov Complexity** — 어떤 문자열을 생성하는 가장 짧은 프로그램에 대한 교과서적 정리. 기술 길이(description length)와 알고리즘적 무작위성의 형식적 뼈대.
- **Variational Lossy Autoencoder** — 변분 오토인코더(VAE)와 자기회귀 디코더를 결합하고, 잠재 코드가 어떤 정보를 유지하도록 강제할지 제어하는 방법을 제시.
- **Machine Super Intelligence** — 기계 지능의 형식적·보편적 척도를 제안하고 매우 유능한 에이전트에 대한 함의를 탐구한 박사 학위 논문.

## 관련 노트

- [[cs336-llm-course-value]] — 밑바닥부터 LLM을 만드는 스탠퍼드 CS336 강의의 가치
- [[cs336-study-advice]] — CS336 학습 조언
- [[karpathy-nanochat]] — 카파시가 처음부터 만든 nanochat LLM

> 원문: [The reading list Ilya Sutskever gave John Carmack](https://30papers.com/)
> 원본 클립: [[2026-07-14-The reading list Ilya Sutskever gave John Carmack]]
