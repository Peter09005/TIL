

## 이미 만들어진거 

법률 QA시스템은 KorBERT 에 기반한 검색 재순위화와 기계독해 기술을 적용하여 보다 정확한 정답을 제시할 수 있습니다.
https://epretx.etri.re.kr/apiDetail?id=70

korbert ? 
법령문서에서 조 내용을 검색하고 정답을 추론하여 제공하는 질의응답 API 입니다.

e5 bm25 
### 한국어 법률 질의응답 시스템의 최적화된 RAG 아키텍처는 무엇인가?

연구 결과, 한국어 법률 질의응답 시스템의 최적화된 RAG 아키텍처는 의미 기반 청킹, 한국어 법률 QA 데이터로 파인튜닝한 E5 임베딩 모델, 그리고 파인튜닝된 E5와 BM25를 8:2 가중치로 결합한 하이브리드 검색 방식을 결합한 형태이다. 이는 검색 정확도와 응답의 사실성 모두에서 우수한 성능을 보인다.

hybrid -> why not bm25?  e5가 파인튜닝해서 그런가 -> 아마 의미추론떄문에 

recall? 

https://www.kci.go.kr/kciportal/ci/sereArticleSearch/ciSereArtiView.kci?sereArticleSearchBean.artiId=ART003234176