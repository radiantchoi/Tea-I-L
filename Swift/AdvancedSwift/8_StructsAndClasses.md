# Structs and Classes
- Swift에는 `struct`와 `class`라는 굉장히 비슷해 보이는 두 가지 선택지가 있다.
- 하지만 이쯤 되면 알지, 둘은 근본적으로 다르다는 것을.
- `struct`는 값 타입, `class`는 참조 타입.

## Value Typees and Reference Types
- 둘의 가장 근본적인 차이는 값의 복사 할당의 방식에 있다.
- 값 타입은 각각의 독립적인 값을 가지며, 이를 value semantics라고도 부른다.
- 그런데 도대체 변수가 뭘까? 한 마디로 말하면, 어떤 타입의 값을 저장하고 있는 메모리 위치에 붙인 이름이다.
- `var a = 1; a += 1`은, a라고 이름붙인 메모리 주소 상의 값을 따와서, 1을 더하고, 더한 값을 a 메모리에 다시 저장하는 것이다.
- 값 타입의 경우, 변수 뒤에 있는 메모리에 바로 할당이 되어 있다. 에잇 뭔 말인지 알지?
    - 이는 기본 타입부터 한층 복잡한 `struct`까지 모두 유효한 말이다.
- 한편 참조 타입은 복사를 해도..
```Swift
var view1 = UIView(frame: CGRect(x: 0, y: 0, width: 100, height: 100))
var view2 = view1
view2.frame.origin = CGPoint(x: 50, y: 50)

view1.frame.origin // (50, 50)
view2.frame.origin // (50, 50)
```
- 뷰 1과 2가 같다는 것을 어렴풋이 느낄 수 있지?!
- 참조 타입 프로퍼티는 값 그 자체가 아닌 값이 할당되어 있는 메모리 주소를 가리키는 참조를 갖기 때문이다.
- 참조 타입은 존재하는 무언가 그 자체를 갖지 않고, 참조한다. 포인터를 갖는다고 하자!
    - 이는 또한 reference semantics라고도 할 수 있다.
- 이 "포인터" 덕분에, 프로그램의 여러 다른 부분에서 이 객체에 접근할 수 있다.
- 아무튼! `struct`를 다른 변수에 할당하는 것은 독립적인 복사본을 만든다.
- 비슷하게, 함수 파라미터로 전달하는 것 또한 독립적인 복사본을 만든다!
- 그리고 이 파라미터로 전달된 값은 불변값이다.
- 값 타입을 직접 바꾸려면, `inout` 파라미터를 사용해야 한다.
- 클래스가 좀 더 유능하긴 한데, 이 유능함은 또한 비용이 되기도 한다.
- Vice versa. 구조체는 더 제한이 많지만, 이 제한이 이득이 될 때도 있다.

## Mutation
- 구조체와 클래스는 값을 바꿔야 할 때 그 차이가 더욱 더 크게 와 닿는다.
- 구조체의 변경은 지역적 변경인 반면, 클래스의 변경은 전역적인 영향을 끼칠 소지가 있다.
- `let` 변수에 클래스를 할당하면 클래스 인스턴스를 변경할 수 있지만, 구조체는 그렇게 안 된다.
- `let`으로 선언한다는 것은 사실 초기화 이후에 변경을 하지 않을 것이라는 선언이다.
- 하지만 참조 타입에 있어, 이는 단지 이 변수에 할당한 클래스 인스턴스를 바꿀 수 없다는 정도의 제약이다.
    - 이 인스턴스 내부의 `var` 프로퍼티를 변경하는 것은 자유롭다.
- 값 타입을 `let`으로 선언했다면, 이건 진짜로 그 실제 값을 가지고 있는 경우다. 따라서 바꿀 수 없다. 프로퍼티조차도.
    - 구조체의 프로퍼티를 변경한다는 것은, 새 구조체를 할당한다는 말과 같기 때문이다.
    - 이는 구조체 내부의 프로퍼티(구조체) 내부의 프로퍼티.. 를 바꾸는 경우에도 마찬가지이다.
- 한편 클래스 내부의 값 타입 프로퍼티가 `let`으로 설정되어 있다면, 마찬가지로 바꿀 수 없다.
- 값 타입 프로퍼티에서는 `var`를 기본적으로 쓰는 것을 추천한다.
- 참조 타입 프로퍼티는 `var`로 정의하면 전역적 변화의 위험이 있지만, 값 타입은 그렇지 않다.

