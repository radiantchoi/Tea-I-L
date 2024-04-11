# Built-in Collections
- 프로그래밍 언어에서, 데이터의 묶음(Collection)은 굉장히 중요하다.
- 여러 가지 다른 컨테이너를 제공하는 것이 좋다.
- Swift 또한 여러 가지 Collection을 제공한다!

# Arrays (배열)
- 순서를 가지고 있는 원소(Element)들의 컨테이너이다.
  - 이 원소들은 전부 같은 타입을 가진다.
  - 각각의 원소에 대해 Random Access 기능을 지원한다.
```Swift
let fibo = [0, 1, 1, 2, 3, 5, 8]
```

## Arrays and Mutability
- 배열의 내용을 바꾸고 싶다면, 위에서 `let`이 아니라 `var`로 선언해야 한다. 당연한 얘기인가?
- `append` 등의 메서드를 활용해 이제 바꿀 수 있다.
```Swift
var fibo = [0, 1, 1, 2, 3, 5]
fibo.append(8)
fibo.append(contentsOf: [13, 21])

print(fibo) // [0, 1, 1, 2, 3, 5, 8, 13, 21]
```
- `var`과 `let`을 구분함으로써 얻는 몇 가지 이점이 있다.
- 먼저 `let`으로 선언한 것은 불변함이 보장된다. 코드를 읽을 때도 도움이 된다!
  - 단 이것은 value semantics가 적용될 때, 즉 원소가 값 타입일 때에 한정된다.
  - 참조 타입 원소가 들어 있다면 배열이 `let`으로 선언되었다 하더라도 그 원소는 바뀔 수 있다.
- 배열은 다른 stdlib 컬렉션 타입들처럼, value semantics를 따른다.
  - `=` 연산자로 로컬 복사본을 가져올 수 있고, 복사본에 가하는 연산이 원본에 영향을 미치지 않는다.
```Swift
var a = [1, 2, 3]
var b = a
b.append(4)

print(a) // [1, 2, 3]
print(b) // [1, 2, 3, 4]
```
- Obj-C 포함 다른 많은 언어는 Reference Semantics 보유 언어인 경우가 많다. JavaScript의 예를 들자면..
```Javascript
const a = [1, 2, 3];
const b = a;
b.push(4);

// 분명 b에만 원소를 추가했지만...
console.log(a); // [1, 2, 3, 4]
console.log(b); // [1, 2, 3, 4]

// 이를 회피하기 위해 똑같은 배열을 따로 만들어 주어야 한다..
c = [1, 2, 3];
d = c.slice();
d.push(5);

console.log(c); // [1, 2, 3]
console.log(d); // [1, 2, 3, 5]
```
- Reference semantics 배열의 경우 이런 특징을 까먹으면 오류가 나기 쉽다.
- 하지만 Swift 언어의 경우 배열에 value semantics를 적용시킴으로써 이를 막는다.
- 이 때문에 메모리/퍼포먼스 문제가 발생할 수 있지만, Swift 내부적으로 Copy on Write 방법을 사용함으로써 이를 방지한다.
  - 이는 Swift 안의 모든 Collection 타입에 적용되어 있다.
  - 위의 예시의 경우 a, b 두 배열은 b의 값이 바뀌기 전까지는 같은 메모리를 공유한다.

## Array Indexing
- Swift 배열은 isEmpty, count, 그리고 subscripting을 통해 특정 인덱스의 원소에 직접 접근하는 방법 등을 지원한다.
  - subscript 방법을 통해 원소를 얻고자 할 경우, 배열의 크기보다 크지 않은 값을 사용해야 함에 유의!
- Swift 배열을 통해 다양한 작업을 할 수 있다.
```Swift
var a = [1, 2, 3, 4, 5]

for x in array { }

a.dropLast(2) // [1, 2, 3]

for (i, number) in a.enumerated() { }

for (index, element) in zip(a.indices, a) { }

if let idx = a.firstIndex { $0 % 2 == 0 } { }

a.map { $0 * 2 } // [2, 4, 6, 8, 10]

a.filter { $0 % 2 == 1 } // [1, 3, 5]
```
- 위의 여러 방법들을 보면, 전통적 C 스타일로 인덱스를 직접 활용해서 뒤지는 것은 Swift 스타일은 아님을 알 수 있다.
- 하지만 가끔은 그런 작업도 필요한 법..! 
- 이런 경우 Swift 차원에서는 조회하기 위해 넣은 index 번호가 유효한지 딱히 검증하진 않는다.
  - 만약 배열의 크기보다 index 번호가 크다면? "Index out of range" (펑)
