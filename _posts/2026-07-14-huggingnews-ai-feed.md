---
title: 'HuggingNews — 읽을 가치 있는 AI 뉴스만 골라주는 AI 큐레이션 피드'
date: 2026-07-14 14:07:30 +0900
categories: [학습, 아티클]
tags: [ai, news, huggingface, curation, llm, open-weights]
description: 'Hugging Face의 clem(Clement Delangue)이 소개한 HuggingNews는 실제로 읽을 가치가 있는 AI 뉴스만 골라주는 AI 큐레이션 피드다. 곧 Hugging Face 프로필 기반 개인화도 지원할 예정이다.'
---
Hugging Face의 clem(Clement Delangue)이 소개한 **HuggingNews**는 실제로 읽을 가치가 있는 AI 뉴스만 골라주는 AI 큐레이션 피드다. 곧 Hugging Face 프로필 기반 개인화도 지원할 예정이다.

## 발표 내용

- "AI 뉴스를 따라가는 게 이제 풀타임 직업이 되어버렸다"는 문제의식에서 출발했다.
- clem의 친구 [@ivan_bezdomny](https://x.com/ivan_bezdomny)(Nikolai Yakovenko)가 만든 서비스로, 읽을 가치가 있는 뉴스를 골라주는 AI 큐레이션 피드다.
- 곧 Hugging Face 프로필을 활용해 피드를 개인화할 수 있게 될 예정이다.
- clem 본인이 몇 주째 사용 중이라며, 북마크해두거나 **에이전트에게 매일 아침·밤에 톱 10 스토리를 보내달라고 부탁하는** 활용법을 권했다.
- 슬로건: 노이즈는 줄이고, 시그널은 늘리고, 더 많이 빌드하라.

## 제작 비하인드

- ivan_bezdomny는 HuggingNews를 어떻게 구축했는지에 대한 X 기사를 썼다(7월 11일): "HuggingNews: How We Tuned a Small Open Weights Model to Write AI News Better Than Claude and GPT".
- 몇 주에 걸쳐 만든, 회사 발표부터 모델 출시까지 오늘의 AI 관련 뉴스 전반을 다루는 가볍지만 포괄적인 시스템이며, 특히 **최종 글쓰기용으로 오픈 웨이트 모델을 직접 튜닝**한 점이 특징이다.

## 반응

- HF 프로필 기반 커스텀 피드가 킬러 피처라는 반응 — 일반 AI 피드를 읽는 것만으로도 이미 소방호스에서 물을 마시는 기분이라는 공감.
- NotebookLM으로 이 피드에서 매일 AI 뉴스 팟캐스트를 만들어보겠다는 아이디어.
- "이름값대로 모델이나 Hugging Face에서 일어나는 일도 다뤘으면 좋겠다. 지금은 그날의 AI 트위터 모음일 뿐"이라는 비판적 의견도 있었다.
- "메타: HuggingNews의 릴리스가 HuggingNews에 올라갔나요?"라는 clem의 자문 농담도.

## 관련 노트

- [hugging-face](/posts/hugging-face/) — 같은 Hugging Face 생태계의 모델 카드 사례(clem/HF 플랫폼 맥락)

> 원문: [clem 🤗 on X: "Keeping up with AI news is becoming a full-time job. …"](https://x.com/ClementDelangue/status/2075605593973194999)
> 원본 클립: 2026-07-14-X에서 clem 🤗 님  "Keeping up with AI news is becoming a full-time job.So my friend @ivan_bezdomny built HuggingNews, an AI-curated feed that surfaces the news actually worth reading. Soon, it will even personalize the feed
