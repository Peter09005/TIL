
time 3 5 1 1 
cost 10 20 10 ..

주어진 시간안에 최대값구하는 문제는 
dp[i] <- i일에 가질수있는 최대값 

loop돌면서 
dp를 업데이트해준다

dp[i] = max(dp[i],dp[i-1]) 
i 정보를 가지고 업데이트
dp[i + time] = max(dp[i] + price, dp[i + time])

핵심은 i 의 정보는 신경쓰지 않고 dp[i] 에 들어간 값은 i 일일때 가질수있는 최대값으로 설정해둬야함

나 같은 경우는 그냥 currentMax = max(dp[i],currentMax) 
하면서 업데이트 해뒀음. 




