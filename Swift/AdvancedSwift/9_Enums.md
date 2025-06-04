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

## Modeling State with Enums
- 위에서, 존재하면 안 되는 케이스를 열거형을 통해 제거하는 것에 대해 논했다.
- 이를 프로그램의 "상태"를 관리하는 데 사용할 수 있다.
- 프로그램의 상태란, 해당 시점에 있는 모든 변수들과 그것들의 현재 실행 상태를 말한다.
- 상태는 지금 앱이 어떤 모드인지, 무엇이 표시되고 있는지, 유저 인터랙션은 어떤지 등을 "기억"한다.
- 가령 웹 개발자들은 stateless한 HTTP 통신과 함께 뭔가 하려면, 쿠키와 같은 기능을 사용해야 한다.
    - HTTP는 stateless할지언정 프로그램은 내부 상태 관리를 위해서라도 stateful하다.
- 앱이 복잡해질 수록, 이 앱이 가질 수 있는 가능한 상태에 대해 미리 정의해 두는 것은 도움이 된다.
    - 이를 state space라고도 한다.
- 프로그램의 state space를 가능한 작게 유지하자. (중요)
- 열거형은 유한한 갯수의 상태를 모델링하기 때문에, 상태와 상태 간의 변화를 모델링하기에 좋다.
- 각각의 케이스가 관련값을 데이터로서 가지고 있기 때문에, 있을 수 없는 케이스는 자연스럽게 애시당초 불가능하게 만들기도 좋다.
- 어떻게 그걸 다 알아요, 할 수도 있지만, 대부분 프로그램의 중요한 부분은 유한한 가짓수의 상태를 가진다.
    - 그리고 심지어 생각보다 갯수도 적다! 그렇지 않으면 코드로 모델링할 수가 없지.
- 채팅 앱을 작성한다고 가정해 보겠다. 로딩중, 메세지 불러옴, 에러의 세 가지 상태를 다루어야 한다.
```Swift
// 열거형 없이 구현하면?
struct StateStruct {
    var isLoading: Bool
    var messages: [Message]?
    var error: Error?
}
```
- 위 코드는 총 8가지의 상태를 가질 수 있다.
    - 로딩중/아님 * 메세지있음/없음 * 오류있음/없음
- 하지만 이 중 우리가 필요한 케이스는 세 가지 뿐이다.
    - 로딩중이거나, 로딩중이 아니고 메시지가 있거나, 로딩중이 아니고 오류가 있거나.
```Swift
enum StateEnum {
    case loading
    case loaded([Message])
    case failed(Error)
}
```
- 훨씬 낫다. 또한 불가능한 케이스(예를 들면, 로딩은 끝났는데 메시지와 오류가 동시에 있다던지)가 소멸하였다.
- 또한 우리의 채팅 앱이 특정 상태에 있을 때 필요한 데이터가 있으리라고 보장이 가능하다.
- 뭐 한편으로는, 상태 전환이 완전하지 않은 면도 있기는 하다.
    - 가령 위 예시는 `loaded` 케이스에서 `failed` 케이스로의 이동은 불가능하다.
    - and vice versa
- 특정한 상태에서만 있는 데이터를 사용하고자 하면, 상태값을 바꾸도록 강요받게 된다.
- 이러한 안전성 확보 기능은 개발자가 "항상" 모든 상태를 컨트롤하게 하기 때문에, 불편하지만 중요하다.
- 한편 기존의 구조체에서 "있을 수 없는" 상태를 제거함으로써 목적을 이룰 수도 있다.
```Swift
// 이 방식은 가능한 경우의 수를 4가지로 줄인다.
// 하지만 그래도, 메세지와 에러가 동시에 존재할 수 있다.
struct StateStruct2 {
    var messages: [Message]?
    var error: Error?

    var isLoading: Bool {
        get {
            return messages == nil && error == nil
        }

        set {
            messages = nil
            error = nil
        }
    }
}
// 그렇다면 Result Type을 사용하면 어떨까?
// 이 경우 나머지 하나의 무효 케이스도 소거된다.
// 하지만 nil이 꼭 "로딩중"을 나타내지는 않는다는 문제는 여전히 있다.
typealias State2 = Result<[Message], Error>?
```
- 상기 예시는 하나의 모듈만을 다루지만, 이 패턴을 확장할 수 있다.
- 전체 프로그램의 상태를 하나의 열거형으로 관리하는 것도 가능하다!!
    - 물론 대개 그 안에 nested enum과 실제 상태를 구체적으로 나타내는 구조체를 여럿 덧붙이게 된다.
    - 하지만 여전히, 프로그램의 상태를 변수 하나로 컨트롤할 수 있는 데에 의의가 있다.
