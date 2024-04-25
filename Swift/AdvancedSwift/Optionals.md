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