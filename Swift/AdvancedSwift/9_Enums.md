# Enums
- 구조체와 클래스는 "레코드 타입"이다.
    - 레코드는 0개 이상의 필드를 가지고, 각각의 필드는 각기 타입이 있다.
    - 그렇다, DB에서 쓰이는 그 레코드다!
    - 튜플도 레코드 타입인데, 가볍고 익명인 구조체라고 보면 된다.
    - 어셈블리 프로그래머도, 메모리에 데이터를 적재할 때 레코드의 컨셉을 사용해 왔다.
- Swift의 `enum`은 뿌리부터 이것과 다른 카테고리에 들어 있다.
    - sum type이라고도 부르는데, 이는 주류 언어보다 함수형 언어에서 더 보편적이다.

## Overview
- `enum`은 0개 이상의 `case`를 가진다.
- 각각의 케이스는 옵셔널 튜플의 형태인 associated value, 연관값을 가질 수 있다.
- Swift의 `Optional`도 사실 일종의 `enum` 즉 열거형이다.
- 한편 `Result` 타입도 열거형인데, 이 경우 실패 케이스를 위한 `Error` 값이 하나 더 붙는다.
```Swift
@frozen enum Optional<Wrapped> {
    case none
    case some(Wrapped)
}

@frozen enum Result<Success, Failure: Error> {
    case success(Success)
    case failure(Failure)
}
```

### Enums are Value Types
- 열거형은 값 타입이다.
- 구조체와 거의 비슷한 특성을 가지고 있다.
    - 메서드를 가질 수 있다.
    - `mutating` 메서드도 정의될 수 있다.
    - `extension`도 만들 수 있다.
    - 프로토콜을 채택할 수도 있다.
- 단! 열거형은 저장 프로퍼티를 가질 수 없다.
- 열거형의 상태는 채택된 케이스와 그 연관값으로만 표현된다.
- 열거형의 `mutating` 함수는 구조체의 그것과 같은 방식으로 작동한다.
    - 열거형은 저장 프로퍼티가 없으므로, 새로운 값을 `self`에 바로 할당한다.
- 열거형은 명시적 초기화 함수가 필요없다.
- 하지만, `conveinence init`을 본체나 `extension`에 더해 주는 것은 가능하다.
```Swift
enum TextAlignment {
    case left
    case center
    case right
}

extension TextAlignment {
    init(defaultFor locale: Locale) {
        guard let language = locale.languageCode else {
            self = .left
            return
        }

        switch Locale.characterDirection(forLanguage: language) {
        case .rightToLeft: 
            self = .right
        case .leftToRight, .topToBottom, .bottomToTop:
            self = .left
        @unknown default:
            self = .left
        }
    }
}
```

# Sum Types and Product Types
- 열거형 값은 가능한 케이스들 중 오직 하나를 갖는다. or 타입
    - 거기에 연관값이 있으면 그 연관값도 포함해서!
    - 가령 상술한 `Result` 타입이라면 성공과 실패 중 하나는 반드시 수행할테지
- 그러나 레코드 타입은 그것이 가진 필드의 "모든" 값을 갖는다. and 타입
    - `(String, Int)` 타입 튜플은 `String` 하나와 `Int` 하나를 가질 거다.
    - 구조체 타입으로 보면, 모든 프로퍼티에 값이 존재한다는 것.
- 열거형이 가진 이런 특성은 구조체, 튜플, 클래스로 분명히 표현되기 어려운 케이스에서, 강타입의 이점을 유지하면서 더 표현력이 좋은 코드를 쓸 수 있다는 점에서 유용하다.
- 물론 프로토콜이나 상속 또한 같은 목적, 즉 or 타입으로 쓰일 수 있다.
    - 하지만 그 트레이드오프나 작동은 사뭇 다르다.
