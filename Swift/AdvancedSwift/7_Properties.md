# Properties
- Swift에서는 저장 프로퍼티와 연산 프로퍼티의 두 가지가 있다.
- 저장 프로퍼티는 값을 저장한다(...)
- 연산 프로퍼티는 함수랑 비슷한 역할을 하기도 한다. 그럼 문법이 다른 함수 아님?
- 프로퍼티는 변수 중 특별한 케이스라고 생각해보면 어떨까?
- 프로퍼티의 중요한 두 가지 기능으로는 Key path와 Property wrappers가 있다.
- 키 패스는 프로퍼티를 직접 참조하지 않으면서 참조하는 방법이다.
    - 견고하고 제네릭한 코드를 쓰기 위해 많은 라이브러리 등지에서 채택한다.
- 프로퍼티 래퍼는 아주 적은 문법을 통해 프로퍼티의 작동 방식을 정할 수 있다.
```Swift
struct GPSTrack {
    // 외부에서는 읽기만 하게 하고 싶을 경우 활용할 수 있다.
    private(set) var record: [(CLLocation, Date)] = []

    // setter가 없는 연산 프로퍼티는 read-only이다.
    // 매번 접근이 일어날 때마다 연산을 수행한다.
    // Swift 가이드라인에 의하면, O(1)이 아닌 모든 연산 프로퍼티는 주석을 달아야 한다.
    var timestamps: [Date] {
        record.map { $0.1 }
    }
}
```

## Change Observers
- 변경사항에 대한 옵저버, `willSet`과 `didSet`.
- 프로퍼티나 변수에 값이 (바뀌지 않더라도) 할당되기 직전/직후에 즉시 호출되는 핸들러이다.
```Swift
class MyView: UIView {
    var pageSize: CGSize = CGSize(width: 800, height: 600) {
        didSet {
            self.setNeedsLayout()
        }
    }
}
```
- 상기 두 옵저버는 프로퍼티가 정의된 부분에 함께 정의되어야 한다.
- 그렇기 때문에 타입을 처음 설계한 사람을 위한 도구인 셈.
- 사실은, 한 쌍의 프로퍼티를 설정하고 조작하는 것에 대한 문법적 설탕에 가깝다.
- 그렇기 때문에 KVO와는 본질적으로 다르다.
    - KVO는 코드의 "사용자"가 설계자의 의도와 관계없이 내부적 변화를 감지하기 위한 수단이니까.
- 이 옵저버들을 사용하기 위해 extension에 프로퍼티를 아예 오버라이딩하는 것도 가능은 하다.
- KVO는 클래스의 setter에 옵저버를 동적으로 더하기 위해 Obj-C 런타임을 사용한다.
- 반면 프로퍼티 옵저빙은 순수하게 컴파일 타임 기능이다.

## Lazy Stored Properties
- `lazy` 프로퍼티는 반드시 `var`로 정의되어야 한다.
    - Swift에서 `let`은 반드시 인스턴스의 초기화 이전에 값을 가져야 한다.
- 메모이제이션의 한 형태라고도 볼 수 있다.
- 비용이 많이 드는 연산을, 실제 접근이 이루어질 때까지 미룰 수 있다.
```Swift
final class GPSTrackViewController: UIViewController {
    var track = GPSTrack()

    lazy var preview: UIImage = {
        for point in track.record {
            // 비용이 많이 드는 연산
        }

        // 아무튼 결과가 반영된 이미지
        return UIImage()
    }()
}
```
- `lazy` 프로퍼티 정의부를 보면, 저장하고자 하는 값의 타입을 반환하는 클로저의 형태이다.
- 한 줄로 정의하기 힘든 `lazy` 프로퍼티를 쓸 때 가장 흔히 사용되는 방법이다.
- `lazy` 변수는 어쨌든 저장 공간이 필요하다.
- 그래서인지 저장 프로퍼티와 `lazy` 프로퍼티는 `extension`에서 정의될 수 없다.
- 마찬가지로, 접근될 때마다 다시 계산되지 않는다. 뭔가 저장이 되어 있을 테니까.
- `lazy` 프로퍼티는 그래서, 연산 결과에 영향을 미치는 다른 프로퍼티의 값이 바뀌더라도 다시 계산되지 않는다.
    - 애시당초 이럴거면 그냥 연산 프로퍼티를 처음부터 썼겠지
