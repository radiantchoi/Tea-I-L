# Functions
- 먼저 함수에 대해 몇 가지 정리해 보자.
1. 함수는 변수에 넣을 수 있고, 매개변수 즉 파라미터로 줄 수 있다.
2. 함수는 스코프 밖의 변수를 "캡처"할 수 있다.
3. 함수를 만들려면 `func` 혹은 `{ }` 두 가지 방법을 쓸 수 있다. 후자의 경우 Swift 상에서는 클로저라고 부른다.
- 이것들을 하나씩 뜯어 볼 것이다.

## 변수와 매개변수로 활용 가능한 함수
- 많은 모던 언어가 그렇듯 Swift 안에서 함수는 일급시민이고, 그래서 이게 가능하다.
- 이 일급시민이라는 걸 이해하는 건.. 마치 C에서 포인터를 이해하는 것과도 같은 중요도랄까.
- 함수를 변수에 할당할 때는 함수 이름을 값으로 사용한다.
```Swift
func printInt(i: Int) {
    print(String(i))
}

let functionVariable = printInt
functionVariable(2) // 2
```
- 전달 인자 레이블은 포함하면 안 된다.
- Swift 함수는 정의할 때만 전달 인자 레이블을 쓰는 것을 허용한다. 함수의 "타입"에는 포함되지 않는다.
  - (근데 나중 버전에서는 바뀔 수도 있대)
- 함수를 인자로 받는 함수도 짤 수 있다.
```Swift
func functionInFunction(function: (Int) -> Void) {
    function(3)
}

functionInFunction(function: printInt) // 3
functionInFunction(function: functionVariable) // 3
```
- 그래서, 함수를 이렇게 다루는 게 뭐 어쨌는데? 사실 이건 고차함수의 작성과 관련이 있다.
- 고차함수는 함수를 인자로 받거나 반환값이 함수인 함수.

## 스코프 밖의 값을 캡처하는 함수
- 바깥에 있는 변수 같은 걸 참조하면, 이걸 "캡쳐"했다고 말한다.
```Swift
func counter() -> (Int) -> String {
    var counter = 0
    
    func innerFunc(i: Int) -> String {
        counter += 1
        return "count: \(counter)"
    }
    
    return innerFunc
}
```
- 이 경우, `innerFunc`는 `counter`를 캡쳐했다.
- 원래대로라면 `counter`는 `counter()` 함수의 지역 변수로, 리턴 부분에서 없어져야 한다.
- 하지만 `innerFunc`에서 캡쳐되었기 때문에 이 함수가 없어질 때까지 메모리에 남아 있다.
  - 아래 예처럼 변수에 할당하고 내장 함수를 여러 번 부르면, 그 변수 값이 남아 있는 것이다.
  - 주: 클래스의 할당과 사뭇 닮아 있다.
```Swift
let f = counter()
f(3) // count: 3
f(4) // count: 7

// 새롭게 캡처하는 변수 생성
let g = counter()
g(2) // count: 2
g(4) // count: 6
```
- 프로그래밍 방법론에서, 함수와 외부의 캡쳐된 변수의 콤비네이션을 클로저라고 부른다.
- `f`와 `g`는 클로저의 한 예시인 셈이다.

## 함수는 `{ }`로도 정의될 수 있다.
- 잘 알다시피 Swift에서 함수 정의 키워드는 `func`.
- 상술한 중괄호, 즉 클로저 표현법으로도 함수를 정의할 수 있다.
```Swift
func doubler(i: Int) -> Int {
    return i * 2
}

let doublerClosure = { (i: Int) -> Int in 
    i * 2
}

let numbers = [1, 2, 3, 4]
numbers.map(doubler)
numbers.map(doublerClosure)
numbers.map { $0 * 2 }
// [2, 4, 6, 8]
```
- 클로저 표현법으로 정의된 함수는 한편으로는 함수 리터럴이라고 봐도 된다.
- 또 클로저로 정의한 함수는 이름이 없다. 그래서 쓰려면 변수에 할당해야 한다. 아니면 다른 함수에 인자로 넘기던가.
- 그럼 아래에 넣은 건 도대체 뭔데?
  - 클로저를 인자로 넘긴다면, 굳이 변수에 할당하지 않고 쓸 수 있다. (`map` 함수의 인자로 넘기므로)
  - 컴파일러가 문맥상 타입 추론이 가능하다면, 클로저의 파라미터 타입 및 반환 타입을 명시하지 않아도 된다. (`Int`의 배열을 사용하므로)
  - 클로저 안에 한 개의 동작만 들어가 있다면 자동으로 `return`하기 때문에, 키워드를 생략해도 된다. (곱하기 2의 동작 한 줄만 들어가므로)
  - 함수 인자에 대한 짧은 버전의 이름을 쓸 수 있다. (`i`에서 `$0`으로. 두 번째 인자는 `$1`, ...)
  - 클로저가 마지막 인자라면, 괄호를 미리 닫고 뒤에 클로저만 덧붙일 수 있다. 이를 `trailing closure`라고 한다.
    - Swift 5.3부터는 다중 후행 클로저도 지원한다. SwiftUI 하다 보면 종종 볼 수 있다.
  - 바로 위의 것의 연장선으로, 인자가 클로저 하나뿐이라면 `()`조차도 생략할 수 있다.
