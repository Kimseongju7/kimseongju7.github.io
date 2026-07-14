---
title: "baidu/Unlimited-OCR — 3B DENSE 문서 파싱 모델"
date: 2026-07-01 23:40:00 +0900
categories: [학습, 아티클]
tags: [ocr, vllm, baidu, document-parsing, multimodal, llm]
slug: baidu-unlimited-ocr
publish: true
---
바이두(Baidu)의 문서 파싱 특화 모델 **Unlimited-OCR**은 R-SWA(Reference Sliding Window Attention)를 적용해 전체 페이지 OCR과 마크다운 변환에 최적화된 3B 규모의 멀티모달 모델이다. 한 번의 추론으로 긴 문서를 통째로 마크다운으로 뽑아내는 것이 목표다.

## 개요

`baidu/Unlimited-OCR`은 DeepSeek-OCR 계열의 문서 파싱 모델이다. DeepSeek-OCR의 *gundam* 비전 스택(SAM-ViT-B + CLIP-L DeepEncoder, `base_size=1024` / `image_size=640` / crop)을 그대로 공유하며, 반복 없이 깔끔하게 문서를 마크다운으로 변환하기 위한 n-gram logits processor도 동일하게 탑재하고 있다.

- 종류: dense, 3B, 컨텍스트 32,768 토큰, 멀티모달
- 요구 버전: [vLLM 0.25.0+](https://vllm.ai/#quick-start)
- 모델 카드: [HuggingFace](https://huggingface.co/baidu/Unlimited-OCR)

## 사전 준비물

- **하드웨어**: BF16 추론 기준 VRAM 8GB 이상 GPU 1장이면 충분하다.
- **vLLM**: 이 아키텍처는 아직 안정 pip wheel에 포함돼 있지 않아, 전용 릴리스 이미지로 서빙해야 한다. Install → Docker 탭에 나오는 이미지를 받아 쓴다.

## 서빙 레시피 (필수)

이 모델은 **채팅 템플릿이 없고** 특정 프롬프트/디코드 레시피에 맞춰 학습돼 있다. 이 레시피를 지키지 않으면 모델이 **빈 출력**을 낸다. 필수 요소는 다음과 같다.

- 서버에 no-repeat-ngram logits processor 등록: `--logits_processors vllm.model_executor.models.unlimited_ocr:NGramPerReqLogitsProcessor`
- 프롬프트 텍스트는 반드시 리터럴 `<image>` 로 시작해야 한다 (예: `<image>document parsing.`)
- `skip_special_tokens=False`
- 요청마다 processor 인자 전달: `ngram_size=35`, `window_size=128` (다중 페이지 / PDF 입력에는 `window_size=1024` 사용)

## Docker 실행

```
docker run --gpus all \
  --privileged --ipc=host -p 8000:8000 \
  -v ~/.cache/huggingface:/root/.cache/huggingface \
  vllm/vllm-openai:unlimited-ocr baidu/Unlimited-OCR \
  --trust-remote-code \
  --logits_processors vllm.model_executor.models.unlimited_ocr:NGramPerReqLogitsProcessor \
  --no-enable-prefix-caching \
  --mm-processor-cache-gb 0 \
  --tensor-parallel-size 1
```

기본적으로 노드의 8개 GPU 중 1개에서 돌아간다. 규모를 키우려면 Advanced에서 `--tensor-parallel-size N` 을 추가한다. 이 방식은 단일 노드 텐서 병렬(single-node tensor parallel)로, 모델을 로컬 GPU들에 나눠 싣는다. TP 크기는 배포 시점의 GPU 수로 설정되며, 모든 모델 아키텍처에서 동작하는 가장 단순한 다중 GPU 전략이다.

## 클라이언트 사용법

### 온라인 OCR 서빙

```bash
docker run --rm --gpus all --network host --ipc host \
  vllm/vllm-openai:unlimited-ocr \
  baidu/Unlimited-OCR \
  --trust-remote-code \
  --logits_processors vllm.model_executor.models.unlimited_ocr:NGramPerReqLogitsProcessor \
  --no-enable-prefix-caching \
  --mm-processor-cache-gb 0
```

```python
import time
from openai import OpenAI

client = OpenAI(api_key="EMPTY", base_url="http://localhost:8000/v1", timeout=3600)

messages = [
    {
        "role": "user",
        "content": [
            {"type": "text", "text": "<image>document parsing."},
            {"type": "image_url", "image_url": {"url": "https://huggingface.co/baidu/Unlimited-OCR/resolve/main/assets/baidu.png"}},
        ],
    }
]

start = time.time()
response = client.chat.completions.create(
    model="baidu/Unlimited-OCR",
    messages=messages,
    max_tokens=8192,
    temperature=0.0,
    extra_body={
        "skip_special_tokens": False,
        "vllm_xargs": {"ngram_size": 35, "window_size": 128},
    },
)
print(f"Response costs: {time.time() - start:.2f}s")
print(f"Generated text: {response.choices[0].message.content}")
```

원시(raw) 출력에는 `<|ref|>…<|/ref|>` / `<|det|>…<|/det|>` grounding 토큰이 함께 담겨 나온다. 깔끔한 마크다운을 얻으려면 `<|ref|>` 안의 텍스트를 벗겨내고 `<|det|>` 좌표 박스는 버리면 된다.

## 트러블슈팅 / 설정 팁

- **커스텀 logits processor는 필수다.** 이걸 빼면 긴 문서에서 `<|det|>` 좌표 토큰이 무한 반복(loop)된다.
- 빈 출력이 나온다면 십중팔구 프롬프트에 리터럴 `<image>` 접두사가 빠졌거나 `skip_special_tokens` 가 `True` 로 남아 있는 경우다.
- OCR 작업은 prefix caching / 이미지 재사용의 이점이 없으므로, 레시피에서 prefix caching과 mm-processor 캐시를 모두 꺼둔다.
- 단일 이미지 입력은 gundam(crop) 모드를 쓰고, 다중 이미지 요청은 자동으로 non-crop(base) 모드로 전환된다 — 이 경우 `window_size=1024` 를 사용한다.

## 관련 노트

- [[hugging-face]] — 같은 Unlimited-OCR 모델의 Hugging Face 모델 카드(Transformers·SGLang 추론 경로와 평가 결과)

> 원문: [baidu/Unlimited-OCR — 3B · DENSE](https://recipes.vllm.ai/baidu/Unlimited-OCR)
> 원본 클립: [[2026-07-01-baiduUnlimited-OCR — 3B · DENSE]]
