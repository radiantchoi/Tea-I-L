# SwiftUI under the hood
https://medium.com/@vladislavshkodich/mastering-swiftui-are-you-really-as-good-as-you-think-40a4953f7e88
- SwiftUI, 그리고 그로 인해 일어나는 "매직"에 관한 이야기.

## The Basics
- SwiftUI `View`가 `struct`인 이유는 무엇일까? 가벼워서? 그것도 한 가지 이유기는 하다.
- SwiftUI 특성상 라이프사이클을 치르는 동안 매우 여러 번 다시 그려진다. 이렇게 여러 번 그릴 떄 `struct`의 장점이 분명 있다.
  - 가볍다.
  - 고정적인 메모리 크기를 차지한다.
  - 스택 메모리에 들어간다.
  - static dispatch를 사용한다. 상속으로 인한 오버헤드가 없다.
- 당연히 개발자가 코드 짜기 나름이긴 하지만.
- 그럼 왜 SwiftUI는 자꾸 뷰를 다시 그릴까? `State`가 바뀌면 뷰는 다시 그려진다. 예를 들자면..
``` Swift
import SwiftUI

struct ContentView: View {
    @State private var counter: Int = 99

    var body: some View {
        VStack {
            Text("Counter: \(counter)")
            Button {
                counter += 1
            } label: {
                Text("Counter +1")
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }
}
```
- 버튼을 누르면 카운터 값이 바뀌고, 뷰가 다시 그려진다. `@State` 프로퍼티가 바뀌었으니까!
- 우리가 익히 알던 구조체의 동작과는 아무래도 다르다.
  - 다들 `'self' is immutable` 에러 메시지 한 번쯤은 보셨으리라 생각한다. `mutating`을 쓰게 되는 이유기도 하고.
- SwiftUI View는 상태의 함수다, 라고 보면 된다. (이하 WWDC19에서 실제로 한 말)
  - `State`가 바뀌면 뷰는 자동으로 업데이트된 상태에 맞춰 재설정되며, 직접 조작하지 않는다.
  - 이것이 선언형의 정수다!
- SwiftUI가 뷰를 업데이트하게 하기 위한 몇 가지 수단이 있다. 대표적으로..
  - `@Binding`
  - `@StateObject`
  - `@Environment`
  - `@EnvironmentObject`
  - `@Observable` 매크로가 붙은 모델

## Visualization
- 그런데 위의 예시에서, 숫자가 변하는 것을 보여 주는 레이블이 없어진다면 (== 내부적으로만 값이 바뀐다면) 어떨까?
- 과연 뷰가 다시 그려졌을까? 그렇지 않다. 근데 분명 `@State` 값이 바뀐 건 맞잖아!

### Printing Changes
- 이럴 때 쓸 수 있는 꿀 메서드, `let _ = Self.printChanges()`. `body` 프로퍼티 안에 넣어 주자.
  - 이거 공식 문서 없는 함수다! 아무래도 디버깅할 때 쓰던 걸 그대로 남겨 둔 듯.
- 레이블이 없는 상태(== 업데이트할 뷰가 없을 때)에서는 뷰가 한 번 초기화되었을 때, 그리고 counter 값이 바뀔 때마다 콘솔에 한 번씩 출력이 된다.
```Bash
ContentView init()
ContentView: @self, @identity, _counter changed.
```

### Canvas Visualization
- 자 이제 뷰에 변화를 가해 보자.
- 이 글의 원작자는 커스텀 라이브러리를 통해, 뷰가 다시 그려질 때마다 배경색이 랜덤하게 바뀌게끔 했다.
  - https://github.com/vdshko/DebugFrame
