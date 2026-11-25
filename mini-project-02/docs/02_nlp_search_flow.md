# NLP 검색 흐름 (CP-01~10)

질문을 입력하면 검색 대상 문단 200개에서 관련성이 높은 문단을 찾는다.

```mermaid
flowchart TD
    D["CP-01 · 검색 데이터 이해"]
    T["CP-02 · TF-IDF 검색기"]
    B["CP-03 · BERT 토큰 벡터"]
    P["CP-04 · Masked Mean Pooling"]
    E["CP-05 · 문단 벡터 저장과 유사도 계산"]
    S["CP-06 · BERT 검색기"]
    V["CP-07 · Validation 검색 성능 평가"]
    C["CP-08 · Sentence-BERT 추가 및 세 방식 비교"]
    A["CP-09 · 실패 사례 분석 및 TF-IDF 선택"]
    F["CP-10 · 최종 검색기와 Test 평가"]
    D --> T
    D --> B --> P --> E --> S
    T --> V
    S --> V
    V --> C --> A --> F
```

## CP-01. 검색 데이터와 초기 가설

```mermaid
flowchart TD
    K["KLUE-MRC에서 검색 데이터 구성"]
    K --> C["Corpus · 검색 대상 문단 200개"]
    K --> V["Validation · 질문 60개"]
    K --> T["Test · 질문 20개"]
    C --> L["문단과 질문의 길이 분석"]
    V --> L
    L --> O["질문과 정답 문단의 문자 겹침 분석"]
    O --> H["TF-IDF와 BERT의 검색 성능 가설 설정"]
    T --> F["CP-10에서 최종 평가"]
```

문자 겹침이 높으면 TF-IDF가, 표현은 달라도 의미가 비슷하면 BERT가 유리할 것으로 예상했다.

## CP-02. TF-IDF 검색기

```mermaid
flowchart TD
    D["문단 200개"] --> N["문자 2-gram과 3-gram 추출"]
    N --> V["사용할 문자 묶음 최대 30,000개"]
    V --> W["TF × IDF로 문자 묶음의 중요도 계산"]
    W --> E["문단별 TF-IDF 벡터"]
    Q["질문 입력"] --> QE["문단과 같은 기준으로 질문 벡터 생성"]
    W -.-> QE
    E --> C["질문과 문단의 코사인 유사도 200개"]
    QE --> C
    C --> S["내림차순 정렬"] --> K["Top-K 문단 반환"]
```

## CP-03. BERT 입력과 토큰 벡터

```mermaid
flowchart TD
    Q["질문 2개"] --> T["토큰화와 Token ID 변환"]
    T --> B["Batch 구성 · 유효 토큰 15개와 19개"]
    B --> P["첫 번째 질문에 Padding 4개 추가"]
    P --> M["Attention Mask · 유효 토큰 1, Padding 0"]
    M --> E["BERT Encoder · Self-Attention과 FFN"]
    E --> V["토큰별 문맥 벡터 · (2, 19, 768)"]
```

## CP-04. Masked Mean Pooling

```mermaid
flowchart TD
    V["토큰 벡터 · (2, 19, 768)"] --> M["Attention Mask를 곱해 Padding 위치를 0으로 변환"]
    M --> S["토큰 벡터를 차원별로 더함"]
    S --> A["유효 토큰 수로 나눔"]
    A --> E["질문마다 768차원 문장 벡터 · (2, 768)"]
```

## CP-05. 문단 벡터 만들고 저장하기

```mermaid
flowchart TD
    D["문단 200개"] --> B["16개씩 나눔 · 마지막 Batch는 8개"]
    B --> T["토큰화, Padding, Truncation · 최대 256토큰"]
    T --> E["BERT Encoder"] --> P["Masked Mean Pooling"]
    P --> V["Batch 결과 합치기 · (200, 768)"]
    V --> F["문단 벡터를 파일에 저장"]
    F --> C["검색 시 질문 벡터와 코사인 유사도 계산"]
```

## CP-06. BERT 검색과 질문 벡터 캐시

```mermaid
flowchart TD
    Q["질문 입력"] --> C{"캐시에 질문 벡터가 있는가?"}
    C -->|YES · Cache Hit| H["기존 질문 벡터 불러오기<br/>BERT 연산 생략"]
    C -->|NO · Cache Miss| M["토큰화 → BERT → Masked Mean Pooling"]
    M --> W["질문 벡터 캐시에 저장"]
    H --> S["질문 벡터와 문단 벡터의 코사인 유사도 계산"]
    W --> S
    F["CP-05에서 저장한 문단 벡터 200개"] --> S
    S --> R["점수 200개를 내림차순으로 정렬"]
    R --> K["Top-K 문단 반환"]
```

첫 Validation 질문 `QV001`에서 두 검색기 모두 정답 문단 `D015`를 1위로 찾았다. 두 검색기의 유사도는 각각 순위를 매기는 데 쓰고, 전체 성능은 Validation 질문 60개에서 정답 문단이 나온 순위로 평가한다.

## CP-07. TF-IDF와 BERT의 검색 성능 평가