- `lazy` 프로퍼티에 접근하는 것은 `mutating` 연산에 해당한다.
    - 그래서 구조체에 `lazy` 프로퍼티가 있다면, 이 구조체를 활용한 변수도 반드시 var로 지정해야 한다.
    - 그래서 구조체랑은 영 궁합이 안 좋은 것 같기도
- 또한 `lazy` 키워드 하에서 스레드 동기화가 일어나지 않도록 주의해야 한다.

## Property Wrappers
- 사실 프로퍼티 래퍼의 등장 동기는, `lazy` 프로퍼티를 컴파일러 내부 기능으로 때우기보다 라이브러리로 사용하고 싶었기 때문.
- 프로퍼티의 정의 과정에서 일어나는 동작을 설정 가능하다.
```Swift
struct MyView: View {
    // isOn을 일반적인 Bool로 사용할 수 있지만, 동작은 다르다.
    // Binding의 경우는 값이 이 자리에 저장되지 않고 부모 뷰에 저장된다.
    @Binding var isOn: Bool

    var body: some View {
        if isOn {
            // ...
        }
    }
}
```
- 클래스와 구조체의 프로퍼티, 지역 변수, 함수 인자에 프로퍼티 래퍼를 쓸 수 있다.
- 위에서 옵저버를 통해 레이아웃을 다시 정의했던 것을 프로퍼티 래퍼를 활용해 재작성할 수 있다.
    - 부분적으로 invalidate한다고 부른다.
```Swift
class MyView: UIView {
    // Invalidating 프로퍼티 래퍼의 경우 값을 내부적으로 저장한다.
    // 그리고 값이 바뀔 경우 자체적으로 setNeedsLayout 메서드를 호출한다.
    @Invalidating(.layout) var pageSize: CGSize = CGSize(width: 800, height: 600)
}
```
- 이외에도 반응형 프로그래밍, `UserDefaults` 등 여러 가지와 관련해서 사용할 수 있다.

### Usage
```Swift
@propertyWrapper
class Box<A> {
    var wrappedValue: A

    init(wrappedValue: A) {
        self.wrappedValue = wrappedValue
    }
}

struct Checkbox {
    @Box var isOn: Bool = false

    func didTap() {
        isOn.toggle()
    }
}

// Under the hood
struct Checkbox {
    // 프로퍼티 래퍼로 마킹된 프로퍼티에 대해 컴파일러는 언더바를 앞에 붙인 실제 저장 프로퍼티를 만든다.
    private var _isOn: Box<Bool> = Box(wrappedValue: false)

    // 감싸인 값에 접근하기 위한 연산 프로퍼티도 정의된다.
    // 그렇기 때문에 프로퍼티 래퍼를 정의할 때는 wrappedValue에 대한 getter는 무조건 정의해야 한다.
    // 한편 setter 정의는 선택사항이다.
    var isOn: Bool {
        get {
            _isOn.wrappedValue
        }

        // nonmutating인 이유는 Box가 클래스기 때문
        nonmutating set {
            _isOn.wrappedValue = newValue
        }
    }

    func didTap() {
        isOn.toggle()
    }
}
```

#### Projected Values
- SwiftUI에서, `@State` 혹은 `@ObservedObject`는 값의 소유권과 저장 장소를 나타낸다.
- 하지만 많은 UI 컴포넌트들은 값이 어디에 저장되는지 중요히 여기지 않는다. 단지 변화시킬 값이 필요할 뿐.
- 이를 위해 밖에 있는 값에 대한 getter와 setter라고 할 수 있는 `@Binding` 프로퍼티 래퍼가 있다.
- `@State` 혹은 `@ObservedObject` 같은 프로퍼티 래퍼가 projected value라는 특별한 기능을 통해 하위 뷰와의 바인딩을 구성한다.
- `projectedValue` 프로퍼티를 직접 구현할 수 있다..
    - `$` 표현을 사용한다. `$foo`는 `foo.projectedValue`와 같은 의미이다.