- 다시 수를 표시해 주는 레이블을 부활시키고, 버튼을 눌러보면!
- 배경색이 마구 바뀌면서!
```Bash
ContentView init()
ContentView: @self, @identity, _counter changed.
ContentView: _counter changed.
ContentView: _counter changed.
ContentView: _counter changed.
ContentView: _counter changed.
ContentView: _counter changed.
```
- SwiftUI는 해당 값의 변화를 "보고" 있는 뷰 요소가 없다면, 다시 그려지지 않는 걸까?
- 한편 `@ObservableObject`를 사용할 때는, SwiftUI가 조금 다르게 동작한다.
- 예시 코드를 바꿔 보자!
```Swift
import SwiftUI

final class ContentViewModel: ObservableObject {
    @Published var counter: Int = 99
}

struct ContentView: View {
    @StateObject private var model: ContentViewModel = ContentViewModel()

    init() {
        print(Self.self, #function)
    }

    var body: some View {
        let _ = Self._printChanges()
        VStack {
//            Text("Counter: \(counter)")
            Button {
                model.counter += 1
            } label: {
                Text("Counter +1")
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
        .debugBackground()
    }
}
```
- 이 경우 찍히는 로그는..
```Bash
ContentView init()
ContentView: @self, @identity, _model changed.
ContentView: _model changed.
ContentView: _model changed.
ContentView: _model changed.
ContentView: _model changed.
```
- `ObservableObject`를 사용하면 값의 변화를 "보고" 있는 뷰가 없음에도! 뷰가 다시 그려진다.
- 한편 `@Observable` 모델을 사용했을 때는 `@State`처럼, 뷰가 다시 그려지지 않는다.

## Task
- 이 "다시 그려지는 타이밍"이 실제 사례에서 문제가 될 수 있을까?
- 10만 개의 원소가 있느 리스트를 뷰에 더한다. 잘 빌드된다.
- 하지만 뷰가 다시 그려질 때, 이는 버벅임을 유발한다.
```Swift
import SwiftUI

@Observable
final class ContentViewModel {
    var counter: Int = 99 {
        didSet {
            print(counter)
        }
    }

    var values: [Int] = [Int](1...100_000)
}

struct ContentView: View {
    var model: ContentViewModel

    init(_ model: ContentViewModel) {
        self.model = model
        print(Self.self, #function)
    }

    var body: some View {
        let _ = Self._printChanges()
        VStack {
            List {
                ForEach(model.values, id: \.self) {
                    Text(String($0))
                        .padding()
                        .debugBackground()
                }
            }
            .listStyle(.plain)
            .frame(height: 200.0)
            Text("Counter: \(model.counter)")
            Button {
                model.counter += 1
            } label: {
                Text("Counter +1")
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
        .debugBackground()
    }
}
```
- 버벅이는 이유는, 리스트 셀 10만개를 다시 그리기 때문..
  - 리스트가 화면에 자리를 잡으려면 일단 안에 몇 개 든지 알아야  하기 때문이다.
- 그럼 `List`를 `LazyVStack`이랑 `ScrollView`로 바꾸면 되지 않으려나? 같은 소리 하지 말고..

## SwiftUI Under the Hood
- SwiftUI 세계에는 뷰하고, 뷰와 **관련 있는** 몇몇 값이 있다.
  - 뷰가 상태를 갖지 않는 UIKit 세계에서는 굉장히 생소한 개념! SwiftUI 뷰는 상태를 가지니까.
- State 하나가 바뀌면, 뷰가 의존하는 새로운 "상태"가 생긴다.
- 이 떄 SwiftUI 상에서 가장 먼저 일어나는 일은, 새 상태에 의존하는 새 뷰를 만든 다음 기존 상태와 비교하는 것이다.
  - 만약 기존 상태와 지금 상태가 같다면, SwiftUI는 만들었던 새 뷰를 버려버리고, 아무 일도 일어나지 않는다.
  - 하지만 다르다면, 각각의 뷰의 `body`를 호출하여 서로 같은지 비교한다.
