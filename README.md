# AI Mini Projects

멀티캠퍼스 AI 엔지니어 과정에서 진행한 모델 실험과 분석을 기록한다.

## Mini Project 01 · 회귀와 분류

- **공유자전거 대여량 예측:** 랜덤 포레스트 선택, Test MAE 54.118대.
- **온라인 쇼핑 구매 여부 예측:** 가중 손실 MLP 선택, Test F1 0.658.

두 Test 데이터는 과거에도 모델 결과 확인에 사용했으므로 독립적인 새 데이터의 성능으로 볼 수 없다.

[회귀 노트북](mini-project-01/code/01_regression_bike_demand.ipynb) · [분류 노트북](mini-project-01/code/02_classification_online_shopping.ipynb) · [분석 문서와 의사결정](mini-project-01/docs/)

## Mini Project 02 · 한국어 문단 검색

질문과 관련된 문단을 찾기 위해 TF-IDF, BERT, Sentence-BERT를 비교했다. 검색 대상은 문단 200개이며 Validation 질문 60개로 비교하고 Test 질문 20개로 최종 평가했다.

| 검색 방식 | Validation Hit@1 | Recall@3 | MRR@5 |
| --- | ---: | ---: | ---: |
| TF-IDF | 0.8833 | 0.9667 | 0.9167 |
| BERT | 0.6667 | 0.8333 | 0.7589 |
| Sentence-BERT | 0.7667 | 0.8833 | 0.8208 |

첫 번째 결과의 정확도를 우선해 TF-IDF를 선택했다. Test Hit@1은 0.8000으로 질문 20개 중 16개를 1위로 찾았다. Validation과 Test는 같은 문단을 사용하므로 새로운 문서에 대한 평가는 아니다.

[NLP 노트북](mini-project-02/code/NLP_BERT_MiniProject_Student%20%281%29.ipynb) · [검색 흐름과 상세 결과](mini-project-02/docs/02_nlp_search_flow.md) · [데이터 분석](mini-project-02/docs/02_nlp_data_analysis.md) · [의사결정](mini-project-02/docs/ADR.md)

## 실행

VS Code에서 노트북을 열고 Python 가상환경을 Jupyter 커널로 선택한 뒤 셀을 순서대로 실행한다. 프로젝트 01은 저장소 루트에서 `uv sync --locked`로 환경을 설치한다. 프로젝트 02는 노트북의 첫 셀에서 필요한 패키지를 확인한다.
