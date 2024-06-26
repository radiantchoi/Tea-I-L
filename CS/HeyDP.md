# DP 네 이놈
- 점화식을 세운다는 것이 핵심
- 메모이제이션: 재귀의 결과를 리스트에 기록
- 태뷸레이션: 연산해서 리스트에 기록하고 마지막 아이템 반환
### 격자를 이동하는 DP
- 이동 자체의 패턴은 비슷하다. 
- 특정 지점에서 나올 수 있는 최댓값을 구하기 위해 특정 방향으로만 이동한다면, 반대로 말하면 지금 숫자로 올 수 있는 이전 숫자가 그래봐야 두 개 정도라는 의미. (위에서 오거나 왼쪽에서 오거나)
- 왜 DP냐면, 사각형의 크기가 다를 뿐 해당 지점을 거쳐가는 연산은 결국 이 지점의 값 + 이전의 값이기 때문이다. 방향이 바뀐다면 이것만 잘 계산해서 하면 된다
### 타일을 배치하는 DP
- 타일의 갯수에 따라, 이전에 채웠던 갯수의 기록에서 따 오면 된다.
- 가령 이 타일을 넣으려면 앞에서 몇 칸이 빠져야 하는지, 같은 것들.
### 적절히 고르기 - 거스름돈
- 지금 거슬러줘야 되는 돈에서 이 동전 하나를 빼면 얼마가 남을까? 를 하나도 안 남을 때까지 / 몇 개만 남을 때까지 등등 반복해주면 된다.
### 적절히 고르기 - Knapsack
- row는 생각할 수 있는 보석(동전)의 갯수, col은 총 무게
- 그래서 row col은 합산 가치. 이 보석을 넣냐 안넣냐를 고려하는 것이다.
### LCS, 편집거리
- dp 테이블을 통해서 정의를 하긴 하는데, 이게 뭘 의미하는지 정의하는 게 중요하다.
- 전자의 경우 이 글자까지 고려했을 경우 가질 수 있는 최대 공통 부분 길이
- 후자의 경우 이 글자까지 고려했을 경우의 편집거리
### 내 생각
- DP는 처음 몇 번 해 보기, 그리고 거꾸로 생각하기가 중요한 것 같다.
- 점화식은 생각보다 다양한 형태로 나올 수 있고, 의사 코드도 점화식이 될 수 있다.