```Swift
var a = [1, 2, 3]

// first와 last는 optional을 반환한다. 빈 배열일 수도 있으니까!
a.first // Optional(1)
a.last // Optional(3)

// removeLast 메서드는 빈 배열에서 불러 오면 프로그램 크래쉬를 발생시킨다
a.removeLast() // 3

// popLast 메서드는 빈 배열에서 불러 오면 nil을 반환한다
a.popLast() // Optional(2)
```

## Transforming Arrays

### map
- 배열에 있는 모든 값을 변화시키는 작업은 흔히 수행하게 된다.
  - 새로운 배열을 만들고, 기존 배열의 모든 원소를 순회하면서, 특정한 동작을 취하고, 그 결과를 새로운 배열에 삽입하는 동작.
- `map`은 함수형 프로그래밍의 세계에서 온 함수!
```Swift
let fibo = [0, 1, 1, 2, 3, 5]

// map을 쓰지 않는다면
var squared = [Int]()
for number in fibo {
  squared.append(number * number)
}

// map을 쓴다면
let mapped = fibo.map { $0 * $0 }
```
- `map`을 씀으로써 얻는 몇 가지 장점이 있다.
  - 짧아서 읽기 좋다(...). 근데 무시할 수 없는 장점
  - 짧다 보니 에러가 날 만 한 구석도 적다.
  - 무엇보다, 명확하다! `map` 함수를 사용해 작성된 코드를 보면 무슨 일이 일어나는지 직관적으로 알 수 있다.
  - 결과 배열을 `var`가 아닌 `let`으로 선언할 수 있게 된다. 이에 따른 장점이 당연히 있다.
  - 결과 배열의 타입을 엄밀히 정해줄 필요도 없다. 기존의 컨테이너의 타입을 따를 테니까.
- Swift 안에서 `map` 함수의 정의는 `Sequence` 프로토콜 내부에 이런 원리로 되어 있다.
```Swift
extension Array {
  func map<T>(_ transform: (Element) -> T) -> [T] {
    var result: [T] = []
    
    // 배열을 위한 저장 공간을 미리 마련해두는 것은 성능상 도움이 된다
    result.reserveCapacity(count)
    
    // 사실 for문이다
    for x in self {
      result.append(transform(x))
    }
    
    // 물론 실제 구현에서는 transform 함수가 element를 넣었을 때 에러가 날 수 있다
    // 그래서 throws, rethrows로 표기한다
    // Collections 챕터에서 마저 다룰 예정
    return result
  }
}
```

### Parameterizing Behavior with Functions
- `map`이 이렇게 범용성 있게 쓰이면서도 유용한 이유는 무엇일까?
- `map` 함수로 하여금 보일러플레이트를 걷어낼 수 있다는 데에 의미가 있다.
- Parameterizing! 한글로 말하면 매개변수화 정도 된다.
  - `map` 함수에는 각각의 원소에 취할 변환 동작을 "매개변수"로서 삽입해서 보일러플레이트를 걷어 냈다.
- 매개변수화는 `Array`의 수많은 빌트-인 메서드에서 쓰이고, 물론 이외의 다른 함수에서도 많이 쓰인다.
```Swift
// 원소의 변환
map, flatMap

// 일부 원소만 포함
filter

// 모든 원소에 대해 조건에 맞는지 테스트
allSatisfy

// 원소들을 하나의 값으로 "fold"
reduce

// 각각의 원소 순회
forEach

// 원소 재배치
sort(by:), sorted(by:), lexicographicallyPrecedes(by:), partition(by:)

// 있나요? 없나요?
firstIndex(where:), lastIndex(where:), first(where:), last(where:), contains(where:)

// 최소 최대
min(by:), max(by:)

// 다른 배열과 원소 비교하기
elementsEqual(_:by:), starts(with:by:)

// 배열 여러 개로 쪼개기
split(whereSeparator:)

// 조건이 true인 동안 앞에서부터 몇 개의 원소를 가져오기
prefix(while:)

// 조건이 true인 동안 앞에서부터 몇 개의 원소를 버리기
drop(while:)

// 조건에 맞는 원소 모두 없애기
removeAll(where:)
```
- 이 모든 함수의 목적은, 코드의 잡다한 부분을 없애는 데 있다.
  - 결과를 담기 위한 새 배열 만들기라던지, `for`문이라던지, ...
  - 대부분의 이러한 함수들에서는 기본값을 제공하는데, 이는 "자연스러운" 방향으로 작동한다.
    - `sort()`만 사용하면 오름차순 정렬이 되는 것.
  - 매개변수화 자체가 한 눈에 어떤 일이 일어나는지 파악하는 데 의의가 있다 보니 더 그런지도.