- 프로토콜 타입 변수(혹은 Existential type, 존재 타입)는 그 프로토콜을 만족하는 하나의 타입을 넣어 줄 수 있다.
- 비슷하게, 어떤 클래스 타입은 이를 상속받는 자식 타입을 넣어 줄 수도 있다.
- 프로토콜이나 클래스 타입은 다이나믹 디스패치를 사용하나, 열거형은 스위치문을 사용한다.
- 제한 사항에서도 차이가 있다.
    - 열거형이 가질 수 있는 경우의 수는 고정되어 바뀌지 않는다.
    - 프로토콜은 하나 이상을 채택해 여러 가지 모습을 띠게 할 수 있다.
    - 상속은 하나 이상이 받을 수 있다.
- 값 타입으로서, 열거형은 더 가볍고, 전통적인 바닐라한 값을 저장하는 데 특화되어 있다.
- 사실 "타입"이라는 것은 여러 가지 모습으로 나타난다.
- 타입을, 그 타입 인스턴스가 가질 수 있는 모든 상태(혹은 경우의 수)의 집합이라고 해 보자.
    - `Bool`은 2가지(`true`, `false`)
    - `UInt8`은 256가지(0 ~ 255)
- 구조체, 클래스, 튜플 등은 각각의 프로퍼티를 모두 가진다. 이를 "곱 타입"이라고 한다.
    - 상술한 두 가지 프로퍼티를 가지는 구조체라면, 이 "타입"이 가질 수 있는 가짓수는 256 * 2가지이다.
    - 여기에 `Bool` 타입이 하나 더 추가된다면 곱하기 2를 하는 거고.
- 한편 열거형에 케이스 하나를 추가하는 것은 단지 경우의 수를 하나 늘릴 뿐이다. 이를 "합 타입"이라고 한다.
    - 물론 연관값의 타입에 따라 경우의 수가 더해진다.
- 타입이 내포한 가짓수가 적어진다는 것은 안전한 코드를 쓰는 데 도움이 된다.

# Pattern Matching
- 열거형 값으로 뭔가 유용한 걸 하려면, 연관값을 빼내고자 하는 것이 보통이다.
- 옵셔널 체이닝, 바인딩 이런 것도 사실 `Optional`이라는 열거형에서 연관값을 빼는 시도다.
- 가장 보편적인 방법은 `switch`문을 쓰는 것이다.
- `switch`문을 쓸 때, 값을 비교해서 일치할 경우 값을 뽑아낼 수 있다.
- 이를 패턴 매칭이라고 하며, 사실 스위치문의 전유물은 아니다.
- 이것이 유용한 이유는, 특정 값을 콕 집지 않아도 값의 와꾸만으로 연관값을 뽑아낼 수 있기 때문이다.
- `switch`문의 각각의 케이스는 입력값과 매치되는 하나 이상의 패턴으로 구성된다.
```Swift
// 예시 1. 성공했고, 튜플 타입을 반환받을 때, 첫 번째 원소가 42인 경우 처리해야 한다면
switch result {
case .success((42, _)):
    // 처리
case .success(_):
    // 다른 성공 케이스 처리
case .failure(let error):
    // 에러 처리    
}
```
- Swift가 지원하는 여러 매칭 가능한 패턴 타입이 있다.
- 와일드카드 패턴(`_`)은 아무 값과도 매칭이 되고, 그 값을 무시한다.
    - 위처럼 특정 패턴이 매칭되면 나머지는 아무래도 좋을 때 유용하다.
    - `switch`에서의 `case _:`는 `default:`와 같다.
- 튜플 패턴은 0개 이상의 서브패턴을 콤마로 묶어 둔 것이다.
    - 상기 예시 중 `(42, let str)`라고 한다면..
        - 숫자는 42와 같아야 한다.
        - 결과로 넘어온 문자열을 `str` 프로퍼티에 바인딩한다.
- 열거형 케이스 패턴은 특정한 케이스와 매칭시킨다.
    - 그냥 모든 `case` 붙는 거라고 생각하면 된다. `switch`문에서도.
    - 연관값에 관한 서브패턴을 가질 수 있다.
    - 무시하려면 와일드카드 패턴을 쓰면 된다. `.success(_)`와 `.success`는 같다.
    - 연관값을 뽑아내기 위한 유일한 방법이다.
