
지금 문제는 조문을 너무 큰 단위로 쪼개서 생긴 문제 

조문 단위로 sparse 에 집어넣으니 "기숙사" 같은 세부사항은 내부에서 낮은 점수를 측정함 

sparse collection <-- 조/항/호/목 단위까지 학습

dense_collection <-- 조까지 학습  

### dense input

- 조문 full_text
- 또는 조/항 수준의 문맥 있는 텍스트

### sparse input

- 조번호
- 조제목
- 항/호/목 경로
- 현재 chunk 본문

으로 넣어보겠음 

**dense 전용 스키마**
```json
{  
  "chunk_id": "chunk::law::main::101::paragraph::hang_2__jo_101__base",  
  "article_uid": "law::main::101",  
  "chunk_type": "paragraph",  
  "dense_text": "식품위생법 제101조 과태료 제2항 다음 각 호의 어느 하나에 해당하는 자에게는 500만원 이하의 과태료를 부과한다."  
}
```

**sparse 전용 스키마**
```json
{
"chunk_id": "chunk::law::main::2::item::ho_6__jo_2__base",  
"article_uid": "law::main::2",  
"chunk_type": "item",  
"sparse_text": "식품위생법 제2조 정의 제6호 “위해”란 식품, 식품첨가물, 기구 또는 용기ㆍ포장에 존재하는 위험요소로서 인체의 건강을 해치거나 해칠 우려가 있는 것을 말한다."
}
```


```text 
		 / -- dense_collection ----> top 15 조문 ---- \
		|                                              \
query-->|                                               hybrid --> reranking --> output 
		|                                              /
		 \ -- sparse_collection ----> top 15 조문 ----/
```


일단 이렇게 검색해보고 그래도 잘 안된다 싶으면 **bm25** 기반의 Elastic Search 도 도입해서 하이브리드 + reranking 으로 성능 개선시켜보겠음 

[[08 sparse , dense collection 따로 둠]] 로그를 보면 기존에 있었던 대부분의 문제를 해결할수있었음. 

게다가 기존에 안올라오던 질의들도 전부 top 5안에 들어온거보면 지금 기조를 유지하고 reranking으로 성능개선해보겠음 

하지만 아직도 "기숙사는 집단급식소야?" 같은 질의는 잘 해결하지 못함 
이건 bge/m3의 문제인데, 이쪽모델 내부에서 "기숙사" 같은 키워드를 중요하지 않다고 
파악하고 weight 을 내려버린거임. 

하지만 개선은 분명히 있었음. 

기존에는 "기숙사" 라고 쳐도 sparse에 조문 chunk 가 들어가서 sparse 가 아예 해당 조문을 가져오지 못했는데 이젠 sparse 는 목 단위로 chunk 했기에 "기숙사" 라고 치면 해당 조문이 나옴 

sparse 특징을 보니 , 특정 조문에 집단급식소라는 정의가 붙은 조문이 있음 
해당 조문의 정의가 자식까지 붙어서 집단급식소라는 키워드가 너무 이쪽으로 쏠리는 현상 발생 

따라서 sparse는 법 + 항 + 호 + 목 을 임베딩 하는 방식으로 바꿔봄 

[[10 sparse embedding chunk 바꾸기]]

지금보니 dense의 항부분이 항 아래에 있는 내용을 못 긁어가고 있었음. 
sparse 도 마찬가지. 해당 문제 고치고 다시 돌려보겠음....

[[10-1 sparse, dense text에 child 내용까지 포함]]

> 지금 얘가 현재 avg 는 좀 낮지만 거의 모든 질의에 대해 hybrid 기준 top 5 안에 들었다. 지금까지 가장 발전한 버전 






