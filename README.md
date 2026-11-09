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

## 개발 환경 및 실행 방법

- Python 3.11과 `uv`, VS Code의 Python 및 Jupyter 확장을 사용합니다.
- 저장소 루트에서 `uv sync --locked`로 `uv.lock` 기준 가상환경을 설치합니다.
- VS Code에서 `mini-project-01/code/` 아래의 노트북을 열고 저장소의 `.venv` Python 커널을 선택합니다. 노트북의 작업 폴더는 `mini-project-01/code/`로 설정합니다. 데이터 경로와 결과 저장 경로는 이 폴더를 기준으로 합니다.
- 각 노트북의 셀을 위에서부터 순서대로 실행합니다. 재실행 시 `mini-project-01/results/`의 모델과 결과 파일이 다시 생성되거나 덮어써질 수 있습니다.

## 프로젝트 자료

- [회귀 노트북](mini-project-01/code/01_regression_bike_demand.ipynb)과 [분류 노트북](mini-project-01/code/02_classification_online_shopping.ipynb)
- [회귀 데이터 분석](mini-project-01/docs/01_regression_data_analysis.md)과 [회귀 모델 분석](mini-project-01/docs/01_regression_model_analysis.md)
- [분류 데이터 분석](mini-project-01/docs/02_classification_data_analysis.md)과 [분류 모델 분석](mini-project-01/docs/02_classification_model_analysis.md)
- [의사결정 기록](mini-project-01/docs/adr.md)
- [모델, 설정 및 실험 결과](mini-project-01/results/)
- [미니프로젝트 명세서](mini-project-01/미니프로젝트1_명세서.docx)

원본 데이터는 `mini-project-01/data/`에 있습니다. 모델과 CSV는 재현 및 결과 확인을 위해 저장소에 포함했습니다. 추적 중인 결과 파일은 일반적인 `git add`로 변경 사항을 추가할 수 있으며, 새로 생성되는 다른 결과 파일은 기본적으로 무시됩니다.