- 값 바인딩 패턴은 매칭된 모든 값을 새로운 상수나 변수에 할당한다.
    - `let (x, y)`, `(let x, let y)`는 같은 뜻이다.
    - `case let`처럼 사용할 수도 있다.
    - 한편 `(let x, y)`는 튜플의 첫 번째 원소를 변수에 할당하고, 두 번째 원소는 `y`와 같을 때 매칭 성공하여 다음 동작을 수행한다.
    - 값 바인딩은 `where`문과 합쳐져서 사용될 수도 있다.
```Swift
// 이 경우 값이 먼저 바인딩된 다음 조건을 점검한다.
if case let .success(let httpStatusCode) where 200..<300 ~= httpStatusCode = code { 
    // do something...
}
```
```Swift
struct Point {
    let x: Double
    let y: Double
}

enum Shape {
    case line(from: Point, to: Point)
    case rectangle(origin: Point, width: Double, height: Double)
    case circle(center: Point, radius: Double)
}

let shape: Shape 
// 적당히 정의하는 코드...

// 여러 가지 패턴을 하나의 분기에서 사용하려면, 모든 패턴은 값을 바인딩할 때 같은 이름과 타입을 사용해야 한다.
// 아래 패턴의 경우 origin으로 명명되는 모든 associated value가 Point 타입이다.
// 모든 케이스에서 동일한 타입과 이름을 붙여야 하므로 위의 타입에서 나머지 값들은 무시해야 한다.
switch shape {
case .line(let origin, _), .rectangle(let origin, _, _), .circle(let origin, _):
    print("Origin Point: \(origin)")
}

// line에서 length 따위의 Double 프로퍼티가 있다면 사용 가능할 수 있다.
enum SecondShape {
    case line(from: Point, length: Double, to: Point)
    case rectangle(origin: Point, width: Double, height: Double)
    case circle(center: Point, radius: Double)
}

let secondShape: SecondShape
// 적당히 값을 삽입

// 패턴 매칭이 가능함을 보여주지만, 크게 실용적이지 않은 코드. 이렇게 쓸 수 있다, 정도만 알면 된다.
switch secondShape {
case .line(let origin, let path, _), .rectangle(let origin, let path, _), .circle(let origin, let path):
    print("Origin Point: \(origin), travels \(path)")
}
```
- 옵셔널 패턴은 `?` 문법을 통해 옵셔널 "열거형"을 패턴 매칭하는 데 대한 문법적 설탕을 제공한다.
    - `value?`는 `.some(let value)`와 같다.
- 타입 캐스팅 패턴은 값의 런타임 타입이 매칭하고자 하는 타입과 같거나 그 자식 클래스일 경우 발동한다. 
```Swift
let input: Any
// 적당히 할당

// 엄밀히 말하면 다운캐스팅이다.
switch input {
    case let integer as Int: // 정수일 경우
    case let string as String: // 문자열일 경우
    default: // 터뜨리기
}

// 참고: 업캐스팅과 다운캐스팅이 있다.
class UIView {
    // ...
}

class UIControl: UIView {
    // ...
}

var view = UIView()

// 다운캐스팅 (실패 가능)
if let view = view as? UIControl {
    // ...
}

let control = UIControl()
// 업캐스팅 (반드시 성공)
view = control as UIView

// 단순한 타입 확인은 is를 사용
if view is UIControl { 
    // ...
}
```
- 표현식 패턴의 경우, 패턴 매칭 연산자인 `~=`를 활용한다. `if (200..<300) ~= httpStatusCode { }`
    - 기본적으로 `Equatable`한 타입이어야 한다.
    - 직접 구현할 수도 있다. `func ~=(pattern: ???, value: ???) -> Bool`
    - 위 함수는 타입을 자유롭게 정해도 되며, 둘이 꼭 같지 않아도 된다.
    - 컴파일러가 알아서 가장 알맞는 `~=` 연산자를 쓴다.
    - 하지만 굳이 재정의할 필요는 보통 없긴 하다..

