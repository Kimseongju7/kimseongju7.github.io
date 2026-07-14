---
title: 'baidu/Unlimited-OCR Hugging Face 모델 카드 — Transformers·vLLM·SGLang 추론'
date: 2026-07-14 14:29:51 +0900
categories: [학습, 아티클]
tags: [ocr, huggingface, baidu, vllm, sglang, transformers, document-parsing, multimodal]
description: '바이두의 문서 파싱 모델 Unlimited-OCR의 Hugging Face 모델 카드 정리. "원샷 롱호라이즌 파싱(One-shot Long-horizon Parsing)"을 내세우는 3B BF16 모델로, Transformers·vLLM·SGLang 세 가지 추론 경로와 릴리스 이력, 평가 결과를 담고 있다.'
---
바이두의 문서 파싱 모델 **Unlimited-OCR**의 Hugging Face 모델 카드 정리. "원샷 롱호라이즌 파싱(One-shot Long-horizon Parsing)"을 내세우는 3B BF16 모델로, Transformers·vLLM·SGLang 세 가지 추론 경로와 릴리스 이력, 평가 결과를 담고 있다.

## 릴리스 이력

- [2026/06/22] DeepSeek-OCR을 한 단계 더 발전시키는 것을 목표로 [Unlimited-OCR](https://github.com/baidu/Unlimited-OCR) 공개
- [2026/06/23] 논문이 [arXiv](https://arxiv.org/abs/2606.23050)에 공개, [ModelScope](https://modelscope.cn/models/PaddlePaddle/Unlimited-OCR)에도 등록
- [2026/06/24] AK가 만든 데모가 [Hugging Face Spaces](https://huggingface.co/spaces/baidu/Unlimited-OCR)에 공개
- [2026/06/28] vLLM 커뮤니티와 Tianyu Guo의 지원으로 vLLM 추론 지원
- [2026/07/03] 바이두 클라우드(Baidu Cloud) 팀 지원으로 [Baidu Cloud](https://cloud.baidu.com/doc/OCR/s/fmr1p39gb)에서도 사용 가능

## Transformers 추론

NVIDIA GPU에서 Huggingface transformers로 추론한다. 테스트 환경은 python 3.12.3 + CUDA 12.9이며, `torch==2.10.0`, `torchvision==0.25.0`, `transformers==4.57.1`, `Pillow==12.1.1`, `pymupdf==1.27.2.2` 등을 요구한다.

- 모델 로드: `AutoModel.from_pretrained('baidu/Unlimited-OCR', trust_remote_code=True, use_safetensors=True, torch_dtype=torch.bfloat16)` 후 `.eval().cuda()`
- **단일 이미지**는 두 가지 설정 지원:
  - *gundam*: `base_size=1024`, `image_size=640`, `crop_mode=True`
  - *base*: `base_size=1024`, `image_size=1024`, `crop_mode=False`
- 단일 이미지 추론은 `model.infer(...)`에 프롬프트 `'<image>document parsing.'`, `max_length=32768`, `no_repeat_ngram_size=35`, `ngram_window=128` 등을 넘긴다.
- **다중 페이지 / PDF**는 base 모드(`image_size=1024`)만 사용하며 `model.infer_multi(...)`에 프롬프트 `'<image>Multi page parsing.'`, `ngram_window=1024`를 쓴다.
- PDF는 PyMuPDF(fitz)로 페이지를 dpi=300 이미지로 변환한 뒤 다중 페이지 파싱으로 처리한다.

## vLLM 추론

배포 상세는 공식 vLLM 레시피([https://recipes.vllm.ai/baidu/Unlimited-OCR](https://recipes.vllm.ai/baidu/Unlimited-OCR)) 참고. GPU 플랫폼에 따라 Docker 이미지를 고른다.

- 기본(CUDA 13.0): `docker pull vllm/vllm-openai:unlimited-ocr`
- Hopper GPU(CUDA 12.9): `docker pull vllm/vllm-openai:unlimited-ocr-cu129`

## SGLang 추론

uv 가상환경(python 3.12)에서 로컬 SGLang wheel을 먼저 설치하고, `kernels` 버전을 고정한 뒤 PDF 변환용 PyMuPDF를 설치한다(`uv pip install kernels==0.11.7`, `pymupdf==1.27.2.2`).

서버는 `python -m sglang.launch_server`로 띄우며 핵심 옵션은 다음과 같다.

- `--model baidu/Unlimited-OCR --served-model-name Unlimited-OCR`
- `--attention-backend fa3`, `--page-size 1`, `--mem-fraction-static 0.8`
- `--context-length 32768`, `--enable-custom-logit-processor`, `--disable-overlap-schedule`, `--skip-server-warmup`

클라이언트는 OpenAI 호환 API(`/v1/chat/completions`)로 스트리밍 요청을 보낸다. 요청 페이로드의 핵심:

- `temperature: 0`, `skip_special_tokens: False`
- `images_config: {"image_mode": "gundam"}` (단일 이미지) 또는 `"base"` (다중 이미지·PDF)
- `custom_logit_processor`에 `DeepseekOCRNoRepeatNGramLogitProcessor.to_str()` 지정
- `custom_params`: `ngram_size=35`, `window_size=128`(단일) / `1024`(다중·PDF)

## 모델 정보와 평가

- 파라미터 3B, 텐서 타입 BF16, Safetensors 배포. 최근 한 달 다운로드 1,506,937회.
- 평가([llamaindex/ParseBench](https://huggingface.co/datasets/llamaindex/ParseBench)): Mean 46.17, Text Content 86.81, Text Formatting 0.97 (외 3개 지표).
- Inference Provider에는 아직 배포되지 않았다 (Image-Text-to-Text 태스크).
- DeepSeek-OCR, DeepSeek-OCR-2, PaddleOCR의 모델과 아이디어에 감사를 표하고 있다.

## 관련 노트

- [baidu-unlimited-ocr](/posts/baidu-unlimited-ocr/) — 같은 모델의 vLLM 서빙 레시피(Docker·logits processor·프롬프트 규칙) 정리

> 원문: [Hugging Face](https://huggingface.co/baidu/Unlimited-OCR)
> 원본 클립: 2026-07-14-Hugging Face