- 이러한 상태 관리는 또한, 재시작시에 상태 복원을 용이하게 한다.
- 그리고 처음부터 전체 앱을 하나의 열거형으로 관리하려 들 필요도 없다.
- 요약하면?
    - 열거형은 상태 관리 및 모델링에 아주 잘 맞는다.
    - 불가능한 상태를 막고, 앱의 모든 가능한 상태를 하나의 변수로 컨트롤할 수 있다.
    - 이를 통해 잠재적인 에러를 크게 줄일 수 있다.
    - 또한 상태(기능)의 추가는, 모든 `switch`문을 업데이트하도록 강요함으로서, 빼먹는 부분 없이 업데이트도 할 수 있게 한다.

## Choosing between Enums and Structs
- 열거형과 구조체의 차이를 다시 생각해보면..
    - 열거형은 가능한 케이스 중 오직 하나만 표시한다.
    - 구조체는 가지고 있는 모든 프로퍼티의 상태를 표시한다.
- 이런 차이에도 불구하고, 열거형과 구조체 둘 다 해결할 수 있는 문제가 있다.
- 분석을 위한 자료를 만든다고 했을 때..
```swift
// 열거형으로 정의했을 때
enum AnalyticsEvent {
    case loginFailed(reason: LoginFailureReason)
    case loginSucceeded
    // ...
}

// 실제 사용하는 값들을 위한 확장
extension AnalyticsEvent {
    var name: String {
        switch self {
        case .loginSucceeded:
            return "loginSucceeded"
        case .loginFailed:
            return "loginFailed"
        }
    }

    var metadata: [String: String] {
        switch self {
        case .loginSucceeded:
            return [:]
        case .loginFailed(let reason):
            return ["reason": String(describing: reason)]
        }
    }
}

// 구조체로 정의했을 때
struct AnalyticsEvent {
    let name: String
    let metadata: [String: String]

    // private init이므로 함부로 초기화할 수 없다.
    private init(name: String, metadata: [String: String] = [:]) {
        self.name = name
        self.metadata = metadata
    }

    // 열거형의 실패 케이스와 비슷하게 작동한다.
    static func loginFailed(reason: LoginFailureReason) -> AnalyticsEvent {
        return AnalyticsEvent(
            name: "loginFailed",
            metadata: ["reason": String(describing: reason)]
        )
    }

    // 열거형의 성공 케이스와 비슷하게 작동한다.
    static let loginSucceeded = AnalyticsEvent(name: "loginSucceeded")
}
```
- 그럼에도 불구하고 열거형이냐 구조체냐는 사뭇 다른 면이 있다.
- 구조체의 초기화 함수를 `private`하게 만들지 않으면, 다른 파일이나 모듈에서 새로운 정적 함수나 변수를 더 구현할 수 있다.
    - 이는 또한 API에 새로운 이벤트 분석 기능을 더한다던지, 하는 식으로 사용할 수 있다.
- 열거형이 데이터를 더 섬세하게 다룬다.
    - 사전 정의된 케이스만 쓸 수 있다는 점에서 그렇다.
    - 이러한 섬세함은, 이벤트 이후에 추가적인 처리를 해야 할 경우 장점이 된다.
- 구조체에서 정적 함수/변수로 정의된 일종의 "케이스"는, `private`할 수 있다.
- 하지만 열거형 케이스는 반드시 해당 열거형과 같은 공개 범위를 지닌다.
    - `private enum`의 모든 케이스는 `private`하다.