## Pattern Matching in Other Contexts
- 패턴 매칭은 열거형에서 연관값을 빼내는 유일한 수단이다.
- 하지만 사실 패턴 매칭은 열거형에서만 쓰지도 않고, `switch`에서만 쓰지도 않는다.
- 사실 `let x = 1`과 같은 식도 할당 연산자 `=` 왼쪽의 패턴에 오른쪽 표현을 할당하는 값 바인딩 패턴이라고도 할 수 있다.
- 튜플 분해 할당 패턴
    - `let (word, pi) = ("Hello", 3.141592)`라던지, `for (index, value) in someArray.enumerated() where index % 2 == 0 { }` 라던지.
    - 특히 `for`문의 경우 값 바인딩을 위해 별도의 `let`을 쓰지 않는다.
    - 이 경우 기본적으로 모든 식별자는 값 바인딩이다.
- 와일드카드를 통한 값 무시
    - `for _ in 0..<someArray.count { }` 이 경우 루프를 위한 값을 만들지 않는다.
    - `_ = someFunction()` 함수 실행을 하고, unused value에 대한 컴파일러 경고를 끌 수 있다.
- `catch`를 통한 에러 캡쳐 `do { } catch let error as NetworkError { }`
- `if case`와 `guard case`는 케이스가 1개인 `switch`문과 비슷한다.
    - 할당 연산자 `=`를 사용하기 때문에, 헷갈리기 쉽다.
```Swift
let color: Color = ...

if case .gray = color {
    print("회색")
}

// 오른쪽에 있는 값을 왼쪽에 있는 패턴과 매칭시킨다, 라고 생각하면 이해하기 더 쉽다.
// 더 분명하게 이해할 수 있는 예시
if case .gray(let alpha) = color {
    print("회색, 알파: \(alpha))")
}

// 사실 옵셔널 바인딩도 이와 비슷하다
if let optionalSomething = optionalSomething {

}
```
- `for case`, `while case` 루프
    - `if case`와 비슷하고, 패턴 매칭을 통해 루프를 돌린다는 점에서 차이가 있다.
- 한편 클로저 안의 인자 리스트는 튜플 분리 패턴처럼 보이나, 그렇지 않다.
```Swift
// 튜플이 아니라, 단지 map의 transformer 함수가 인자를 두 개 받을 뿐!
someDictionary.map { (key, value) in
    // ...
}

// 튜플 분리는 이렇게 해야 한다
someDictionary.map { element in
    let (key, value) = element
    
    // ...
}
```

# Designing with Enums
- 열거형은 구조체, 클래스와 다르기 때문에 사용법도 달라야 한다.
- 현재 주류 프로그래밍 언어에서 이러한 순수 합 타입이 있는 경우는 드물다.

## Switching Exhaustively
- `switch`는 대부분의 경우 `if`에서 선택지가 여러 개 있을 때, 더 편하게 쓰는 방법으로 쓰인다.
- 하지만 가장 큰 하나의 차이점이 있다. `switch`는 모든 가능한 경우의 수를 커버해야 한다.
    - 이를 exhaustive하다고 한다.
- exhaustive 여부를 체크하는 것은 안전한 코드를 쓰고, 프로그램의 변경에 따라 코드를 올바르게 유지하는 데 도움이 된다.
- 하지만 이는 `default` 케이스가 있는 `switch`에서는 사라지는 장점이다.
- 그렇기 때문에, `switch`문을 쓸 때는 `default`를 되도록 피하자.
- exhaustive check는 `Bool`, `enum`, 튜플 모두에서 사용할 수 있다.
- `default`를 사용하기보다, 기본 동작으로 묶어 처리할 모든 케이스를 명시하자.
- 컴파일러는 또한, 이전의 케이스로부터 모두 커버되어, "무효"인 케이스에 대해서도 warning을 통해 알려 준다.
- exhaustive check는 열거형이나 코드가 동기화될 필요가 있을 경우 유용하다.
    - 특히 라이브러리 등에서 코드에 접근 가능할 경우.
- 라이브러리가 코드가 아닌 바이너리로 배포될 경우 좀 복잡하다.
    - 라이브러리가 업데이트가 되어서 적용되어도 코드가 문제없이 동작해야 할 경우.
    - 이 때는 어쩔 수 없이 `default`를 포함해야 한다.
    - 자세한 것은 `@frozen enum` 파트에서 설명하기로.

