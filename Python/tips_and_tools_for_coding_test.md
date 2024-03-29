# 코딩테스트 때 써먹기 좋은 파이썬 도구
- Swift 엔지니어의 입장에서 쓴 Python3 꿀팁모음!

## 들어가기 전에
- 먼저 파이썬은 컴파일 언어기 때문에 위에서부터 코드를 읽는다.
- 그렇기 때문에 솔루션 함수 안에서 쓸 함수를 밖에 분리해서 적는 건 좋지만, 그 경우 솔루션 함수보다 위에 쓸 것.


## 문자열 포매팅
``` python
name = "Gordon"
age = 30
height = 183.0

# 각각 str, int, decimal 타입임을 나타냄
description = "name %s age %i height %d" % (name, age, height)
description = "name {name} age {age} height {height}".format(name=name, age=age, height=height)

# 소수 몇째 자리까지 포매팅하는 방법
# 콜론 다음에 표시할 정수의 자릿수, 점 다음에 표시할 소수 자릿수를 넣어 준다.
# 이런 방식의 포매팅을 Rust도 쓰다 보니, 이쪽으로 익히는 것이 좋아 보인다.
print("{} {:.2f}".format(5, 5 / 2))

```

## 삼항 연산자
``` python
a = 100
result = "{a} is even".format(a=a) if a % 2 == 0 else "{a} is odd".format(a=a)
```

## 대소문자 판별
``` python
a = "Python"
a.islower() # false
a.isupper() # false
a = a.lower() # "python"
a.islower() # true
```

## 키 코드와 진법 변환
``` python
# 아스키 코드와 문자
chr(65) # "A"
ord("a") # 97

# 기본 지원 진법 변환
int("1010", 2) # (수 스트링, 진법)
bin(30) # "0b11110"
oct(30) # "0o36"
hex(30) # "0x1E"
```

## 고차함수
``` python
numbers = [1, 2, 3]

# list로 꼭 변환을 해 줘야 한다 - 고차함수의 결과로 나오는 컨테이너는 list가 아니다
# 여기서는 람다 함수를 쓰지만, 당연히 따로 정의한 함수도 조건 파라미터에 들어갈 수 있다
mapped = list(map(lambda x: x ** 2, numbers)) # [1, 4, 9]
filtered = list(filter(lambda x: x % 2 == 1, numbers)) # [1, 3]

from functools import reduce
# 초기값을 잘 넘겨야 한다
reduced = list(reduce(lambda x, cur: cur * x, numbers, 0)) # 6
```

## itertools
``` python
import itertools

nums = [1, 2, 3]

# permutations
# 뒤에 갯수를 안 넣으면 모든 리스트의 인수에 대해 수행한다
list(itertools.permutations(nums, 2))
# [(1, 2), (1, 3), (2, 1), (2, 3), (3, 1), (3, 2)]

# combinations
list(itertools.combinations(nums, r=2))
# [(1, 2), (1, 3), (2, 3)]
list(itertools.combinations_with_replacement(nums, r=2))
# [(1, 1), (1, 2), (1, 3), (2, 2), (2, 3), (3, 3)]

# products
list(itertools.product(nums, repeat=3))
# [(1, 1, 1), (1, 1, 2), (1, 1, 3), (1, 2, 1), (1, 2, 2), (1, 2, 3), (1, 3, 1), (1, 3, 2), (1, 3, 3), (2, 1, 1), (2, 1, 2), (2, 1, 3), (2, 2, 1), (2, 2, 2), (2, 2, 3), (2, 3, 1), (2, 3, 2), (2, 3, 3), (3, 1, 1), (3, 1, 2), (3, 1, 3), (3, 2, 1), (3, 2, 2), (3, 2, 3), (3, 3, 1), (3, 3, 2), (3, 3, 3)]
```

## index
``` python
nums = [1, 2, 2, 3, 3]

nums.index(2) # 1
```
- 파이썬이라기보단 재귀의 특성인데, 콜 스택의 특성상 사전 순 정렬은 신경쓸 필요가 없다.
- 12345를 가지고 사전을 만들면
  - 1 11 111 1111 11111 11112 11113 ... 이렇게 진행되기를 바랄텐데
  - 문제 조건상 1에서 불러온 재귀 함수의 콜 스택의 리턴 조건은 5글자! 그렇기 때문에 자동으로 추가되고 돌리고.. 하는 스택이 쌓인다.

## 재귀 제한 늘리기
``` python
# 기본적으로는 1000번까지 가능하지만
import sys
sys.setrecursionlimit(2000)
# 이렇게 하면 2000번으로 수정된다
# 많아 봐야 10000번이면 코테 수준은 커버 가능하다
```

