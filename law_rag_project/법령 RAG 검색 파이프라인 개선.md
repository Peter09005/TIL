

## 1. 프로젝트 목적

이 프로젝트의 목적은 특정 법령 문서를 구조화하고, 사용자의 자연어 질문이 들어왔을 때 관련 조문을 정확하게 검색하여 RAG 기반 QA에 활용하는 것이다.

현재 주요 대상은 식품위생법과 같은 한국 법령이며, 사용자는 다음과 같은 질문을 던질 수 있다.

```text
위해식품의 의미가 뭐야?
식품 관련 영업을 하려면 허가나 신고를 받아야 한다는 조문은?
HACCP 같은 식품안전관리인증기준은 몇 조에서 다뤄?
기숙사 급식시설처럼 집단으로 식사를 제공하는 시설에 대한 규정은 어디 조문이야?
제3조제2항을 어기면 어떻게 돼?
```

단순 키워드 검색이 아니라, 조문 제목, 본문, 항/호/목 구조, 참조 관계, 의미적 유사성까지 고려하여 최종적으로 관련 조문 top-k를 안정적으로 반환하는 것이 목표다.

---

## 2. 전체 파이프라인 개요

현재까지 설계한 법령 처리 파이프라인은 다음과 같다.

```text
┌──────────────────────────────┐
│        법령 데이터 입력        │
│  (HTML / marker JSON)        │
└─────────────┬────────────────┘
              ↓
┌──────────────────────────────┐
│   01_raw_markers.json        │
└─────────────┬────────────────┘
              ↓
┌──────────────────────────────┐
│ 02_structure_relations.json  │
└─────────────┬────────────────┘
              ↓
┌──────────────────────────────┐
│ 03_versioned_articles.json   │
└─────────────┬────────────────┘
              ↓
┌──────────────────────────────┐
│ 04_original_chunks.jsonl     │
└───────┬───────────┬──────────┘
        ↓           ↓
┌──────────────┐   ┌──────────────┐
│ Dense Input  │   │ Sparse Input │
└──────┬───────┘   └──────┬───────┘
       ↓                  ↓
┌──────────────┐   ┌──────────────┐
│   Milvus     │   │   Milvus     │
│ Dense Vector │   │ Sparse Vector│
└──────────────┘   └──────┬───────┘
                          ↓
                   ┌──────────────┐
                   │ ElasticSearch│
                   │ (BM25 + Nori)│
                   └──────────────┘


====================================================
                  🔍 검색 파이프라인
====================================================

┌──────────────────────────────┐
│        사용자 질문 입력        │
└─────────────┬────────────────┘
              ↓
┌──────────────────────────────┐
│ Query Embedding (Dense)      │
└─────────────┬────────────────┘
              ↓
┌──────────────────────────────────────────────┐
│ Dense Search (20)                            │
│ Sparse Search (40)                           │
│ BM25 Search (40)                             │
└─────────────┬───────────────────────────────┘
              ↓
┌──────────────────────────────┐
│ article_uid 기준 중복 제거     │
└─────────────┬────────────────┘
              ↓
┌──────────────────────────────┐
│ retriever별 score 정규화 (0~1)│
└─────────────┬────────────────┘
              ↓
┌──────────────────────────────┐
│ sparse-BM25 top5 overlap 분석 │
│ → fusion weight 동적 조정     │
└─────────────┬────────────────┘
              ↓
┌──────────────────────────────┐
│ Hybrid Fusion                │
│ - dense 중심                 │
│ - sparse/bm25 보조           │
│ - multi-hit bonus 적용       │
└─────────────┬────────────────┘
              ↓
┌──────────────────────────────┐
│ Hybrid Top 15 생성           │
└─────────────┬────────────────┘
              ↓
┌──────────────────────────────┐
│ (Query + Passage) Pair 생성   │
└─────────────┬────────────────┘
              ↓
┌──────────────────────────────┐
│ Cross Encoder Reranking      │
│ (bge-reranker-v2-m3)         │
└─────────────┬────────────────┘
              ↓
┌──────────────────────────────┐
│   union with hybrid result   │
└─────────────┬────────────────┘
			  ↓
┌──────────────────────────────┐
│        🔥 Final Top 3        │
└──────────────────────────────┘
```