- 바디 간의 비교를 수행해 같다면, 마찬가지로 아무 일도 일어나지 않는다.
- 하지만 다르다면!! SwiftUI는 새 뷰를 가져와서 뷰 구조에 넣고, 변경사항을 반영하기 위해 새로이 렌더링한다.
- 그런데 그렇다면 그 "비교"는 어떻게 일어나는 걸까? 뒤에서 성능에 대해 이야기하면서 살펴보자.

## Performance

### Rule 1: Keep your View body simple
- `body` 안에서 복잡한 작업을 하지 말자.
  - 대표적으로 `ForEach` 안에 넣는 컬렉션에 `filter`, `map` 등이 덕지덕지 붙은 경우! 계산은 밖에서 다 하고 들어오자!
- 시스템이 한 번의 레이아웃 결정 주기 동안 바디를 여러 번 호출할 수 있기 때문에 이는 중요한 요소이다.
  - `GeometryReader`의 사용과 같은 경우가 대표적.

### Rule 2: Preferring no effect modifiers over conditional views
- 예시로 볼까?
```Swift
// Avoid if possible ...
var body: some View {
  VStack {
    if isHighlighted {
      CustomView()
        .opacity(0.8)
    } else {
      CustomView()
    }
    // ...
  }
}

// Prefer ...
var body: some View {
  VStack {
      CustomView()
        .opacity(isHighlighted ? 0.8 : 1.0)
    // ...
  }
}
```
- 뷰 트리에 뷰 한 개를 더할 거를, 두 개를 더하는 격이다.
  - SwiftUI는 두 뷰가 같은 것임을 알지 못한다! 애니메이션도 안 되고, 그냥 갈아끼기 작업을 수행할 뿐이다.
  - 그러니 조건의 만족을 위해 뷰를 두 개 들고 있어야 하는..
- 기본적으로, 같은 뷰의 두 가지 상태에 대해 다룰 때 어떤 식으로 코드를 쓸지 생각해보고 쓰도록 하자.

### Rule 3: Split "states" parts into custom view types
- 크고 아름다운 뷰, 되게 긴 바디 코드. 바디는 이 뷰의 상태에 따라 분기로 다른 뷰 조각을 쓴다고 가정한다.
- 하지만 이게 하나의 바디에 다 구겨져있기 때문에
  - 뷰 전체를 다시 렌더링할 때 바디 전체를 검사하게 될 테고
  - 비교하는 데 시간도 오래 걸리고
  - 분기에 따라 다양한 선택지의 뷰를 들고 있어야 할지도 모르고
  - 그런데 이게 상태가 바뀔 때마다 일어난다면..
- 기억하면 좋을 것은, 애플 SwiftUI 튜토리얼에도 드러나 있는 방법이다. "작은 뷰의 조합으로 큰 뷰 만들기"
  - 정확히는, 각각의 subview에 각각이 가져야 할 `state`를 떠넘기자. `@Binding`이라는 좋은 게 있잖아!
- 이렇게 하면 `body`의 비교 과정에 있어서, 영향을 받지 않는 Subview들은 비교할 필요도 없다! 이 뷰의 `body`들은 일차적으로는 안 불러와진다.
  - 물론 차이점이 발생하면 호출되겠지만.
  - 대부분의 경우 이 subview들의 `body`의 코드를 비교하는 것이 아닌, 그저 새로 생성하는 과정이 일어난다.

## Comparing
- 들어가기 앞서, 공식 의견은 아님에 주의! 애플 개발자들이 트위터나 포럼 등지에서 한 얘기들을 끌어모은 것이다.
- SwiftUI에서 뷰를 비교하기 위해 쓰이는 세 가지 메서드가 있다.
  - `memcmp`: memory comparition. C 함수라 굉장히 빠르다. 
    - 두 구조체 간에 실제 차지하는 메모리 바이트를 비교한다. 이전 상태와 새로운 상태 간의 비교에 쓰인다.
  - `== (Equality)`: 조금 느리지만 오케이! `View`가 `Equatable`하다면 써봄직하다.
  - `Reflection`: 느리다. 최후의 보루이며 기본값 정도 된다.