- 열거형이 exhaustive해야 한다는 특성은, 빼먹고 처리하는 이벤트가 없게끔 한다.
- 하지만 열거형에 케이스를 더한다는 것은 소스 코드 전체를 체크해야 하는 변화일 수 있다.
- 반면 구조체에 새로운 이벤트 타입을 더하는 정적 함수를 얼마든지 더 만들 수 있으며, 다른 코드에 영향이 없다.
    - 이러한 특성은 라이브러리를 만들 때 유용하다.

## Drawing Parallels between Enums and Protocols
- 열거형과 프로토콜은 꽤 흥미로운 공통점이 있다.
- 열거형이 "~중 하나"의 관계를 나타내는 유일한 수단이 아니다! 프로토콜도 이렇게 쓰일 수 있다.
- 위에서 썼던 "모양" 관련 열거형을 가져와 보면..
```swift
enum Shape {
    case line(from: Point, to: Point)
    case rectangle(origin: Point, width: Double, height: Double)
    case circle(center: Point, radius: Double)
}

// 렌더링을 위한 확장
extension Shape {
    func render(into context: CGContext) {
        switch self {
        case let .line(from, to): // ...
        case let .rectangle(origin, width, height): // ...
        case let .circle(center, radius): // ...
        }
    }
}

// 동일한 구현을 프로토콜로
protocol Shape {
    func render(into context: CGContext)
}

struct Line: Shape {
    var from: Point
    var to: Point

    func render(into context: CGContext) {
        // ...
    }
}

struct Rectangle: Shape {
    var origin: Point
    var width: Double
    var height: Double

    func render(into context: CGContext) {
        // ...
    }
}
```
- 열거형 기반의 구현은 메서드로 묶인다.
    - 하나의 메서드가 케이스별로 `switch`문을 통해 동작을 수행한다.
- 프로토콜 기반의 구현은 케이스로 묶인다.
    - 각각의 구현은 또한 각각의 케이스별로 특화된 렌더 메서드 구현을 가진다.
- 이러한 차이점은 확장성 측면에서 중요하다.
    - 열거형 구현은 새로운 메서드를 쉽게 더할 수 있다.
    - 하지만 새로운 케이스를 더하는 것은 소스 코드 전체에 영향을 미칠 수 있다.
    - 프로토콜 구현은 새로운 케이스를 쉽게 만들 수 있다. 타입을 만들고 채택만 하면 된다.
    - 반면 새로운 기능을 가진 메서드를 추가하기는 어렵다. 프로토콜 정의 밖에서 새로운 함수를 더할 수 없기 때문.
        - `extension`을 통한 방법이 있기는 하나, 많은 경우 "새로운 기능의 추가" 측면에서는 적합하지 않다.
        - Dynamic Dispatch를 따르지 않기 때문에 그렇다.
        - (주) 이러한 특성은 "특화된 메서드 제공" 측면에서 의미가 퇴색된다고 보여진다.
- 열거형과 프로토콜 기반의 구현은 이렇게 각각 정확히 반대되는 장점과 단점이 있다.
- 이러한 차이점은 또한 라이브러리를 만들 때, 어떤 측면에서 유저에게 확장성을 줄 지 결정하는 과정에서 고려된다.

## Modeling Recursive Data Structures wwith Enums
- 열거형이 재귀적인 데이터 모델을 만들기 정말 좋다는 사실 알고 계십니까?!
    - 재귀적인 데이터 모델이라 함은, 자기 자신을 포함할 수 있는 데이터 타입을 말한다.
    - HTML이나 XML같은 것이 그러하다.
