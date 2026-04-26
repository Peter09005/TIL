
===== FINAL =====
num_cases = 81
total_score = 71.0
avg_score = 0.8765
mode = hybrid_hit
use_reranking = True
reranker_model = BAAI/bge-reranker-v2-m3
rerank_candidate_top_k = 15
fusion_weights = dense_base:0.75, sparse_base:0.15, bm25_base:0.1, weight_shift_max:10.0, similarity_top_k:5, overlap_bonus_step:0.03
matched_within_top3 = 74
low_score_queries(<= 0.4) = 10
top1_accuracy = 0.8272
top5_hit_count = 80
miss_count = 1