### Mutating Methods
- 구조체에 있는 일반적인 메서드들은 그 구조체의 어떠한 프로퍼티도 변경할 수 없다.
- 모든 함수의 파라미터에 암시적으로 할당되는 `self`가, `let`으로 설정된 것 처럼 불변하기 때문.
- 정 프로퍼티를 변경하고 싶다면, `mutating` 키워드를 함수 앞에 붙여 줘야 한다.
- 컴파일러는 `mutating` 키워드를, `let`으로 선언되었을 때 불러올 수 없는 메서드를 표시하는 수단으로 인식한다!!
    - `let` 변수는 불변한데, `mutating` 함수는 내부의 프로퍼티를 바꿈으로써 새 구조체 인스턴스를 변수에 넣을 수 있으니.
- 프로퍼티와 `subscript`의 setter는 암시적으로 `mutating`이다.
- 가끔 setter가 `mutating`하지 않은 연산 프로퍼티를 정의하고 싶다면, `nonmutating` 키워드를 `set` 앞에 붙여 주면 된다.
- 한편 클래스는 이런 키워드가 필요하지 않다. 앞에서 봤듯, `var`로 정의되어 있다면 그냥 바꾸면 된다.

### inout Parameters
- `mutating` 메서드는 변경 가능한 `self`에 접근 가능하게 함으로써 그 모든 프로퍼티에 접근 가능하다.
- `inout` 파라미터를 가진 메서드는 프로퍼티 중 하나를 콕 집어서 접근할 수 있게 한다.
- `inout`을 쓰려면 두 가지를 먼저 해야 한다.
    - 당연히 `var` 프로퍼티에 접근해야 한다.
    - 변수 이름 앞에 `&`를 붙여 줘야 한다.
- 이 앰퍼샌드는 C 혹은 Obj-C에서의 포인터 같은 느낌이 들겠지만, 그것과는 다르게 동작한다.
- 여타 파라미터처럼, 파라미터로 입력받는 값의 복사본을 전달한다.
- 단지 함수 실행이 끝난 후 원래 프로퍼티로 다시 복사되어 반영된다는 차이가 있는 것이다.
- 함수를 실행한 사람은 단지 실행이 끝나고 나니 프로퍼티 값이 바뀌어 있더라, 정도만 볼 수 있다.
- 그렇기 때문에, `inout` 파라미터로 전달받은 값에 변환을 가하지 않더라도 이 복사 - 재복사의 과정은 동작한다.
    - `willSet` 혹은 `didSet`이 작동하는 걸 볼 수 있을 것

# Lifecycle
- 구조체와 클래스는 그 라이프사이클의 측면에서 사뭇 다르다.
- 구조체는 오우너가 둘 이상이 될 수 없기 때문에 해당 변수의 라이프사이클에 매여 있다.
- 한편 클래스는 여러 오우너로부터 소유당할 수 있고, 더 정밀한 메모리 관리가 필요하다.
    - 그래서 등장한 ARC!
    - reference count가 0이 되면 deinit을 호출하고 메모리를 비운다.
    - 이러한 클래스의 "공유 가능" 특성 덕에 파일 핸들링이나, 뷰 컨트롤러 등에 쓰인다.

## Reference Cycles
- 둘 이상의 객체가 서로를 강하게 참조하고 있어서(참조를 들고 있어서) 해제될 수 없게 되는 경우.
- 메모리 누수, 클리닝 과정의 누락 등의 문제가 있다.
- 당연하지만 값 타입인 구조체 사이에는 순환 참조가 발생할 수 없다.
    - 근데 오히려 그래서, 사이클릭한 자료구조는 구조체로 만들 수 없다.
- 순환 참조는 두 개의 객체 간에서 발생하는 것부터 객체와 객체 간의 복잡한 의존 관계까지, 다양하다.
```Swift
class Window {
    var rootView: View?
}

class View {
    // 예시 코드긴 한데 사실 여기서부터 순환 냄새를 맡아버림
    // window 프로퍼티는 옵셔널이 되는 게 맞다
    var window: Window
    
    init(window: Window) {
        self.window = window
    }
}

var window: Window? = Window() // 1
window = nil // 0

var window: Window? = Window() // 1, 0
var view: View? = View(window: window!) // 2, 1
window?.rootView = view // 2, 2
view = nil // 2, 1
window = nil // 1, 1
```
- 그래프 같은 자료구조를 다룰 때, 이러한 순환 참조로 인한 메모리 누수에 대해 유의할 필요가 있다.