- `Node` 타입을 만들면서 생각해 보자..
```swift
enum Node: Hashable {
    case text(String)
    // indirect 키워드는 컴파일러가 해당 케이스를 참조 타입처럼 사용하게 한다.
    // 이런 명시가 없다면, 이 열거형은 크기가 무한히 커질 것이다.
    indirect case element(
        name: String,
        attributes: [String: String] = [:],
        children: Node = .fragment([])
    )
    // (주) 여기서 쓰인 Fragment는 React의 그것과 비슷하다고 생각하면 된다고 한다.
    case fragment([Node])
}

// 아래 코드는 <h1>Hello <em>World</em></h1>; 코드와 같은 모양을 나타낸다.
let header: Node = .element(
    name: "h1",
    children: .fragment(
        [
            .text("Hello"),
            .element(name: "em", children: .text("World"))
        ]
    )
)

// ExpressibleByArrayLiteral을 통해 fragment를 더 깔끔하게 만들 수 있다.
extension Node: ExpressibleByArrayLiteral {
    init(arrayLiteral elements: Node...) {
        self = .fragment(elements)
    }
}

// ExpressibleByStringLiteral을 채택하여 text를 더 쉽게 만들 수 있다.
extension Node: ExpressibleByStringLiteral {
    init(stringLiteral value: String) {
        self = .text(value)
    }
}

// element 케이스를 쉽게 만들기 위해 wrapper 메서드를 만든다.
extension Node {
    func wrapped(in elementName: String, attributes: [String: String] = [:]) -> Node {
        .element(
            name: elementName,
            attributes: attributes,
            children: self
        )
    }
}

// 상기 모든 확장을 적용하고 나면 같은 표현을 이렇게 쓸 수 있다.
let contents: Node = [
    "Hello",
    ("World" as Node).wrapped(in: "em")
]

let headerAlt = contents.wrapped(in: "h1")

// 더 예쁘고 세련되게 만들고 싶다면 앞에서 배운 result builder를 사용하는 방법도 있다.

// 한편 enum의 mutating 함수를 이용하는 방법도 있다.
extension Node {
    mutating func wrap(in elementName: String, attributes: [String: String] = [:]) {
        self = .element(name: elementName, attributes: attributes, children: self)
    }
}

var greeting: Node = "Hello"
greeting,wrap(in: "strong")
```
- Swift 열거형은 이것보다 더 추상적인 모델링도 할 수 있다.
    - 연결 리스트, 이진 트리, ...
- 그러나 Swift의 기본 내장 자료구조보다 대부분의 경우 성능이 떨어지는 건 어쩔 수 없다..

### Indirect
- 값 타입은 원래 자기 자신을 포함할 수 없다.
    - 타입 크기를 계산할 때 무한히 커질 수 있기 때문.
- 한편 컴파일러는 프로그램 컴파일 시점에 타입의 고정된/유한한 크기를 알아야 한다.
- 참조는 일종의 우회를 한 단계 더한다. Indirect는 우회하다라는 뜻이니까..
    - 컴파일러는 참조를 저장하는 저장소의 크기가 (64비트 시스템에서)항상 8바이트인 것을 안다.
- 한편 위쪽에 `.fragment` 케이스에는 왜 `indirect` 키워드를 쓰지 않았을까? `[Node]` 타입을 연관값으로 포함함에도?
    - 그것은 연관값이 어쨌든 배열이기 때문에, 정해진 크기를 갖기 때문이다. 
    - 이 경우 실제로 메모리에서 배열이 차지하는 크기를 토대로 결정하게 된다.
- `indirect` 구문은 열거형에서만 사용할 수 있다.
- 만약 이것이 없다면, 클래스에서 우회 값을 박싱함으로써 수동으로 우회를.. 에이 하지 말자. 
- 만약 최상위 레벨 fragment를 허용하지 않는 정책이라면, 이런 구현도 가능하다.
```swift
enum NodeAlt {
    case text(String)
    case element(
        name: String,
        attributes: [String: String] = [:],
        children: [Node]
    )
}
```
- 열거형 자체도 `indirect enum`으로 선언할 수 있다.
    - 이는 모든 케이스에 대해 `indirect`를 붙이는 것과 같다.
- `indirect case`에 여러 연관값이 있을 경우, 참조는 그것들을 묶어서 참조한다.
- 열거형의 크기는 가장 큰 케이스의 크기에 따라 결정된다.
```swift
enum TwoInts {
    case nothing
    case ints(Int, Int)
}

// 가장 큰 케이스(8바이트가 두개지요) + 태그를 저장할 1바이트
MemoryLayout<TwoInts>.size // 17

enum PairedInts {
    case nothing
    indirect case ints(Int, Int)
}

// "참조"에 배정된 크기는 8바이트
// 하지만 태그 비트를 구겨넣을 남는 바이트 정도는 있다.
MemoryLayout<PairedInts>.size // 8
```
- 특정 케이스의 크기를 줄이기 위해 `indirect`를 쓰고 싶을 지도 모른다.
- 이론상으로 컴파일러는 여기에 `indirect`가 필요한지 아닌지 추론할 숭 ㅣㅆ다.
- Swift에서는 그러나 그러지 않고, 직접 컨트롤하게끔 구현되어 있다.
- 그리고 사실 컴파일러가 이를 합리적으로 추론하는 것도 쉬운 일은 아니다..
    - 가령 제네릭한 연관값을 가지는 열거형에서 그 크기를 어떻게 판단할지는 어려운 일이다.