- 위에서 정의한 `Box`를 확장해서, `@Binding`의 동작을 모사해 보자.
```Swift
// 값을 어떻게 만질 건지 정의하는 Reference 타입
@propertyWrapper
class Reference<A> {
    private var _get: () -> A
    private var _set: (A) -> ()

    var wrappedValue: A {
        get { 
            _get()
        }

        set {
            _set(newValue)
        }
    }

    init(get: @escaping () -> A, set: @escaping (A) -> ()) {
        _get = get
        _set = set
    }
}

extension Box {
    var projectedValue: Reference<A> {
        Reference<A>(
            get: { self.wrappedValue }
        ) {
             self.wrappedValue = $0 
        }
    }
}

struct Person {
    var name: String
}

struct PersonEditor {
    @Reference var person: Person
}

func makeEditor() -> PersonEditor {
    @Box var person = Person(name: "Gordon")
    PersonEditor(person: $person)
}
```
- Projected value는 Key path를 활용한 동적인 탐색과 혼용했을 때 유용하다.
- 뭐 이를테면, 위에 정의한 데이터 중 사람 이름만 텍스트 에디터에 넘기고 싶을 수 있다.
- 커스텀하게 정의한 `Reference` 타입에 동적 멤버 탐색을 더함으로써 해결할 수 있다.
```Swift
@propertyWrapper
@dynamicMemberLookup
class Reference<A> {
    private var _get: () -> A
    private var _set: (A) -> ()

    var wrappedValue: A {
        get { 
            _get()
        }

        set {
            _set(newValue)
        }
    }

    init(get: @escaping () -> A, set: @escaping (A) -> ()) {
        _get = get
        _set = set
    }

    subscript<B>(dynamicMember keyPath: WritableKeyPath<A, B>) -> Reference<B> {
        Reference<B>(
            get: { self.wrappedValue[keyPath: keyPath] }
        ) {
            self.wrappedValue[keyPath: keyPath] = $0
        }
    }
}

// $person 대신 $person.name을 사용할 수 있게 된다!
// 원래는 $person이 person.projectedValue와 같았다.
// 지금은 $person.name도 person.projectedValue[dynamicMember: \.name]과 같다.
```

#### Enclosing Self
- 몇몇 프로퍼티 래퍼는 그 프로퍼티가 속한 객체에 대해 접근 권한이 있을 때만 작동한다.
    - 가령 `@Published`의 경우, `@Published` 프로퍼티를 가지고 있는 객체의 `objectWillChange` 프로퍼티에 접근한다.
    - 비슷하게, `@Invalidating`으로 마크된 프로퍼티의 변경은, 이 프로퍼티를 가진 뷰의 레이아웃을 비활성화한다.
- 비슷하게 동작하는 것을 구현하려면 비공식 API를 써서 조금 다른 모습으로 구현해야 한다.
```Swift
@propertyWrapper
struct InvalidatingLayout<A> {
    private var _value: A

    // 프로퍼티 래퍼에서 static subscript를 쓰는 것은 아직 정식 API는 아니다
    // https://github.com/swiftlang/swift-evolution/blob/main/proposals/0258-property-wrappers.md
    static subscript<T: UIView>(
        // object를 통해 뷰에 접근
        _enclosingInstance object: T,
        // 프로퍼티가 언더바인 이유는 사용하지 않겠다는 뜻
        // 다만 subscript의 요구사항상 wrapped 값이 필요하므로 추가
        wrapped _:ReferenceWritableKeyPath<T, A>,
        // storage를 통해 자기 자신에 접근 (static 메서드니까)
        storage storage: ReferenceWritableKeyPath<T, Self> 
    ) -> A {
        get {
            object[keyPath: storage]._value
        }

        set {
            object[keyPath: storage]._value = newValue
            object.setNeedsLayout()
        }
    }
    // 참조가 아닌, 원래 목적대로 프로퍼티 래퍼를 사용한다면 불러와질 wrappedValue
    @available(*, unavailable, message: "@InvalidatingLayout is only available on UIView subclasses")
    var wrappedValue: A {
        get { fatalError() }
        set { fatalError() }
    }

    init(wrappedValue: A) {
        _value = wrappedValue
    }
}
```
- SwiftUI의 `@State`도 비슷하게 작동할 것 같지만, 다르다. 감싸인 외부 타입이 값 타입이기 때문.
    - 뷰 계층 상의 뷰의 위치에 따라 SwiftUI 내부적으로 저장하고 다룬다.
    - 이는 Introspection을 통해 이루어진다.

### Property Wrapper Ins and Outs
- 열거형에서는 프로퍼티 래퍼가 동작하지 않는다.
    - 열거형은 독립적인 저장 공간을 가질 수 없기 때문.
    - 프로퍼티 래퍼는 저장 프로퍼티를 항상 갖기 때문.