---

## 3. 데이터 구조화 단계

### 3.1 01_raw_markers.json

법령 원문에서 장/절/조/항/호/목 등 구조 마커를 추출하는 단계다.

주요 마커 예시는 다음과 같다.

```text
제1장
제4조
①
1.
가.
```

각 marker는 대략 다음과 같은 정보를 가진다.

```json
{
  "node_id": "LAW001_jo_4_ho_1",
  "marker_text": "1.",
  "pattern_name": "ho",
  "kind": "ho",
  "main_no": 1,
  "sub_no": null,
  "display_no": "1.",
  "parent_id": "LAW001_jo_4",
  "order_index": 123,
  "title": null,
  "body_text": "썩거나 상하거나 설익어서 인체의 건강을 해칠 우려가 있는 것",
  "raw_text": "1. 썩거나 상하거나 설익어서 인체의 건강을 해칠 우려가 있는 것"
}
```

### 3.2 02_structure_relations.json

01 단계에서 추출된 marker들을 부모-자식 관계로 연결한다.

예:

```text
제4조
 ├─ 제4조제1호
 ├─ 제4조제2호
 ├─ 제4조제3호
 └─ ...
```

향후에는 이 단계에서 조문 간 참조 관계도 일부 저장할 수 있다.

예:

```json
{
  "source_uid": "law::main::22",
  "target_uid": "law::main::21::p1",
  "relation_type": "references",
  "anchor_text": "제21조제1항에 따른 검사"
}
```

### <mark style="background: #FF5582A6;">3.3 03_versioned_articles.json</mark>  

조문 단위로 versioned article을 구성하는 단계다.

주요 필드 예시는 다음과 같다.

```json
{
  "article_uid": "law::main::4",
  "source_anchor_node_id": "jo_4",
  "article_no": {
    "display_no": "제4조",
    "main_no": 4,
    "sub_no": null
  },
  "article_title": "위해식품등의 판매 등 금지",
  "effective_from": "YYYY-MM-DD | null",
  "effective_to": "YYYY-MM-DD | null",
  "deleted_flag": false,
  "text": {
    "lead_text": "누구든지 다음 각 호의 어느 하나에 해당하는 식품등을 판매하거나...",
    "full_text": "...",
    "clean_text": "...",
    "display_text": "..."
  },
  "content": {
    "annotations": [],
    "paragraphs": [],
    "items": [],
    "subitems": []
  }
}
```

기존에는 `effective_date`만 있었지만, 이후 버전 관리를 위해 `effective_from`, `effective_to`로 나누는 방향을 고려했다.

### <mark style="background: #FF5582A6;">3.4 04_original_chunks.jsonl</mark>

검색과 임베딩을 위해 조문을 chunk 단위로 나누는 단계다.

chunk 예시는 다음과 같다.

```json
{
  "chunk_id": "chunk::law::main::4::item::1",
  "article_uid": "law::main::4",
  "chunk_type": "item",
  "article_no": "제4조",
  "title": "위해식품등의 판매 등 금지",
  "text": "1. 썩거나 상하거나 설익어서 인체의 건강을 해칠 우려가 있는 것",
  "parent_chunk_id": "chunk::law::main::4::article"
}
```


### 3.5 03_dense_chunks.jsonl - dense collection 

**dense** 는 핵심 축이다. 

질의가 날아왔을때 이 질의는 어느 조랑 관련 있겠구나를 1차적으로 걸러줘야되기에 
조 / 항 chunk 를 넣기로 했다. 