- 변천사를 보자면
```Swift
numbers.map({ (i: Int) -> Int in return i * 2 })
numbers.map({ i in return i * 2 })
numbers.map({ i in i * 2 })
numbers.map({ $0 * 2 })
numbers.map() { $0 * 2 }
numbers.map { $0 * 2 }
```
- 클로저식 표현이 익숙하지 않다면 모든 것을 명시해보는 것도 좋다.
- 그럼에도 어쨌든 파라미터를 명시는 해야 하는데, `_` 와일드카드 문자를 씀으로써 무시할 수 있다.
```Swift
numbers.map { _ in 3 } // [3, 3, 3, 3]
```
- 변수에 클로저를 넣을 때도 타입 추론이 기능한다.
```Swift
// 클로저 내용상 isEven은 (Int) -> Bool로 추론된다.
// 숫자 리터럴은 기본적으로 Int로 추론되기 때문.
let isEven = { $0 % 2 == 0 }

// 이는 Swift Standard Library의 ExpressibleByIntegerLiteral 타입과 관련이 있다.
// 해당 타입의 associatedType인 IntergerLiteralType은 Int의 typealias다.
protocol ExpressibleByIntegerLiteral {
    init(integerLiteral value: IntegerLiteralType
}

typealias IntegerLiteralType: Int
```
- 물론 다른 타입이 필요할 때 명시해줄 수도 있다.
```Swift
let isEven8: (UInt8) -> Bool = isEven 
```
- `func` 키워드로 정의된 함수도 클로저가 될 수 있고, 클로저란 캡쳐된 변수와 함께 존재하는 함수이다.
- 그러니까, 다 함수일 수 있고 다 클로저일 수 있다는 말이다.

## Flexibility through Functions
- Swift Collection 안에는 네 가지의 정렬 함수가 있다.
  - `sort(by:)`, `sorted(by:)`
  - `sort()`, `sorted()`: 위 함수의 기본 버전으로, 오름차순 정렬한다.
- `by` 파라미터 안에 값을 제공해 주기만 하면 된다.
```Swift
numbers.sorted(by: >)
// 클로저 형태로 안에 복잡한 비교를 해줄 수도 있다.
numbers.sorted { $0 % 3 > $1 % 3 } 

// 참고사항인데, 튜플의 대소 비교는 첫 번째 인자를 기준으로 이루어진다.
// 이를 활용해 여러 조건이 걸린 정렬을 할 수 있다.
struct Person {
    let name: String
    let age: String // 후술알 localizedStandardCompare는 Int에 적용되지 않기 때문..
}

let gordon = Person(name: "Gordon", age: "30")
let greg = Person(name: "Greg", age: "27")
let annie = Person(name: "Annie", age: "30")

let people = [gordon, greg, annie]
people.sorted { ($0.age, -$0.name) > ($1.age, -$1.name) } 
// [annie, gordon, greg]
// 나이가 많은 순서대로, 나이가 같다면 이름이 사전순으로 오름차순
// 만약 나이와 이름 오름차순으로 가려면?
people.sorted { ($0.age, $0.name) > ($1.age, $1.name) } 
```
- 한편, Obj-C의 `NSSortDescriptor` 또한 도움이 된다.
- 위의 정렬을 리팩토링해보면
```Swift
people.sorted { p1, p2 in
    // localizedStandardCompare는 정렬 과정에서 로케일의 영향을 받는다.
    switch p1.age.localizedStandardCompare(p2.age) {
        case .orderedAscending:
            return false
        case .orderedDescending:
            return true
        case .orderedSame:
            return p1.name.localizedStandardCompare(p2.name) == .orderedAscending
    }
}
```

