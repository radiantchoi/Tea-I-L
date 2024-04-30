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
- `Never`는 `case`가 없는 열거형의 형태로 구현되어 있다.

## Optional Chaining
- Obj-C 상에서는 `nil`인 것에 메시지를 보내는 것은 작동하지 않는다. Swift 상에서는 어떨까?
- 옵셔널 체이닝을 통해 같은 효과를 누릴 수 있다!
```Swift
// mark의 뇌가 있다면 그 무게를 출력
print(mark.brain?.weight) // Optional(1320.0)
```
- ? 기호가 붙어 있는 것은 코드를 읽는 사람으로 하여금 이 코드가 작동하지 않을 수 있다는 확실한 메시지를 준다.
- 이 코드의 결과는 옵셔널로 출력된다.
- 또한 옵셔널 "체이닝"이기 때문에, 여러 번 걸어버릴 수도 있다.
```Swift
home?.guardian?.brain?.weight 
```
- 위 코드의 결과는 `Optional(Optional(Optional(Double)))`이 될 텐데, 이게 맞을까?
- 옵셔널 체이닝은 flattening 동작을 수행한다.
- 한편 옵셔널 체이닝은 subscript 과정에서도 적용될 수 있다.
```Swift
let users = ["Steam": [1, 2, 3, 4]-
print(users["Steam"]?[2]) // Optional(3)

// 당연히 옵셔널 함수를 사용할 때도 가능하다
// 참고: 연산자를 함수로 취급할 땐 이렇게 써 줄 수도 있다는 거!
let functions = ["add": (+), "subtract": (-)]
print(functions["add"]?(1, 3) // Optional(4)
```
- 상기한 활용법은, 클래스가 이벤트 발생 시 콜백 함수의 결과값을 주인 객체에게 전달할 때 유용하다.
```Swift
// 이건 진짜 텍스트 필드에 있는 함수
class TextField {
  private(set) var text = ""
  var didChange: ((String) -> ())?
  
  // 프레임워크 차원에서 호출한다
  func textDidChange(newText: String) {
    text = newText
    didChange?(text)
  }
}

class Box<T> {
  private(set) var value: T
  var listener: ((T) -> ())?
  
  init(value: T) {
    self.value = value
  }
  
  func subscribe(newValue: T) {
    value = newValue
    listener?(value)
  }
}
```
- 상기 코드의 `textDidChange`, `changeValue` 함수는 안에 콜백 함수를 가지고 있다. 또한 값이 바뀔 때마다 불려 오기도 하고.
- 그런데 유저는 이 콜백을 초기화해 줄 필요는 없다. 왜냐면 옵셔널 값이니까, 기본값은 `nil`이기 때문.
  - 옵셔널 `let` 말고 옵셔널 `var`은 따로 할당하지 않아도 `nil`이 할당된다.
  - 옵셔널은 사용의 편의를 위해 "분명한 기본값"이라고 할 수 있는 `nil`이 있는 것이다. 다만 Swift 팀에서는 후회한다고(소근)

### Optional Chaining and Assignments
- 체인이 걸려 있는 옵셔널 값에도 다른 값을 할당할 수 있다!
- 이 경우 `nil`이 아니라면! 값을 할당한다.
  - 일반 프로퍼티는 아묻따 값을 할당하지만, 옵셔널 프로퍼티는 먼저 값이 있나 없나 체크한다는 것이다.
  - `nil`이 뜨면 하던 모든 연산을 집어치운다는 것이 옵셔널 체이닝의 특징이니까.
```Swift
optionalCard?.value += 1
```