호/ 목 까지 chunk 해서 넣어버리게 된다면, 벡터값이 문맥을 잘 파악하지못하고 엉뚱한 
조를 가르킬수도있기 때문이다.

### 3.6 03_sparse_chunks.jsonl - sparse collection 

**sparse** 는 bm25와 유사한 lexial matching을 지원한다. sparse는 문맥 + bm25 느낌이라고 생각하면 되는데, 쉽게 말해서 키워드 기반 검색이되, 내부적으로 키워드에 문맥 , 위치 등으로 값을 조정한거라고 생각하면 편하다. 

예를 들면 기존 bm25 는 통계를 활용한 매칭을 사용해서 "목적" , "의도" 가 비슷한 내용이라고 잘 구별하지 못하지만

**sparse** 는 1차적으로 키워드를 한번 손본 벡터이기 때문에 **목적** , **의도** 비슷한 벡터를 가지게 된다. 

**sparse** 는 **dense** 를 보조할 축으로 설계했기때문에, **문맥** 에 집중하기 보다는 조안에 세부적인 데이터에 집중하는게 좋을거같아 조 chunk 를 제외한 **항, 호 , 목** 을 embedding 값으로 넣었다. 

### 3.7 03_sparse_chunks.jsonl - bm25 elastic search

sparse collection , dense collection 만 활용하면 문제가 있었다 

```text 
"기숙사는 집단급식소야?"
```

해당 법령에는 **기숙사** 라는 키워드는 한 조문에서 밖에 나와있지 않고, 
**집단급식소** 라는 키워드는 수십개의 조문에서 **집중적** 으로 다룬다. 

**dense**, **sparse** 모두 문맥을 고려하다보니, 문서 여러곳에서 수십번 나온 **집단급식소** 를 
**기숙사** 보다 훨씬 더 중요한 토큰으로 생각하고, 해당 질의의 벡터를 **집단 급식소** 쪽으로 
너무 강하게 당기는 문제가 있었음. **sparse** 도 TF-IDF 로 적은 단어에 큰 가중치를 두지만, 
내부적으로 **기숙사**는 한번밖에 안나오니, **노이즈**로 처리한거같음 

이에 적은 키워드에 확실하게 높은 점수를 줄수있는 

**elastic search** 에 bm25로 sparse_chunk.jsol 을 학습시켰음. 



---

## 4. 기존 검색 구조

초기 검색 구조는 dense, sparse, bm25를 각각 사용하고 결과를 fusion하는 방식이었다.

```text
query
 ↓
dense search
sparse search
bm25 search
 ↓
score fusion
 ↓
hybrid candidate top15
```

현재 실험에서 가장 효율이 좋았던 기본 가중치는 다음과 같았다.

```text
dense 0.75
sparse 0.15
bm25 0.10
```

이 결과는 현재 질의셋이 대체로 자연어/해석형 질문이 많고, exact keyword보다는 의미적 유사성이 더 중요하다는 것을 보여준다.

---

## 5. Dense / Sparse / BM25 역할 정리

### 5.1 Dense retrieval

Dense retrieval은 query와 passage를 각각 vector로 변환한 뒤 의미적 유사도를 계산한다.

장점:

- 자연어 질문에 강함
    
- 표현이 달라도 의미가 비슷하면 검색 가능
    
- 해석형 질문에 유리함
    

단점:

- 정확한 법률 용어, 조문번호, 특정 키워드 매칭에는 약할 수 있음
    
- 비슷한 의미의 다른 조문이 함께 올라올 수 있음
    

현재 기본 비중은 가장 높게 설정했다.

```text
dense weight = 0.75
```

### 5.2 Sparse retrieval

Sparse retrieval은 BGE-M3의 sparse vector처럼 transformer 기반 sparse 표현을 활용한다.

BM25와 달리 단순 키워드 빈도만 보는 것이 아니라, 어느 정도 문맥 기반 sparse representation을 활용한다.

장점:

- BM25보다 문맥 반영 가능
    
- 법률 용어와 의미 단서를 함께 활용 가능
    
- dense와 bm25 사이의 중간 역할
    

단점:

- 특정 키워드가 강하면 그쪽으로 쏠릴 수 있음
    
- full_text에 반복 키워드가 많으면 일부 조문으로 편향될 수 있음
    

현재 기본 비중:

```text
sparse weight = 0.15
```

### 5.3 BM25

BM25는 전통적인 lexical retrieval 방식이다.

장점:

- 정확한 키워드 매칭에 강함
    
- 조문번호, 법률 고정 표현, 제목 검색에 유리함
    
- 빠르고 구현이 쉬움
    

단점:

- 의미적 paraphrase에 약함
    
- 특정 키워드가 반복되는 조문으로 쏠릴 수 있음
    
- 자연어 해석형 질문에는 한계가 있음
    

현재 기본 비중:

```text
bm25 weight = 0.10
```

---

## 6. 초기 문제점

### 6.1 정답은 top15 안에 있지만 top3로 올라오지 않음

예시 질문:

```text
위해식품의 의미가 뭐야?
```

관련 조문:

```text
제4조(위해식품등의 판매 등 금지)
```

제4조는 위해식품에 해당하는 유형을 각 호로 열거한다.

```text
1. 썩거나 상하거나 설익어서 인체의 건강을 해칠 우려가 있는 것
2. 유독ㆍ유해물질이 들어 있거나 묻어 있는 것
3. 병을 일으키는 미생물에 오염되었거나...
```

그러나 초기 hybrid search에서는 제4조가 rank 6 정도로 나오는 문제가 있었다.

이는 정답을 전혀 못 찾는 문제가 아니라, 정답 후보를 상위로 재정렬하지 못하는 문제였다.

```text
문제 유형: recall 문제 X, ranking 문제 O
```

### 6.2 같은 조문 chunk가 top15를 많이 차지함

현재 chunking 방식은 다음과 같다.

```text
dense: 조 / 항 단위
sparse: 조 / 항 / 호 / 목 단위
bm25: 별도 lexical 문서
```

이 때문에 검색 결과 top15 안에 같은 article_uid를 가진 chunk가 여러 개 등장할 수 있다.

예:

```text
제4조 전체
제4조 제1호
제4조 제2호
제4조 제3호
```

이 상태에서 chunk를 그대로 reranker에 넣으면 다음 문제가 생긴다.

1. 같은 조문이 final top3에 중복 등장할 수 있음
    
2. top15처럼 보이지만 실제 unique article 후보는 훨씬 적음
    
3. chunk 단위 경쟁 때문에 조문 단위 판단이 어려워짐
    

따라서 chunk-level 검색 결과를 article_uid 기준으로 grouping/dedup하는 단계가 필요했다.

---
### 7.1 `article_uid` 기준 dedupe  
  
각 retriever의 결과는 chunk 단위로 여러 개가 나올 수 있다.    
하지만 최종 fusion은 조문 단위 비교가 목적이므로 retriever 내부에서 먼저 `article_uid` 기준 dedupe를 수행한다.  


핵심 규칙:  
- 같은 retriever 결과 안에서 동일한 `article_uid`가 여러 번 나오면,  
- 가장 먼저 나온 결과, 즉 가장 rank가 높은 결과만 남긴다.  
  
예:  
- Dense 결과에서 같은 조문이 article chunk, paragraph chunk로 여러 번 잡혀도  
- 해당 retriever 내부에서는 가장 높은 순위 1개만 유지한다.  
  
이 단계의 목적:  
- chunk 중복으로 특정 조문이 과도하게 유리해지는 현상을 줄임  
- 이후 fusion 단계에서 비교 단위를 `article_uid`로 통일  
  
---
### 7.2 Retriever별 score normalize  
  
