# Dictionaries
- 유일한 값인 키와 그에 대응되는 값을 가지고 있다. 
- 키를 토대로 값을 가져올 때 O(1)의 시간 복잡도를 지닌다.
- Element 간의 순서가 매겨져 있지 않다!
- subscripting 방법을 활용해 내부의 값을 가져 오며, 이 떄 항상 `Optional`을 반환한다.
    - 대응되는 키에 대한 값이 없다면 `nil`을 반환한다.
- 배열의 인덱스를 subscript 하는 것과는 꽤 차이가 있다.
  - 배열의 경우 알고 쓰는 것! 따라서 subscript 메서드에 index out of range 값을 넣는다면 그것은 휴먼 에러이다.
  - 반면 딕셔너리의 경우 이 값이 있나 없나~ 를 찔러 봄으로써 조회하는 것은 굉장히 자연스러운 동작이다.

## Mutating Dictionaries
- 딕셔너리에서 값을 제거하고자 한다면 두 가지 방법을 쓸 수 있다.
```Swift
var dict = ["One": 1, "Two": 2, "Three": 3]

// 첫 번째 방법. 해당 키에 대한 값을 nil로 만든다.
dict["One"] = nil

// 두 번째 방법. removeValue(forKey:) 메서드를 사용한다.
// 이 경우 메서드가 제거된 값을 반환도 해 준다. 해당 키가 원래 없었다면 nil을 반환한다.
dict.removeValue(forKey: "Two")
```
- 상수로 선언한 딕셔너리를 변수로 복사해 오면 값을 바꿔가며 활용할 수 있다. 무슨 당연한 소리지?
  - 예를 들어 기본 설정 목록이 저장되어 있고, 개인별 설정을 이에 기반해 짜고 싶을 때! 유용하다.
- 상술한 `removeValue(forKey:)` 이외에 `updateValue(_:forkey:)` 메서드도 있다.
```Swift
// 딕셔너리 안에서 다양한 종류의 값을 망라하기 위한 래핑 열거형
enum Setting {
    case text(String)
    case number(Int)
    case bool(Bool)
}

let defaultSettings = ["name": .text("John Doe"), "age": .number(50)]

var settings = defaultSettings
settings["name"] = .text("Gordon Choi")
settings["age"] = .number(30)

// 이렇게 기존에 없던 키가 주가됐을 수도 있다!
settings["doNotDistrurb"] = .bool(true)
```

## Some Useful Dictionary Methods
- 위에 쓴 설정의 예시에서, 기본 설정과 개인 설정을 합치고 싶다면 어떻게 해야 할까?
```Swift
// merge 메서드를 활용
// 각각의 딕셔너리에 있는 키, 값 쌍을 합쳐서 새로운 딕셔너리를 만든다.
// 겹치는 키가 있을 경우 $1, 즉 파라미터에 들어가는 딕셔너리의 값을 사용한다.
let combinedSettings = defaultSettings.merge(settings, uniqueingKeysWith: { $1 })

// 키가 유니크할 때만 사용할 수 있는 게 아니다!
// 본질적으로는 같은 키를 가지고 있는 값에 대해 함수를 가하는 동작.
// uniqueingKeysWith에 함수를 어떤 걸 주냐에 따라 아래와 같은 활용도 가능하다.
extension Sequence where Element: Hashable {
    // 하나의 시퀀스 안에 같은 원소가 몇 개나 나왔는지 측정하는 연산 프로퍼티
    var frequencies: [Element: Int] {
        let frequencyPairs = self.map { ($0, 1) }
        return Dictionary(frequencyPairs, uniqueingKeysWith: +)
    }
}
```
- 딕셔너리에서 `map`을 하는 것도 가능하다. 딕셔너리는 `Sequence` 프로토콜을 만족하기 때문.
- 그런데 딕셔너리는 냅두고 값만 받아서 처리하고 싶으면, `mapValues` 메서드를 활용할 수 있다.
```Swift
let settingsAsStrings = settings.mapValues { setting in
    switch setting {
    case .string(let text):
        return text
    case .number(let value):
        return String(value)
    case .bool(let status):
        return String(status)
    }
}
```

## Hashable Requirement for Keys
- 딕셔너리는 기본적으로 해시 테이블이다. 따라서 키의 해시 값을 인덱스로 활용해 내부 배열에 값을 저장한다.
- 그래서 딕셔너리의 키가 `Hashable` 프로토콜을 만족해야 하는 것이다.
- 대부분의 기본 타입은 `Hashable`하다.
- 성능을 위해 해시 테이블은 그 자신에게 할당되는 타입을 알고자 한다.
  - 그래야 더 좋은 해시 함수를 사용해 충돌을 줄일 수 있기 때문.
  - 대부분 개발자가 직접 할 필요는 없는 일이다.
- 커스텀 타입이라 해도 사용할 수 있는 표준 라이브러리 내부의 해시 함수가 있다.
- 값 타입의 경우 Swift 자체적으로 `Hashable` 프로토콜에 부합하게 해 준다. 
  - 단! `struct`의 경우 내부 프로퍼티가 전부 `Hashable`하다는 전제 하에!
  - 마찬가지로 `enum`도 associated value 모두가 `Hashable`하다는 전제 하에!
- 커스텀 타입을 짜느라, 예를 들면 해시 과정에서 무시되어야 할 값 등이 있어서 이 이점을 누리지 못한다면, 먼저 `Equatable`을 채택해야 한다.
- 그 다음 `Hashable`에서 구현해야 하는 `hash(into:)` 함수를 적용하면 된다.
  - `Hasher` 타입을 입력받는다. 이미 `Hashable`한 모든 값을 받아서 엮는 친구.
- 모든 중요한 타입을 해셔에게 전달해야 한다! 그래야 좋은 해시 값이 나오겠지.
  - 자료의 아이덴티티 등을 따지는 것은 중요한 타입이라고 할 수 있겠다.
- 반면 사용자에게 필요없는 값도 있다. 예를 들어 배열의 버퍼 사이즈는 사실 유저에게는 필요가 없다. 내용이 중요하지. 이런 건 해시하지 않는다.
- 동일성의 체크를 위해 같은 자료들을 비교해야 한다. 사실 == 함수를 오버로딩하면서 이미 경험했을 것이다.
- 인스턴스가 같다! 라는 것은 같은 해시 값을 갖고 있다! 이기도 하다.
  - 의외로 역은 성립하지 않는데, 서로 다른 해시 값의 갯수는 유한해서 다른 인스턴스가 같은 해시 값을 가질 수도 있기 때문.
- Swift 표준 라이브러리의 해시 함수는 랜덤 시드를 입력 중 하나로 받는다.
  - 이 말인즉, 매번 프로그램이 실행될 때마다 같은 인스턴스에 대해서도 해시 값은 달라질 수 있다는 말이다.
  - 끌 수도 있는데, 환경 변수 `SWIFT_DETERMINISTIC_HASHING=1`로 설정해주면 된다. 다만 릴리즈/프로덕션 과정에선 엄금!
- 한편 밸류 시맨틱으로 작동하지 않는 타입을 딕셔너리 키로 쓸 때는 각별히 조심해야 한다.
  - 참조 타입의 뭔가를 딕셔너리 키로 썼는데 나중에 그것의 값을 바꿔 버리면? 딕셔너리에 저장한 값은 미아가 되어 버린다.
  - 물론 값 타입을 사용할 때는 상관이 없다.