```mermaid
flowchart TD
    V["Validation 질문 60개"] --> T["TF-IDF로 문단 상위 5개 검색"]
    V --> C["BERT 질문 벡터를 파일에서 불러오거나 계산"]
    C --> B["질문 벡터를 캐시에 넣고 BERT로 문단 상위 5개 검색"]
    T --> R["상위 5개에서 정답 문단 ID와 순위 확인"]
    B --> R
    R --> M["질문별 Hit@1, Recall@3, RR@5 계산"]
    M --> S["summary · 검색기별 평균 점수"]
    M --> D["detail · 질문별 순위와 점수"]
```

Hit@1은 정답이 1위에 나온 비율이고, Recall@3은 정답이 상위 3개 안에 들어온 비율이다. MRR@5는 정답 순위의 역수를 평균 낸 값이며, 상위 5개에 정답이 없으면 해당 질문의 점수는 0이다. TF-IDF는 60개 중 53개를 1위로 찾았고, BERT는 40개를 1위로 찾았다.

## CP-08. Sentence-BERT 추가 및 세 검색기 비교

```mermaid
flowchart TD
    M["Sentence-BERT 로드 · 최대 256토큰"] --> C{"저장된 벡터가 있는가?"}
    C -->|있음| L["문단 또는 질문 벡터 불러오기"]
    C -->|없음| E["해당 벡터를 계산하고 정규화한 뒤 저장"]
    L --> S["문단 벡터 200개와 Validation 질문 벡터 60개 준비"]
    E --> S
    S --> R["질문과 문단의 코사인 유사도 계산 및 Top-K 반환"]
    R --> V["같은 Validation 질문 60개로 Sentence-BERT 평가"]
    P["CP-07의 TF-IDF와 BERT 평가 결과"] --> F["세 검색기의 비교표와 그래프 만들기"]
    V --> F
```

| 검색 방식 | Hit@1 | Recall@3 | MRR@5 |
| --- | ---: | ---: | ---: |
| TF-IDF | 0.8833 | 0.9667 | 0.9167 |
| BERT | 0.6667 | 0.8333 | 0.7589 |
| Sentence-BERT | 0.7667 | 0.8833 | 0.8208 |

같은 질문 60개에서 세 지표 모두 TF-IDF, Sentence-BERT, BERT 순이었다. 어떤 질문에서 이 차이가 났는지는 CP-09에서 순위와 문자 겹침 비율을 같이 봤다.

## CP-09. 실패 사례 분석 및 검색 방식 선택

```mermaid
flowchart TD
    D["세 검색기의 질문별 순위와 점수"] --> J["질문 ID로 합치고 문자 겹침 비율 붙이기"]
    J --> F["세 검색기 중 정답을 1위로 찾지 못한 개수 계산"]
    F --> E["실패가 많은 질문 5개 선택"]
    E --> A["질문별 순위와 문자 겹침을 보고 CP-01 가설과 비교"]
    U["사용 목적 · 첫 번째 결과에서 관련 문단 확인"] --> H["Hit@1을 우선 지표로 선택"]
    A --> C["전체 점수와 실패 사례를 보고 TF-IDF 선택"]
    H --> C
    C --> S["선택 근거를 cp09_selection.json에 저장"]
```

대표 사례 5개의 평균 문자 겹침 비율은 35.7%로, 전체 평균 47.4%보다 11.7%p 낮았다. `QV018`은 겹침 비율이 15.8%였고, 정답 순위는 TF-IDF 3위, BERT 2위, Sentence-BERT 1위였다. `QV020`에서는 세 검색기 모두 정답을 상위 5개에서 놓쳤다.

첫 번째 검색 결과에서 관련 문단을 바로 확인하는 상황으로 정했다. 그래서 `Hit@1`을 우선으로 봤고, Validation에서 53개를 1위로 찾은 TF-IDF를 선택했다.

## CP-10. 최종 검색기와 Test 평가

```mermaid
flowchart TD
    Q["질문 입력"] --> S["search_docs 함수 · 기본값은 TF-IDF"]
    S --> K["유사도 순으로 Top-K 문단 반환"]
    K --> N["새 질문의 상위 3개 결과 확인"]
    T["보류 Test 질문 20개"] --> E["CP-09에서 확정한 TF-IDF로 평가"]
    C["같은 검색기의 Validation 점수"] --> V["Validation과 Test의 점수 및 순위 비교"]
    E --> V
    V --> F["비교 점수와 질문별 결과를 CSV·JSON으로 저장"]
```

| 지표 | Validation | Test | 감소량 |
| --- | ---: | ---: | ---: |
| Hit@1 | 0.8833 | 0.8000 | 0.0833 |
| Recall@3 | 0.9667 | 0.8000 | 0.1667 |
| MRR@5 | 0.9167 | 0.8125 | 0.1042 |

Test 질문 20개 중 16개에서 정답을 1위로 찾았다. 나머지 4개 중 하나는 정답이 4위였고, 세 개는 상위 5개에 정답이 없었다. 검색 대상은 같은 문단 200개였고, CP-10에서는 새 질문으로 확정된 검색 방식을 평가했다.
