
질의 라우터에서부터 dense/ sparse 로 나누니, avg_score , top 3 hit 갯수는 증가했지만 
기존 hybrid에서 잘 뽑혔던 질의가 안뽑히는 케이스가 생김 

이에 router에서 이분화로 나누지말고 

router에서 해당 질의가 sparse인지, dense 인지 confidence를 계산해봄 

confidence가 80% 이상이면 해당 top 15을 선택하고 , 그 아래라면 hybrid 로 조문을 뽑아보겠음 
hybrid 는 기존 가장 효율이 좋았던 (dense: 0.8 , sparse: 0.2) 를 사용했고, 추후에 confidence 를 반영해서 위 비율을 조정해볼까 생각중임 

오히려 기존 [[04 query router 적용]] 보다 성능이 안나옴. 

그냥 confidence 를 hybrid ratio로 넣어보겠음. 이게 맞는듯? 

여기서 confidence 를 계산 하는게 진짜 쉽지않음 

sparse 비율이 높으려면 해당 조문에서 실제 단어나 이런게 데이터가 있어야되는데 
내가 일일히 추가하는것도 가능하겠지만 추후 다른 법령 데이터셋이나 이런거에 아예 
대응을 못함 

사실 조문 단위 full_text 로 안하고 다른 chunk로 학습시켰으면 좀 더 수월했을거같긴한데 
full_text 기반으로 학습시키라고 하니 해당 조문 full_text 를 사전화 시켜서 
n-gram.json 을 만들어야겠음 

query 가 날라오면 n-gram 사전에 가서 confidence를 계산하는 방법임. 





