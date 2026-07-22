---
title: 'karpathy/nanochat — 100달러로 만드는 최고의 ChatGPT'
date: 2026-07-14 14:13:47 +0900
categories: [개발, AI]
tags: [nanochat, karpathy, llm, pretraining, gpt-2, pytorch, training]
description: 'nanochat은 안드레이 카파시(Andrej Karpathy)의 가장 단순한 LLM 학습 실험 하네스다. 단일 GPU 노드에서 토크나이징·사전학습·파인튜닝·평가·추론까지 LLM의 모든 주요 단계를 커버하며, 2019년 약 43,000달러였던 GPT-2 수준의 모델을 이제 약 48달러에 직접 학습할 수 있다.'
---
nanochat은 안드레이 카파시(Andrej Karpathy)의 가장 단순한 LLM 학습 실험 하네스다. 단일 GPU 노드에서 토크나이징·사전학습·파인튜닝·평가·추론까지 LLM의 모든 주요 단계를 커버하며, 2019년에 학습 비용이 약 43,000달러였던 GPT-2 수준의 모델을 이제 약 48달러(8XH100 노드 약 2시간)에 직접 학습해 CLI로 대화할 수 있다.

## 개요

- 코드가 최소한이고 해킹 가능(hackable)하게 설계됐다. 스팟 인스턴스를 쓰면 총비용이 약 15달러까지 내려간다.
- 복잡도 다이얼이 `--depth`(GPT 트랜스포머의 레이어 수) **단 하나**다. 이 정수 하나로 트랜스포머 너비, 헤드 수, 학습률 조정, 학습 기간, weight decay 등 나머지 하이퍼파라미터가 모두 compute-optimal하게 자동 계산된다. GPT-2 수준은 대략 depth 24~26 범위다.
- depth를 스윕하면 다양한 크기의 compute-optimal 모델 "미니시리즈"를 얻을 수 있다.
- 질문은 DeepWiki, GitHub Discussions, Discord #nanochat 채널에서 받는다.

## Time-to-GPT-2 리더보드

현재 개발의 주 초점은 가장 많은 컴퓨트를 쓰는 사전학습 단계 튜닝이다. modded-nanogpt에서 영감을 받아, DCLM CORE 점수로 측정한 GPT-2 수준 도달까지의 벽시계 시간(wall-clock time)을 겨루는 "GPT-2 스피드런" 리더보드를 운영한다. 기준 스크립트는 `runs/speedrun.sh`.

| # | 시간 | val_bpb | CORE | 설명 | 날짜 |
| --- | --- | --- | --- | --- | --- |
| 0 | 168시간 | - | 0.2565 | OpenAI GPT-2 원본 체크포인트 | 2019 |
| 1 | 3.04 | 0.74833 | 0.2585 | d24 베이스라인, 약간 과학습 | 2026-01-29 |
| 2 | 2.91 | 0.74504 | 0.2578 | d26 약간 미달학습 + fp8 | 2026-02-02 |
| 3 | 2.76 | 0.74645 | 0.2602 | 총 배치 사이즈 1M 토큰으로 증가 | 2026-02-05 |
| 4 | 2.02 | 0.71854 | 0.2571 | 데이터셋을 NVIDIA ClimbMix로 변경 | 2026-03-04 |
| 5 | 1.80 | 0.71808 | 0.2690 | autoresearch 라운드 1 | 2026-03-09 |
| 6 | 1.65 | 0.71800 | 0.2626 | autoresearch 라운드 2 | 2026-03-14 |

핵심 지표는 8XH100 노드에서 GPT-2(1.6B)의 CORE 점수 0.256525를 넘어서는 데 걸리는 시간이다. 현재 GPU 시세(~$3/GPU/hr, 노드당 ~$24/hr) 기준 2시간이면 약 48달러다.

## 시작하기

