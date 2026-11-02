---
date: 2026-11-02
created: 2026-11-02
collected: 2026-11-02
document_role: source
domain: article
type: article-source
title: "Memento: 성공과 실패 사례를 별도 공간에 저장하는 방식으로, LLM 파인튜닝 없이 성능을 높이는 딥 리서치 에이전트"
source_url: "https://discuss.pytorch.kr/t/memento-llm/12309"
publisher: "PyTorch 한국 사용자 모임 (PyTorch KR)"
published: 2026-10-27
language: ko
tags:
  - learning
  - source
  - article
  - agent-memory
  - llm-agent
  - case-based-reasoning
learning_notes:
  - "[[Learning/학습노트/2026-11-02-Memento-CBR-에이전트-메모리와-실무-적용-노트]]"
---

# Memento: 성공과 실패 사례를 별도 공간에 저장하는 방식으로, LLM 파인튜닝 없이 성능을 높이는 딥 리서치 에이전트

- **출처:** [PyTorch 한국 사용자 모임](https://discuss.pytorch.kr/t/memento-llm/12309)
- **원문 논문:** *Memento: Fine-tuning LLM Agents without Fine-tuning LLMs* (Huawei Noah's Ark Lab & UCL AI Centre, 2025.08, [arXiv:2508.16153](https://arxiv.org/abs/2508.16153))
- **코드 저장소:** [GitHub (Memento-Teams/Memento)](https://github.com/Memento-Teams/Memento)

> [!info] 원문 보관 범위
> 파이토치 한국 사용자 모임(PyTorch KR) 기술 소개 아티클 원문 보관본입니다.
> 상단 사이트 로고·네비게이션 및 하단 사이트 푸터·약관·회원가입 안내를 정리하고 순수 본문 내용과 링크, 표, 코드 블록을 온전히 보존했습니다.

## 핵심 요약

* [Memento](https://github.com/Memento-Teams/Memento)는 LLM 가중치를 고치지 않고, 과거 과제의 (질문, 계획, 성공 여부)를 케이스 뱅크에 쌓아 다음 과제의 계획에 참고하게 하는 딥 리서치 에이전트입니다.
* [논문](https://arxiv.org/abs/2508.16153)은 GAIA 검증(validation) 세트 87.88%(Pass@3)와 테스트(test) 세트 79.40%를 보고하지만, 사례 기반 추론(CBR)만 따로 더한 몫은 F1 기준 3.7~6.7점입니다.
* 공개된 모수(parametric) 메모리는 논문의 온라인 soft Q-learning이 아니라, 미리 모은 데이터로 SimCSE RoBERTa 분류기를 오프라인 학습하는 방식입니다.
* 성공 여부를 정답과 비교해 판정하므로, 정답이 있는 데이터셋에서 실험하려는 연구자에게 맞고 정답 없이 쓰는 배포 환경에는 그대로 옮기기 어렵습니다.

[![Memento: 성공과 실패 사례를 케이스 뱅크에 쌓아 LLM 파인튜닝 없이 성능을 높이는 딥 리서치 에이전트](https://discuss.pytorch.kr/uploads/default/optimized/3X/d/b/db54ed1704559c4e8f2bbdd9f19651ef7c79deff_2_1028x671.jpeg)

Memento: 성공과 실패 사례를 케이스 뱅크에 쌓아 LLM 파인튜닝 없이 성능을 높이는 딥 리서치 에이전트1536×1003 277 KB](https://discuss.pytorch.kr/uploads/default/original/3X/d/b/db54ed1704559c4e8f2bbdd9f19651ef7c79deff.jpeg "Memento: 성공과 실패 사례를 케이스 뱅크에 쌓아 LLM 파인튜닝 없이 성능을 높이는 딥 리서치 에이전트")

## Memento 소개

LLM 에이전트가 배포 뒤에도 나아지게 하는 방법은 크게 두 가지였습니다. 하나는 고정된 워크플로우와 반성(reflection) 프롬프트를 손으로 짜 두는 방식인데, 배포한 뒤에는 새 상황에 적응하지 못합니다. 다른 하나는 지도 학습이나 강화학습으로 LLM 파라미터를 직접 갱신하는 방식으로, 유연하지만 학습 데이터와 연산 비용이 커서 계속 갱신하기 어렵습니다. 이번에 소개하는 Memento는 그 사이에서 LLM은 고정하고, 에이전트가 과거에 성공하거나 실패한 사례를 메모리에 쌓아 비슷한 문제를 만났을 때 꺼내 쓰게 하는 딥 리서치 에이전트 프로젝트입니다.

Memento는 이 아이디어를 사례 기반 추론(Case-Based Reasoning, CBR)으로 정리합니다. 새 과제가 들어오면 플래너(planner)가 케이스 뱅크(Case Bank)에서 비슷한 과거 사례를 찾아 프롬프트에 붙이고, 그 사례를 참고해 과제를 하위 과제로 나눕니다. 실행기(executor)는 [MCP(Model Context Protocol)](https://modelcontextprotocol.io/) 클라이언트로서 검색, 크롤링(crawling), 문서 파싱, 코드 실행 도구를 호출해 하위 과제를 풉니다. 과제가 끝나면 결과의 성공 여부와 함께 새 사례가 케이스 뱅크에 추가되므로, 같은 LLM을 쓰더라도 참고할 사례가 점점 늘어납니다.

Memento는 2025년 8월 arXiv에 공개된 논문 "Memento: Fine-tuning LLM Agents without Fine-tuning LLMs"의 공식 구현입니다. 논문은 [Huawei Noah's Ark Lab](https://www.noahlab.com.hk/)과 UCL(University College London) AI Centre 등의 연구진이 썼습니다. 논문 실험에서는 플래너로 GPT-4.1, 실행기로 o3(GAIA) 또는 o4-mini(그 밖의 벤치마크)를 쓰고, 사례 검색에는 [SimCSE](https://github.com/princeton-nlp/SimCSE) 문장 임베딩을 씁니다. 저장소는 Python 3.11 이상을 요구하며, 실행기의 도구는 모두 FastMCP 서버로 구현되어 있습니다. 공개 코드에는 학습이 없는 비모수(non-parametric) 메모리와 분류기를 학습하는 모수 메모리가 각각 별도 실행 스크립트로 들어 있습니다.

## Memento vs. 다른 에이전트 메모리 방식

경험을 메모리에 쌓아 에이전트를 개선한다는 목표는 다른 프로젝트들과 같지만, 무엇을 저장하고 성공을 어떻게 판정하는지가 다릅니다. 아래는 PyTorchKR에서 소개한 두 프로젝트와 비교한 표입니다:

| 항목 | **Memento** | [ReasoningBank](https://discuss.pytorch.kr/t/reasoningbank-feat-google/9854) | [Mem0](https://discuss.pytorch.kr/t/mem0-ai/5179) |
| --- | --- | --- | --- |
| 저장하는 것 | 과제별 (질문, 계획, 성공 여부) 원본 사례 | 성공과 실패 궤적에서 증류(distillation)한 추론 전략 | 사용자, 세션, 에이전트 수준의 기억 |
| 성공 판정 | 정답과 비교하는 LLM 판정(gpt-4o-mini) | 정답 없이 LLM-as-a-judge로 자체 평가 | 해당 없음 |
| 주요 적용 과제 | GAIA, DeepResearcher, SimpleQA, HLE(Humanity's Last Exam) | WebArena, SWE-Bench-Verified | 개인화 AI 어시스턴트, 고객 지원 챗봇 |

Memento와 ReasoningBank는 둘 다 실패 사례를 버리지 않고 저장하지만, 그 실패를 어떻게 아는지가 다릅니다. ReasoningBank는 에이전트가 자기 궤적을 LLM 판정기로 평가하므로 정답이 없어도 동작합니다. 반면 Memento의 공개 스크립트는 데이터셋의 정답(`ground_truth`)과 예측을 비교해 성공 여부를 기록하므로, **정답이 없는 실제 질의에서는 케이스 뱅크에 넣을 보상을 만드는 단계를 직접 설계해야 합니다.** Mem0는 사용자 선호와 대화 맥락을 기억하는 메모리 계층이라, 과제 해결 계획을 재사용하는 Memento와는 쓰임새가 다릅니다.

## Memento를 사용하면 좋을 사용자

정답이 있는 질의응답 벤치마크에서 에이전트 메모리나 사례 기반 추론을 실험하려는 연구자에게는 Memento가 좋은 출발점입니다. 플래너와 실행기, MCP 도구 서버, 케이스 뱅크 기록과 분류기 학습 스크립트까지 한 저장소에 있고, 논문의 DeepResearcher 평가 데이터도 함께 들어 있습니다. 반면 정답 없이 들어오는 사용자 질의로 에이전트를 계속 개선하려는 팀이나, 논문의 온라인 soft Q-learning 구현을 기대하는 독자에게는 맞지 않습니다. **API 비용과 정답 라벨 없이 바로 서비스에 붙일 메모리를 찾는 팀에게는 Memento가 아직 적절한 선택지가 아닙니다.**

## Memento의 동작 원리: 메모리 기반 MDP와 케이스 뱅크

Memento 논문은 에이전트의 의사결정을 메모리 기반 마르코프 결정 과정(Memory-Based Markov Decision Process, M-MDP)으로 정식화합니다. 표준 MDP에 과거 경험의 집합인 메모리 공간 \mathcal{M} 을 더한 것이 차이입니다. 매 단계에서 에이전트는 검색 정책 \mu 로 케이스 뱅크 M 에서 사례 c=(s,a,r) 를 고르고, 고정된 LLM이 현재 상태 s 와 그 사례를 보고 행동을 만듭니다. 그래서 전체 정책은 다음과 같이 쓰이고, 학습 대상은 LLM이 아니라 검색 정책 \mu 뿐입니다:

\pi(a \mid s, M) = \sum\_{c \in M} \mu(c \mid s, M)\, p\_{\text{LLM}}(a \mid s, c)

저자들은 "어떤 사례를 고를지"를 행동으로 보고 최대 엔트로피 강화학습(soft Q-learning)을 적용합니다. 그러면 최적 검색 정책은 Q 값에 대한 소프트맥스(softmax) \mu^\*(c \mid s, M) \propto \exp(Q^\*(s, M, c)/\alpha) 가 됩니다(유도는 논문 부록 A). 다만 자연어로 된 상태에서 Q 함수를 바로 학습하기는 어렵습니다. 그래서 논문 3절은 같은 사례를 썼던 과거 상호작용의 Q 값을 학습 가능한 커널로 가중 평균하는 에피소드 제어 방식(Algorithm 1)도 제안합니다.

아래 그림은 논문 Figure 3의 전체 구조로, 왼쪽이 사례 기반 계획(Case-Based Planning), 오른쪽이 도구 기반 실행(Tool-based Execution) 단계입니다:

[![Memento의 전체 구조 다이어그램, 케이스 메모리를 쓰는 플래너와 MCP 도구를 호출하는 실행기](https://discuss.pytorch.kr/uploads/default/optimized/3X/8/5/85880b77955031dbaef0249f38586bcd1a6f1f06_2_1028x423.png)

Memento의 전체 구조 다이어그램, 케이스 메모리를 쓰는 플래너와 MCP 도구를 호출하는 실행기1870×770 292 KB](https://discuss.pytorch.kr/uploads/default/original/3X/8/5/85880b77955031dbaef0249f38586bcd1a6f1f06.png "Memento의 전체 구조 다이어그램, 케이스 메모리를 쓰는 플래너와 MCP 도구를 호출하는 실행기")

플래너는 Case Memory에서 사례를 읽어 계획을 만들고, Subtask Memory에 하위 과제와 결과를 기록합니다. 실행기는 하위 과제마다 Tool Memory의 이력을 보고 MCP 서버의 도구(Search, Crawl, Video, Image, Code, Math, Doc)를 호출합니다. 플래너는 결과를 보고 다시 계획하거나 최종 답을 내며, 케이스 뱅크에는 과제가 끝났을 때 마지막 단계의 (상태, 계획, 보상)만 기록합니다. 논문 4.2절에 따르면 실험의 플래너는 사례를 계획에만 쓰므로, 저자들은 문제를 단일 단계로 줄였습니다. 그러면 시간차(Temporal Difference, TD) 학습의 부트스트랩이 사라지고, 학습 목표는 "이 사례가 이 과제에 좋은 참고인가"를 맞히는 이진 분류가 됩니다.

### Memento의 두 가지 케이스 메모리

비모수 메모리는 학습 없이 현재 질문과 저장된 질문의 SimCSE 임베딩(`sup-simcse-bert-base-uncased`) 코사인 유사도로 상위 K개 사례를 고릅니다. 모수 메모리는 (질문, 사례와 계획) 쌍을 입력받아 그 사례가 도움이 될 확률을 내는 분류기를 학습하고, 점수가 높은 K개를 고릅니다. 공개된 [`memory/train_memory_retriever.py`](https://github.com/Memento-Teams/Memento/blob/main/memory/train_memory_retriever.py)의 분류기는 다음과 같습니다:

```
class MemoryRetrieverClassifier(nn.Module):
    def __init__(self, sentence_bert: AutoModel):
        super().__init__()
        hidden = sentence_bert.config.hidden_size
        self.sentence_bert = sentence_bert
        self.classifier = nn.Sequential(
            nn.Linear(hidden * 2, 512),
            nn.ReLU(),
            nn.Dropout(0.2),
            nn.Linear(512, 2)
        )

    def forward(self, ids1, mask1, ids2, mask2):
        o1 = self.sentence_bert(ids1, attention_mask=mask1).last_hidden_state[:, 0]
        o2 = self.sentence_bert(ids2, attention_mask=mask2).last_hidden_state[:, 0]
        return self.classifier(torch.cat([o1, o2], dim=1))
```

백본은 [`princeton-nlp/sup-simcse-roberta-base`](https://huggingface.co/princeton-nlp/sup-simcse-roberta-base)이고, 학습 스크립트는 백본을 고정하지 않고 분류 헤드와 함께 AdamW로 학습합니다. 손실은 클래스 가중치를 준 교차 엔트로피이고, 라벨(`truth_label`)은 그 사례를 참고해 푼 과제가 정답으로 판정됐는지입니다. 검색할 때는 질의마다 케이스 뱅크의 모든 사례와 쌍을 만들어 점수를 매기므로, 사례 수에 비례해 비용이 늘어납니다. 저자도 [이슈 #25](https://github.com/Memento-Teams/Memento/issues/25)에서 현재 메모리 모듈이 연구용이며 규모보다 단순성을 우선했다고 답했습니다.

## Memento의 공개 코드와 논문의 차이

논문의 수식과 공개 코드 사이에는 독자가 미리 알아야 할 차이가 있습니다. 논문의 Algorithm 1과 Figure 3은 메모리를 쓸 때마다 Q 함수를 온라인으로 갱신한다고 설명합니다. 그러나 공개된 모수 메모리는 [`client/parametric_memory_cbr.py`](https://github.com/Memento-Teams/Memento/blob/main/client/parametric_memory_cbr.py)가 과제를 풀 때마다 케이스 뱅크와 학습 데이터 파일에 줄을 추가하고, 분류기 학습은 별도 스크립트로 오프라인에서 실행하는 구조입니다. 실행 중에 분류기를 갱신하는 코드는 없고, 커널 기반 에피소드 제어(논문 식 9~11)의 구현도 없습니다. README의 완료 항목에는 학습 데이터 수집과 재학습 파이프라인이 올라 있지만, 프로젝트 구조도에 있는 재학습 스크립트 `memory/retrain.sh`는 저장소에 없습니다. 같은 질문을 담은 [이슈 #31](https://github.com/Memento-Teams/Memento/issues/31)과 [이슈 #34](https://github.com/Memento-Teams/Memento/issues/34)에는 2026년 10월 26일 기준 저장소 관리자의 답이 없습니다.

저장소에 함께 들어 있는 메모리 파일도 확인이 필요합니다. [`memory/memory.jsonl`](https://github.com/Memento-Teams/Memento/blob/main/memory/memory.jsonl)에는 사례 1,278개(성공 897개, 실패 381개)가 들어 있는데, 그중 1,263개가 평가용 질문 파일 [`data/deepresearcher.jsonl`](https://github.com/Memento-Teams/Memento/blob/main/data/deepresearcher.jsonl)의 질문과 같습니다. 모수 스크립트는 기본값으로 이 메모리를 불러와 평가 파일을 순서대로 풀기 때문에, 그대로 실행하면 중복을 뺀 평가 질문 3,193개 가운데 약 40%에서 같은 질문의 과거 계획과 성공 여부를 참고하게 됩니다. 이 메모리에는 논문이 분포 밖(OOD) 평가에 쓴 MusiQue, Bamboogle, PopQA 질문도 들어 있으므로, 논문의 수치를 재현하려면 빈 메모리에서 시작하거나 데이터를 분리해야 합니다. 비모수 스크립트의 기본 메모리는 예시 8개짜리 `memory/dummy_memo.jsonl`인데, 이 8개도 모두 평가 질문입니다.

모수 메모리용 체크포인트(`memory/ckpts/retriever/best.pt`)는 저장소에 없어서, 함께 들어 있는 학습 데이터 7,000줄로 분류기를 먼저 학습해야 사례 검색이 동작합니다. 체크포인트나 메모리 의존성이 없어도 모수 스크립트는 경고 로그만 남기고 사례 없이 계속 실행됩니다. 따라서 실행 로그에서 메모리를 실제로 불러왔는지 확인해야 합니다.

## Memento 벤치마크

Memento 연구진이 논문에서 보고한 주요 결과와 측정 조건은 다음과 같습니다:

| 벤치마크 | 결과 | 조건 |
| --- | --- | --- |
| GAIA validation | 87.88% | 빈 메모리에서 validation 문제를 세 번 반복하며 사례를 쌓은 뒤의 Pass@3 |
| GAIA test | 79.40% | validation에서 쌓은 케이스 뱅크만 사용 (2025년 6월 26일 리더보드) |
| DeepResearcher 7종 평균 | F1 66.6 / PM 80.4 | 비교 대상은 Qwen2.5-7B를 학습한 DeepResearcher(F1 51.8 / PM 60.5) |
| SimpleQA | 95.0% | 플래너 GPT-4.1, 실행기 o4-mini |
| HLE | PM 24.4% | 같은 구성, 공식 리더보드 모델과 비교 |

GAIA validation 수치는 같은 문제에서 얻은 경험이 메모리에 들어간 값이므로, 다른 시스템과 비교할 때는 test의 79.40%가 더 공정합니다. 논문 본문은 test 순위를 4위로 적었고, 표 2 캡션은 오픈소스 에이전트 프레임워크 가운데 1위라고 적었습니다. DeepResearcher 비교도 7B 모델을 학습한 시스템과 GPT-4.1, o4-mini 조합을 비교한 결과라, 학습 기반 방법보다 메모리 방식이 낫다는 근거로 읽기에는 조건이 다릅니다.

구성 요소별 분석(논문 표 5)에서는 큰 폭의 향상이 플래닝에서 나옵니다. 도구만 붙인 실행기에 플래너를 더하면 F1이 HLE에서 11.0점, SimpleQA에서 32.5점, DeepResearcher에서 29.1점 오르고, 여기에 CBR을 더한 몫은 각각 4.5점, 3.7점, 6.7점입니다. 분포 밖 데이터셋 세 개(MusiQue, Bamboogle, PopQA)에서는 Natural Questions, TriviaQA, HotpotQA, 2WikiMultihopQA의 사례만 케이스 뱅크에 쌓아 두고 평가했습니다. 저자들은 이때 CBR을 끈 구성보다 F1과 PM이 4.7~9.6점 높았다고 보고합니다(논문 Figure 1d). 검색하는 사례 수는 K=4일 때 평균 F1 64.5, PM 78.5로 가장 좋았고, 그보다 늘리면 정체하거나 조금 떨어졌습니다. 공개 스크립트의 기본값(`MEMORY_TOP_K`)은 8이므로 논문 설정을 따르려면 4로 바꿔야 합니다.

아래 그림은 DeepResearcher 데이터에서 다섯 번 반복하며 측정한 지속 학습 곡선(논문 Figure 1c, 표 4)입니다. 가로축은 Iteration, 세로축은 Accuracy (%)이고, 세 선은 Memento w/o CBR, Memento w/ Non-Parametric CBR, Memento w/ Parametric CBR입니다:

[![Memento의 반복 횟수별 정확도 곡선, CBR 없음과 비모수 CBR과 모수 CBR 비교](https://discuss.pytorch.kr/uploads/default/optimized/3X/9/1/919e952462d02509165f1b69977dafeded929049_2_1028x647.png)

Memento의 반복 횟수별 정확도 곡선, CBR 없음과 비모수 CBR과 모수 CBR 비교1431×902 79 KB](https://discuss.pytorch.kr/uploads/default/original/3X/9/1/919e952462d02509165f1b69977dafeded929049.png "Memento의 반복 횟수별 정확도 곡선, CBR 없음과 비모수 CBR과 모수 CBR 비교")

CBR이 없는 기준선도 78.65에서 84.47까지 오르는데, 저자는 [이슈 #1](https://github.com/Memento-Teams/Memento/issues/1)에서 매 반복이 이전 반복에서 틀린 문제만 다시 푸는 설정이라 여러 번 샘플링한 효과가 생긴다고 설명했습니다. 그래서 이 그림에서 메모리의 기여는 선 사이의 간격이고, 다섯 번째 반복 기준으로 비모수 CBR은 0.38점, 모수 CBR은 0.97점 높습니다. 그 밖에 GAIA validation의 pass@1 비교(논문 표 6)에서는 GPT-4.1 플래너와 o3 실행기 조합이 70.91%로, o3를 플래너로 쓴 조합(63.03%)보다 높았습니다. 저자들은 추론 모델을 플래너로 쓰면 계획을 건너뛰고 바로 답하거나 장황한 계획을 내는 경우가 있었다고 분석합니다. 비용 면에서는 GAIA validation 과제 10개를 무작위로 골라 측정한 과제당 입력 토큰이 Level 1에서 약 2만 6천 개, Level 3에서 약 12만 1천 개였고, 출력 토큰은 4.7천~9.8천 개 수준이었습니다(논문 6.2절).

## Memento 설치와 사용

README의 설치 절차는 저장소를 받아 의존성을 설치하는 것으로 시작합니다. README의 clone 주소는 예전 조직명(`Agent-on-the-Fly/Memento`)이지만 현재 `Memento-Teams/Memento`로 연결되므로, 아래 명령에서는 새 주소를 씁니다:

```
git clone https://github.com/Memento-Teams/Memento
cd Memento

python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
# requirements.txt에 없는 크롤러, SerpAPI 클라이언트, 비디오와 문서 도구용 패키지
pip install crawl4ai google-search-results ffmpeg-python PyPDF2 openpyxl xls2xlsx tabulate

# 웹 크롤링 도구 설정
crawl4ai-setup
crawl4ai-doctor
playwright install
```

README가 권장하는 `uv sync`는 `pyproject.toml`을 따르는데, 여기에는 `torch`와 `transformers`가 없어 케이스 메모리 코드를 불러오지 못합니다. 그래서 위 명령은 두 패키지가 들어 있는 `requirements.txt`를 쓰고, 그 파일에 빠진 [Crawl4AI](https://github.com/unclecode/crawl4ai) (![:pytorch:](https://discuss.pytorch.kr/uploads/default/original/2X/f/fa98c2196c22febe7475e503792febf39ba7a0de.svg?v=15 ":pytorch:")![:kr:](https://discuss.pytorch.kr/uploads/default/original/2X/0/095f13ff3222d3a60c752712faf5af2e6995a838.png?v=15 ":kr:") [Crawl4AI: LLM 및 AI 애플리케이션을 위한 오픈소스 웹 크롤러(Crawler)](https://discuss.pytorch.kr/t/crawl4ai-llm-ai-crawler/5282))와 SerpAPI 클라이언트, 도구 서버가 불러오는 패키지를 따로 설치합니다. 특히 비디오 도구는 시작할 때 `ffmpeg-python`을 불러오는데, 클라이언트는 도구 서버 하나만 실패해도 연결 단계에서 멈춥니다. 비디오 처리에는 시스템 FFmpeg가 필요하고, `.env`에 `OPENAI_API_KEY`와 문서 파싱용 `CHUNKR_API_KEY`, 음성 인식용 `ASSEMBLYAI_API_KEY` 등을 채웁니다.

메모리 없이 플래너와 실행기만 써 보려면 대화형 클라이언트를 실행합니다. 이 클라이언트는 실행기로 o3를 쓰고, 검색에는 자체 호스팅한 [SearXNG](https://github.com/searxng/searxng)를 씁니다:

```
cd searxng-docker
docker compose up -d

cd ../client
python agent.py
```

모수 메모리는 분류기를 먼저 학습한 뒤 실행합니다. README의 실행 안내에는 `parametric_memory.py`로 적혀 있지만 실제 파일은 [`client/parametric_memory_cbr.py`](https://github.com/Memento-Teams/Memento/blob/main/client/parametric_memory_cbr.py)입니다:

```
cd memory
python train_memory_retriever.py \
  --train training_data.jsonl \
  --output_dir ./ckpts/retriever \
  --use_plan \
  --val_ratio 0.1 \
  --batch_size 32 \
  --lr 2e-5 \
  --epochs 10 \
  --save_best

cd ../client
python parametric_memory_cbr.py
```

두 CBR 스크립트는 대화형 도구가 아니라 `data/deepresearcher.jsonl`의 질문을 차례로 풀고 정답과 비교해 기록하는 배치 실행기입니다. 검색 도구로는 [SerpAPI](https://serpapi.com/)를 쓰는 [`server/serp_search.py`](https://github.com/Memento-Teams/Memento/blob/main/server/serp_search.py)를 연결하는데, 이 파일은 API 키를 빈 문자열로 코드에 적어 두었으므로 직접 채워야 합니다. 이 도구가 쓰는 `serpapi` 모듈은 PyPI의 [`google-search-results`](https://pypi.org/project/google-search-results/) 패키지가 제공하며, 저장소의 두 의존성 파일 어디에도 들어 있지 않아 위에서 따로 설치했습니다. 자기 데이터로 실험하려면 같은 형식(`question`, `ground_truth`)의 파일로 바꾸면 됩니다. 비모수 스크립트([`client/no_parametric_cbr.py`](https://github.com/Memento-Teams/Memento/blob/main/client/no_parametric_cbr.py))는 임베딩 모델을 불러오자마자 `.to('cuda')`로 옮기므로, NVIDIA GPU가 없는 환경에서는 그 줄을 고쳐야 실행됩니다.

실행기를 [vLLM](https://github.com/vllm-project/vllm)으로 띄운 로컬 모델로 바꾸는 [`client/agent_local_server.py`](https://github.com/Memento-Teams/Memento/blob/main/client/agent_local_server.py)도 있으며, 기본 실행기 모델은 `qwen3-8b`이고 서버 주소는 `DIRECT_BASE_URL`로 지정합니다. 다만 README의 한계(Limitations) 항목은 완전히 오픈 모델로만 구성한 파이프라인에서 실행기 검증이 제한적이라고 밝히고 있습니다.

## Memento의 라이선스

Memento는 [MIT 라이선스](https://github.com/Memento-Teams/Memento/blob/main/LICENSE)로 공개되어 있어 개인 및 상업적 목적으로 자유롭게 사용할 수 있습니다.

## :scroll: Memento 논문

![](https://discuss.pytorch.kr/uploads/default/original/3X/7/7/7737f9c766957e34da6871902e1e7a9d2aca40f3.png)
[arXiv.org](https://arxiv.org/abs/2508.16153)

![](https://discuss.pytorch.kr/uploads/default/original/3X/c/d/cd49b65780faf86c14ed9761c9c522acfb73adde.png)

### [Memento: Fine-tuning LLM Agents without Fine-tuning LLMs](https://arxiv.org/abs/2508.16153)

In this paper, we introduce a novel learning paradigm for Adaptive Large Language Model (LLM) agents that eliminates the need for fine-tuning the underlying LLMs. Existing approaches are often either rigid, relying on static, handcrafted reflection...

## :github: Memento 프로젝트 GitHub 저장소

[github.com](https://github.com/Memento-Teams/Memento)

![](https://discuss.pytorch.kr/uploads/default/optimized/3X/e/0/e079096fd3502d28c15e701a3d32658b14f0f3d7_2_695x347.png)

### [GitHub - Memento-Teams/Memento: Official Code of Memento: Fine-tuning LLM Agents...](https://github.com/Memento-Teams/Memento)

Official Code of Memento: Fine-tuning LLM Agents without Fine-tuning LLMs

## 더 읽어보기

* [ReasoningBank: 성공과 실패 경험으로부터 스스로 진화하는 에이전트 추론 메모리 프레임워크 (feat. Google)](https://discuss.pytorch.kr/t/reasoningbank-feat-google/9854)
* [Mem0: AI 애플리케이션을 위한 맞춤형 메모리 레이어 (멤-제로)](https://discuss.pytorch.kr/t/mem0-ai/5179)
* [OWL, 자동회된 작업 수행을 위한 개방형 다중 에이전트 협력 프레임워크 (feat. CAMEL)](https://discuss.pytorch.kr/t/owl-feat-camel/6549)
* [DeerFlow: 웹 브라우징, 검색 엔진, MCP 등을 통합한 멀티 에이전트 리서치 어시스턴트 (feat. ByteDance)](https://discuss.pytorch.kr/t/deerflow-mcp-feat-bytedance/6956)
* [Crawl4AI: LLM 및 AI 애플리케이션을 위한 오픈소스 웹 크롤러(Crawler)](https://discuss.pytorch.kr/t/crawl4ai-llm-ai-crawler/5282)

## 관련

- [[Learning/학습노트/2026-11-02-Memento-CBR-에이전트-메모리와-실무-적용-노트|한국어 정리]]
- [[Learning/원문/index|원문 인덱스]]
- [[Learning/index|Learning 전체 목록]]