### Weak References
- 순환 참조를 깨기 위해, 참조 중 하나를 `weak` 혹은 `unowned`로 지정해야 한다.
- `weak` 변수에 객체를 할당하는 것은 참조 카운트를 올리지 않는다.
- 또한 원본 객체가 참조 해제될 경우 `nil`로 변한다.
- 그래서 `weak` 변수는 항상 `Optional`이어야 한다.
- 약한 참조는 delegate 패턴을 사용할 때 특히 유용하다.

### Unowned References
- 그런데 가끔은 약한 참조지만 옵셔널이 아닌 것을 쓰고 싶을 수도 있다.
- `unowned`는 이런 경우의 해법.
- `unowned`를 사용할 경우, 참조당하는 객체가 참조하는 객체보다 라이프사이클이 길 것을 보장해야 한다.
    - 그것을 보장할 책임은 개발자에게 있다..
- 흔히 아는 것과 다르게, Swift 런타임은 `unowned` 참조 카운트를 추적하는 또다른 참조 카운터가 있다.
    - 강한 참조만 다 해제되면 모든 리소스를 비우긴 한다.
- `unowned` 참조가 모두 해제되지 않으면 메모리 자체는 해제되지 않으며, invalid로 표시된다.
    - 좀비 메모리가 되는 것.
    - 만약 해당 메모리 구역에 접근하면 런타임 에러가 나는 것이다. 예기치 못한 동작이 아니다.
    - 이러한 제한 또한 `unowned(unsafe)`로 우회할 수 있지만, 이 때는 진짜 예기치 못한 동작이 생길수도.

## Closures and Reference Cycles
- 참조 타입에는 `actor`하고 함수도 포함되어 있지롱~
- 클로저가 참조 타입 변수를 캡쳐한다면 이 클로저는 그것에 대해 강한 참조를 갖는다.
    - 이것이 또한 흔히 일어나는 순환 참조 패턴!!
```Swift
class Window {
    weak var rootView: View?
    // Swift에서는 함수 프로퍼티는 약한 참조로 정의할 수 없다
    var onRotate: (() -> Void)? = nil
}

var window: Window? = Window()
// view에서 만약 window를 약하게 참조한다면, 바로 deinit되어 버릴 것이다. 참조 카운트가 0일 것이기 때문
var view: View? = View(window: window!)
// 윈도우는 클로저를 참조하고, 클로저는 뷰를 참조하며, 뷰는 윈도우를 참조한다.
window?.onRotate = { 
    print("We now also need to update the view: \(String(describing: view))")
}

// 방법은 클로저의 약한 캡쳐를 사용하는 것 뿐
window?.onRotate = { [weak self] in 
    print("We now also need to update the view: \(String(describing: view))")
}

// 캡쳐 리스트는 생각보다 여러 가지를 할 수 있다
// 임의의 프로퍼티 캡쳐하여 초기화하기, 전혀 관련없는 값 넣기 등
// 스코프 빼고는 클로저 바로 위에 변수 정의하기랑 똑같다
window?.onRotate = { [weak self, weak myWindow = window, x = 5 * 5] in 
    print("We now also need to update the view: \(String(describing: view))")
    print("window: \(String(describing: myWindow)), x: \(x)")
}
```

## Choosing between Unowned and Weak References
- 이 질문의 답은 사용하는 객체의 라이프사이클과 관련이 있다.
- 참조하는 객체가 참조당하는 객체보다 라이프사이클이 짧거나 같다는 게 보장되면, `unowned`가 의외로 편할 수 있다.
    - 옵셔널 다루기를 안 해도 된다.
    - let으로 선언할 수 있다.
- 하지만 그게 아니라면 `weak`만이 유일한 선택지이다.
- 객체가 부모-자식 관계라면, 의외로 라이프사이클이 같은 경우는 흔하다.
- 또 `unowned`가 `weak`보다 오버헤드가 적고, 그래서 쬐금 더 빠르다.
- 역시 그래도 `unowned`가 가진 크래시 가능성을 무시할 순 없다. 라이프사이클 예상을 잘못 했다면?
    - 이러한 케이스는 리팩토링 과정, 그리고 전임자 이슈(...)로 정말 흔히 일어날 수 있다.
    - 하지만 오히려 일부러 앱이 터지게 만들어서, 테스트를 할 때 발견되게 하는 경우도 있다(!!).