## 비트마스킹
``` python
installed = [0, 0, 1, 1, 1]
installed_bin = 0b00111

# 소프트웨어 1~5번까지 각각을 깔았나 확인하려면 원래는 하나씩 봐야 하지만!
if installed_bin & 0b00111 == 0b00111:
    print("Install complete")
else:
    print("Install incomplete")

# 비트 연산자로 &(and), |(or), ^(xor), ~(not) 정도는 알아 두자.
# Swift에서는 let example: UInt8 = 0b000111 이런 식으로 쓸 수 있다! 
# 비트를 이동하는 << >> 연산자도 있다. 이동하고 나서 빈 공간은 0으로 채우는 방식.
```

## 접두사와 접미사
``` python
string = "Gordon"
string.startswith("Gor") # True
string.endswith("on") # True

# 아니면 그냥
suffix = "on"
string[-len(suffix):] == suffix # True
string[:len(suffix)] == suffix # False
```

## 정규 표현식
``` regex
[]: 안쪽에 범위 내의 하나와 매치
    [a-zA-Z], [가-힣], [abc], ...
.: \n을 제외한 모든 문자
    [] 안에 쓰면 실제 . 문자를 찾는다..
*: 앞에 있는 문자가 0개부터 무한개까지 반복된다
    a*
+: 앞에 있는 문자가 1개부터 무한개까지 반복된다
{}: 문자 뒤에 붙어서 안에 있는 갯수의 범위 내에서 반복된다
    a{2,5}: a가 2번에서 5번까지 반복
    a{2}: a가 2번 반복
?: {0,1}과 같다.
\d: 숫자
\s: 화이트스페이스 문자 - 공백, 개행, ...
\w: 문자와 숫자
이상의 표현식들을 대문자로 쓰면 not의 의미를 가진다.
    \D: 숫자가 아닌 것
```

``` python
import re

pattern = re.compile("[0-9]+aa")

# match되는 패턴이 처음에 있는 경우
example = pattern.match("1234aa35")

# 매칭되는 패턴이 어디에라도 있는 경우
example = pattern.search("1234aa35")

# 완전히 매칭되어야 하는 경우
example = pattern.fullmatch("1234aa35")

# 매칭되는 모든 패턴을 추출할 경우
example = pattern.findall("1234aa 4aa 123aa")

# 생 스트링 넣으려면
re.match("[0-9]+aa", "1234aa35")
re.fullmatch("[0-9]+aa", "1234aa35")

# 참조: 분리하기
# findall을 배열로 만든 건가
# r을 스트링 앞에 붙인 건 raw string이라는 뜻으로, 이스케이핑 문자를 쓰지 않아도 된다.
numbers = re.split(r'([-+*/()])|/s+', expression)
```

### 참조: Swift 정규표현식
``` swift
import Foundation

// 슬래시 사이에 넣어 주면 컴파일러가 정규표현식 패턴임을 인식한다
let pattern = /[0-9]+aa/
let input = "1234aa35"

// 문자열 어디서든 처음으로 매칭되는 부분
input.firstMatch(of: pattern)?.output

// 문자열 처음부터 매칭되어야 하는 부분
input.prefixMatch(of: pattern)?.output

// 문자열 전체 일치
input.wholeMatch(of: pattern)?.output

// 일치하는 모든 부분
input.matches(of: pattern)?.map { $0.output }

// 참조: 정규식 대신 스트링 넣기
input.firstMatch(in: "[0-9]+aa")
input.wholeMatch(in: "[0-9]+aa")
input.range(of: "[0-9]+aa", options: .regularExpression) // firstMatch와 같다
```

## 후위 연산자 만들기
``` python
# 일반적인 중위 연산자 식 스트링을 후위 연산자 식으로 바꾸기
# 리스트 tokens, 우선순위 딕셔너리 priority
def toPostfix(tokens, priority):
    stack = []
    postfix = []
    
    for token in tokens:
        if token.isdigit():
            postfix.append(token)
        else:
            if not stack:
                stack.append(token)
            else:
                while stack:
                    if priority[token] <= priority[stack[-1]]:
                        postfix.append(stack.pop())
                    else:
                        break
                
                stack.append(token)
    
    while stack:
        postfix.append(stack.pop())
        
    return postfix

# 후위 연산자 식 계산하기
def assemble(tokens):
    numbers = []
    
    for token in tokens:
        if token.isdigit():
            numbers.append(int(token))
        else:
            second = numbers.pop()
            first = numbers.pop()
            numbers.append(calculate(first, token, second))
    
    return numbers.pop()

# 어따 쓰냐? 연산자의 우선순위나 순서를 바꾸는 문제는 후위 연산자로 대응하는 것이 정석이다.
# 어쩌면 계산기 기능도 이걸로 만들 만 할지도
```

## 소수 판별
``` python
from math import sqrt

def is_prime(n):
	for i in range(2, int(sqrt(n))):
		if n % i == 0:
			return False

	return True
```

