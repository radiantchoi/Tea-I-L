# Day 1

# 코딩 테스트

## 코딩 테스트를 보는 이유
- 문제 해결 과정 보기, 생각하는 실력 다지기, 빠르고 효율적으로 풀기.

### 문제 해결 과정 보기
- 결국 인터넷에서 복붙한 코드로는 아무것도 할 수 없으니까.
- 여러 변수 없이, 순수하게 코드를 얼마나 잘 짜는지만 보는 거다.
- 효율적으로 풀고, 언어의 한계점을 염두해야 한다.

### 생각하는 실력 다지기
- 단순 지식이 아닌 논리적으로 생각해서 풀어야 하는 문제도 있다.
- 나만의 방법으로 규칙을 찾고 코드에 적용하는 능력을 길러야 한다.

### 같은 결과를 빠르고 효율적으로
- 배열, 문자열, 재귀, 완탐, 정렬, 해시, DP, 그리디, 스택, 큐, 이진 탐색, pq, 그래프 등을 본다.
- 먼저 문제를 읽고 풀어보기
- 과정을 되짚으면서 해설을 읽고 비교하기
- 설명이 이해가 되지 않으면 표시하고 과감히 넘어가기
- 모두 이해했다면 한 문제를 푼 거지만, 다른 방법으로도 시도해 보기

## 코딩과 디버깅

### 잘 짠 코드의 기준?
- 시간 복잡도
- 공간 복잡도
- 가독성

### 코딩할 떄 흔히 하는 실수
- 런타임 에러로 드러난다.
1. 존재하지 않는 요소에 접근하지 않는지?
2. Python 배열 슬라이싱은 `[시작값:끝값+1:간격]`
3. 부등호 쓸 때 경계값 체크
4. 연산자 우선순위
5. 자료 형변환
6. 최댓값, 최솟값 파악
7. 반복이나 재귀에서 종료 조건 체크
8. 곱하거나 나눌 때 0을 조심

### 디버깅과 시행착오 줄이기
1. 개발을 과정별로 분리
2. 상수, 전역 변수, 지역 변수를 구분하고 구획화
3. 의미 있는 변수 이름
4. 퍼스트 파티 지향
5. 언어의 한계 - 지원하지 않는 기능을 파악하고 어떻게 대체할 것인지 고민

# 시간 복잡도

## 시간 복잡도란
- 간단히 말해 얼마나 오래 걸리는지

### Big-O
- O(1), O(logn) O(n), O(n^2), O(nlogn), and so on...
- 문제에서 주어지는 제한조건을 고려해 적합한 시간 복잡도를 골라야 한다.
  - 컴퓨터는 대략 1초에 1억 번을 계산한다. 당연히 요즘 기계는 이거보다 성능이 좋지만, 이건 약간 국룰 같은 거다.
  - 인풋 최댓값이 10만이면 못해도 nlogn을 선택해야 한다.

### 시간 복잡도 선택시 참고사항
- 최대 시간 1초 기준으로
1. 1000개는 O(n^2) 이하
2. 10000개면 O(n^2) 미만
3. 100000개면 O(nlogn) 이하
4. 1000000개면 가급적 O(n)
5. 그거보다 많으면 특정 알고리즘을 써야 할 가능성이 높다
- 자료구조의 탐색/삽입/삭제 시간도 고려해볼 수 있다.

## 시간 복잡도 계산
- 계산 횟수를 잘 따져 봐야 한다..!
- 배열, 딕셔너리, 집합이나 삽입, 정렬, 삭제, 슬라이싱 연산에 대해서도 염두하면 좋다.

### 시간 복잡도 줄이기
- `input()` 대신 `sys.stdin.readLine()`: 입력을 받아야 할 경우, 데이터가 많을수록 인풋은 비효율적
- `[0 for _ in range(1000)]` 대신 `[0] * 1000`: `for`문으로 하나씩 넣기보다 할당을 동시에 해 주는 곱셈
- `+`보다는 `"".join()`: 파이썬 문자열은 바꿀 수 없기 때문에, + 연산을 하는 각각의 문자열을 복사한다.
- 다중 조건문의 경우 빨리 실행되는 쪽을 앞으로 배치하기
- 슬라이싱: 파이썬 활용의 꽃! iterable 자료형이라면 무엇이든 자를 수 있다. 자료의 범위를 좁혀 불필요한 연산을 줄여보자.
- 퍼스트 파티 표준 라이브러리 쓰기. 속도와 안전함을 모두 잡자!
  - `heapq`: 이진 트리 기반의 최소 힙 구조
  - `collections`: 동일한 원소 갯수를 잴 수 있는 `counter`가 포함되어 있다! `deque`도 여기 있다.
  - `itertools`: 경우의 수 문제에 사용한다. 
    - `permutations`, `combinations`, `product`
    - `permutations_with_replacement`, `combinations_with_replacement`
  - `math`: 수학 계산!
    - `ceil`, `floor`
      - `round`는 기본 구현 함수다.
    - `factorial`
    - `sqrt`
    - `fmod`
    - `gcd`, `lcm`
  - `bisect`: 이진 탐색! `bisect_left`와 `bisect_right`가 있다. 당연히 정렬된 데이터가 필요하겠지!
- 리스트 컴프리헨션과 제너레이터
  - `[x for x in range(1, 10)]`: 편하게 구현되나, 메모리 공간을 먹는다.
  - `(x for x in range(1, 10))`: 제너레이션! `next()` 호출시까지 대기한다.
    - 제너레이션은 함수에서 `yield`를 `return` 대신 써서 만들거나, 컴프리헨션을 소괄호로 감싸면 된다.
    - 제너레이션은 메모리 소모가 항상 고정적이다. 대신 값을 저장하려면 list() 등의 방법을 써야 한다.
- 중복 피하기: 의미없는 데이터의 복사나 연산에 주의. 코딩 테스트 문제는 일련의 풀이 흐름이고, 흐름을 최적화하는 것은 개발자의 역량.