- 이런 관점에서, 퍼포먼스 문제를 겪는다면 뷰를 다시 만들어 보시라! SwiftUI가 더 빠르게 비교할 수 있을 것.
- 그리고 여기서 새롭게 제시하는 개념, POD(Plain Old Data).
  - 값들이 모여 있는 간단한 구조체라고 생각하면 편하다.
- 예시를 통해 비교해보자면..
```Swift
// POD
struct ContentView: View {
    var counter: Int
    let isHighlighted: Bool

    // ...
}

// POD 아님
struct ContentView: View {
    @StateObject private var counter: ContentViewModel
    @State var isHighlighted: Bool

    // ...
}
```
- 내 뷰가 POD인지 어떻게 아느냐? `_isPOD` 함수를 활용해보면 된다.
```Swift
print(_isPOD(ContentView.self))
```
- 오로지 값 타입만 들어있는 구조체라면 POD라고 할 수 있다.
  - 하지만 여기서도 `String`은 value semantic을 가지고 있을 뿐 COW 메커니즘을 사용한다! POD 탈락!
- 근데 POD인 게 왜 중요해? 
- SwiftUI에서 뷰를 비교할 때, POD인 뷰는 `memcmp`를 사용하고, 그렇지 않으면 Reflection을 사용하기 때문이다!!
  - 당연히 비교 속도가 엄청 차이날 수 밖에 없다!!

## Back to the Task
- 자 다시 10만 개의 셀을 가진 리스트로 돌아왔다. 예시 코드를 조금 바꿔 보자.
```Swift
import SwiftUI

@Observable
final class ContentViewModel {
    var counter: Int = 99 {
        didSet {
            print(counter)
        }
    }

    var values: [Int] = [Int](1...100_000)
}

struct ContentView: View {
    var model: ContentViewModel

    init(_ model: ContentViewModel) {
        self.model = model
        print(Self.self, #function)
    }

    var body: some View {
        let _ = Self._printChanges()
        VStack {
            ContentListView(values: model.values)
            Text("Counter: \(model.counter)")
            Button {
                model.counter += 1
            } label: {
                Text("Counter +1")
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
        .debugBackground()
    }
}

// MARK: - Inside types
private extension ContentView {
    // Rule 3를 활용해, 리스트를 활용하는 뷰를 별도의 서브뷰 타입으로 분리했다.
    // 한편 이를 private nested type으로 만들어서, 그저 ContentView의 일부일 뿐 불필요하게 드러나지 않게 했다.
    // Equatable 프로토콜을 채택해 최악의 경우에도 Reflection 비교를 사용하지 않게 했다.
    struct ContentListView: View, Equatable {
        // POD 뷰가 아닌 이유는, 배열도 value semantic이 있을 뿐 COW를 사용하기 때문.
        let values: [Int]

        init(values: [Int]) {
            self.values = values
            print(Self.self, #function)
        }

        var body: some View {
            let _ = Self._printChanges()
            List {
                ForEach(values, id: \.self) {
                    Text(String($0))
                        .padding()
                        .debugBackground()
                }
            }
            .listStyle(.plain)
            .frame(height: 200.0)
            .padding()
            .debugBackground()
        }
    }
}
```
- 이러한 방법들을 활용하면, 불필요하게 다시 비교하고 다시 그려지는 부분을 확 줄여서, 빠른 퍼포먼스를 낼 수 있다!

## Conclusion
- SwiftUI와 싸우지 말고 이용하자!
- `ObservableObject`보다는 `@Observable`을 사용하자!
- `body`의 비교, modifiers, 그리고 분리에 대한 원칙을 숙지하자!
- 비교에도 종류가 있음을 상기하자! - `memcmp`, Equality, Reflection
- POD 여부를 체크하고 싶으면 `_isPOD(T.self)` 메서드도 활용해 보자!!