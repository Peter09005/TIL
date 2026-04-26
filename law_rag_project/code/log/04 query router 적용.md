간단한 router 설정으로 sparse / dense 구분하니 성능 개선 확인됨 

===== FINAL =====
num_cases = 81
total_score = 64.80000000000003
avg_score = 0.8000
mode = routed
use_reranking = False
matched_within_top3 = 72
low_score_queries(<= 0.4) = 13

하지만 기존에 잘 나왔던게 라우터에서 2분법으로 나눠버리니 잘 안나오는 상황 생김 