## The nil-Coalescing Operator
- 옵셔널을 깔 때, `nil`을 특정한 기본값으로 지정하고 싶을 때 사용하는 오퍼레이터.
```Swift
// 여기서 ??가 바로 Coalescing Operator다.
let message = readLine() ?? "No Message"

// 삼항 연산자랑 비슷하다고 느낄 사람들을 위해
let message = readLine() != nil ? readLine()! : "No Message"
```
- ?? 기호 뒤에 나오는 것이 기본값임을 한 눈에 알 수 있어서 좋다.
- 조금 응용해보자면, 배열의 인덱스로 접근할 때, 배열의 크기를 넘는 인덱스를 조회하는 것은 옵셔널을 반환하지 않지만...
```Swift
let numbers = [1, 2, 3, 4]
let fifth = numbers.count > 5 ? numbers[5] : 0 // 0

extension Array {
  subscript(guarded idx: Int) -> Element? {
    guard (startIndex..<endIndex) ~= idx else {
      return nil
    }
    
    return self[idx]
  }
}

let sixth = numbers[guarded: 6] ?? 0 // 0
```
- Coalescing Operator 역시 체이닝이 가능하다.
```Swift
let a: Int? = nil
let b: Int? = nil
let c: Int? = 34

a ?? b ?? c ?? 0 // 34

// 체이닝과 먼저 까는 것은 구분되어야 한다!!
// 아래 예시는 Optional(Optional(String))의 경우.
let firstString: String?? = nil
let secondString: String?? = .some(nil)

// 첫 번째 String은 inner 지점에서 nil이므로
(firstString ?? "inner") ?? "outer" // inner

// 두 번째 String은 일단 inner 지점에선 some이므로, 언래핑되어 안에 있던 값이 나온다.
// outer 지점으로 나왔을 때 값이 nil이므로, 기본값을 대신 제공한다.
(secondString ?? "inner") ?? "outer" // outer
```

## Using Optionals with String Interpolation
- 옵셔널 값을 프린트하려고 하면 컴파일러가 노랑 오류를 뱉어내는 것을 본 적이 있을 것이다.
  - `Warning: Expression implicity coerced from 'Int?' to Any.`
- 출력 과정에서 값을 `String`으로 바꾸는 "문자열 보간(String Interpolation)"을 하면서 일어나는 현상.
- 이렇게 하면 `nil`이 프린트되거나, `Optional(3)` 따위가 프린트되거나 할 텐데..
- 문제는 유저가 보는 화면에는 "옵셔널" 이런 표현은 치워야 된다는 것이다.
- 치우는 몇 가지 방법이 있긴 하다.
  - `as Any`로 타입캐스팅하기
  - 강제 언래핑하기
  - `String(describing:)` 활용하기
  - 기본값 제공하기: 가장 바람직! 다만, 연산자 좌, 우의 타입이 같아야 한다. Swift 언어는 타입 미스매치를 용납하지 않는다!
    - 그런데 도대체 common type을 찾을 수 없는 경우는 어떻게 해야 할까? 직접 만들어 보자.
```Swift
// 왼쪽 값이 nil이면 오른쪽의 String 기본값을 반환하는 오퍼레이터
infix operator ???: NilCoalescingPrecedence

// @autoclosure를 사용하는 이유는, 여기는 진짜 필요할 때만 불러올거라는 표시다.
// 말하자면 연산을 지연시키는 것.
// Functions 파트에서 자세히 다룬다.
public func ???<T>(optional: T?, defaultValue: @autoclosure () -> String) -> String {
  switch optional {
  case let value?: 
    return String(describing: value)
  case nil:
    return defaultValue()
  }
}
```

## Optional Map
- 옵셔널을 받고, `nil`이 아닌 값을 변환한다~
- 옵셔널에도 `map` 함수가 있어서 이런 역할을 수행한다. `Sequences`에 있는 그것과 굉장히 유사하다.
- 하지만! 옵셔널을 대상으로 하는 맵 함수는 단 하나의 값에서만 작동한다.
```Swift
let message = "Hello World!"
let firstCharacter = message.first.map { String($0) } // Optional("H")
```
- 결과를 옵셔널로 원할 때 유용하다.
```Swift 
extension Array {
  func reduce(_ nextPartialResult: (Element, Element) -> Element) -> Element? {
    guard let f = first else { return nil }
    
    return dropFirst().reduce(f, nextPartialResult)
  }
}

[1, 2, 3, 4].reduce(+) // Optional(10)
```