# Deciding between Structs and Classes
- 애시당초 타입 설계할 때부터, 이 타입의 인스턴스가 같은 상태로 프로그램의 다른 여러 곳에 공유 사용될 만 한지 생각해봐야 한다.
    - 이것이 그냥 클래스와 구조체를 고르는 기준이다(...).
- 공유되지 않는 타입, 뭐 예를 들면 `Int`, `Bool`, `URL`... 이런것들은 구조체다.
    - 이런 것들은 그냥 값만 같으면 된다.
- 그런데 `UIView`로 넘어오면 얘기가 확 다르다.
    - 모든 값이 똑같은 객체가 하나 더 있더라도, 이는 아이덴티티가 같다고는 볼 수 없다.
    - 뷰 계층 상에 있는 특정한 요 뷰! 거기에 조작을 가해야 한다. 이 과정에서 아이덴티티가 보존되어야 한다.
    - 이럴 때는 클래스를 쓴다. `UIView`만 해도 상위 뷰와 하위 뷰 모두에게서 참조가 된다. 뷰 컨트롤러도 얘를 참조하고.
- 바꿔 말하면, 소유권 공유가 필요하지 않은 경우는 굳이 클래스를 쓸 필요가 없다.
- 반대로, 소유권 공유 개념 자체가 클래스 없이는 성립이 안 되기도 한다.
    - 만약 이런 케이스에서 뭔가 바꾸려면.. 앱의 머리부터 발끝까지 다시 계산하라고?
- 구조체는 클래스보다 한결 무능하지만, 한결 심플하다. 걱정거리가 적단 얘기.
- 구조체는 특히 작은 값에서 더 좋은 성능을 보장하기도 한다.
    - `Int`가 만약 참조 타입이고, 이를 담은 배열이 있다면?
    - 포인터 저장을 위한 추가 메모리가 필요하고, 메모리상에 값이 흩어져 있다면 순회도 더 오래 걸리고...

# Classes with Value Semantics
- 밸류 시맨틱은 구조체나 열거형에 쓰는 거 아닌가?
- 밸류 시맨틱처럼 작동하게 할 수는 있다.
    - 모든 프로퍼티를 `let`으로
    - 클래스 자체는 `final`로
    - 모든 사용 시나리오에서 독립적인 값을 가진 것처럼 쓸 수 있다. 어쨌든 절대 바뀔 수 없는 값뿐이니까(...).
- `NSArray`가 놀랍게도 이런 케이스!
    - 그 자체로는 어떠한 값을 변화시키는 API가 없다.
    - `NSMutableArray`는 따로 있다.. 에이 뭐야
    - 그런데 그렇기 때문에 값을 받아올 때, 조작을 가하기 전에는 `NSArray`로 받아오는 것이 효과적인 것이다.

# Structs with Reference Semantics
- 한편 참조 타입 프로퍼티를 가진 구조체도 재밌는 방식으로 작동한다.
```Swift
struct ScoreStruct {
    var home: Int
    var guest: Int
    let scoreFormatter: NumberFormatter
    
    init(home: Int, guest: Int) {
        self.home = home
        self.guest = guest
        scoreFormatter = NumberFormatter()
        scoreFormatter.minimumIntegerDigits = 2
    }

    var pretty: String {
        let h = scoreFormatter.string(from: home as NSNumber)!
        let g = scoreFormatter.string(from: guest as NSNumber)!
        return "\(h) – \(g)"
    }
}

let score1 = ScoreStruct(home: 2, guest: 1)
score1.pretty // 02 - 01

let score2 = score1
// 이게 되는 것부터 눈치를 깔 수가 있다
score2.scoreFormatter.minimumIntegerDigits = 3

score1.pretty // 002 - 001
```
- 위와 같은 현상이 벌어지는 이유는 `numberFormatter`가 참조 타입이기 때문이다.
    - `score1`과 `score2`는 독립적인 인스턴스를 갖지만, 각각의 프로퍼티는 `numberFormatter`에 대한 참조를 갖는다.
