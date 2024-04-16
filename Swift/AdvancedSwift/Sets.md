# Sets
- `Set`은 각각의 원소가 한 번씩만 등장하며 정해진 순서가 없는 컬렉션이다.
- 거칠게 요약하면, 딕셔너리지만 값 없이 키만 있는 형태라고 볼 수도 있다.
- 근데 이거 진짜임. 내부적으로는 해시 테이블로 구현되어 있다.
  - 그래서 `Set`의 원소는 반드시 `Hashable` 프로토콜에 부합해야 한다.
- 각각의 원소에 대해 테스트를 수행할 때 퍼포먼스가 중요하다면, 그리고 순서가 중요하지 않다면, 혹은 중복이 없어야 한다면, `Set`의 사용을 고려해 보자.
- 한편 `Set`은 `ExpressibleByArrayLiteral` 프로토콜을 채택한다.
  - 이게 무슨 말이냐면, 배열 리터럴을 활용해 선언할 수 있다는 이야기이다.
```Swift
let nums: Set = [1, 2, 3, 2]

// 이 와중에 Set이라서 중복 원소가 하나 날아갔다
print(nums) // Set([2, 1, 3])  
```
- `Set` 역시 다른 컬렉션에서 사용할 수 있는 많은 메서드를 쓸 수 있다.
  - `for` 문을 활용한 순회, `map`, `filter`, ...

## Set Algebra
- 이름에서 알 수 있듯, `Set`은 수학의 집합과 유사한 부분이 많다.
- 집합에서 사용할 수 있는 공통 연산을 모두 지원한다.
```Swift
let firstSet: Set = [1, 2, 3]
let secondSet: Set = [2, 3, 4]

// 교집합
let intersectedSet = firstSet.intersection(secondSet) // [2, 3]

// 합집합
// formUnion은 mutating 메서드다.
var thirdSet: Set = [3, 4, 5]
thirdSet.formUnion(secondSet) // [2, 3, 4, 5]

// 차집합
let subtractedSet = firstSet.subtracting(secondSet) // [1]
```
- 합집합 연산 같은 일부 경우를 제외하면, 대부분의 집합 연산은 `mutating` 버전이 있고 아닌 버전이 있다.
- `SetAlgebra`[프로토콜](https://developer.apple.com/documentation/swift/setalgebra)도 체크해보자!

## Using Sets inside Closures
- 딕셔너리와 집합 모두 함수 안에서 쓰기 좋은 자료구조이다.
- 예를 들면 중복 원소 걸러내기 작업은 사실 집합에 넣었다 빼면 자동으로 수행되는 작업이다.
  - 하지만 불안정한 면도 있는데, 결국 순서가 무시되기 때문이다.
  - 이를 해결하기 위해 순서는 보장해 주는 extension 하나 작성해 보자.
```Swift
extension Sequence where Element: Hashable {
  func unique() -> [Element] {
    var seen: Set<Element> = []
    
    return filter { element in 
      if seen.contains(element) {
        return false
      } else {
        seen.insert(element)
        return true
      }
    }
  }
}
```
- 이 방식은 전체 원소가 `Hashable`해야 한다는 제한 사항이 있긴 하다.
- `filter` 클로저 안에서 클로저 외부에 집합을 정의하고, 그 "상태"를 참조해 가면서 연산한다.
- 결과적으로 배열의 순서를 해치지 않으면서도 중복 원소를 걸러낼 수 있다.