## eval
``` python
expression = "1 + 2"
absolute = "abs(-4)"

# 간단히 말해, raw string으로 표현되어 있는 코드를 실행시켜주는 함수
# 그래서 보안상 문제가 있다고는 한다
eval(expression) # 3
eval(absolute) # 4
```

## 정렬
``` python
# 퀵 정렬 - 피벗은 첫 번째 원소
def quicksort(data):
    if len(data) == 1:
    	return data
    	
    pivot = data[0]
    rest = data[1:]

    left = [x for x in rest if x < pivot]
    right = [y for y in rest if x > pivot]

    return [*quicksort(left), pivot, *quicksort(right)]

# 병합 정렬
def merge(left, right):
    result = []
    while len(left) > 0 and len(right) > 0:
    	if left[0] <= right[0]:
    	    result.append(left[0])
    	    left = left[1:]
        else:
            result.append(right[0])
            right = right[1:]

    # extend는 안에 들어가는 iterable의 컨텐츠를 모두 추가하겠다는 뜻이다
    # Swift의 Array.append(contentsOf:) 메서드와 비슷하군
    result.extend([*left, *right])
    return result

def mergesort(data):
    if len(data) <= 1:
    	return data

    mid = len(data) // 2

    left = mergesort(data[:mid])
    right = mergesort(data[mid:])
	
    return merge(left, right)

# 힙 정렬 - 근데 이걸 정렬 함수라고 해야 할지
from heapq import heappush, heappop

def heapsort(iterable):
    h = []
    for value in iterable:
    	heappush(h, value)

    return [heappop(h) for _ in range(len(h))]

# 직접 구현하면
def heapify(data, n, i):
    largest = i
    left = 2 * i + 1
    right = 2 * i + 2
    
    if left < n and data[i] < data[left]:
    	largest = left

    if right < n and data[largest] < data[right]:
    	largest = right

    if largest != i:
    	data[i], data[largest] = data[largest], data[i]
    	heapify(data, n, largest)

def heapsort(data):
    n = len(data)
    for i in range(n, -1, -1):
        heapify(data, n, i)

    for i in range(n - 1, 0, -1):
        data[i], data[0] = data[0], data[i]
        heapify(data, i, 0)
```

## 여러 조건으로 정렬하기
``` python
def solution(strings, n):
    return sorted(strings, key = lambda x: x[n] + x)
    # 혹은
    return sorted(strings, key = lambda x: (x[n], x))
```

## 이진 탐색
``` python
from bisect import bisect_left, bisect_right

nums = list(range(0, 10))

bisect_left(nums, 5) # 5
bisect_right(nums, 5) # 6
# 그냥 bisect 함수는 bisect_right와 똑같다

# 특정 범위 내의 원소 수 구하기
def calCountsByRange(nums, left_value, right_value):
    r_i = bisect_right(nums, right_value)
    l_i = bisect_left(nums, left_value)
    return r_i - l_i

nums = [-1, -3, 5, 5, 4, 7, 1, 7, 2, 5, 6]
nums.sort()
calCountsByRange(nums, 5, 7) # 6
```

## defaultdict
``` python
from collections import defaultdict

# defaultdict(키 추가시 기본으로 넣어줄 밸류)
# 이 경우에는 키를 조회하면, 키가 없을 때 빈 리스트를 밸류로 해서 자동으로 키가 할당된다
# 단 그러다보니 del을 사용해도 해당 키가 없어지질 않는다..
example = defaultdict([])
```

## 예외처리하기
``` python
# 별다른 종결 조건이 주어지지 않는 경우, 예를 들어 EOF(End Of File) 오류가 발생하는 경우 적합
while True:
    try:
    	n, m = map(int, input().split())
    	print(n + m)
    except:
	# 여기서 예외 처리를 하면 되는듯?
	break
```

## 입출력 시간 줄이기
``` python
import sys

# input과 print 함수를 바꿔 준다
input = sys.stdin.readline
print = sys.stdout.write

# 단, input 함수는 공백 문자까지 읽어들이게 되며, print는 줄바꿈이 없어진다
# 먼저 rstrip으로 오른쪽의 공백 문자 없애기
command = list(input().rstrip().split())

# 다음으로 프린트할 때는 줄바꿈 문자 붙여주기
for r in result:
    print("%s\n" % r)
```

## 최대 힙 큐 만들기
``` python
# 사실 마이너스 플러스 구분만 신경써서 잘 해 줘도 되긴 하다
class PriorityQueue:
    def __init__(self):
    	self.items = []
    def push(self, item):
    	heapq.heappush(self.items, -item)   
	
	def empty():
    	return not self.items
    	
    def pop(self):
    	if self.empty():
    		raise Exception("empty")
    		
    	return -heapq.heappop(self.items)

    def size(self):
    	return lens(self.items)

    def top(self):
        if self.empty():
        	raise Exception("empty")
			
        return -self.items[0]
```