## Optional FlatMap
- 배열에 맵을 하면서, 변환 함수는 배열을 반환하지만, 진짜 결과는 배열의 배열이 아닌 그냥 배열로 돌려받고 싶을 때가 항상 있다.
- 옵셔널에서도, 맵을 하고 싶은데 변환 함수가 옵셔널이라면 `Optional(Optional(Element))` 타입이 나오는 꼴이 보기 싫을 수도.
- `flatMap`을 사용하면 한 번만 옵셔널로 감싸진 값을 반환한다!
- 그런데 사실 이러한 연산은 여러 대체재가 있다.
  - 앞에서 알아본 `if let`으로 여러 개의 변수/상수 만들기
  - 옵셔널 체이닝
```Swift
let urlString = "https://objc.io/logo.png"
let view = URL(string: urlString)
  .flatMap { try? Data(contentsOf: $0) }
  .flatMap { UIImage(data: $0) }
  .map { UIImageView(image: $0) }

if let view = view {
  PlaygroundPage.current.liveView = view
}

// 체이닝 및 if let과 유사한 대목
i?.advance(by: 1)
i.flatMap { $0.advance(by: 1) }
if let i {
  i.advance(by: 1)
}
```

## Filtering out nils with compactMap
- 만약 컬렉션 안에 `nil`이 있는데, 신경쓰기 싫다면..!
```Swift
let probablyNumbers = ["1", "2", "three", "4"]

// compactMap을 모르는 우리는 이렇게 했겠지만
var gross = 0
for case let i? in numbers.map { Int($0) } {
    gross += i
}
print(gross) // 7

// 또는 이렇게 했겠지만
let coalescingSum = probablyNumbers.map { Int($0) }.reduce(0) { $0 + ($1 ?? 0) }
print(coalescingSum) // 7

// 이제는 이렇게 할 수 있다
let compactSum = probablyNumbers.compactMap { Int($0) }.reduce(0, +)
print(sum) // 7
```
- `compactMap`을 직접 구현해볼 수 있다. 예시에서는 먼저 배열을 순회한 다음 `nil`이 아닌 것만 반환한다.
  - 이 때 퍼포먼스를 위해 `lazy`하게 배열을 생성한다.
```Swift
extension Sequence {
  func compactMap<B>(_ transform: (Element) -> B?) -> [B] {
    return lazy.map(transform).filter { $0 != nil }.map { $0! }
  }
}  
```
- 뭐 원래 Collection stdlib 안에서는 이런 짓을 안 하긴 하지만.

## Equating Optionals
- 값이 `nil`이든 아니든 신경 안 쓰는데, 아무튼 같은지 아닌지 해야 할 경우.
- `Optional`은 `Equatable`하다! 단, 감싸고 있는 값도 `Equatable`할 경우에만.
```Swift
extension Equatable where Wrapped: Equatable {
  static func ==(lhs: Wrapped?, rhs: Wrapped?) -> Bool {
    switch lhs, rhs {
    case (nil, nil): return true
    case let (x?, y?): return x == y
    case (_?, nil), (nil, _?): return false
    }
  }
}
```
- `Optional`을 비교할 땐 위와 같은 네 가지 케이스가 있다. 각각에 대해 처리해 주면 된다.
- 한편 옵셔널이 아닌 값과 옵셔널을 비교하고자 한다면, 옵셔널이 아닌 값을 자동으로 옵셔널로 감싸서 타입 매칭을 한다.
  - 이게 아니었으면 위의 네 가지 케이스를 전부 따로따로 구현해야 했을걸?
