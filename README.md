# AI Mini Projects

멀티캠퍼스 AI 엔지니어 과정에서 진행한 모델 실험과 분석, 의사결정을 기록한 저장소입니다.

## Mini Project 01

회귀와 분류 문제를 각각 해결하고, 검증 데이터에서 모델을 비교한 뒤 오류를 분석했습니다. 두 노트북은 완료 상태입니다.

| 과제 | 문제 | 최종 선택 | 주요 결과 |
| --- | --- | --- | --- |
| 1일차 | 시간당 공유자전거 대여량 예측 | 랜덤 포레스트, 트리 200개, 깊이 제한 없음 | Validation MAE 86.194대, 최종 Test MAE 54.118대 |
| 2일차 | 온라인 쇼핑 세션의 구매 여부 예측 | 가중 손실 MLP, 임계값 0.5, 16에포크 | Validation F1 0.657, 최종 Test F1 0.658 |

회귀 모델은 출퇴근 특성을 제외한 설정을 Validation에서 선택했습니다. 분류 모델은 구매 클래스 가중치를 기본 클래스 비율의 절반으로 적용했습니다. 최종 Test 평가는 두 과제 모두 Train과 Validation을 합쳐 재학습한 모델의 결과입니다.

**평가 이력과 한계:** 두 과제의 Test 데이터는 과거에도 모델 결과를 확인하는 데 사용했습니다. 따라서 위 Test 점수를 독립적인 새 데이터의 성능으로 볼 수 없습니다. 회귀는 검증 기간과 시간대에 따라 결과가 달랐고, 분류는 중복값과 `PageValues`의 예측 시점 가용성을 추가로 확인해야 합니다. 세부 결과와 한계는 분석 문서와 ADR에 기록했습니다.

### Mini Project 01 실행 방법

- Python 3.11과 `uv`, VS Code의 Python 및 Jupyter 확장을 사용합니다.
- 저장소 루트에서 `uv sync --locked`로 `uv.lock` 기준 가상환경을 설치합니다.
- VS Code에서 `mini-project-01/code/` 아래의 노트북을 열고 저장소의 `.venv` Python 커널을 선택합니다. 노트북의 작업 폴더는 `mini-project-01/code/`로 설정합니다. 데이터 경로와 결과 저장 경로는 이 폴더를 기준으로 합니다.
- 각 노트북의 셀을 위에서부터 순서대로 실행합니다. 재실행 시 `mini-project-01/results/`의 모델과 결과 파일이 다시 생성되거나 덮어써질 수 있습니다.

### Mini Project 01 자료

- [회귀 노트북](mini-project-01/code/01_regression_bike_demand.ipynb)과 [분류 노트북](mini-project-01/code/02_classification_online_shopping.ipynb)
- [회귀 데이터 분석](mini-project-01/docs/01_regression_data_analysis.md)과 [회귀 모델 분석](mini-project-01/docs/01_regression_model_analysis.md)
- [분류 데이터 분석](mini-project-01/docs/02_classification_data_analysis.md)과 [분류 모델 분석](mini-project-01/docs/02_classification_model_analysis.md)
- [의사결정 기록](mini-project-01/docs/adr.md)
- [모델, 설정 및 실험 결과](mini-project-01/results/)
- [미니프로젝트 명세서](mini-project-01/미니프로젝트1_명세서.docx)

원본 데이터는 `mini-project-01/data/`에 있습니다. 모델과 CSV는 재현 및 결과 확인을 위해 저장소에 포함했습니다. 추적 중인 결과 파일은 일반적인 `git add`로 변경 사항을 추가할 수 있으며, 새로 생성되는 다른 결과 파일은 기본적으로 무시됩니다.

## Mini Project 02 · 한국어 문단 검색기

한국어 질문을 입력하면 관련된 문단을 찾아주는 검색기이다. 같은 질문과 문단을 대상으로 TF-IDF, BERT, Sentence-BERT를 비교하고 사용 목적에 맞는 방법을 선택했다.

답변을 새로 생성하거나 모델을 재학습하는 프로젝트는 아니다. 질문과 문단의 유사도를 계산해 관련성이 높은 문단을 순서대로 보여준다.

### 데이터

KLUE-MRC의 Validation 데이터로 검색용 데이터셋을 구성했다.

| 구분 | 개수 | 용도 |
| --- | ---: | --- |
| Corpus | 200개 문단 | 검색 대상 |
| Validation | 60개 질문 | 검색기 비교와 선택 |
| Test | 20개 질문 | 선택한 검색기의 최종 평가 |