# Raw Values
- 열거형에 있는 케이스들을 뭔가 다른 값과 연동시키고 싶을 때가 있다.
- C나 Obj-C의 열거형은 기본적으로 이렇게 작동하는데, 사실 까보면 열거형 케이스는 숫자의 alias다.
- Swift 열거형의 케이스는 그렇지 않지만, 원한다면 하나하나 뭔가 다른 값을 매칭시켜줄 수 있다.
- 이것을 Raw Value(원시값)라고 한다.
- 역발상! C API와 상호작용할 때 유용할 수 있는 기능이다.
- 혹은 열거형을 JSON과 같은 데이터 포맷으로 인코딩하고자 할 때 유용하다.
- 열거형에 원시값을 추가하면, 열거형 타입과 별개로 원시값 타입 또한 추가된다.
```swift
enum HTTPStatusCode: Int {
    case ok = 200
    case created = 201
    // and so on...
}
```
- (일단 적용했다면) 열거형의 모든 케이스는 고유한 원시값을 가져야 한다.

## The `RawRepresentable` Protocol
- 원시값을 통해 나타낼 수 있는 타입은 두 개의 API를 새로 얻는다.
    - `rawValue` 프로퍼티
    - 실패 가능한 `init?(rawValue:)` 초기화 함수
- 열거형에 원시값을 적용했다면, 컴파일러는 이 두 가지를 자동으로 구현한다.
- 초기화 함수가 실패 가능한 이유는, 케이스와 매칭이 안 되는 원시값이 들어올 수도 있기 때문이다.

## Manual RawRepresentable Conformance
- 원시값의 타입은 `String`, `Character`, 혹은 숫자 타입만 일단은 가능하다.
- "일단은". 만약 직접 상기한 두 가지를 구현한다면, 다른 타입이더라도 `RawRepresentable`을 채택할 수 있다.
    - 사실 컴파일러가 자동으로 구현해주는 것도 일종의 문법적 설탕일 뿐!!
- 아래 예시에서는 원래 원시값으로 지원되지 않는 `(Int, Int)` 튜플 타입을 써 볼 거다.
```swift
enum AnchorPoint {
    case center
    case topLeft
    case topRight
    case bottomLeft
    case bottomRight
}

extension AnchorPoint: RawRepresentable {
    typealias RawValue = (x: Int, y: Int)

    var rawValue: (x: Int, y: Int) {
        switch self {
        case .center: return (0, 0)
        case .topLeft: return (-1, 1)
        case .topRight: return (1, 1)
        case .bottomLeft: return (-1, -1)
        case .bottomRight: return (1, -1)
        }
    }

    init?(rawValue: (x: Int, y: Int)) {
        switch rawValue {
        case (0, 0): self = .center
        case (-1, 1): self = .topLeft
        case (1, 1): self = .topRight
        case (-1, -1): self = .bottomLeft
        case (1, -1): self = .bottomRight
        default: return nil
        }
    }
}

AnchorPoint.topLeft.rawValue // (x: -1, y: 1)
AnchorPoint(rawValue: (x: 0, y: 0)) // Optional(AnchorPoint.center)
```
- 사실 위와 같은 모양의 코드가, 컴파일러가 자동으로 만들어 주는 코드와 정확히 같다.
- 한편 이렇게 커스텀하게 원시값을 정해줄 때, "같은 원시값 문제"에 대해 주의해야 한다.
- 원래 모든 케이스는 "고유한" 원시값을 가져야 한다는 것을 기억하는지?
- 하지만 이렇게 직접 구현하면 같은 원시값을 가져도 컴파일러가 에러를 뱉지 않는다.
- 물론 하위호환을 위해 다른 케이스에 같은 원시값을 할당해야 하는 케이스도 있을 수 있지만, 이건 예외!
- 그렇기 때문에 직접 구현한다면 원시값을 다루는 데 신경쓰자.

