
## 프로젝트 목표
~~~
주택 양도세 비과세 판단을 위해 법령·예규·판례 근거를 빠르게 큐레이션하는 세무사 전용 RAG 에이전트
~~~

## rag 품질 및 평가 

~~~
Automatic eval (Ragas)  
+  
Small golden set  
+  
Human review
~~~

## 내부 llm 
~~~
chatgpt openapi 
~~~

## 온톨로지 (완성도 측면)
~~~
json 스키마 기반
soft ontology 
~~~

## 임베딩
~~~
bge/m3 or e5 
text-embedding-3-small (OpenAI)

법률 용어 사전 전처리 
~~~

## db 
~~~
postgreSQL
pgvector  

---후보----
elasticsearch 
milvus
neo4j
~~~

## 데이터 소스 
~~~
국가법령정보센터 openapi 
~~~

## 리랭커 

~~~
bge-reranker 
llm reranking 
~~~

## langchain 

### 데이터 임베딩
~~~
[오프라인 인덱싱]

국세청/법령/예규/내부문서
↓
문서 로딩
↓
세목/자산유형/쟁점별 청킹
↓
metadata 부착
↓
임베딩
↓
벡터DB 저장
~~~

### 실시간 쿼리 
~~~
[실시간 질의응답]

세무사 질문
↓
쿼리 rewrite 
↓
metadata filter
↓
hybrid search (vector + bm25) + llm hypotheical query embedding 
↓
reranking (bge-m3)
↓
근거 chunk 선택
↓
LLM 답변 생성
↓
답변 검증 
↓
멘트 출력 
~~~

## 모니터링 
~~~
LangSmith 
~~~


## LLM 호출 포인트 
~~~
1. 질문 시나리오, 속성 추출 
   
2. 정보 누락 판단 및 역질문 생성 
   
3. RAG 기반 근거 매칭 및 추론 
   
4. 출력 
~~~


## 필요 기술 스택 

~~~
python 
sql 
milvus 
온톨로지 개념
~~~