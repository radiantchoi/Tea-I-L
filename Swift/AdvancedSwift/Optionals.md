# Optionals

## Sentinel Values
- 프로그래밍에서 자주 등장하는, "값이 있을 때 혹은 없을 때"
  - 그냥 파일 따위를 끝까지 읽어서 말 그대로 더 이상 값이 없을 수도 있고
  - 함수 따위를 실행하다가 뭔가 잘못돼서 더 이상 값이 없을 수도 있고
- 여러 언어의 이러한 예시 케이스에서, "값이 없음"을 나타내는 특별한 "값"을 반환함을 알 수 있다.
  - 이러한 값을 Sentinel Value라고 부른다.
- 하지만 문제가 있다. 이러한 값을 통해 값이 없음을 나타낸다면, 마치 유효한 값을 반환한 것 같은 착각이 들지 않는가??
- 이러한 착각의 연장선에서 값을 "사용"하려고 든다면? (끔찍)
- 한편 Sentinel Value 자체를 어떻게 다루려고만 한다 해도, 유의미한 지식이 필요하다.

## Replacing Sentinel Values with Enum
- 이런 문제들을 해결하기 위해 나온 `Optional`!
- 내부적으로는 `enum`으로 이루어져 있다.
  - 정확히는 `@frozen enum`이며, `.some(value)`과 `.none`의 두 가지 케이스가 있다.
- 아래는 `firstIndex(of:)` 메서드의 예시.
```Swift
extension Collection where Element: Equatable {
  func firstIndex(of element: Element) -> Optional<Index> {
    var idx = startIndex
    
    while idx != endIndex {
      if self[idx] == element { 
        return .some(idx) 
      }
      
      formIndex(after: &idx)
     }
    
    // Not found, return .none.
    return .none
  }
}
```
- 한편 `?` 기호를 붙임으로써 옵셔널을 나타낼 수도 있다.
  - 이는 옵셔널 열거형이 `ExpressibleByNilLiteral` 프로토콜을 채택하기 때문이다.
  - 이 덕분에 위 예시에서 `return .some(idx)` 대신 `return idx`를 사용할 수 있다.
- 이제 사용자는 실수로 Sentinel Value를 직접 사용할 수 없게 됐다..! 대신 unwrap 동작을 수행할 것이 강제된다.

## A tour of Optional Technics
- `Optional`은 언어 기본 기능이기 때문에, 이를 서포트하기 위한 여러 문법이 있다.

### `if let`
- 옵셔널 바인딩이라고도 한다.
- 옵셔널 값이 `nil`인지 아닌지 확인하고, 아니라면 언래핑 후 다음 과정을 진행한다.
```Swift
var numbers = [1, 2, 3]

if let index = numbers.firstIndex(of: 2) {
  numbers.remove(at: index)
}
```
- `if let`과 함께 `Bool` 타입의 무언가를 엮어서 조건을 더할 수 있다.
- 같은 구문 안에 여러 개의 값을 바인딩할 수도 있다.
  - 이 경우 나중에 등장하는 값이 먼저 등장한 언래핑된 값에 의존한다.

### `while let`
- `if let`과 비슷하다. 그런데 `nil`이 되는 순간 루프를 해제하는
```Swift 
// readLine() 함수는 한 줄씩 읽어서 값을 반환하고, 문서의 끝에서 nil을 반환한다
while let inputString = readLine() {
  print(inputString)
}
```
- 또한 마찬가지로 `Bool` 조건을 더해줄 수 있다. 이를 통해 filter처럼 사용해볼 수도 있다.
```Swift
// 아래의 두 코드는 같은 동작을 한다.
for i in (0..<10) where i % 2 == 0 {
  print(i)
}

var iterator = (0..<10).makeIterator()
while let i = iterator.next(),
      i % 2 == 0 {
  print(i)
}
```

### Doubly-Nested Optionals
- `Optional`이 감싸는 타입 그 자체도 `Optional`이 될 수 있다..?!
```Swift
var wannaBeInts = ["1", "2", "three"]
var mappedInts = wannaBeInts.map { Int($0) }
 
var iterators = mappedInts.iterator()
while let number = iterators.next() {
  print(number, terminator: ", ")
}
// Optional(1), Optional(2), nil,
// nested optional이 없다면 nil도 값으로 쳤을 것이다.
// .some(nil)

// 한편 패턴 매칭을 사용할 수도 있다 - enum 챕터에서 자세히 설명
for case let i? in mappedInts {
  // i는 여기서 Int? 가 아닌 Int가 된다. 
  print(i)
}
// 여담이지만 case 패턴 매칭은 for, if, while 등에서 다 사용할 수 있다.
```

### `if var`, `while var`
- `if let, while let`이 있으면 `var`도 당연히 있겠제!
- 다만 이를 통해 바인딩 및 할당되는 변수는 로컬 변수이다. 즉, 원본 값에 변경사항이 반영되지 않는다.
  - 옵셔널은 값 타입이기 때문!

### Scoping of Unwrapped Optionals
- 위에서 살짝 언급했듯, 바인딩을 통해 옵셔널을 언래핑하면 해당 스코프 안에서만 효력이 있다.
- 그런데 가끔 스코프 밖에서 이 값을 쓰고 싶을 때도 있기 마련이다.
- 이 경우 변수/상수가 들어갈 자리를 먼저 마련해 두고, 이후에 옵셔널 바인딩을 통해 자리에 값을 채워넣는 방법이 있다.
  - 이 경우 컴파일러가 코드를 체크하여, 값을 사용할 거라면 반드시 할당하고, 그렇지 않으면 빠르게 반환하게 한다.
```Swift
extension String {
  var fileExtension: String? {
    // 먼저 선언함으로써 자리를 비워 둔다
    // 이 경우 당연히 값은 한 번만 할당될 수 있다 - let이기 때문
    let period: String.Index
    
    if let idx = lastIndex(of: ".") {
      period = idx
    } else {
      return nil
    }
    
    let extensionStartingPoint = index(after: period)
    return String(self[extensionStartingPoint...]
  }
}
```
- 빠르게 반환? 우리는 이 상황에서 쓰이는 구문을 이미 알고 있다. 바로 `guard`.
```Swift
extension String {
  var fileExtension: String? {
    guard let period = lastIndex(of: ".") else {
      return nil
    }
    
    let extensionStartingPoint = index(after: period)
    return String(self[extensionStartingPoint...]
  }
}
```
- 잘 알다시피 `guard`를 사용하려면, `else`의 경우에 무조건 이 스코프를 빠져나갈 수만 있으면 된다.
- 약간 참고사항인데, `Never`를 반환하는 것에 관한 이야기가 있다.
  - 주로 프로그램을 멈춰야 하거나 계속 돌아가는 프로그램일 때 이를 리턴한다.
  - `fatalError` 를 발생시키는 함수라면 `Never`로 리턴되는 타입이다.
- Swift 하에서 `Never`를 위시한 아래 각각의 표현이 지니는 의미를 보면 다음과 같다.
  - `nil`: 무언가가 없음
  - `Void`: 아무것도 없는 것이 있음
  - `Never`: 있어서는 안 될 것