## Making Illegal States Impossible
- 컴파일러가 프로그램 내의 변수 타입에 대해 더 많이 알 수록, 코드를 더 빨리 생성할 수 있다.
    - 이 경우 아마 컴파일 후의 결과물
- 타입은 또한, API가 어떻게 사용되어야 하는지에 대한 가이드를 개발자에게 줄 수 있다.
- 이를 CDD, Compilier Driven Development라고 불러 볼까?
    - 컴파일러는 싸워 이겨야 할 적이 아니라, 옳은 답을 낼 가능성을 비약적으로 높여 주는 도구다.
- 함수의 입력 및 출력 타입은 신중히 정해져야 한다.
    - 함수의 "상위 차원"에서 함수의 동작을 결정하기 때문이다.
    - 가령 파라미터가 옵셔널이 아니라면, 함수 내에서 해당 수는 `nil`이 아님을 보장할 수 있다.
    - 이럴 때 열거형은 예측 가능한 값의 범위를 섬세하게 정할 수 있게 한다.
- 정적으로 체크되는 타입은 몇몇 종류의 에러를 막을 수 있다.
    - 런타임 아니고 컴파일 타임에 체크된단 얘기~
- 일종의 문서 역할을 하는 타입은, 주석과 달리 동기화가 되는 것을 보장한다.
    - 상술한 exhaustive check
- 한편 타입 시스템이 모든 것을 커버할 수는 없기도 하고, 그래서 문서가 필요하다.
- 그럼에도 불구하고 열거형 및 타입 시스템의 활용을 통해 컴파일러에게서 꽤 많은 도움을 받을 수 있다.
- 컴파일러에게 더 많은 정보를 준단 것은, 개발자가 할 일이 더 늘어난다는 것이기도 하다.
- 또한 타입을 섬세하게 적용할 수록 형변환을 할 때 써야 할 코드가 더 많아진다.
- 아무튼, 타입을 활용해서 "불가능한" 상태를 아예 사용할 수 없게 만들자.
- 열거형은 합 타입으로, 열거형에 케이스를 하나 더하는 것은 단지 선택지를 하나 늘릴 뿐이다.
    - 좋은 예시가 바로 `Optional`.
- 이를 적용하기 어려운 케이스도 있는데, 기존 Swift의 Completion Handler를 활용한 네트워크 콜백.
- 비동기 함수는 언제나 실패할 수 있다.
- Core Location을 예로 들면, `[CLPlacemark]?, Error?` 타입을 핸들러가 반환한다.
    - 이러면 있을 수 있는 경우의 수는 네 가지다.
    - 에러가 없는데 값도 없다면? 혹은 에러가 있는데 값도 있다면?
- 이를 보완하기 위해 분명한 상태를 가지는 `Result` 타입을 사용할 수 있다.
- 많은 iOS API가 이처럼 Swift의 강력한 타입의 수혜를 받지 못한다. Obj-C로 많이 쓰였기 때문.
- 단, Core Location에서도 iOS 15 이후 `async` API를 제공하여, 성공 시 값, 실패 시 에러 던지기, 둘 중 하나만 하게 바뀌었다.
- 함수를 작성할 때는 파라미터 타입과 반환 타입에 대해 깊이 고민하자. (2)
    - 가능한 범위를 분명히 하여 타입을 꽉 잡을수록, 컴파일러의 도움을 더 많이 받을 수 있다.
- 그런데 위의 예시에서 `.success`가 반환되긴 했지만 연관값으로 빈 배열이 반환된다면?
    - 통신은 성공했는데 결과가 없는 것. 이걸 성공이라고 할 수 있을까?
    - 이를 보완하기 위해 원소가 최소 1개 있는 배열을 구현해볼 수 있다.
```Swift
struct NonEmptyArray<Element> {
    // 헤드가 옵셔널이 아니므로 최소 하나 이상의 원소가 있을 것을 보장한다
    private var head: Element
    private var tail: [Element]

    // 여기에 Array가 채택하는 다른 모든 프로토콜을 채택한다면 편히 사용할 수 있을 것
}
```