의존성 관리는 [uv](https://docs.astral.sh/uv/)를 쓴다.

```
uv sync --extra gpu    # CUDA (A100/H100 등)
uv sync --extra cpu    # CPU 전용 / MPS
source .venv/bin/activate
```

개발용(pytest, matplotlib, ipykernel, transformers 등 추가)은 `uv sync --extra gpu --group dev`.

GPT-2를 직접 재현하려면 8XH100 노드를 빌려(예: Lambda) 다음을 실행한다.

```
bash runs/speedrun.sh
```

약 1.5시간 걸리므로 screen 세션에서 돌리는 게 좋다. 끝나면 `python -m scripts.chat_cli`로 대화할 수 있다. 스피드런 모델은 4e19 FLOPs 수준이라 유치원생과 대화하는 느낌이다(예시 대화에서 "하늘이 왜 파랗냐"는 질문에 그럴듯하지만 틀린 설명을 한다).

추가 참고 사항:

- 8XA100(Ampere) 노드에서도 잘 돌아가지만 조금 느리다.
- `torchrun`을 빼면 단일 GPU에서도 동작하며(자동으로 gradient accumulation 전환) 결과는 거의 동일하지만 8배 오래 걸린다.
- GPU 메모리가 80GB 미만이면 `--device-batch-size`를 기본 32에서 16, 8, 4, 2, 1로 줄여 맞춘다.
- 대부분 바닐라 PyTorch 코드라 xpu, mps 등에서도 돌아갈 수 있으나 모든 경로가 검증된 것은 아니다.

## 연구 활용

연구자에게 유용한 스크립트는 `runs/scaling_laws.sh`와 `runs/miniseries.sh`. 빠른 실험(~5분 사전학습)에는 12레이어(GPT-1 크기) 모델이 적합하다.

```
OMP_NUM_THREADS=1 torchrun --standalone --nproc_per_node=8 -m scripts.base_train -- \
    --depth=12 \
    --run="d12" \
    --model-tag="d12" \
    --core-metric-every=999999 \
    --sample-every=-1 \
    --save-every=-1 \
```

개선 여부는 wandb에서 다음을 모니터링한다.

1. `val_bpb` — 어휘 크기에 무관한 bits per byte 단위의 검증 손실 (step, 총 학습 시간, 총 FLOPs 대비)
2. `core_metric` — DCLM CORE 점수
3. VRAM 사용률, `train/mfu`(Model FLOPS utilization), `train/tok_per_sec`(학습 처리량)

레포에 넣을 변경은 모든 depth 설정에서 동작할 만큼 원칙적(principled)이어야 한다.

## CPU / MPS 실행

`runs/runcpu.sh`가 CPU나 Apple Silicon에서 돌리는 간단한 예시다. 학습 모델을 대폭 축소해 수십 분 안에 끝나게 하지만, 강한 결과는 기대할 수 없다.

## 정밀도 / dtype

`torch.amp.autocast`를 쓰지 않고, `nanochat/common.py`의 전역 `COMPUTE_DTYPE` 하나로 정밀도를 명시적으로 관리한다. 하드웨어별 기본값:

| 하드웨어 | 기본 dtype | 이유 |
| --- | --- | --- |
| CUDA SM 80+ (A100, H100 등) | `bfloat16` | 네이티브 bf16 텐서 코어 |
| CUDA SM < 80 (V100, T4 등) | `float32` | bf16 없음. `NANOCHAT_DTYPE=float16`으로 fp16 가능(GradScaler 사용) |
| CPU / MPS | `float32` | 안전한 기본값. 최신 macOS의 MPS는 bf16도 잘 돌아감(메모리 ~25% 절약, 속도 유사) |

`NANOCHAT_DTYPE` 환경변수로 오버라이드할 수 있다. 가중치는 fp32로 저장하되 커스텀 `Linear` 레이어가 forward에서 `COMPUTE_DTYPE`으로 캐스팅하고, 임베딩은 메모리 절약을 위해 `COMPUTE_DTYPE`으로 직접 저장한다. autocast와 같은 혼합 정밀도 이점을 얻으면서 어떤 연산이 어떤 정밀도로 도는지 완전히 통제한다. fp16 학습 시 `base_train.py`에서 GradScaler가 자동 활성화된다(SFT도 지원, RL은 아직 미지원, fp16 추론은 어디서나 동작).

## 구조와 철학

- 레포는 `nanochat/`(GPT 모듈, 토크나이저, 데이터로더, KV 캐시 추론 엔진, Muon+AdamW 옵티마이저 등), `scripts/`(base/chat 학습·평가·SFT·RL·추론 벤치), `tasks/`(ARC, GSM8K, HumanEval, MMLU, SmolTalk), `runs/`(speedrun, miniseries, scaling laws, CPU 예시), `tests/`로 구성된다.
- 목표는 1,000달러 미만 예산으로 end-to-end로 다룰 수 있는 마이크로 모델의 최신 기술을 끌어올리는 것. 접근성은 비용뿐 아니라 인지적 복잡도의 문제이기도 해서, 거대한 설정 객체·모델 팩토리·if-then-else 괴물이 없는 단일하고 응집력 있는 "강한 베이스라인" 코드베이스를 지향한다.
- AI 기여 정책은 공개(disclosure): PR 제출 시 LLM이 실질적으로 기여했거나 본인이 완전히 이해하지 못한 부분을 밝혀야 한다.
- 이름은 사전학습만 다뤘던 전작 nanoGPT에서 왔고, 명확한 지표와 리더보드로 nanoGPT를 게임화한 modded-nanoGPT에서 많은 아이디어를 빌렸다. 라이선스는 MIT.

## 관련 노트

- [cs336-llm-course-value](/posts/cs336-llm-course-value/) — "직접 LLM을 손으로 만든다"는 목표를 강의로 다루는 CS336
- [cs336-study-advice](/posts/cs336-study-advice/) — 같은 CS336을 추천하며 밑바닥 구현을 강조하는 학습 조언
- [ilya-sutskever-reading-list](/posts/ilya-sutskever-reading-list/) — 사전학습·트랜스포머를 깊게 이해하기 위한 딥러닝 읽기 목록
- [도대체 루프가 뭔데?](/posts/what-is-a-loop-anyway/) — 리더보드 개선에 쓰인 카파시의 autoresearch(시스템 루프) 개념을 다루는 글

> 원문: [karpathy/nanochat: The best ChatGPT that $100 can buy.](https://github.com/karpathy/nanochat)
> 원본 클립: 2026-07-14-karpathynanochat The best ChatGPT that $100 can buy.