Dense, sparse, BM25는 점수 체계가 서로 다르므로 raw score를 그대로 합치지 않는다.  
  
  
정규화 방식:  
  
```python  
normalized_score = (score - min_score) / (max_score - min_score)  
```
---
### 7.3 동적 weight 조정  
  
`test2.py`는 고정 weight만 쓰지 않는다.    
sparse와 BM25의 상위 결과가 서로 얼마나 겹치는지 보고 dense 비중을 조금 줄이거나 유지한다.  
  

기본 weight:  
- `dense = 0.75`  
- `sparse = 0.15`  
- `bm25 = 0.10`  
  
비교 대상:  
- sparse top5 `article_uid`  
- bm25 top5 `article_uid`  
  
계산:  
- `overlap_count`  
- `overlap_ratio = overlap_count / top_k`  
- `jaccard_similarity`  
  
그 다음:  
  
```python  
dense_shift = overlap_ratio * (weight_shift_max / 100.0)  
shifted_dense_weight = dense_weight - dense_shift  
```  
  
기본값 기준:  
- `weight_shift_max = 10.0`  
- 따라서 sparse/BM25 top5가 많이 겹칠수록 dense weight는 최대 0.10까지 감소 가능  
  
감소된 dense weight의 나머지 비중은 sparse와 BM25가 기존 비율대로 나눠 가진다.  
  
의도:  
- sparse와 BM25가 서로 같은 문서를 강하게 지지하면 lexical evidence가 강하다고 보고  
- dense 비중을 약간 줄여 lexical 쪽 신호를 더 반영  
---
### 7.4 Fusion score 계산  
  
이제 dense / sparse / BM25 결과를 `article_uid` 기준으로 하나로 합친다.  
  
#### Weighted score  
  
```python  
dense_weighted_score = dense_normalized_score * dense_weight  
sparse_weighted_score = sparse_normalized_score * sparse_weight  
bm25_weighted_score = bm25_normalized_score * bm25_weight  
```  
  
#### Hit count  
  
```python  
hit_count = int(dense_hit) + int(sparse_hit) + int(bm25_hit)  
```  
  
의미:  
- 한 문서가 몇 개 retriever에서 동시에 잡혔는지
  
####  Overlap bonus  
  
```python  
overlap_bonus = max(hit_count - 1, 0) * OVERLAP_BONUS_STEP  
```  
  
기본값:  
- `OVERLAP_BONUS_STEP = 0.03`  
  
즉:  
- 1개 retriever에서만 잡히면 bonus 0  
- 2개 retriever에서 잡히면 0.03  
- 3개 retriever에서 잡히면 0.06  
  
의미:  
- 여러 retriever가 동시에 지지하는 문서에 소폭 가산점 부여

---
### 7.5 최종 fusion score  
  
```python  
fusion_score =  
    dense_weighted_score    + sparse_weighted_score    + bm25_weighted_score    + overlap_bonus  
```  

이 fusion score 를 통해 top 15개를 뽑았다. 
[[09-2 similarity apporach]]

여기까지만 해도 꽤 좋은 결과가 나왔지만, top 1 , 2 , 3 간 score 점수의 차이가 너무 비슷했고, 
세부적인 내용은 top 15안에는 나오지만, dense 의 성질이 너무 강한탓에 정답 조문이 dense 조문에게 밀려 비교적 낮은 순위로 나오는 경우가 있었다. 

---
### 8. Cross-Encoder Reranker 도입

#### 8.1 Retriever와 Reranker의 차이

현재 dense/sparse/bm25는 1차 검색기이다.

```text
Retriever = 전체 조문 중 관련 있을 만한 후보를 찾는 단계
```

반면 reranker는 이미 가져온 후보들 안에서 순서를 다시 매기는 단계다.

```text
Reranker = 후보들을 query와 직접 비교해서 재정렬하는 단계
```

즉 reranker는 새로운 조문을 찾아오지 않는다.