# `RawRepresentable` for Structs and Classes
- `RawRepresentable`이 열거형에서만 쓸 수 있는게 아닌 거 아십니가?
- 간단히 감싸는 타입을 만들면서 타입 안전성을 보장하고자 할 때 유용하다.
- 가령, 유저 아이디를 `UserID` 타입으로 정의해 관리함으로써 다른 `String` 타입과의 혼동을 방지할 수 있다.
```swift
struct UserID: RawRepresentable {
    var rawValue: String
}
```
- `rawValue` 프로퍼티는 만족했는데, `init?(rawValue:)`는 어디갔지?
- Swift 구조체의 memberwise initializer 자동 생성 기능으로 인해 생성이 된다!
    - 그리고 컴파일러는 이 경우 `init`이 failable initializer가 아닌 것도 파악을 한다.
- 그럼에도 불구하고 failable한 `init?(rawValue:)` 함수가 필요하면, 마찬가지로 직접 구현하면 된다.
    - 모든 문자열이 유효한 유저 아이디가 아닐 수도 있으니까..

## Internal Representation of Raw Values
- 몇몇 추가 기능을 제외하면, 원시값이 있는 열거형은 그냥 얼거형이랑 다를 게 없다.
- C와는 다르게, Swift 열거형은 타입을 보존한다. 열거형이 가질 수 있는 "값"은 케이스 뿐이다.
- 마찬가지로 원시값에 접근할 수 있는 수단은 상술한 두 가지 API 뿐이다.
```swift
enum MenuItem: String {
    case undo = "Undo"
    case cut = "Cut"
    case copy = "Copy"
    case paste = "Paste"
}

// 단지 케이스 태그를 저장할 바이트만이 필요할 뿐
MemoryLayout<MenuItem>.size // 1
```
- 열거형은 원시값을 내부적으로 저장하지 않고, 연산 프로퍼티처럼 동작한다.

# Enumerating Enum Cases
- 열거형의 모든 케이스를 가지고 있는 컬렉션이 있으면 순회나 카운팅에 유용하지 않을까?
- 이를 위해 안배된 `CaseIterable` 프로토콜!
- 구현은 대략 아래와 같다.
```swift
protocol CaseIterable {
    associatedtype AllCases: Collection where AllCases.Element == Self

    static var allCases: AllCases { get }
}
```
- 연관값이 없는 열거형이라면, 컴파일러는 `CaseIterable`의 채택만으로도 위 구현을 자동으로 수행한다.
```swift
enum MenuItem: String, CaseIterable {
    case undo = "Undo"
    case cut = "Cut"
    case copy = "Copy"
    case paste = "Paste"
}

MenuItem.allCases // [MenuItem.undo, MenuItem.cut, MenuItem.copy, MenuItem.paste]
```
- `allCases` 프로퍼티는 `Collection`이므로, 배열이나 다른 컬렉션에서 쓸 수 있는 유용한 기능을 쓸 수 있다.
- 컴파일러가 자동 생성해주는 코드가 갖는 이점이 있다면..
    - 코드의 변경에 따라 항상 최신 상태로 유지된다는 것!
- `CaseIterable` 문서에 따르면, 케이스의 선언 순서대로 반환할 것을 보장한다고 한다.

## Manual `CaseIterable` Conformance
- 연관값이 있는 열거형은 이론상 무한한 경우의 수가 있기 때문에 `CaseIterable`을 기본적으론 못 쓴다.
- 하지만 몇몇 타입은 직접 구현할 수 있다! 심지어 구조체나 클래스일 때도 그렇다.
```swift
extension Bool: CaseIterable {
    public static var allCases: [Bool] {
        return [false, true]
    }
}

extension UInt8: CaseIterable {
    // 꼭 배열이 아니고, 어떤 Collection이더라도 가능하다!
    // ClosedRange 또한 Collection을 충족한다.
    public static var allCases: ClosedRange<UInt8> {
        return .min ... .max
    }
}

UInt8.allCases.count // 256
```
- `CaseIterable`을 경우의 수가 많거나 연산이 많이 필요한 타입에 적용하려면...
    - `allCases`를 `lazy` 프로퍼티로 만들어 실제 연산을 호출 시점까지 미뤄 보자.
