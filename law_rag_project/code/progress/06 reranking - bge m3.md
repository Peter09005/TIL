
기존 하이브리드로 뽑아낸 15 개의 조문을 

조문 + query 단위로 bge - reranking 에 넣어서 reranking 을 시키니 
기존 세부적인 내용 못잡는부분 개선됨. 

하지만 전체적인 성능은 내려감 

[[11 reranking]] 

### 문제 1 

reranker가 hybrid 순위를 망가뜨리는 케이스가 있음

-> hybrid_score + rerank_score 혼합하겠음 

보니 지금 중복되는 조들이 과하게 상승되는 문제가 있음 

-> 중복되는 조문이 나오면 점수가 더 조문을 택하고 나머지는 버리겠음 

-> **top 5** 에 중복으로 올라오는 조문이 없어져 중복되는 조문의 점수의 reranking이 
과도하게 높아지지 않게 바뀜 

**개선 전**
```json
{  
  "query": "기숙사 급식시설처럼 집단으로 식사를 제공하는 시설에 대한 규정은 어디 조문이야?",  
  "expected_article_uid": "law::main::88",  
  "score": 0.4,  
  "fused_top5": [  
    {  
      "rank": 1,  
      "article_uid": "law::main::2",  
      "article_no": "law::main::2",  
      "title": "정의",  
      "chunk_type": "subitem",  
      "hit_count": 2,  
      "rank_sum": 3,  
      "fusion_score": 0.36,  
      "hybrid_score": 0.36,  
      "rerank_score": 0.6870366896506998,  
      "overlap_bonus": 0.03,  
      "dense_normalized_score": 0.0,  
      "sparse_normalized_score": 1.0,  
      "bm25_normalized_score": 1.0,  
      "dense_weighted_score": 0.0,  
      "sparse_weighted_score": 0.198,  
      "bm25_weighted_score": 0.132,  
      "dense_hit": false,  
      "sparse_hit": true,  
      "bm25_hit": true,  
      "dense_rank": null,  
      "sparse_rank": 2,  
      "bm25_rank": 1  
    }
```

**개선 후 
```json
{  
  "rank": 1,  
  "canonical_uid": "law::main::2",  
  "article_uid": "law::main::2::2026-12-31",  
  "article_no": "law::main::2::2026-12-31",  
  "title": "정의",  
  "chunk_type": "subitem",  
  "hit_count": 2,  
  "rank_sum": 2,  
  "dense_score": 0.0,  
  "sparse_score": 1.0,  
  "bm25_score": 1.0,  
  "fusion_score": 0.3,  
  "hybrid_score": 0.3,  
  "rerank_score": 0.21629131220919348,  
  "overlap_bonus": 0.03,  
  "dense_normalized_score": 0.0,  
  "sparse_normalized_score": 1.0,  
  "bm25_normalized_score": 1.0,  
  "dense_weighted_score": 0.0,  
  "sparse_weighted_score": 0.162,  
  "bm25_weighted_score": 0.108,  
  "dense_hit": false,  
  "sparse_hit": true,  
  "bm25_hit": true,  
  "dense_rank": null,  
  "sparse_rank": 1,  
  "bm25_rank": 1,  
}
```


### 문제 2

기존 hybrid 는 잘 뽑혔는데, rerank 차원에서 hybrid 를 밀었음. rerank에서 다시 해석을 해버리니까 비슷한 조문끼리 비교를 잘 못함 

기존 hybrid 로 해당 문제를 해결했는데 

이에 hybrid + reranker 로 final 산출물을 내야겠음 

#### apporach 1 

```
final_score = hybrid_score x 0.8 + rerank_score x 0.2 
```
#### apporach 2

```
hybrid top 3의 hybrid_score = final_score , 나머지 top 4 ~ 10 에서 reranking 후 rerank_score = final_score  
```
#### apporach 3

```
rerank_score >= 0.05 hybrid 만 적용 
```
#### apporach 4

```
hybrid_score > rerank_score -> hybrid 적용 
```

## 결과 테이블 

![[problems_query_report_test2_comparison.xlsx]]


## 결과 정리

4개의 apporach 전부 기존 low score 이였던 질의를 확실하게 top 5 위로 끌어올리는 모습을 보여줌. 

기본적으로 괜찮다고 생각했던게 apporach 2, apporach 4 인데, 정확도는 챙기고, reranking 에서 
질의의 의도도 잘 잡아내는걸로 보임. 


### 문제점들 

- 일단 bge-m3 reranking 이 너무 무거움 
	- 이건 내 컴퓨터가 그래픽 카드가 없어서 그런걸수도 
	- 그래도 조문 전체를 넣는거보다 좀 쪼개서 넣으면 더 의도 파악이나 쏠림 현상 사라지지 않을까 생각 중 


- hybrid search에서 너무 많은 일을 함 
	- dense , sparse , bm25 비율 
	- 유사도 계산 
	- 가중치 더해서 hybrid 10 반환 
	- hit ratio 계산
     
    너무 복잡하게 뽑아서 해당 법령에 **overfitting** 한 상황일수도있음 


- 다양한 테스트 케이스의 부재 
	- 직접 만든 케이스를 사용하다보니 진짜 문제를 못찾고있을수도있음 