- 구조체의 할당은 "복사"지만, 이 경우 복사한 것은 `numberFormatter`를 가리키는 일종의 포인터.
- 결국 값이란 무엇인가 라는 관점에서 갈릴 수가 있다.
- 위 예시와 같은 경우는 타입을 클래스로 바꿈으로써 혼란을 피할 수 있다.
- 아니면 `numberFormatter`를 별도로 정의하던지.
- 위에 나온 포매터 프로퍼티를 private하게 만드는 방법도 있긴 할건데, 완전한 방법은 아니다.
- 구조체 안에 참조를 넣는 것은 그래서 매우 경계되어야 한다. 예기치 못한 동작이 일어날 수 있다.
- 하지만 가끔, 참조를 저장하는 것이 진짜 필요하고 의도된 경우가 있다. 예를 들면~

# Copy-On-Write Optimization
- 값 타입은 복사가 굉장히 많이 이루어진다.
- 그렇다면 복사를 굳이 하지 않아도 될 때는 복사하지 않는 방법이 있지 않을까?
- 이럴 때 사용되는 최적화 기법이 바로 Copy On Write!
- 특히 큰 데이터를 가질 수 있는 타입에서 유용하다. `Collection` 타입이라던지.
- 핵심은, (복사) 초기 상태에는 구조체에 들어 있는 데이터를 여러 변수가 공유한다는 것이다.
    - 이후 일부 변수에서 값의 변화가 일어나면 그제서야 값을 복사해서 다른 메모리 버퍼에 할당한다.
    - 이는 곧, 여러 변수에서의 변경이 독립적이라는 이야기도 된다.
- COW는 직접 구현해야 하지만, 대부분의 경우 그럴 일은 없다.
    - 대개 많은 데이터를 갖는 건 `Collection`이고, Swift는 이미 이들에 대한 COW를 잘 구현해 두었기 때문.

## Copy-On-Write Tradeoffs
- 내부 구현에 대해 탐구하기 전! COW의 트레이드오프부터 알아보자.
- 통상적인 값 타입과 달리, COW 구조체는 내부적으로 참조를 사용한다.
    - 이 때문에 COW 구조체가 변수에 복사되면 참조 카운트가 늘어난다.
    - 상술한 "참조를 저장하는 것이 진짜 필요하고 의도된 경우".
- 참조 카운트를 올렸다 내렸다 하는 건 의외로 꽤 느린 작업이다.
    - 스레드-세이프해야만 하는 작업이고, 락 등의 오버헤드가 그에 따라 발생한다.
- 이는 또한 COW 구조체를 프로퍼티로 가진 타입에도 적용되는 문제이다.
    - SwiftNIO 프로젝트의 예시. HTTP Request와 같은 타입은, 하다못해 `String`을 많이 가지고 있고, 그에 따라 이러한 오버헤드가 엄청나게 발생한다.
    - Request를 함수의 파라미터로 넘긴다면? (끔찍)

## Implementing Copy-On-Write
- 위에서 말한 예시를 아주 간략하게 만들어서 시작해 볼까?
```Swift
struct HTTPRequest {
    // 1. 원래 상태. 생략된 프로퍼티들도 대개 Collection 타입이므로 오버헤드가 엄청나다
    var path: String
    var headers: [String: String]
    // 생략된 다른 프로퍼티들...

    // 참조 카운팅 오버헤드부터 줄여 보자
    fileprivate class Storage {
        var path: String
        var headers: String

        init(path: String, headers: [String: String]) {
            self.path = path
            self.headers = headers
        }
    }

    // 2. HTTPRequest가 갖는 참조는 이제 storage 하나밖에 없게 됐다
    private var storage: Storage

    init(path: String, headers: [String: String]) {
        storage = Storage(path: path, headers: headers)
    }
}

// 3. path와 headers를 노출하기 위한 연산 프로퍼티
// 4. COW처럼 작동하게 하려면 path 혹은 headers가 바뀌었을 때마다 값을 새로 할당한다
extension HTTPRequest {
    var path: String {
        get { storage.path }
        set { 
            storage = storage.copy()
            storage.path = newValue
        }
    }

    var headers: [String: String] {
        get { storage.headers }
        set {
            storage = storage.copy()
            storage.headers = newValue
        }
    }
}

// 테스트용 print 오버로딩
var printedLines = [String]()
func print(_ value: Any) {
    var output = ""
    Swift.print(value, terminator: "", to: &output)
    printedLines.append(output)
    Swift.print(output)
}

extension HTTPRequest.Storage {
    func copy() {
        print("Making a copy...")
        return HTTPRequest.Storage(path: path, headers: headers)
    }
}

// 지금까지의 구현으로는 인스턴스가 하나일 때도 mutation만 일어나면 복사본이 생성된다.
// 이를 해결하기 위해 객체 참조 카운트가 하나인지 알아야 할 필요가 있다.
// 오우너가 하나라면 그 자리에서 변경하고, 둘 이상이면 일단 복사 후 변경하는 것

// 5. isKnownUniquelyReferenced의 활용
extension HTTPRequest {
    private var storageForWriting: HTTPRequest.Storage {
        mutating get {
            if !isKnownUniquelyReferences(&storage) {
                self.storage = storage.copy()
            }

            return storage
        }
    }

    // path, headers 연산 프로퍼티의 재작성
    var path: String {
        get { storage.path }
        set { storageForWriting.path = newValue }
    }

    var headers: [String: String] {
        get { storage.headers }
        set { storageForWriting.headers = newValue }
    }
}

// isKnownUniquelyReferenced에 전달하는 클래스에 다른 스레드가 접근하지 않음을 보장해야 한다.
// Race Condition 발생 우려가 있음..
// 이는 모든 inout 파라미터에서 주의할 점인데, Swift 함수에서는 inout 없이는 컴파일러가 값을 "복사"하기 때문에 어쩔 수 없다.
// 복사된 객체는 함수 바디 안에서 참조가 1 늘어나니까, 절대 true가 반환될 수 없는 셈.
// 또한 Obj-C 클래스는 인자로 넘길 수 없다.
```