Validation과 Test는 서로 다른 질문이지만 같은 문단 200개를 검색한다. 따라서 Test 결과는 새로운 질문에 대한 평가이며, 새로운 문서에 대한 성능을 보여주지는 않는다.

### 검색 방식

- **TF-IDF:** 문단과 질문을 문자 2~3개 묶음으로 표현하고 코사인 유사도를 계산한다.
- **BERT:** `klue/bert-base`의 토큰 벡터를 패딩을 제외하고 평균 내 문장 벡터로 만든다.
- **Sentence-BERT:** `snunlp/KR-SBERT-V40K-klueNLI-augSTS`로 문단과 질문의 벡터를 만든다.

각 방식은 질문과 문단의 벡터를 비교해 유사도가 높은 문단부터 반환한다. BERT와 Sentence-BERT는 최대 입력 길이를 256토큰으로 맞췄다.

### 진행 과정

1. **CP-01:** 문단과 질문의 길이, 질문과 정답 문단의 문자 겹침을 살펴보고 가설을 세운다.
2. **CP-02:** TF-IDF 검색기를 만든다.
3. **CP-03~05:** BERT의 입력과 출력을 확인하고, 토큰 벡터를 평균 내 문단 벡터를 만든다.
4. **CP-06:** BERT 검색기를 만들고 질문 벡터를 캐시에 저장한다.
5. **CP-07~08:** Validation 질문으로 세 검색기의 성능을 비교한다.
6. **CP-09:** 실패 사례를 살펴보고 사용 목적과 평가 지표에 따라 검색 방식을 선택한다.
7. **CP-10:** 선택한 검색기를 새로운 질문과 보류 Test 질문으로 평가한다.

단계별 그림과 설명은 [NLP 검색 흐름](mini-project-02/docs/02_nlp_search_flow.md)에 정리했다.

### 실험 결과

Validation 질문 60개로 비교한 결과이다.

| 검색 방식 | Hit@1 | Recall@3 | MRR@5 |
| --- | ---: | ---: | ---: |
| TF-IDF | 0.8833 | 0.9667 | 0.9167 |
| BERT | 0.6667 | 0.8333 | 0.7589 |
| Sentence-BERT | 0.7667 | 0.8833 | 0.8208 |

- **Hit@1:** 정답 문단이 1위인 질문의 비율이다.
- **Recall@3:** 정답 문단이 상위 3개에 포함된 질문의 비율이다.
- **MRR@5:** 상위 5개에서 정답 문단 순위의 역수를 평균 낸 값이다. 정답이 없으면 0으로 계산한다.

첫 번째 결과에서 관련 문단을 바로 확인하는 상황을 사용 목적으로 정하고 Hit@1을 우선 지표로 선택했다. TF-IDF는 Validation 질문 60개 중 53개에서 정답을 1위로 찾아 최종 검색 방식으로 선택했다.

선택 후 Test 질문 20개로 평가한 결과, TF-IDF의 Hit@1은 0.8000, Recall@3은 0.8000, MRR@5는 0.8125였다. 20개 중 16개에서 정답을 1위로 찾았다. 이 결과는 정해진 문단과 각 질문에 지정된 정답 1개를 기준으로 평가한 값이다.

### 실행 방법

1. VS Code에서 [NLP 노트북](mini-project-02/code/NLP_BERT_MiniProject_Student%20%281%29.ipynb)을 열고 Python 가상환경을 Jupyter 커널로 선택한다.
2. 첫 셀부터 순서대로 실행한다. 필요한 패키지와 데이터, 모델 가중치가 없으면 준비 과정이 실행된다.
3. BERT와 Sentence-BERT의 벡터 계산에는 시간이 걸릴 수 있다. GPU를 사용할 수 있으면 연산을 빠르게 진행할 수 있으며, 저장된 벡터는 조건에 따라 재사용한다.

### 프로젝트 파일

- [NLP 노트북](mini-project-02/code/NLP_BERT_MiniProject_Student%20%281%29.ipynb): 검색기 구현과 실행 결과
- [데이터 분석 및 초기 가설](mini-project-02/docs/02_nlp_data_analysis.md): CP-01 관찰 결과
- [검색 성능 실험 및 오류 분석](mini-project-02/docs/02_nlp_model_analysis.md): 실험 기록 문서. CP-07 이후 항목은 작성 중이다.
- [NLP 검색 흐름](mini-project-02/docs/02_nlp_search_flow.md): CP-01~10 과정과 결과
- [의사결정 기록](mini-project-02/docs/ADR.md)

이 폴더에는 별도로 진행한 CV 프로젝트의 노트북과 분석 문서도 있다.
