# SwiftUI vs UIKit: Lifecycle 차이를 통한 탐구
https://o-kaledin.medium.com/swiftui-vs-uikit-navigating-through-lifecycle-differences-f6182b31b960
- iOS의 대표적인 UI 프레임워크로 꼽히는 두 가지! 그런데 생명 주기 관리방법이 다르다??

## UIKit: 전통적인 접근법
- iOS UI 개발의 "Backbone".
- View를 관리하기 위해 View Controller에 매우 강하게 의존한다.
- 몇몇 중요한 이정표를 가지고 있으며, 문서화가 잘 되어 있고 예상 가능한 길을 따른다.

### UIKit Life Cycle
- Initialization(`init(coder:)`): 스토리보드에서 뷰 컨트롤러가 생성되는 단계.
- Load(`loadView`, `viewDidLoad`): 뷰가 메모리에 로드되었을 경우 단 한 번 호출된다. 초기 설정 및 단 한 번 설정하는 요소가 있을 경우 중요하다.
- Visibility(`viewWillAppear`, `viewDidAppear`): 매번 뷰가 화면에 나타날/나타났을 때 호출된다.
- Layout(`viewWillLayoutSubviews`, `viewDidLayoutSubviews`): 뷰 컨트롤러의 뷰가 하위 뷰들의 레이아웃을 다잡아야 할 때 호출한다. 뷰가 호출되기 전, 최신 변경사항을 반영하는 데 유용하다.
- Disappearance(`viewWillDisappear`, `viewDidDisappear`): 뷰가 화면에서 사라질/사라졌을 때 호출된다.
- Deallocation(`deinit`): 뷰 컨트롤러가 메모리에서 해제되기 직전에 호출된다.
- 이러한 라이프사이클 메서드들은 개발자들이 각각의 단계에서 어떤 것을 할지 명시적으로 컨트롤할 수 있게 만든다.
- 다만 관리 측면, 그리고 보일러플레이트 코드의 범람에는 주의가 필요하다.

## SwiftUI: 현대적이고 선언적인 접근법
- 2019년에 소개된 선언적 UI 프레임워크.
- 상태(`State`) 기반 업데이트를 UI에 적용한다.
- 많은 UI 생명 주기 컨트롤을 추상화하여 Under the hood로 숨김으로써, UI 개발을 간결하게 한다.

### SwiftUI Life Cycle
- Initialization: 부모 뷰나 시스템이 "이 뷰를 보여줘야겠다!" 결정한 시점에 초기화된다. SwiftUI 뷰는 꽤 가벼워서, 대개의 경우 UIKit 뷰 컨트롤러보다 초기화 비용이 더 적게 든다.
- State Updates(`@State`, `@Binding`, `@ObservedObject`, ...): SwiftUI 뷰들은 이러한 State들이 바뀔 때 업데이트된다. 일종의 반응형 동작인지라, `viewDidLoad`와 같은 라이프사이클 메서드가 필요가 없어진다.
- Recomposition: `viewWillLayoutSubviews`처럼 레이아웃 메서드를 쓰는 대신, 상술한 State가 바뀐다면 SwiftUI 뷰들은 자동으로 다시 생성되거나 다시 그려진다.
- Disappearance: `viewWillDisappear`처럼 지정되어 호출되는 것은 없다. 단지 뷰 계층에서의 제거라는 "변경사항"을 반영하기 위해 다시 그려지고, 이 과정에서 없어지고, 하는 것.
    - *역주: 그런데 `.onDisappear` 오퍼레이터는 있지 않나?*
- Deallocation: SwiftUI 관리 하에서, 쓰이지 않는 뷰는 자동으로 허물어지고 리소스가 확보된다.

## 생명 주기 관리 방법의 비교
- 컨트롤 vs 간편함
    - UIKit은 생명 주기 메서드를 통해 좀 더 자세한 컨트롤을 할 수 있게 한다. 이는 복잡한 앱에 적합하다.
    - 한편 SwiftUI는 편하고, 코드를 덜 쳐도 된다. 상대적으로 심플한 UI와 빠른 개발 사이클에 적합하다.
- 성능
    - 심플한 화면에서는 SwiftUI의 효율적인 관리/렌더링 시스템을 통해 더 좋은 성능을 낼 수 있다.
    - 하지만 UIKit이 빛나는 순간은 커스터마이징이 많이 들어간 뷰와 동적인 UI 사용 시나리오가 예상될 때. 렌더링에 대한 좀 더 직접적인 컨트롤을 제공하기 때문에 이런 상황에서는 더 좋은 성능을 낼 수 있다.
- 학습 곡선
    - UIKit의 생명 주기 관리는 다소 복잡하지만, 이미 많은 iOS 개발자가 이에 익숙하다.
    - SwiftUI는 더 간단한 생명 주기를 가져서 이해하기는 쉽다. 다만 특유의 반응형 패러다임을 이해하는 것이 기본적으로 어려울 수 있다.

## 결론
- 언제나처럼 조직과 상황에 맞게 어쩌구
- UIKit은 깊이와 컨트롤 여부, 그리고 앱을 위한 특수한 동작을 지원한다. 복잡한 앱에 적합하다.
- SwiftUI는 속이 뻥 뚫리는 간편함을 통해 직관적 앱을 만듬에 있어 엄청난 생산성을 제공한다.
- 아무튼 햅삐 코딩!