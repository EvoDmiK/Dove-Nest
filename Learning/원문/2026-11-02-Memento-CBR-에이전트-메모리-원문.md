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