```text
정답이 top15 안에 있음 → reranker가 위로 올릴 수 있음
정답이 top15 밖에 있음 → reranker는 못 살림
```

현재 실험에서는 정답 조문이 top15 안에 거의 다 들어오는 상태였으므로, reranker를 적용하기 좋은 조건이었다.

#### 8.2 Cross-Encoder 구조

Cross-Encoder는 query와 passage를 따로 임베딩하지 않고, 하나의 pair로 함께 입력한다.

```text
[Query] 위해식품의 의미가 뭐야?
[Passage] 제4조(위해식품등의 판매 등 금지) 누구든지 다음 각 호의...
 ↓
Cross-Encoder
 ↓
relevance score
```

BI-Encoder와 비교하면 다음과 같다.

|구분|역할|장점|단점|
|---|---|---|---|
|BI-Encoder|1차 후보 검색|빠름, 대규모 검색 가능|세밀한 관계 판단 약함|
|Cross-Encoder|2차 후보 재정렬|정밀도 높음|계산 비용 큼|

#### 8.3 사용 모델

현재 사용할 reranker 모델은 다음과 같다.

```text
BAAI/bge-reranker-v2-m3
```

사용 라이브러리:

```bash
pip install -U FlagEmbedding
```

기본 사용 예:

```python
from FlagEmbedding import FlagReranker

reranker = FlagReranker("BAAI/bge-reranker-v2-m3", use_fp16=True)

pairs = [
    [query, passage_1],
    [query, passage_2],
    [query, passage_3]
]

scores = reranker.compute_score(pairs, normalize=True)
```

---
### 9. 상위 15개 rerank  
  
fusion ranking이 끝났다고 바로 final이 되는 것은 아니다.    
상위 후보만 다시 cross-encoder reranker로 재평가한다.  
  
기본값:  
- `RERANK_TOP_K = 15  
- reranker model: `BAAI/bge-reranker-v2-m3`  
  
동작:  
- `hybrid_results[:15]`만 rerank 후보로 잡는다.  
- 각 후보에 대해 아래 pair를 만든다.  
  
```python  
[question_query, build_rerank_passage(candidate)]  
```  
  
`build_rerank_passage()`는 대략 아래 텍스트를 합친다.  
- `article_no`  
- `title`  
- `full_text` 또는 `clean_text` 또는 `text`/`snippet`  
  
즉 reranker는:  
- 질문과  
- 조문 번호 + 조문 제목 + 본문/스니펫  
  
의 쌍을 받아 relevance score를 계산한다.  
  
rerank 점수는 `normalize=True`로 받아 `rerank_score`에 저장된다.

---
### 9. rerank 문제점 

rerank 모델을 bge-reranker-v2-m3 을 쓰다보니, 의미 분석은 잘해주는데, 너무 쿼리의 의미에 중요도를 두다보니 쿼리의 의미 해석은 훨씬 잘하는데, 의미 해석을 하다보니 비슷한 의미를 갖는 조를 잘 구별하지 못하는 상황이 발생함. 

원래 이건 hybrid로 개선을 확실히 한 상태였는데, rerank를 한번 더 하면서 다시 생긴 문제

### 10. 최종 개선 구조

현재 가장 효과가 좋았던 구조는 다음과 같다.

```text
1. dense / sparse / bm25로 chunk-level search 수행
2. hybrid score 계산
3. chunk 후보 top15 확보
4. article_uid 기준 grouping / dedup
5. unique article candidate 생성
6. reranker input 생성 
7. Cross-Encoder reranker 적용
8. rerank_score 기준 final top3 출력
```

---
### 10. 한 줄 요약  
  
`test2.py`의 최종 `top3`는  `dense + sparse + BM25 후보를 article_uid 기준으로 통합`한 뒤,    
`정규화된 가중합 + overlap bonus`로 1차 정렬하고,    
`상위 15개를 reranker로 다시 정렬한 결과의 첫 3개`다.  
  
---