- 상기 두 가지 예시는, 내가 own하지 않는 타입에 내가 own하지 않는 프로토콜을 적용하지 말라는 법칙과 어긋나긴 한다.
    - `Bool`도 `UInt8`도 Swift 기본 타입이고, `CaseIterable`도 기본 프로토콜이라 소유권이 없다.
    - 무슨 부작용이 일어날지 모른단 얘기.

# Frozen and Non-Frozen Enums
- 열거형의 장점 중 하나는 처리의 완전성(Exhaustiveness)의 보장이 강요된다는 것이다.
- 그러나 이것은 컴파일 타임에 모든 케이스에 대해 컴파일러가 알아야지만 가능하다. 대부분 가능하지만..
- 열거형 타입인데, 인터페이스 이외의 내용물은 바이너리로 제공되는 (은닉화된)타입이 있다.
    - Swift Foundation을 위시한 퍼스트 파티 프레임워크 혹은 라이브러리
    - 네이버 지도처럼 서드 파티 라이브러리 중 오픈 소스가 아닌 것들
- 이러한 라이브러리들이 업데이트됨으로써, 이전 버전에는 없던 케이스가 생긴다면?
    - 핸들링이 불가능하므로 크래시가 날 것이다.
- 이렇듯, 미래에 케이스가 추가될 수도 있는 열거형을 non-frozen enum이라고 부른다.
    - 이 경우 미래에 추가될 수도 있는 케이스를 위해 반드시 `default` 분기를 작성해야 한다.
```swift
switch error {
    // previous implementation...
    case .dataCorrupted:
        // do something...
    @unknown default: 
        // handle unknown cases...
}
```
- `@unknown default`는 런타임에는 그냥 `default`랑 똑같이 작동한다.
- 하지만 한편 컴파일러에게 "컴파일 타임에 파악하지 못한 케이스에 대해서만 써라" 라는 메시지를 남기는 것과 같다.
- 이를 통해 바이너리로 제공되는 열거형에도 처리의 완전성을 보장할 수 있다.
    - 심지어 미래에 업데이트 될 새로운 라이브러리 인터페이스에도!
    - 다만 컴파일 타임에 존재를 알았지만 커버되지 않는 케이스가 있다면, 업데이트하라는 컴파일러 경고가 뜨기는 할 거다.
- `@frozen enum`과 그렇지 않은 열거형의 구분은, library evolution mode로 컴파일되었을 때만 의미가 부여된다.
- library evolution mode는 기본적으로 비활성화되어 있다. `-enable-library-evolution` 컴파일러 플래그를 통해 활성화할 수 있다.
    - 이것이 활성화된 라이브러리를 탄력적 라이브러리(resilient library)라고도 부른다.
    - 탄력적 라이브러리는 기본적으로 바이너리로 제공되며 소스 코드는 숨겨져 있다.
    - 이것이 활성화된 경우 라이브러리에 변화가 있더라도 ABI(Application Binary Interface)가 일정하게 유지된다.
    - 이러한 특징 덕에, 각기 다른 Swift 버전에서 동일한 라이브러리를 쓸 수 있는 것이다.
    - 애플 표준 라이브러리는 대개 그렇다.
- 탄력적 라이브러리에 있는 열거형은 기본적으로는 non-frozen이다.
- `@frozen` 애트리뷰트로 표시한 열거형은 frozen이다.
    - 라이브러리 개발자가, "더 이상 케이스를 더하지 않겠다" 하고 약속한 것과 같다.
    - 만약 그렇지 않다면(non-frozen이라면), 언제나 열거형을 처리할 때 `@unknown default`를 추가해야 한다.
- 표준 라이브러리 중 `Optional`, `Result` 열거형이 `@frozen enum`이다.
- 비 탄력적 라이브러리에 있는 열거형은 항상 frozen하다고 가정한다.
    - Swift Package Manager 등을 통해 설치하는 대부분의 오픈 소스 라이브러리는 이렇다.
    - 그렇기 때문에 `@unknown default`를 쓸 일이 없다.