- Swift 코드는 전반적으로 이러한 암시적 변환에 의존한다..!
- 한편 딕셔너리를 사용하면서 이런 옵셔널 값을 사용하는 것은 조심해야 할 수도 있다!
  - 딕셔너리에서 `value` 부분에 `nil`이 들어간다는 것은 해당 키를 삭제한다는 것과 같기 때문.
```Swift 
var exampleDict: [String: Int?] = ["one": 1, "two": 2]

// 이렇게 하면 키 two가 없어진다
exampleDict["two"] = nil

// 구태여 nil값을 넣고 싶다면 이렇게 하자
exampleDict["two"] = .some(nil)
exampleDict["two"] = Optional(nil)
exampleDict["two"]? = nil
```

## Comparing Optionals
- `Equatable`하니까.. 당연히 비교도 할 것.
- `nil`이랑 값이 있는 경우를 비교했을 때만 짚고 넘어가자면! `.some`이 더 큰 값으로 판정한다.

# When to Force-Unwrap
- 상술한 모든 방법은 옵셔널을 깔끔하게 언래핑하기 위한 과정이다.
- 그럼 강제 언래핑은 언제 해야 할까? 여러 의견이 있지만, 이 책에서 제시하는 의견은 다음과 같다.
  - 값이 `nil`이 아닌 것이 매우 자명하고, 만약 `nil`일 경우 프로그램을 크래쉬시키고 싶을 경우에 사용하면 된다.
- 위에서 정의한 `compactMap` 예시를 가져오면..
```Swift
extension Sequence {
  func compactMap<B>(_ transform: (Element) -> B?) -> [B] {
    return lazy.map(transform).filter { $0 != nil }.map { $0! }
  }
}  
```
- 위 예시에서는 마지막 `map`에 강제 언래핑이 되어 있다.
- 논리적으로, 이전 부분에서 `nil`을 모두 걸러냈기 때문에, 저 부분에서는 값이 없을 수가 없기 때문이다.
- 물론 이런 경우는 매우 드문 경우. 대부분의 경우는 강제 언래핑보다 더 좋은 선택지가 있다.
```Swift
let ages = ["Gordon": 30, "Greg": 28]

// 강제 언래핑을 써도 값이 있음이 보장되는 경우. 딕셔너리의 키 배열은 당연히 해당하는 값이 있다.
let forceNames = ages.keys.filter { name in ages[name]! >= 30 } // ["Gordon"]

// 다만 이 경우에도 강제 언래핑을 쓰지 않을 수 있다.
ages.filter { (_, age) in age >= 30 }
  .map { (name, _) in name }
// [Gordon]
// 딕셔너리는 키와 값의 시퀀스기 때문에, 이런 식으로 나타낼 수 있다!
```
- 뭐 그렇긴 해도, 이론적으로 불가능한 상황을 커버하기 위해 관련 없는 방법을 사용하면서 언래핑하기보다 강제 언래핑을 쓰는 게 좋을 수도 있다는 말.
- "Prevent sweeping theoretically impossible situation under the carpet"

## Improving force-unwrap Error Messages
- 강제 언래핑은 했으되, 왜 프로그램을 크래쉬 시켰는지 메시지를 던져 주면 더 좋을 것 같다.
```Swift
infix operator !!

func !!<T>(wrapped: T?, failureText: @autoclosure () -> String) -> T {
  if let x = wrapped { return x }
  fatalError(failureText())
}
```
- cf. [공식 문서](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/advancedoperators/)
  - 커스텀 오퍼레이터를 정의하고 싶다면 `infix`를 붙여 줘야 한다.
- 실제로 사용해보자!
```Swift
let wannaBeInt = "Two"
let number = Int(wannaBeInt) !! "Expecting Int, got \(wannaBeInt)"
```