## `willSet` Defeats Copy-On-Write
- 사실 COW 타입 프로퍼티의 `willSet` 옵저버가 이 모든 구현을 쓸모없게 한다는 사실(...)
- `newValue`라는 임시 프로퍼티를 바디 안에 갖고 있어서, 컴파일러로 하여금 임시 복사본을 만들게 하기 때문.
- 결과적으로, 해당 값은 더 이상 유니크하게 참조되지 않게 되며, 모든 mutation이 그 복사본을 만들게 한다.
```Swift
struct Wrapper {
    var req = HTTPRequest(path: "/", headers: [:])
    var reqWithWillSet = HTTPRequest(path: "/", headers: [:]) {
        willSet {
            print("willSet")
        }
    }
    var reqWithDidSet = HTTPRequest(path: "/", headers: [:]) {
        didSet {
            print("didSet")
        }
    }
}

var wrapper = Wrapper()
wrapper.req.path = "/about"
// 재미있게도, didSet에서는 이러한 현상이 일어나지 않는다.
wrapper.reqWithDidSet.path = "/forum" // didSet

wrapper.willSet.path = "/blog"
// Making a copy...
// willSet
```
- `willSet`의 호출 과정을 되짚어보면..
    - 현재 값의 복사본이 만들어진다.
    - 복사본이 변경된다.
    - `willSet` 호출시 그 바디 안에서 이 복사본은 `newValue`라는 이름이 붙는다
    - 그리고 이 복사본이 원본에 반영된다.
- 그런데 `didSet`도 `oldValue`라는 이름으로 이전 값에 액세스 가능한데, 왜 안 불려오나?
    - 컴파일러가 `oldValue`가 쓰이지 않았음을 코드 상에서 감지했기 때문이다.
    - `willSet`의 `newValue`에 대해서는 이 감지가 동작하지 않는다.
- 한편으로는 이해할 수 있는 것이, 만약 `newValue` 유무에 따라 컴파일이 다르게 된다면..
    - 변경 "이전에" 어떤 로직을 실행하는 `willSet` 블록이 만들어진다. (현재의 `willSet`은 어쨌든 일종의 변경 이후)
    - 변경의 사이드 이펙트가 일어나기 이전에 `willSet`에 들어 있는 어떤 사이드 이펙트가 실행될테고..
    - 지금 상태는 어쨌든, `willSet` 이전에 사이드 이펙트가 일어나게끔 보장이 되어 있으니까.
- 이러한 `willSet`의 동작 방식은 SwiftUI 사용시에 퍼포먼스 문제를 일으킬 수 있다.
    - 다름아닌 `ObservedObject` 내부의 `@Published`가 내부적으로 `willSet`을 활용하기 때문.

# Recap
- 값 타입과 참조 타입의 근본적인 작동방식 차이에 대해 논했다.
- `let`, `var`, `mutating`, `inout`에 대해 논했다.
- Copy-On-Write에 대해 논했다.