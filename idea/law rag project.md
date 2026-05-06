
## 프로젝트 목표
~~~
세무사 workflow copilot

expected query: "법인세법 시행령 43조 특수관계인 범위 관련 최근 심판례"
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

## 온톨로지 
~~~
json 스키마 기반
soft ontology 
~~~

## 임베딩
~~~
bge/m3 or e5 

법률 용어 사전 전처리 
~~~

## db 
~~~
milvus 
elasticsearch 
neo4j
postgres 
~~~

## 데이터 소스 
~~~
국가법령정보센터 openapi 
~~~