## Asserting in Debug builds
- 아무리 그래도 릴리즈 빌드에서 앱을 크래쉬 내는 것은.. 매우 "용감한" 행동이지.
- 그래서 디버그/테스트 빌드에서만 검증하고, 실제 프로덕트에서는 기본값을 제공하고 싶을 수 있다.
- `assert` 메서드는 디버그/테스트 환경에서만 작동해서, 값을 비교하고 다르면 크래쉬시키니까 이를 충족시켜줄 수 있다.
```Swift
infix operator !?

func !?<T: ExpressibleByIntegerLiteral>(wrapped: T?, failureText: @autoclosure () -> String) -> T {
  assert(wrapped != nil, failureText())
  return wrapped ?? 0
}
```
- 위 경우는 Integer Literal 한정으로 쓸 수 있지만, Array Literal 혹은 String Literal 등 여러 가지 오버로딩을 통해 써볼 수 있다.
- 이를 위시해서, 프로그램의 실행을 멈출 수 있는 수단은 크게 세 가지가 있다.
  - `fatalError`: 메시지를 로그에 찍고 바로 프로그램을 멈춰 버린다.
  - `assert`: 특정 상태와 지금 상태를 비교하고, 일치하지 않을 경우(`false`인 경우) 프로그램을 멈춘다. Release 빌드에서 제거된다.
  - `precondition`: `assert`와 같은데, 릴리즈 빌드에서도 제거되지 않는다.

# Implicitly Unwrapped Optionals
- 암시적 언래핑 타입. 대표적으로 `UIKit`과 Storyboard 혹은 xib 사용 시 볼 수 있다.
```Swift
final class MyViewController {
  @IBOutlet private lazy var helloLabel: UILabel!
}
```
- 아니 그럼 어차피 옵셔널이어선 안 되는데, 도대체 왜 이렇게 쓰는 걸까?
- 첫 번째 이유! Obj-C 코드와 연동해야 할 경우 임시적으로 이렇게 할 수 있다. 
  - Obj-C는 옵셔널처럼 null 처리가 세심히 되어 있지 않다.
  - 혹은 Swift 스타일 표현이 먹히지 않는 C 라이브러리에 접근해야 할 경우. 똑같다.
  - Obj-C 라이프 사이클 상에서는 참조가 nullable하다는 것을 나타낼 방법이 없다. 하지만 몇몇 Obj-C API는 null 참조를 반환한다(?!).
  - 당연히 이런 이유기 때문에 순수 Swift API 코드에서는 암시적 언래핑 옵셔널이 나오면 안 된다.
- 두 번째 이유! 아주아주 짧은 시간동안만 값이 `nil`인 경우 이렇게 할 수 있다.
  - 대표적인 예시가 상술한 UIKit View Controller.
  - 뷰 컨트롤러는 그 뷰를 lazy하게 만든다. 
    - loadView 시점과 viewDidLoad 시점 사이 짧은 시간에 nil인 자식 뷰가 있을 수 있다.
    - 심지어 본인의 view 프로퍼티도 nil이다..

## Implicit Optional Behavior
- 암시적 언래핑 옵셔널들이 옵셔널 아닌 것처럼 쓰이긴 하지만, 안전한 핸들링을 위해 지금껏 사용하던 옵셔널 언래핑 기술을 사용할 수 있다.
```Swift
let s: String! = "Hello"
s?.isEmpty // Optional(false)

if let s {
  print(s) // Hello
}

s = nil
s ?? "Goodbye" // Goodbye
```

# Recap
- 옵셔널은 안전한 코드를 위한 Swift의 가장 큰 기능 중 하나이다.
- 대개의 개발 언어가 `null` 혹은 `nil`이 있다만, 대개의 개발 언어는 "`nil`이 아니다"로 값을 정의할 수단이 없다.
  - 또는 `null`이 아님을 보장하기 위해 개발자가 기본값을 제공하도록 강요하기도 하고..
- 그러니까, 음.. 잘 활용해보자..!