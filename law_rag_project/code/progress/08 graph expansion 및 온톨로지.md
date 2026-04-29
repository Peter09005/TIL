**reranker** 

**hybrid retrieval** 

만으로는 한계가 있다는걸 느낌 

아무리 dense , sparse 를 쪼개고 해당 질의를 reranker로 돌려도, 이게 **정의** , **참조** , **의무** 조문이냐 
이런걸 확실하게는 못잡는 느낌이였음. 

특히 **이걸 어기면 어떻게 돼**? 같은 질의는 벡터 검색으로는 **이걸** 과 관련된 조문은 잘 찾아올수있지만, 
**어기면 어떻게 돼** 와 관련된 조문은 찾기가 어렵다. 

따라서 현재 구조에서 뽑은 hybrid 15 중에서 **graph ontology** 를 만들어 reranker의 성능을 높혀보겠다. 

현재 큰 pipeline은 다음과 같다 

~~~text
검색 
 |
 |
하이브리드 
 |
 |
reranker + hybrid 
~~~

이를 다음과 같이 개선시키려고 한다 

~~~text
검색
 |
 |
하이브리드 
 |
 |
보강 - graph 도입
 |
 | 
reranker + hybrid 
~~~


### graph ontology 도입 후 나아져야할 것 

~~~text 
1. 참조 조문 연결  
2. 정의 조문 연결  
3. 벌칙/과태료/처분 연결  
4. 조-항-호-목 계층 연결  
~~~

기존에는 **reranker** 에 넣을때 계층 연결을 상위 문맥으로 연결했지만 
**graph** 를 쓰면 더 좋은 reranker input을 넣을수있을거라 기대한다. 

## 개선안 1 

**neo4j** + **lang chain** 기법으로 hybrid top 15에 대해 graph ontology 생성 후 
해당 데이터 reranker에 넣기


## 개선안 2 

**색인** 이전 데이터에 **LLM** 을 활용해 chunk 단위에 **Contextual Retrieval** 데이터 붙이기 
-> **BGE/M3** 의 벡터화가 정확해질 가능성 **매우 높음** 


## 개선안 3

graph extension을 python으로 구현하기 
milvus 구조 바꾸지 않고 유지할수있음 