- cf. Combine에서는 `Publisher`를 시간에 따른 배열이라고 보다 보니, 이처럼 배열에 쓰이는 메서드를 대부분 쓸 수 있다.
- 한편 직접 정의해볼 만 한 몇 가지 키워드가 있는데, 도전해보는 것은 어떨까?
  - `accumulate`: `reduce`와 유사하지만, 중간중간의 부분합을 포함시킨 배열을 반환하는 것.
  - `count(where:)`: 지정한 조건을 만족하는 원소의 갯수를 반환하는 것.
    - Swift 5.0에서 나왔어야 했다만 이름이 겹치는 이슈가 있었다고 한다
  - `indices(where:)`: 지정한 조건을 만족하는 원소의 인덱스를 반환하는 것. `firstIndex(where:)`와 유사하지만 1번에서 멈추지 않는다.
- 같거나 비슷한 동작을 배열 안에서 여러 번 반복하는 경우가 코드 안에서 발생한다면, `Array`의 `extension`을 짜서 해결해 보는 건 어떨까?

### Mutation and Stateful Closures
- `map`을 통해 변환을 거치는 과정에서, 매개변수로 전달한 함수 안에서 다른 함수를 불러와서 사이드 이펙트를 시전하는 것도 가능은 하다.
  - 하지만 권장하지 않는 방법! 마치 배열의 변환처럼 보이지만 사실 그냥 사이드 이펙트기 때문.
```Swift
var a = [1, 2, 3]

a.map { item in
  // 원소의 변환으로 둔갑한 사이드 이펙트
  db.insert(item)
  
  // 실제 원소의 변환
  return item * item
}
```
- 이럴 거면 `for`문을 쓰는 게 차라리 낫다는 거다. 혹은 `forEach`도 괜찮겠고.
- 클로저는 자기 스코프 밖의 변수를 캡처할 수 있다.
- 그렇기 때문에 "고차 함수"를 활용해보자는 것! 로컬 변수마저도 함수 안에 넣어 보자.
```Swift
extension Array {
  func accumulate<Result>(_ initial: Result, _ nextPartial: (Result, Element) -> Result) -> [Result] {
    // Stateful한 클로저를 위해 임시 저장 공간을 할당한다
    var running = initial
    
    // 여기에 등장하는 map은 Stateful하다
    return map { next in 
      running = nextPartial(running, next)
      return running
    }
  }
}
```

### filter
- 많이 하는 작업이 또, 특정 조건을 만족하는 배열의 값들만을 가지고 있는 새로운 배열을 만드는 것.
  - 변수 배열 생성하고, `for`문 `if`문 돌려서 알아내기..? 끔찍하죠
```Swift
var a = [1, 2, 3, 4, 5]

// 흔히 이렇게 클로저 내부를 축약하긴 하지만
// 동작이 복잡할수록 파라미터 이름 같은 걸 다 명시해주는 것이 좋다.
var b = a.filter { $0 % 2 == 0 }
print(b) // [2, 4]

// 축약 표현의 이점을 극대화하는 사례
(1..<10).map { $0 * $0 }.filter { $0 % 2 == 0 } // [4, 16, 36, 64]

// 단, 위와 같은 사례는 배열이 매우 클 경우 퍼포먼스 문제를 야기할 수 있다.
// 어차피 함수 안에서 계산이 끝나고 버려지는 배열을 위해 추가 allocation을 하는 것이기 때문.
// lazy sequence를 사용해볼 수 있다.
(1..<10).lazy.map { $0 * $0 }.filter { $0 % 2 == 0 }

// 조건에 맞는 값이 있는지 확인만 하고 싶다면, contains(where:) 함수가 훨씬 효율적이다.
// 새 배열을 만들지도 않고 탈출도 빠르기 때문.
(1..<10).lazy.map { $0 * $0 }.contains { $0 % 2 == 0 } // true
```

### reduce
- `map`과 `filter` 모두 새로운 컨테이너를 만든다만, 하나의 값으로 그냥 뭉치고 싶을 때가 있다.
- `reduce`는 기본값과 동작에 대한 정보를 파라미터로 받아서 하나로 뭉쳐진 값을 출력한다.
- `reduce`의 반환 타입은 배열의 원소의 타입과 같지 않아도 된다!
```Swift
let a = [1, 2, 3, 4] // [Int]
let result = a.reduce("") { str, number in
  return str + "\(number)"
}

print(result) // "1234" - String
```
- `reduce`의 구현은 이런 식이다.
```Swift
extension Array {
  func reduce<Result>(_ initial: Result, nextPartial: (Result, Element) -> Result) -> Result {
    var result = initial
    
    for x in self {
      result = nextPartial(result, x)
    }
    
    return result
  }
}
```
- `reduce`만을 활용해 `map`과 `filter`를 구현하는 것도 가능하다! 다만 매번 새로 배열을 만들다 보니 시간복잡도가 O(n^2)가 되어 버린다.
  - 하지만 중간 결과를 저장하는 `reduce`를 `inout` 버전으로 사용한다면, 매번 새로 배열을 만들지 않으므로 O(n)의 시간복잡도가 가능하다.