- 글로벌 변수에는 프로퍼티 래퍼를 쓸 수 없다.
- 프로퍼티 래퍼가 붙었다면, `unowned`, `weak`, `lazy`, `@NSCopying`으로 마크할 수 없다.
- Swift 5.5부터 함수 파라미터로도 프로퍼티 래퍼가 적용된 값을 받을 수 있다.
```Swift
func takesBox(@Box foo: String) {
    // ...
}

// 컴파일러가 바꾼 버전
func takesBox(foo initialValue: String) {
    var _foo: Box<String> = Box(wrappedValue: initialValue)
    var foo: String {
        get { _foo.wrappedValue }
        nonmutating set { _foo.wrappedValue = newValue }
    }
}

// 만약 프로퍼티 래퍼에 `init(projectedValue:)` 초기화 함수가 있다면 컴파일러는 다른 버전의 함수도 만든다 
func takesBox($foo initialValue: Reference<String>) {
    var _foo: Box<String> = Box(projectedValue: initialValue)
    var foo: String {
        get { _foo.wrappedValue }
        nonmutating set { _foo.wrappedValue = newValue }
    }
}
```
- 프로퍼티 래퍼를 통해 정의된 프로퍼티는 또한 해당 타입의 자동 생성 이니셜라이저의 일부분이다.
```Swift
struct Test {
    @Box var name: String
    @Reference var street: String

    // 자동 생성 이니셜라이저는 init(name: String, street: Reference<String>)일 것이다
    // init(wrappedValue:) 메서드가 있다면 그냥 감싸인 타입을 받는다
    // 이게 없다면, 그 "성질"이라고 해야 되나.. 그것까지 같이 주입을 해 줘야 한다
    // 물론 커스텀 이니셜라이저 잘 쓰면 해결
}
```
- 또 프로퍼티 래퍼는 중첩할 수 있다.
- `@State`처럼 introspection에 기반해 작동하는 래퍼는, 일반적으로 그것이 가장 바깥의 래퍼여야지만 작동한다.

# Key Paths
- 단적으로 말해, 프로퍼티에 대한 참조. 백슬래시 기호로 나타낸다.
```Swift
let foo = "Hello"
let number = foo.count
let keyPathNumber = \foo.count
```
- 키 패스에 대해서도 타입 추론이 작동한다. `\.count` 꼴로 나타낼 수 있다.
- 타입 계층에 대한 "Path"를 제공하는 셈이다. 루트 값부터 시작해서.
```Swift
struct Address {
    var street: String
    var city: String
    var zipCode: Int
}

struct Person {
    let name: String
    var address: Address
}

let streetKeyPath = \Person.address.street // Swift.WritableKeyPath<Person, Swift.String>
let nameKeyPath = \Person.name // Swift.KeyPath<Person, Swift.String>
```
- 키 패스는 저장, 연산 프로퍼티 및 옵셔널 체이닝 오퍼레이터와도 혼용할 수 있다.
    - 이 경우 컴파일러는 자동으로 모든 타입에 대해 `[keyPath:]` subscript를 만든다.
    - `"Hello"[keyPath: \.count]`는 `"Hello".count`와 같다.
- 위 예시에 보면 `KeyPath`와 `WritableKeyPath`의 차이가 있는데, 후자는 `var`이라서 모든 키 패스가 값이 변화할 수 있기 때문이다.
- 키 패스는 또한 프로퍼티만 보는 게 아니고, subscript를 순회하는 데에도 쓸 수 있다.
```Swift
let simpsonResidence = Address(
    // ...
)
var bart: Person(name: "Bart Simpson", address: simpsonResidence)

let people = [lisa, bart]
people[keyPath: \.[1].name] // Bart Simpson
```

## Key Paths can be Modeled with Functions
- 키 패스가 만약, 베이스 타입 `Root`부터 `Value` 타입을 가진 프로퍼티로 간다면?
- (Root) -> Value 타입의 함수와 굉장히 비슷하다. 혹은 WritableKeyPath라면, 한 쌍의 함수가 있을 수 있겠다.
- 하지만 함수와 달리 키 패스는 "값 타입"이다.
- 한편 컴파일러는 키 패스 표현을 자동으로 함수로 바꿀 수 있다.
```Swift
people.map { $0.name }
people.map(\.name)

// 다만 반드시 키 패스 표현을 직접 써야 한다. 아래와 같은 코드는 컴파일 에러를 뱉는다.
let keyPath = \Person.name
people.map(keyPath)
```
- 키 패스 표현식을 보면, 컴파일러는 두 가지 선택지를 갖게 된다.
    - `\Person.name`은 `KeyPath<Person, String>`일 수도 있고 `(Person) -> String`일 수도 있다.
    - 키 패스로 먼저 해석을 시도하고, 실패한다면 함수로 해석을 시도한다.