### Functions as Data
- 값의 정렬을 추상화해서 직접 정의할 수도 있을 것이다. 
- `typealias`를 사용하던 타입을 정의하던 해서 말이다.
```Swift
typealias SortDescriptor<Root> = (Root, Root) -> Bool

struct SortDescriptor<Root> {
    var areInIncreasingOrder: (Root, Root) -> Bool
}

let sortByAge: SortDescriptor<Person> = .init { $0.age < $1.age }
let sortByName: SoreDescriptor<Person> = .init { $0.name < $1.name }

// 아니면 init 함수를 하나 더 쓰던지
extension SortDescriptor {
    // 파라미터로 들어가는 key 함수는 비교할 만 한 무언가를 뽑아내는 방법이라고 보면 된다.
    // Swift에 있는 key path와 모양이 비슷하다. 실제로 Root와 Value는 거기서 빌려온 표현법이라고 한다.
    init<Value: Comparable>(_ key: @escaping (Root) -> Value) {
        self.areInIncreasingOrder = { key($0) < key($1) }
    }
}

let sortByAgeAlt: SortDescriptor<Person> = .init { $0.age }
```
- `localizedStandardCompare` 비슷한 것도 비슷한 방법으로 만들어볼 수 있다.
- 참조: [ComparisonResult](https://developer.apple.com/documentation/foundation/comparisonresult)
```Swift
extension SortDescriptor {
    // 여기서 파라미터로 주어진 compare 함수는 Value를 인자로 받아 (Value) -> ComparisonResult 값을 반환하는 함수이다.
    // str.localizedStandardCompare의 반환값이 실제로 이런 모양이라고 한다.
    // 다른 표현으로는 String.localizedStandardCompare(str)
    init<Value>(
        _ key: @escaping (Root) -> value,
        by compare: @escaping (Value) -> (Value) -> ComparisonResult
    ) {
        self.areInIncreasingOrder = { compare(key($0))(key($1)) == .orderedAscending }
    }
}

let sortByNameAlt: SortDescriptor<Person> = .init({ $0.name }), by: String.localizedStandardCompare)
```
- 여러 조건을 걸고 비교하고자 할 때, `SortDescriptor` 여러 가지를 하나로 합칠 수 있다.
```Swift
extension SortDescriptor {
    // 다분히 함수형 방법론이 의식되는 방식
    func then(_ other: SortDescriptor<Root>) -> SortDescriptor<Root> {
        SortDescriptor { x, y in 
            // 먼저 자기 자신이 가지고 있는 규칙을 활용해, SortDescriptor 인스턴스를 즉석에서 하나 만들어서 이 기준대로 비교한다.
            if areInIncreasingOrder(x, y) { return true }
            if areInIncreasingOrder(y, x) { return false }
            
            // x와 y가 같다면 비로소 그 다음 기준을 적용한다.
            return other.areInIncreasingOrder(x, y)
        }
    } 
}

let combinedCriteria = sortByAgeAlt.then(sortByNameAlt)
people.sorted(by: combinedCriteria.areInIncreasingOrder)
```
- 물론 `Foundation`의 `SortDescriptor`만큼 좋은 건 아닌 건 맞다~
- 하지만 함수를 데이터로 접근해서, 배열에 넣고 이것을 런타임에 빌드하고 하는 방식에 대해 이해한 것이 중요하다.
- 철저한 컴파일 타임 언어인 Swift에 런타임 언어인 Obj-C의 동작을 조금 넣어줄 수 있다는 거!
- 한편으로는 다른 함수를 엮는 함수를 작성하는 것이 얼마나 유용한지 볼 수 있었다.

# Functions as Delegates
- 콜백을 위한 프로토콜로써 정의되는 `delegate`!
- `delegate`가 메서드 하나만 갖고 있다면, `delegate` 말고 그냥 그 콜백 함수를 프로퍼티에 할당하는 방법이 있다.
  - 물론 염두해야 할 트레이드오프도 있지만.

## Delegates, Cocoa Style
- `delegate`를 정의하는 예시 코드부터 시작하자.
```Swift
// AnyObject인 이유: 약한 참조를 사용하고 싶어서
protocol AlertViewDelegate: AnyObject {
    func buttonTapped(atIndex: Int)
}

final class AlertView {
    var buttons: [String]
    
    weak var delegate: AlertViewDelegate?
    
    init(buttons: [String] = ["OK", "Cancel]) {
        self.buttons = buttons
    }
    
    func fire() {
        delegate?.buttonTapped?(atIndex: 1)
    }
} 
```
- 이런 식의 `delegate` 정의는 클래스에서 사용하기 좋으며, 많이들 이렇게 해 온다.
- `delegate` 프로퍼티를 약한 참조로 할당하는 것은 실제로 국룰에 가깝다. 메모리 관리가 용이하기 때문.

## Functions instead of Delegates
- 상술한 대로 `delegate` 프로토콜이 하나의 함수만 갖고 있다면?
- 콜백 함수를 그 자리에 그대로 구겨넣어 볼 수 있다.
- 원래 함수 타입 안에서는 전달 인자 레이블을 볼 수 없어야 한다고 했는데, 사실 내부 인자 이름을 가진 빈 인자라면 이름을 덧붙일 수 있긴 하다.
- 로깅 함수를 들고 있는 구조체까지 붙여 보자.
```Swift
final class AlertView {
    var buttons: [String]
    var buttonTapped: ((_ buttonIndex: Int) -> ())?
    
    init(buttons: [String] = ["OK", "Cancel]) {
        self.buttons = buttons
    }
    
    func fire() {
        buttonTapped?(1)
    }
}

struct Logger {
    var taps: [Int] = []
    
    mutating func logTap(index: Int) {
        taps.append(index)
    }
}

let alert = AlertView()
var logger = Logger()

alert.buttonTapped = logger.logTap // 컴파일 에러
// partial application of a 'mutating' method is not allowed
// 왜냐면, logger를 복사해서 할당할지, buttonTapped가 taps를 변환을 가해야 할 지 명확하지 않기 때문
// 이럴 때는 클로저로 감싸 준다 - 지금 이미 정의되어 있는 logger를 "캡처"한다는 의미가 된다.
alert.buttonTapped = { logger.logTap(index: $0) }
```
- 콜백과 클래스를 결합할 때 몇 가지 주의사항이 있다.
```Swift
final class ViewController {
    let alert: AlertView
    
    init() {
        alert = AlertView(buttons: ["OK", "Cancel"])
        // 여기가 괜찮아 보인다면? 노우.. 순환 참조를 만들었다.
        alert.buttonTapped = self.buttonTapped(atIndex:)
    }
    
    func buttonTapped(atIndex index: Int) {
        print("Button Tapped: \(index)")
    }
}
```
- 객체의 메서드를 참조하는 것은, 암시적으로 그 객체를 캡처하는 것을 나타낸다.
- 위의 예시의 경우. `AlertView`는 어쨌든 어디서 `buttonTapped` 함수를 가져와서 불러오는지 "알아야" 하기 때문.
- 이런 현상을 막기 위해 클로저 표현을 써 줘야 할 때가 있다.
```Swift
// 파라미터로 정수를 받고
// 함수 자체는 객체에서 가져 오되
// 약한 참조를 통해 캡처한다.
// 두 객체의 라이프사이클이 함께 간다면, weak 대신 unowned도 괜찮다.
alert.buttonTapped = { [weak self] index in 
    self?.buttonTapped(atIndex: index)
}
```
- 프로토콜 `delegate` 만들기와 콜백 함수 직접 할당하기는 일장일단이 있다.
  - 프로토콜은 좀 귀찮고 지저분하지만, 순환 참조를 나도 모르게 만들 가능성은 적다.
  - 콜백 함수 직접 만들기는 구조체나 무명 함수(클로저)를 콜백 자리에 쓸 수 있게 된다.
- 한편 여러 개의 콜백 함수가 밀접하게 연관된 경우, 각각을 콜백 함수 프로퍼티에 할당하기보다 프로토콜로 묶어 주는 게 나을 수 있다.
  - UIKit 상의 여러 delegate 클래스를 생각해 보자.
  - 다만 이런 경우 필요도 없는 함수를 다 구현해야 하는 경우도 있긴 하다..
- `delegate`나 콜백의 지정을 해제하려면, `nil`을 그 자리에 할당하면 된다.

# `inout` Parameters and Mutating Methods
- `&` 기호는 참조 전달 즉 포인터 같겠지만, 사실 아니지롱~
- 값을 전달하고, 값을 복사해서 되돌려주는 것이다.
  - 함수 안에 파라미터로서 "값"을 전달하고, 연산 등을 통해 사용한 다음, 다 쓰고 난 값을 함수에서 다시 빼내어 원래 있던 자리에 할당한다.
- `inout`으로 전달할 수 있는 것에 대해 알려면, `lvalue`와 `rvalue`에 대해 생각해봐야 한다.
  - `lvalue`는 메모리에서의 위치를 나타낸다. (`array[0]`) 왜 left나면, 할당할 때 왼쪽에 나오는 거니까 그렇다.
  - `rvalue`는 값을 나타낸다. (`2 + 2`) 이하동문
- `inout`에는 `lvalue`만 넣을 수 있다. `rvalue`를 "바꾼다"라는 건 조금만 생각해보면 말이 안 된다.
- `inout` 파라미터에 들어가는 `lvalue`는 explicit 해야만 한다. 앰퍼샌드를 붙이는 것.
```Swift
func traverse(area: inout Int) {
    guard area < 10 else { return }
    
    area += 1
}

// 마찬가지로 let 변수를 "mutate"시키는 건 말이 안 되므로, var만 넣을 수 있다.
var area = 0
traverse(area: &area)
```
- 또 뭘 넣을 수 있을까? 우선 `var`로 정의된 배열의 `subscript`를 넣을 수 있다. (`traverse(area: &areas[2])`)
  - 사실 배열뿐만 아니라 모든 `subscript`에 대해 가능하다.
- 프로퍼티의 `get set` 모두 정의가 되어 있을 경우 프로퍼티 또한 넣을 수 있다.
  - read-only 프로퍼티는 쓸 수 없다는 뜻.
- 연산자도 `inout` 값을 받는다만, 앰퍼샌드가 필요하지는 않다.
```Swift
// prefix, infix, postfix 접두어를 연산자 함수에 붙일 수 있다.
// 각각 연산자가 앞, 중간, 뒤에 오는 것을 나타낸다.
postfix func ++(x: inout Int) {
    x += 1
}

var num = 0
num++ // 1
```
- 이러한 `mutating` 연산자는 옵셔널 체이닝도 쓸 수 있다.
- 참조 전달같은 동작을 보이긴 하지만, 진짜 그렇게 생각하면 안 된다고 문서에 명시되어 있다.

## Nested Functions and `inout`
- `inout` 파라미터는 Nested Function 안에서 쓸 수도 있다.
- 그런데, escape 하게 할 수는 없다.
```Swift
func incrementTenTimes(value: inout Int) {
    func increment() {
        value += 1
    }
    
    for _ in 0..<10 {
        increment()
    }
}

// 이거는 내부에 있는 함수가 스코프에서 escape하려고 했기 때문에 오류가 난다.
func escapingIncrement(value: Int) -> () -> () {
    func increment() {
        value += 1
    }
    
    return increment
}
```
- escape 안 될 만도 한 게, `inout` 값은 `return` 직전에 원래 있던 자리에 반환되기 때문.
- 근데 return 이후에 뭔가 동작이 이루어져서 수가 바뀐다면? 이거는 아무래도 안전하진 않은 상황이라, 금지된 거다.

## When `&` Doesn't Mean `inout`
- 앰퍼샌드에는 또 다른 기능이 있는데, 함수 인자를 unsafe pointer로 바꾸는 것이다.
- 함수가 `UnsafeMutablePointer` 타입을 인자로 받는다면, `var` 앞에 `&` 기호를 붙임으로써 인자로 넘길 수 있다.
- 이 경우는 진짜, 참조 전달이다. 진짜 포인터라고.
```Swift
func incref(pointer: UnsafeMutablePointer<Int>) -> () -> Int {
    // 클로저 안에 포인터의 복사본을 저장한 모습이다
    return {
        pointer.pointee += 1
        return pointer.pointee
    }
} 
```
- 여담인데, Swift 배열은 C와의 호환성을 위해 포인터로 어.. 표현된다? 저수준으로 가면 그렇다?
- 만약 결과 함수를 불러오기 전에 스코프를 벗어나는 배열을 포인터 함수에 넘겼다면?
```Swift
let fun: () -> Int

do {
    var array = [0]
    fun = incref(pointer: &array)
}

fun()
```
- "fun"한 결과가 나오는 것을 볼 수 있다..
- 결국 요컨대, 뭘 인자로 넣고 있는지 잘 알아야 한다.
  - 앰퍼샌드가 있을 때는 `inout`일 수도 있고, unsafe pointer 지옥일 수도 있다.
  - 만약 후자라면, 변수의 라이프사이클에 대해 각별한 주의를 기울여야 한다.

# Subscripts
- 대괋호 안에 뭔가 넣어서 찾는 것! Array Index 혹은 Dictionary Key ...
- 함수와 프로퍼티의 혼종에 가까우며, 이것만을 위한 특별한 문법이 있다.
  - 함수처럼 인자를 받는다.
  - 연산 프로퍼티처럼 `get` 혹은 `get set`일 수 있다.
  - 함수처럼 여러 다른 타입의 인자를 줘서 오버라이딩할 수 있다.
    - 배열은 사실 기본적으로 두 가지 `subscript`를 가지고 있다는 놀라운 사실! 개별 원소에 접근하기 하나, 슬라이스 하나.

## Custom Subscripts
- 커스텀 타입에 `subscript`를 더해줄 수도 있고, 이미 존재하는 타입에 새로운 `subscript`를 더해 줄 수도 있다!
```Swift
extension Collection {
    subscript(indices indexList: Index...) -> [Element] {
        var result: [Element] = []
        
        for index in indexList {
            result.append(self[index])
        }
        
        return result
    }
}

var numbers = [1, 2, 3, 4, 5]
var picked = numbers[indices: 1, 2, 4, 4]
print(picked) // [2, 3, 5, 5]
```
- 기존에 `Collection`에 구현되어 있던 `subscript`와의 차이는, 이것이 개별 `Index`나 `Range`가 아닌 `Index` 여러 개라는 점!

## Advanced Subscripts
- 위에서 슬쩍 말했는데, `subscript`에는 하나의 파라미터만 들어가야 하는 것이 아니다.
- 대표적 예시로 `Dictionary`에 구현되어 있는, 키값과 기본값을 받는 `subscript`가 있다. 
- [구현](https://github.com/apple/swift/blob/release/5.5/stdlib/public/core/Dictionary.swift#L884-L903)
```Swift
extension Dictionary {
  @inlinable
  public subscript(
    key: Key, default defaultValue: @autoclosure () -> Value
  ) -> Value {
  
    @inline(__always)
    get {
      return _variant.lookup(key) ?? defaultValue()
    }
    
    @inline(__always)
    _modify {
      let (bucket, found) = _variant.mutatingFind(key)
      let native = _variant.asNative
      if !found {
        let value = defaultValue()
        native._insert(at: bucket, key: key, value: value)
      }
      
      let address = native._values + bucket.offset
      
      defer { _fixLifetime(self) }
      yield &address.pointee
    }
  }
}
```
- `subscript`는 또한 파라미터나 반환 타입에 제네릭을 먹일 수 있다.
```Swift
var japan: [String: Any] = [
    "name": "Japan",
    "capital": "Tokyo",
    "population": 126_440_000,
    "coordinates": [
        "latitude": 35.0,
        "longitude": 139.0
    ]
]

// 그냥 값을 뽑으려고 한다면?
japan["coordinates"]?["latitude"] = 36.0
// Error: Type "Any" has no subscript members.

// Any 타입에는 subscript가 없다, 납득 가능하다.

// 타입 캐스팅을 해서 뽑아 보면?
(japan["coordinates"] as? [String: Double])?["latitude"] = 36.0
// Error: Cannot assign to immutable expression.

// 타입 캐스팅을 한 순간 이 녀석은 lvalue, 즉 할당 가능한 변수 공간이 아니게 된다.
// 타입 캐스팅을 해서 로컬 변수에 먼저 넣고 값을 바꾸고 원래 자리에 있던 걸 바꿔 줘야 한다.

// 빡이 치니까, 원하는 타입을 두 번째 인자로 받는 subscript를 직접 구현하기로 한다.
extension Dictionary {
    subscript<Result>(key: Key, as type: Result.Type) -> Result? {
        get {
            return self[key] as? result
        }
        
        set {
            guard let value = newValue else {
                // 입력값을 nil을 받았을 경우 내부의 값을 삭제하는 과정
                self[key] = nil
                return
            }
            
            guard let value2 = value as? Value else {
                // 입력값의 타입이 맞지 않는다면 그냥 무시
                return
            }
            
            self[key] = value2
        }
    }
}

// 이렇게 하면 다운캐스팅 없이 값을 다룰 수 있다.
japan["coordinates", as: [String: Double].self]?["latitude"] = 36.0
print(japan["coordinates]) // Optional(["latitude": 36.0, "longitude": 139.0])
```
- `subscript`를 통해 일련의 해결 과정을 거쳤음에도, 사실 코드가 보기에 꽤 못생기긴 했다.
- 이 예시는 `subscript`에 파라미터가 여러 가지 들어갈 수 있다는 예시 정도로 참고하고, 대개 이런 경우는 커스텀 타입을 쓰는 게 낫다.

# Autoclosures
- AND 논리 연산자나 `&&` 연산자가 그들의 "파라미터"를 어떻게 받는지, 어떻게 유효함을 감지하는지 알고 있지?
  - 연산자의 왼쪽 값을 먼저 체크해서 이게 `false`라면 바로 반환
  - 그 다음 오른쪽을 체크
- 이러한 동작을 "Short Circuiting"이라고 부른다.
  - 참고로 `if let`도 이 방법의 하나인데, 앞의 조건이 만족되지 않으면 뒤의 조건은 검증조차 하지 않기 때문.
- 대부분의 언어에서 `&&` 혹은 `||` 연산자에 대해 Short Circuiting은 내장되어 있다.
- 그런데 커스텀 연산자에선 이게 안 될 수도 있다.
- 이 경우, 만약 함수를 일급시민으로 가진 언어라면, 값 대신 익명함수를 파라미터에 전달해 Short Circuiting "흉내"를 낼 수 있다.
```Swift
let numbers = [Int]()

func and(_ lhs: Bool, _ rhs: () -> Bool) -> Bool {
    guard lhs else { return false }
    
    return rhs()
}

// 근데 이걸 그대로 쓰면 좀 못생겼다
if and(!numbers.isEmpty, { numbers[0] > 10 }) {
    // ...
}
```
- 위의 못생긴 표현을 교정해줄 수 있는 Swift `@autoclosure` 기능!
- 컴파일러로 하여금 어떤 값을 클로저로 감싸야 한다는 것을 알 수 있게 하는 애트리뷰트이다.
```Swift
// 위의 함수 리팩토링
func and(_ lhs: Bool, _ rhs: @autoclosure () -> Bool) -> Bool {
    guard lhs else { return false }
    
    return rhs()
}

if and(!numbers.isEmpty, numbers[0] > 10) {
    // do something
}
```
- 실제 언어 안에서는 `@autoclosure`가 `assert` 등의 함수에서 널리 쓰인다.
- 왜 이런 짓을 하는가? 함수의 사용을 통해 Short Circuiting을 실현하기 위해서이다.
- 이는 다시 말하면 정말 필요할 때만 인자를 검증하기 위함이기도 하다.
- `@autoclosure`를 사용하게 되는 인자는 대개 연산 비용이 많이 드는 인자이며, 이를 실제 사용할 때까지 계산을 미룰 수 있다.
- `@autoclosure`는 로깅 함수를 작성할 때도 편한데 일단 예시 보실까요?
```Swift
func log(ifFalse condition: Bool,
         message: @autoclosure () -> (String),
         file: String = #fileID,
         function: String = #function,
         line: Int = #line) {
    guard !condition else { return }
    
    print("Assertion failed: \(message()), \(file): \(function) (line \(line))")
}
```
- 위 함수에서, 조건을 만족하지 않는다면 겁나 비싼 `String` 메세지를 작성할 필요가 없는 것이다.
- 한편 `#fileID`, `#function`, `#line` 등의 디버깅 ID는 특히 기본값을 제공할 때 유용하다.
  - 호출부의 파일 이름, 함수 이름, 코드 줄 위치를 바로 받기 때문!
- `@autoclosure`는 그래도, 일반적으로 예측되는 동작과 다르기 때문에 주의해서 사용할 필요는 있다.
  - 가령 실행될 것이라 예상했던 어떤 식이 `@autoclosure`에 래핑되어 있어 실행되지 않았을 경우의 사이드 이펙트는 어떤가?
  - 코드의 가독성을 떨어뜨릴 수 있다.
  - `@autoclosure`를 사용한다면, 해당 인자의 실행은 미뤄질 수 있음을 명시해야 한다.

# The `@escaping` Annotation
- 어떤 클로저 표현은 캡처할 때 `self`를 명시해야 하는 경우가 있다.
```Swift
import Combine

@Published var numbers = [1, 2, 3]
var cancellables = Set<AnyCancellable>()
var bin = [Int]()

$numbers
    .sink { [weak self] nums in
        // 어떤 클로저는 이렇게 안에 self를 명시해야 작동하는 반면
        self?.bin.append(contentsOf: nums)
    }
    .store(in: &cancellables)
    
// 어떤 클로저는 그러지 않아도 된다
bin = numbers.map { $0 * 2 }
```
- 이 둘의 차이점은, 인자로 제공된 클로저가 동기적으로 즉시 실행되고 털어버리는지, 아니면 나중에 호출하기 위해 저장되었다가 사용되는지이다.
- 후자의 경우 탈출 클로저라고 불린다.
- 클로저가 인자로 제공된 함수에서, 클로저가 해당 함수의 스코프를 벗어나지 않는다면 비 탈출 클로저이다.
  - 밖의 프로퍼티 등에 저장하던지, 반환값으로 주던지, 하는 것들이 스코프를 벗어나게 한다.
- 탈출 클로저가 `self`를 명시하게 하는 이유는, 실수로 클로저 밖의 변수를 캡처해 강한 참조를 만드는 것을 막기 위함이다.
  - 막는다기보단, 주의시킨다고 해 둘까?
  - 비 탈출 클로저는 함수가 끝나면 같이 사라지기 때문에 이럴 일이 없다.
- 클로저는 기본적으로 비 탈출 클로저이다.
- 탈출 클로저로 만들고 싶다면 (== 인자로 전달한 함수 밖에서 쓰고 싶다면) `@escaping` 애트리뷰트를 파라미터 앞에 불여줘야 한다.
- 한편 함수 파라미터 말고, 객체의 프로퍼티 등으로 저장되어 있는 클로저는 항상 탈출 클로저이다.
- 놀랍게도(혹은 놀랍지 않게도), 튜플이나 옵셔널 등으로 감싸서 파라미터로 전달되는 함수 역시 기본적으로 탈출 클로저이다.
  - "즉시 실행되는" 클로저가 아니게 됐기 때문.
  - 가령 옵셔널로 감싸여 있다면, 일단 값이 있는지 없는지 먼저 검증하는 기능이 들어가니까.
  - 그래서 옵셔널이면서 비 탈출 클로저인 파라미터는 아예 만들 수가 없다. 이게 싫으면 기본값을 제공하면 된다.
  - 아니면 아래처럼 오버로딩을 사용하는 방법도 있다.
```Swift
// 옵셔널 클로저 파라미터 (탈출 클로저)
// 옵셔널 클로저를 탈출 클로저 기능으로 쓸 때는 그래서, @escaping을 명시할 필요가 없다.
func transform(_ input: Int, with f: ((Int) -> Int)?) -> Int {
    guard let f else { return input }
    
    return f(input)
}

// 클로저 파라미터 (비 탈출 클로저)
func transform(_ input: Int, with f: (Int) -> Int) -> Int {
    return f(input)
}

transform(10, nil) // 10 (첫 번째 메서드)
transform(10) { $0 * $0 } // 100 (두 번째 메서드)
```

## `withoutActuallyEscaping`
- 이 함수가 escaping하지 않다는 것을 분명 알지만, 컴파일러는 모르니까 `@escaping`을 붙이라고 강요할 경우.
- 실제로 그런 경우가 있기는 있다. 예를 들어 `lazy`한 원소의 배열은, 그 원소들의 변경 로직을 메모리에 저장한다.
    - 주: 어찌보면 당연한 것이, lazy하게 불려오는 원소는 접근할 때 연산을 하니까 대비해야 한다고 컴파일러에 설정되어 있을 수 있다.
```Swift
extension Array {
    func allSatisfy2(_ predicate: (Element) -> Bool) -> Bool {
        // 컴파일 에러 발생
        // Error: Closure use of non-escaping parameter 'predicate' may allow it to escape.
        return self.lazy.filter({ !predicate($0) }).isEmpty
    }
}
```
- 어쨌든 상기 함수는 라이프사이클을 고려했을 때 절대 escape하지 않는다는 보장이 있다.
- 이 때 `withoutActuallyEscaping`을 통해, escaping closure가 기대되는 부분에 non-escaping closure를 넣어 줄 수 있다.
- 다만 이렇게 구현은 했더라도 원래 구현되어 있는 allSatisfy보다는 당연히 못하다는 점! 설명을 위한 예시였을 뿐이다.
- `withoutActuallyEscaping`을 사용함으로써 본격적으로 안전하지 못한 영역에 들어서게 된다는 것을 유념하자.

# Result Builders
- 여러 개의 표현으로부터 결과값을 빌드해서 내보내는 특수한 형태의 함수.
- SwiftUI 상에서 `View`가 대표적으로 Result Builder를 사용하는 곳이다.
```Swift
var body: some View {
    HStack {
        Text("Hello")

        Spacer()

        Button("Click me") {
            print("Button tapped")
        }
    }
}
```
- `HStack`의 후행 클로저는 result builder 함수이다. 별다른 표시는 되어 있지 않지만.
- 하지만 `HStack`의 이니셜라이저에서 `@ViewBuilder`라는 어노테이션을 찾을 수 있을 거다.
```Swift
struct HStack<Content>: View where Content: View {
    public init(
        alignment: VerticalAlignment = .center,
        spacing: CGFloat? = nil,
        @ViewBuilder content: () -> Content
    ) {
        // ...
    }
}
```
- Result builder 함수 안에 있다면, 컴파일러는 해당 함수 안에 있는 모든 코드를 다시 쓴다.
- 이 다시 쓴 코드를 통해 하나의 합성된 값을 반환한다.
- 이를 위해 알맞는 Result builder static 함수를 사용한다. `View`의 경우 `buildBlock` 메서드가 있다.
```Swift
// 원래는 뷰 한 개, 두 개, ... 열 개를 파라미터로 받는 buildBlock 함수가 있었다
// Swift 5.9 parameter packs를 통해 아래와 같은 표현이 가능해짐
// 참조: http://developer.apple.com/documentation/swiftui/viewbuilder/buildblock(_:)-2bj1g
// 참조: https://www.swift.org/blog/pack-iteration/
static func buildBlock<each Content>(_ content: repeat each Content) -> TupleView<(repeat each Content)> where repeat each Content : View {
    // ...
}
```
- Result builder 타입은 `@resultBuilder` 어노테이션으로 마크되어 있다.
- 한편 다양한 build 어쩌구 static 메서드를 사용할 수 있다.

## Blocks and Expressions
- 가장 기본적인 빌더 메서드는 `buildBlock`과 `buildExpression`이 있다.
    - 당연히! static 메서드다.
- 한편 `@resultBuilder` 타입의 정의를 위해서는 적어도 하나의 `buildBlock` 메서드가 필요하다.
    - `buildBlock` 메서드 자체는 여러 가지 구현할 수 있다. 숫자나, 파라미터 타입이나 이런 게 다르다면.
- 결국 여러 개의 부분적 결과값을 하나의 혼합된 결과로 만들면 되는 것이다.
- SwiftUI의 `buildBlock` 메서드는 오로지 `View` 타입만 파라미터로 받는다.
- 하지만! `buildBlock` 메서드의 구현에 있어서 오로지 `View` 타입만 받도록 제한해야 하는 건 아니다!!
- 그래도 여러 가지 타입을 지원하는 builder 함수를 만들려면 `buildExpression`을 구현하는 것이 좀 더 세련된 방법이긴 하다.
- `buildExpression` 함수는 빌더 타입에 꼭 요구되는 것은 아니나, 여러 가지 다른 표현을 다룰 때 도움이 된다.
- Swift는 `buildBlock` 함수에 넣기 전의 모든 표현을 `buildExpression`을 적용할 것이다!
- `buildExpression`은 하나의 파라미터를 받는다만, 여러 파라미터 타입을 지원하기 위해 여러 개 정의할 수 있다.
    - 결과값은 이미 구현되어 있는 `buildBlock` 함수에 들어갈 수 있는 타입이라면 뭐든 가능하다.
- 어김없이 등장한 구현타임~ String을 빌드해 보자.
```Swift
@resultBuilder
struct StringBuilder {
    static func buildBlock() -> String {
        ""
    }
}

@StringBuilder func build() -> String { }
build() // 빈 스트링이 프린트된다

// 사실 컴파일러를 통과하면서 이렇게 재작성된다
func build_rewritten() -> String {
    StringBuilder.buildBlock()
}
```
- 이제 여러 개의 파라미터를 받게끔 수정할 수 있다.
```Swift

@resultBuilder
struct StringBuilder {
    static func buildBlock(_ strings: String...) -> String {
        strings.joined()
    }

    // 다양한 타입 지원을 위한 expression 메서드 추가
    static func buildExpression(_ s: String) -> String {
        s
    }

    static func buildExpression(_ n: Int) -> String {
        "\(n)"
    }
}

@StringBuilder func greeting() -> String {
    "Hello "
    2
    " the "
    "World"
}

greeting() // Hello 2 the World

// 실제 과정
func greeting_rewritten() -> String {
    StringBuilder.buildBlock(
        StringBuilder.buildExpression("Hello "),
        StringBuilder.buildExpression(2),
        StringBuilder.buildExpression(" the "),
        StringBuilder.buildExpression("World")
    )
}
```

## Overloading Builder Methods
- 다양한 타입의 지원을 위해 빌더 함수를 오버로딩하는 경우가 잦다.
- 그런데 오버로딩으로써 유용한 기능을 덧붙일 수도 있다는 점~
- 예를 들면, `fatalError` 타입은 빌더 함수 안에서 쓸 수가 없다. 
    - 타입이 `Never`인 "표현식"으로 감지되기 때문.
- `Never` 타입을 위한 `buildExpression`을 만드는 것이 해결책이 될 수 있다.
    - 다만 그다지 실용적인 아이디어는 아니다. 컴파일러가 "이거 실행 안할건데?" 라고 플래그를 띄워 버린다.
- 비슷한 원리로 무언가 `print`하기 위해 `Void` 타입을 위한 `buildExpression`을 만들 수 있다.
- 또한 더 명확한 컴파일러 메시지를 위해 오버로딩할 수도 있다.
```Swift
extension StringBuilder {
    // String과 Int 이외의 타입이 StringBuilder 표현식 안에 들어왔을 경우
    @available(*, unavailable, message: "String Builders only support string and integer values")
    static func buildExpression<A>(_ expression: A) -> String {
        ""
    }
}
```

## Conditionals
- 지금까지 구현한 String Builder는 기존의 String interpolation과 사뭇 다르다.
- `buildIf` 혹은 `buildEither` 메서드를 빌더 타입에 더함으로써 `if`, `else`, `switch` 문에서 활용될 때의 편의성을 더할 수 있다.
```Swift
extension StringBuilder {
    static func buildIf(_ s: String?) -> String {
        s ?? ""
    }
}

@StringBuilder func greet(planet: String) -> String {
     "Hello, Planet"

    if let idx = planets.firstIndex(of: planet) {
        " "
        idx
    }
        
    "!"
}

greet(planet: "Earth") // Hello, Planet 2!
greet(planet: "Spaghetti") // Hello, Planet!

// 실제로 일어나는 일
func greet_rewritten(planet: String) -> String {
    let v0 = "Hello, Planet"
    // 부분 결과값을 위해 옵셔널 값이 정의된다
    var v1: String?

    // 각각의 표현을 위한 익스프레션
    if let idx = planets.firstIndex(of: planet) {
        v1 = StringBuilder.buildBlock(
            StringBuilder.buildExpression(" "),
            StringBuilder.buildExpression(idx)
        )
    }

    // v1이 nil이냐 아니냐에 따라 불려오는 값이 달라진다.
    let v2 = StringBuilder.buildIf(v1)
    return StringBuilder.buildBlock(v0, v2)
}
```
- 여러 개의 `if...else`문을 위해서는 `buildEither(first:)` 와 `buildEither(second:)`가 필요하다.
    - 이 과정에서 `else if` 문은 `else` 안에 중첩된 `if...else` 문으로 해석된다.
```Swift
extension StringBuilder {
    static func buildEither(first component: String) -> String {
        component
    }

    static func buildEither(second component: String) -> String {
        component
    }
}
// 상세 예시는.. 책의 해당 목차 참조. 너ㅓㅓ무 길다
```
- 당연히 이 안에서 뭔가 변환해서 뱉어도 된다.

## Loops
- 그럼 `for`문은 어떡할까?
- `buildArray` 메서드를 더하면 된다.
```Swift
extension StringBuilder {
    static func buildArray(_ components: [String]) -> String {
        components.joined(separator: "")
    }
}

@StringBuilder func greet3(planet: String?) -> String {
    "Hello "

    if let p = planet {
        p
    } else {
        for p in planets.dropLast() {
            "\(p), "
        }

        "and \(planets.last!)!"
    }
}

greet3(planet: nil)
// Swift는 for문을 통해 순회한 결과를 배열에 담고, buildArray에 통과시킨다.
// Hello Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, and Neptune!
```

## Other Build Methods
- 지금까지 소개한 빌드 메서드 이외에 두 가지가 더 있다.
- `buildMethodAvailability`는 결과 타입을 제한할 때 사용할 수 있다.
    - 예를 들면 `if #available`
- 한편 `buildFinalResult`는 말 그대로 result builder 함수들을 모두 거치고 마지막에 적용된다.
    - 내부적인 타입을 숨기고 임의의 반환 타입으로 반환하고자 할 때 유용하다.
    - 가령 String Builder에서 내부적으로는 `[String]` 타입을 쓰다가 반환시에 `String` 타입으로 바꾸고 싶은 경우.

## Unsupported Statements
- 지금까지 알아본 지원하는 표현들은 다음과 같다.
    - 타입으로 정의되는 "표현(Expression)" 그 자체
    - `if`, `if let`
    - `if ... else`
    - `switch`
    - `for`
- 바꿔 말하면, (Swift 5.6 기준) 이 표현들 말고 다른 거의 모든 표현들은 result builder 안에서 지원이 안 된다.
    - `guard`, `defer`, `do...catch`, `break`, `continue`...

# Recap
- 함수는 Swift의 일급시민!
- 함수를 자료로써 대하는 것은, 우리의 코드를 더욱 유연하게 만든다.
- 함수가 어떻게 런타임 프로그래밍을 대체하는지 배웠다.
- delegate에 대해 배웠다.
- `mutating` 및 `inout`에 대해 배웠다.
- 사실은 특별한 종류의 함수인 `subscript`에 대해 배웠다.
- `@autoclosure`, `@escaping`에 대해 배웠다.
- 결과값을 간결하면서도 표현력이 좋은 문법을 통해 반환하는 특별한 함수인 result builder에 대해 배웠다.