```Swift
extension Array {
  func filterTwo(_ isIncluded: (Element) -> Bool) -> [Element] {
    return reduce([]) {
      isIncluded($1) ? $0 + [$1] : $0
    }
  }
  
  // Swift에 이미 정의되어 있는 함수
  public func reduce<Result>(into initialResult: Result,
                             _ updateAccumulatingResult: (_ partialResult: inout Result, Element) throws -> ()
  ) rethrows: Result {
    // ...
  }
  
  // 이걸 활용해 보면
  func filterThree(_ isIncluded: (Element) -> Bool) -> [Element] {
    return reduce(into: []) { result, element in
      if isIncluded(element) {
        result.append(element)
      }
    }
  }
}
```

### A Flattening map
- 배열을 반환하는 변환 함수를 사용해 `map`을 하고 싶을 경우도 있다.
- `flatMap` 함수는 mapping & flattening 두 가지 동작을 수행한다. 한 번에.
- 구현은 `map`과 거의 유사한데, 결과가 T 말고 [T]로 나온다는 것이 차이점이다.
  - 내부적으로는 `append` 대신 `append(contentsOf:)` 메서드를 사용한다. 이걸로 차원을 낮추는 거구만~
```Swift
// 위에 구현해 놓은 야매..? 간단한 버전 map과 비교해보자
extension Array {
  func flatMap<T>(_ transform: (Element) -> [T]) -> [T] {
    var result: [T] = []
    
    for x in self {
      result.append(contentsOf: transform(x))
    }

    return result
  }
}
```
- `flatMap`의 좋은 활용법은 또한 다른 배열에 있는 원소들과 합칠 때 드러난다.
```Swift
let files = ["a", "b", "c"]
let ranks = [1, 2, 3]

let result = files.flatMap { file in
  ranks.map { rank in 
    (file, rank)
  }
}

print(result) // [("a", 1), ("a", 2), ("a", 3), ("b", 1), ("b", 2), ("b", 3), ("c", 1), ("c", 2), ("c", 3)]
```

### Iteration Using ForEach
- `for`문과 거의 비슷한 역할을 수행한다.
- `map`과는 달리, 리턴값이 없다. 사이드 이펙트 수행에 특화된 함수!
- 컬렉션 안에 있는 모든 원소마다 한 번씩 간단한 함수를 불러올 때 사용이 용이하다.
  - `view`에 `addSubview` 메서드를 사용해 자식 뷰들을 추가할 때라던지!
- 다만 `forEach` 클로저 안에서 `return`하는 것은 클로저에서 벗어나는 것일 뿐, 그것이 쓰인 함수의 반환값으로 리턴을 하는 것이 아니다.
```Swift
func findOne(_ nums: [Int]) -> Int? {
  for i in 0..<nums.count {
    if nums[i] == 1 {
      // 이 경우의 return은 findOne 함수의 return 값으로 사용된다
      return i
    }
  }
  
  return nil
}

func findOne(_ numbers: [Int]) -> Int? {
  numbers.enumerated().forEach {
    if $0.1 == 1 {
      // 클로저 안에서만 벗어날 뿐 아무 일도 일어나지 않는다.
      // 에러도 뜰걸?
      return $0[0]
    }
  }
  
  return nil
}
```
- `return`이 불분명하게 동작할 가능성이 크므로, 단순한 반복문을 대체하는 것 이외에 추천하기는 힘든 면이 있다.

## Array Slices
- `subscript`를 활용해 배열의 일부분만 뜯어올 수도 있다.
```Swift
let a = [1, 2, 3, 4, 5]
let sliced = a[1...]

// 함정
print(type(of: sliced)) // ArraySlice<Int>
```
- `Array`가 아니라 `ArraySlice`라니! 말도 안 돼!!
- 데이터베이스 식으로 말하자면, `ArraySlice`는 `Array`의 View.. 라고 할 수 있다.
  - 그렇기 때문에 `ArraySlice`를 만드는 데에는 비용이 크게 들지 않는다! 원소를 직접 복사하지 않기 때문.
- `ArraySlice`는 `Array`에서 쓰이는 메서드를 대부분 쓸 수 있다. `Collection` 프로토콜의 힘이랄까?
- 그냥 `Array` 붙이면 배열로 변환되가도 한다 일단.
- `ArraySlice`는 원본 배열에서 쓰이는 인덱스 번호를 사용해야 한다. 그렇기 때문에, 0부터 시작하지 않을 수도 있다.
  - 상기 예시의 경우 시작 인덱스가 1일 것이다.
  - 인덱스를 잘못 넣으면 Index out of range 오류가 뜰 가능성이 다분하다는 말.
- 사실 그렇기 때문에 `startIndex`와 `endIndex` 메서드를 쓰는 것이 안전하고 좋을 지도 모른다.