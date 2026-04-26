dense ratio 

0.1
  "avg_score": 0.73
  "matched_within_top3_count": 66,
0.2
  "avg_score": 0.7580246913580249,
  "matched_within_top3_count": 68,
0.3
  "avg_score": 0.7617283950617286,
  "matched_within_top3_count": 68,
0.4
  "avg_score": 0.7938271604938273,
  "matched_within_top3_count": 70,
0.5
  avg_score = 0.7506
  matched_within_top3 = 68
0.6 
  avg_score = 0.7926
  matched_within_top3 = 69
0.7
 avg_score = 0.7975
 matched_within_top3 = 70
0.8
 avg_score = 0.7988
 matched_within_top3 = 71
0.9
avg_score = 0.7901
matched_within_top3 = 70
1.0
avg_score = 0.7481
matched_within_top3 = 69


dense: 0.8 
sparse: 0.2 

> dense | sparse top 15안에는 위 쿼리들이 다 들어있는 상태이긴함 -> good! 
> dense + sparse union 해서 그 안에서 reranking 으로 뽑으면 될듯 

아님 bm25 를 추가해서 dense + sparse + bm25 3개를 쓰는것도 방법이긴함 