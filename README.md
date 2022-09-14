# DACON-Korean-sentence-relation-classification
Collaborator: Wonjun-chung

Team Name: 혹성탈출

code share: [https://dacon.io/competitions/official/235875/codeshare/4593?page=1&dtype=recent](https://dacon.io/competitions/official/235875/codeshare/4593?page=1&dtype=recent)

## 프로젝트 개요

한국어 Natural Language Inference Dataset을 활용하여(출처: [https://klue-benchmark.com/tasks/68/overview/description](https://klue-benchmark.com/tasks/68/overview/description)) Premise와 Hypothesis로 구성되어 있는 한쌍의 문장을 premise 문장을 참고해 hypothesis 문장이 참인지(Entailment), 거짓인지(Contradiction), 혹은 참/거짓 여부를 알 수 없는 문장인지(Neutral)를 판별.

출처: [https://dacon.io/competitions/official/235875/overview/description](https://dacon.io/competitions/official/235875/overview/description)

## 데이터셋 증강

1. 제공되는 KLUE 데이터 이외의 한국어 NLI 오픈소스 데이터셋인 XNLI, SNLI, MultiNLI 를 학습에 추가 활용함
    - SNLI, MultiNLI 데이터 (출처: [https://github.com/kakaobrain/KorNLUDatasets](https://github.com/kakaobrain/KorNLUDatasets))
2. 구글 Cloud에서 제공하는 translation을 활용하여 한 -> 영 -> 한으로 돌아오는 Backtranslation 기법을 사용하여 의미는 보존하되 다른 형태의 문장을 생성함.
    - Only hypothesis, Only premise, hypothesis & premise augmentation 모두 사용
    - 상대적으로 품질이 좋은(human annotated) KLUE, XNLI 데이터에 대해서만 진행
    - 예시 1) 어떤 방에서도 흡연은 금지됩니다. -> 모든 객실에서 금연입니다.
    - 예시 2) 10명이 함께 사용하기 불편함이 많았다. -> 10명이 사용하기에는 너무 불편했습니다.

## 모델 Fine-tunning

1. Model: KLUE pretrained Roberta-large (hugging face)
2. Layer-wise lr decay
    - Pre-trained bert 모델은 하위 layer에 일반적인 정보를 갖고 있음
    - 새로운 정보를 학습하는 동안 pre-trained된 정보가 잊혀지는 catastrophic forgetting이 존재함
    - 하위 Layer의 일반적인 정보가 잊혀지는 현상을 완화하기 위하여 하위 layer에 대해 작은 learning rate를 적용하는 Layer-wise lr decay 적용 (reference 1.)
3. Label smoothing
    - Gold Label 에 대한 Over-confidence를 줄이기 위해 label smoothing을 통한 Calibration 효과를 줌. (reference 2, 3)

## 학습

- AdamW & Cosine scheduler with warmup
- Layer-wise learning rate decay
- Label smoothing, Weight decay for generalization
- 2번의 학습
    - KLUE, XNLI, SNLI, MultiNLI 모두 사용하여 학습 진행 (Public: 0.88)
        - Training argument
            - weight decay: 0.01
            - lr: 2e-5
            - label smoothing: 0.05
            - batch size: 512
            - no layer-wise lr decay
            - no cosine scheduler
            - 2 epoch
    - Augmented KLUE, XNLI 에 대해서만 이어서 학습 진행 (Public: 0.896)
        - Training argument
            - weight decay:0.01
            - initinal lr: 2e-5
            - layer-wise lr decay (factor: 0.95)
            - label smoothing: 0.025
            - batch size: 1024
            - cosine scheduler
            - 5 epoch

## References

1. [https://arxiv.org/pdf/1905.05583.pdf](https://arxiv.org/pdf/1905.05583.pdf)
2. [https://papers.nips.cc/paper/2019/file/f1748d6b0fd9d439f71450117eba2725-Paper.pdf](https://papers.nips.cc/paper/2019/file/f1748d6b0fd9d439f71450117eba2725-Paper.pdf)
3. [http://proceedings.mlr.press/v70/guo17a/guo17a.pdf](http://proceedings.mlr.press/v70/guo17a/guo17a.pdf)
