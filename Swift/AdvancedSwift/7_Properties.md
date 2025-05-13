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

### Projected Values
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