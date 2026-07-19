---
title: 'Amazon CloudWatch'
date: 2026-07-04 03:01:50 +0900
categories: [학습, 용어]
tags: [aws, monitoring, observability, cloud]
description: 'AWS 리소스와 애플리케이션에서 나오는 지표·로그·이벤트를 한곳에 모아 감시하고, 임계치를 넘으면 알람·자동 대응을 트리거하는 AWS의 모니터링·관측성 서비스.'
---
## 한 줄 정의
AWS 리소스와 애플리케이션에서 나오는 지표(metric)·로그·이벤트를 한곳에 모아 감시하고, 임계치를 넘으면 알람·자동 대응을 트리거하는 AWS의 모니터링·관측성 서비스.

## 자세히
CloudWatch는 EC2, RDS, Lambda, ELB 같은 AWS 서비스가 자동으로 내보내는 **지표(metric)** 를 수집한다. CPU 사용률, 네트워크 처리량, 요청 수 같은 값이 시간축을 따라 저장되고, 이를 대시보드에서 그래프로 볼 수 있다. 애플리케이션이 직접 커스텀 지표를 올리는 것도 가능하다.

핵심 기능은 **알람(Alarm)** 이다. 특정 지표가 일정 기간 동안 임계치를 넘으면(예: CPU 80% 이상 5분 지속) 알람이 `ALARM` 상태로 바뀌고, 이걸 방아쇠로 다른 동작을 건다. 대표적으로 SNS로 알림을 보내거나, **[Auto Scaling](/posts/amazon-ec2-auto-scaling/)** 을 발동해 인스턴스를 늘리고/줄인다. 이 덕분에 부하에 맞춰 자원을 자동으로 [프로비저닝](/posts/provisioning/)하며 [고가용성](/posts/high-availability/)을 유지하는 흐름의 중심에 CloudWatch가 있다.

지표 외에 **CloudWatch Logs** 로 로그를 중앙에 모아 검색·보존하고, **CloudWatch Events / EventBridge** 로 상태 변화에 반응하는 이벤트 기반 자동화를 구성한다. 즉 "관측(무슨 일이 일어나는지 본다) → 알람(이상을 감지한다) → 대응(자동으로 조치한다)"의 세 단계를 한 서비스로 엮는 게 CloudWatch의 그림이다.

주의할 점은 **CloudTrail과 헷갈리기 쉽다**는 것. CloudWatch는 "리소스가 어떻게 동작하는가(성능·상태)"를 보고, CloudTrail은 "누가 무슨 API를 호출했는가(감사·이력)"를 기록한다. 목적이 다르다.

## 예시
- CPU 사용률 알람으로 Auto Scaling을 트리거:
  1. EC2가 `CPUUtilization` 지표를 CloudWatch로 자동 전송
  2. "5분 평균 CPU ≥ 70%" 조건의 알람 생성
  3. 알람 발동 시 Auto Scaling 정책이 인스턴스 2대 추가
  4. 부하가 내려가면 반대 알람이 인스턴스를 축소
- Lambda 함수 오류를 감시: `Errors` 지표에 알람을 걸어 값이 0보다 커지면 SNS로 Slack 알림 발송.

## 관련
- [Amazon EC2 Auto Scaling](/posts/amazon-ec2-auto-scaling/) — CloudWatch 알람이 방아쇠가 되어 인스턴스를 늘리고 줄이는 짝 서비스
- [사전강의 - AWS Cloud Practitioner Essentials](/posts/aws-cloud-practitioner-essentials/) — CloudWatch 개념이 처음 등장한 강의 정리 노트
- [고가용성](/posts/high-availability/) — CloudWatch의 알람·Auto Scaling 연계로 유지하는 목표
- [프로비저닝](/posts/provisioning/) — CloudWatch 알람이 촉발하는 자원 준비 과정
- [온디맨드](/posts/on-demand/) — 필요한 만큼만 자원을 쓰게 하는, CloudWatch가 뒷받침하는 과금 방식