- 키 패스는 더함으로써 조합할 수 있다. 당연히 타입 매칭은 되어 있어야 하겠지만.
```Swift
let nameCountKeyPath = nameKeyPath.appending(path: \.count)
```

## Writable Key Paths
- 이 키 패스는 값 읽기 및 "쓰기"에 활용할 수 있다.
- 상술했듯, 쓰기를 위한 하나의 함수를 더해 한 쌍의 함수를 갖고 있는 것과 비슷하다.
    - `(Root) -> Value` 하나, `(inout Root, Value) -> Void` 하나.
- `WritableKeyPath`, `ReferenceWritableKeyPath`의 두 가지가 있다.
    - 후자의 경우 참조 시맨틱이 있는 값들과 쓸 수 있고, 전자의 경우 나머지 모두를 포괄한다.
    - 전자는 루트 값이 `mutable`할 것을 요구한다.
- Writable Key Path는 SwiftUI 등의 프레임워크에서 쓰인다.
- 이 값의 일부를 바꾸기 위해 `WritableKeyPath`를 요구하는 특별한 함수를 사용할 수 있다.

## The Key Path Hierarchy
- 다섯 가지 다른 타입의 키 패스가 있다. 각각을 함수로 표현한 결과도 있다.
    - `AnyKeyPath`: `(Any) -> Any?`
    - `PartialKeyPath<Source>`: `(Source) -> Any?`
    - `KeyPath<Source, Target>`: `(Source) -> Target`
    - `WritableKeyPath<Source, Target>`: `(Source) -> Target`, `(inout Source, Target) -> ()`
    - `ReferenceWritableKeyPath<Source, Target>`: `(Source) -> Target`, `(Source, Target) -> ()`
        - 이 경우 참조 타입이기 때문에 `inout` 없이도 값을 바꾸는 동작을 할 수 있다.
- 이러한 키 패스의 계층은 일단 클래스 계층으로 구현되어 있다.
- 이상적인 케이스라면 프로토콜이어야 하겠지만..
- 앞으로 릴리즈될 Swift에서는 예고 없이 내부가 바뀔 수도 있다. 물론 작동은 정상적으로 하겠지만.
- 한편 키 패스는 함수와 달리 `Hashable`하다.

## Key Paths Compared to Objective-C
- `Foundation`과 Obj-C에서는 키 패스는 String으로 설계되었다.
    - 키 패스를 위한 별도의 타입이 없었다. 그런 면에선 `AnyKeyPath`와 비슷.
- 그래서 오타가 난다던지 잘못 적으면 크래시가 날 수도 있다.
- Swift의 키 패스들은 견고한 타입으로 되어 있고, 그런 오류가 날 가능성 자체를 봉쇄한다. 컴파일 에러가 나면 났지..
- 많은 Cocoa API는 String 기반의 Obj-C 키 패스를 아직 사용한다.
- 사실 키 패스는 익명함수 같은 문법보다도 더 오래 돼서 그렇다..

## Future Directions
- 키 패스가 `Codable` 프로토콜을 통해 직렬화를 수행할 수 있게 되지 않으려나?
- 키 패스의 구조를 활용해 타입이 잘 갖춰진 데이터베이스 쿼리를 만들 수 있지 않으려나?
- 한편 키 패스는 프로퍼티에 직접 접근하는 것보다 꽤 느리다. 이미 꽤 널리 알려진 버그.
- 대부분의 경우 키 패스가 가져다 주는 명료함이 속도에 대한 트레이드 오프로서 적절하다.
- 하지만 성능이 좀 필요한 코드라면 염두에 두는 것이 좋겠다.

# Recap
- 프로퍼티나 변수는 Swift에서 중요한 부분.
- 저장 프로퍼티와 연산 프로퍼티는 외부에서 사용할 때는 같은 문법을 쓰지만, 굉장히 다른 역할을 수행한다.
- 그리고 사실 프로퍼티 래퍼와 키 패스는 문법적 설탕에 가깝다. 명료한 문법은 명료한 코드를 위해 중요하니까!
- 하지만 이런 문법적 설탕이나 추상화가 으레 그렇듯, 코드를 이해하기 어렵게 만들 수도 있다.