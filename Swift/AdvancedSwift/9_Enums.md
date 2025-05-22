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