# Ranges
- 일정 간격을 가지고 있는 값들의 모음이며, 아래쪽 그리고 위쪽 경계를 설정해 줌으로써 정의된다.
- closed range, half-opened range 두 가지가 있다.
```Swift
// half-opened range
let singleDigitNumbers = 0..<10
print(Array(singleDigitNumbers)) // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

// closed range
let lowercaseAlphabets = Character(a)...Character(z) // a부터 z까지 아무튼 나옵니다!
```
- one-sided range 개념도 있다.
```Swift 
let fromZero = 0...
let smallerThanTen = ..<10
```
- 이와 같이 다양한 범위를 표현하기 위해 총 5가지의 각기 다른 range 타입이 있으며, 또한 각기 다른 제한 사항을 가진다.
  - 예를 들어 `Range` 타입은 half-open range(..<)를 나타낸다.
  - `ClosedRange` 타입은 말 그대로 (...) 표현을 사용해 나타내는 범위.
  - 두 가지 타입 모두 파라미터로 `Bound` 제네릭을 받는다. `Bound`는 일단 `Comparable`하기만 하면 된다.
  - [Comparable 문서](https://developer.apple.com/documentation/swift/comparable)
- 범위를 쓰는 경우? 원하는 값이 범위 내에 있는지 체크하는 것이 가장 많이 쓰이는 예시.
```Swift
singleDigitNumbers.contains(3) // true
lowercaseAlphabets.overlap("c"..<"f") // true
```
- 상술한 두 가지 범위 타입이 분리된 이유는, 각각의 타입만이 유일하게 할 수 있는 일이 있기 때문이다.
  - 예를 들어 비어 있는 범위는 half-open range 안에서 lower bound, upper bound 둘이 동일한 범위여야만 표현할 수 있다.
  - 한편 자료형에서 제공하는 max 값은 closed range 안에서만 표현할 수 있다.
    - half-open 범위로 표현하려면 범위 내의 upper bound 값보다 더 큰 값이 있어야 한다.
```Swift
// 포함하는 게 아무것도 없으려면
let emptyRange = 5..<5

// Int.max + 1 같은 값은 없으니까
let maximumRange = 0...Int.max
```

## Countable Ranges
- 물론, 범위도 시퀀스/컬렉션이기 때문에, 웬만하면 범위 내를 `for`문으로 순회하거나 컬렉션처럼 취급하는 것도 가능하다.
- 하지만 왜 웬만하면이겠는가? 이게 안 되는 범위 타입도 있기 마련!
  - 범위를 나타낼 때는 `Comparable`한 `Bound`만 있으면 되지만, 순회하려면 `Stridable` 타입이 필요하다.
  - `Character`의 경우 `Stridable`하지 않다.
  - [Stridable 문서](https://developer.apple.com/documentation/swift/strideable)
  - 사실 이거는, 유니코드 문자 사이를 순회하는 것이 보이는 것만큼 직관적이지 않아서 그렇다. 추후 String 챕터에서 알아보자.
- 다시 말하면, `Range`는 `Stridable`할 때만 제한적으로 `Collection` 프로토콜에 부합한다.
- 범위 타입 안에서 순회하고 싶다면 그 원소가 "셀 수 있는" 것이어야 한다.
  - 이를 만족하는 것으로는 정수나 (가변 포인터가 아닌) 포인터 타입이 있다.
- 이를 제공하기 위해!! `CountableRange` 타입과 `CountableClosedRange` 타입이 제공된다!!
  - 실상은 바운드가 `Stridable`한 `Range`의 alias.

## Partial Ranges
- 한 쪽 바운드가 없기 때문에 Partial Range라고 표현한다.
- 세 가지 타입이 있다.
```Swift
let fromOne: PartialRangeFrom<Int> = 1...
let throughTen: PartialRangeThrough<Int> = ...10
let upto10: PartialRangeUpTo<Int> = ..<10
```
- 마찬가지로 `CountablePartialRangeFrom`과 같은, 바운드가 `Stridable`함을 나타내는 alias 타입이 있다.
  - 다만 제한사항이 더 빡세다.
  - 순회는 `advanced(by: 1)` 함수를 내부에서 지속적으로 호출함으로써 이루어진다.
  - 부분 범위의 경우 break 조건을 그래서 잘 정해 주어야 한다.
  - 아래쪽 바운드가 없는 범위 타입의 경우, `relative(to:)` 메서드를 통해 내부적으로 아래 바운드를 정해서 시작점으로 삼는다.
  - 위쪽 바운드가 없는 범위 타입의 경우, `endIndex` 프로퍼티를 통해 내부적으로 위 바운드를 정한다.
- 부분 범위 타입이 강력할 때는 역시 슬라이싱할 때.
```Swift 
let numbers = [1, 2, 3, 4, 5]

numbers[1...] // [2, 3, 4, 5]
numbers[..<3] // [1, 2, 3]
numbers[2...4] // [3, 4, 5]

// 심지어 양쪽 바운드를 모두 생략해서 컬렉션 전체를 불러올 수도 있다.
// 유효한 Range 표현은 아니지만, stdlib에 특별한 케이스로 정의되어 있다.
numbers[...] // [1, 2, 3, 4, 5]
```
- 이와 같은 코드가 가능한 이유는, `Collection` 프로토콜의 `subscript` 메서드가 정해진 범위 타입이 아닌 `RangeExpression`을 받기 때문.
- 표준 라이브러리의 접근법을 모사하여 `RangeExpression`을 받는 나만의 함수를 만들어보는 것도 좋다.
  - 항상 되는 것은 아닌 게, 컬렉션이 아니라면 바운드를 정할 수 없기 때문에..

## RangeSet
- 같은 원소 타입을 가지고 있는 범위 타입의 나열이다.
- 컬렉션의 인덱스 중 연속되지 않은 값을 가지고 작업할 때 좀 더 쉽고 효율적으로 작업할 수 있다.
  - 물론 `Set<Index>` 타입을 써도 되긴 하지만, 좀 더 내부적으로 메모리 효율성이 좋다.
  - 예를 들어 원소가 1000개인 집합이 있고, 이 중 유저가 선택한 부분만 커버하고 싶다면?
    - 최대 1000개의 인덱스를 메모리에 올릴 것인지, 범위별로 2개씩의 인덱스만 메모리에 올릴 것인지의 차이.
```Swift
var indices = RangeSet(1..<5)
indices.insert(contentsOf: 11..<15)

// ranges 프로퍼티가 따로 있다.
Array(indices.ranges)
```
- 삽입 순서에 상관없이 오름차순으로 정렬되어서 나오며, 겹치거나 인접하지 않는다.
- 한편 `RangeSet`은 `SetAlgebra`를 만족하지 않는다.
  - 하지만 각종 집합 연산을 수행할 때는 일부 `SetAlgebra` 동작을 빌려오기도 한다.
- 사실 `RangeSet` 타입은 Swift 5.6 기준 표준 라이브러리에 들어있지는 않고, Swift Evolution SE-0270 이슈에 들어가 있다.