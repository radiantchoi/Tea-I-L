# iOS 접근성
## 접근성?
- 접근성이란 건, 여러 가지 제한 사항을 고려해 더 많은 사람이 서비스를 사용할 수 있도록 하는 것.
- 이게 꼭 장애를 말하는 게 아니고, 가령 운전중이라서 팔을 못 쓴다던지 하는 상황이라면 어떨까 하는 것.
#### 시각
- 텍스트 크기 확대/축소, 읽어주기, 화면 요소 움직임 줄이기
#### 청각
- 경보음 등을 감지해 화면에 띄워 주기, 자막
#### 운동 능력
- 받아쓰기, 자동완성, 후면 탭
#### 인지 능력
- 사파리 읽기 도구, 사용법 유도
### 왜 접근성인가?
- 근본적인 이유는 "사용자 배려"긴 하지만, 의외로 많은 다른 이점이 있다.
    - 사소한 부분에 집중할 수 있다: 생각지 못한 UI 요소를 개발 과정에서 돌아보게 된다. 어떻게 생겼고 어떻게 동작하는지, 레이아웃은 어떤지 더 신경쓰게 됨으로써 완성도 있는 앱을 만들 수 있다.
    - UITest가 쉬워진다. Accessibility Inspector를 이용해 진행하는 UI Test 특성상, Accessibility Label을 잘 지원하면 더 쉬운 테스트가 가능하다.
    - 더 많은 사용자를 위해 앱을 만든다는 것은, 앱의 버그를 고친다는 것과 동등한 존재감을 지닌다.

## 접근성 개선하기 - VoiceOver
### Accessibility Inspector
- 먼저 접근성 검사를 위해 Accessibility Inspector를 사용해봅시다~
    - Xcode - Open Developer Tool - Accessibility Inspector
    - 이후 Target을 현재 돌리고 있는 테스트 디바이스 혹은 시뮬레이터로 지정
#### Inspection 탭
- VoiceOver로 읽어 주는 거라고 보면 된다.
    - Navigation: 보이스오버를 읽어주는 컨트롤
    - Basic: 보이스오버가 읽어주는 순서대로 UI 컴포넌트의 프로퍼티를 나열
    - Actions: UI를 컨트롤
    - Element: 보이스오버가 인식중인 UI 컴포넌트의 정보
    - Hierarchy: 뷰 계층 구조
#### Audit 탭
- Run Audit을 하면, UI 접근성을 개선해야 하는 부분에 대해 검사하고 알려 준다.
- 각각의 검사 사항에 대해
    - 눈 모양을 누르면 해당 UI 요소를 캡쳐해서 알려 준다.
    - 물음표 모양을 누르면 구체적인 개선 제안사항을 알려 준다.
        - 물론 참고사항
#### Settings 탭
- 실제 기기에서도 적용 가능한 접근성 관련 다양한 설정을 바로 적용시킬 수 있다.
### Accessibility API
- Label, Value, Traits, Identifier, Hint 등과 같은 값을 설정해보자.
#### UIAccessibility
- UIKit의 뷰나 컨트롤들은 기본적으로 이 친구를 채택하고 있다.
- 또한 몇 가지 프로퍼티를 지원한다. 아래는 그 중 일부
    - `isAccessibilityElement`: 접근성을 지원하는 요소인지를 결정하는 Bool 타입의 프로퍼티
    - `accessibilityLabel`: 접근성 요소가 무엇인지 설명하는 짧은 localizing된 문자열
    - `accessibilityValue`: 접근성 요소의 값을 설명하는 localizing된 문자열
    - `accessibilityTraits`: 접근성 요소의 특징에 대한 값 (ex: Button, Seleted)
    - `accessibilityHint`: 접근성 요소의 결과를 설명하는 짧은 localizing된 문자열
- 직접 구현한 컨트롤일 때도, UIAccessibilityElement를 상속받아서 구현하면 접근성을 지원할 수 있다.
### VoiceOver 개선하기
- 그냥 코드로 구현하고 읽어보게 하면.. 개발한 우리 입장에선 잘 알아들을 수 있는데, 사용자도 그럴까?
- 이걸 수정해보자. 위에 써 둔 프로퍼티 값을 직접 세팅해 줌으로써 구현할 수 있다.
- 스토리보드나 인터페이스 빌더에서는 오른쪽에 있는 메뉴에서 수정 가능하지만, 대개 코드로 하게 될 것이므로.
``` swift
import UIKit

final class ExampleViewController: UIViewController {
	let exampleLabel: UILabel = {
		let label = UILabel()
		label.traslateAutoresizingMaskToConstraint = false
		label.text = "Hello World!"
		return label
	}()

	override func viewDidLoad() {
		super.viewDidLoad()

		setupView()
		setupAccessibility()
	}

	private func setupView() {
		view.addSubview(exampleLabel)

		// 대충 중간에 넣는 Constraint
	}

	private func setAccessibility() {
		exampleLabel.accessibilityLabel = "테스트 문구"
		exampleLabel.accessibilityIdentifier = "ExampleViewController.exampleLabel"
	}
}
```
- 대부분의 UILabel 요소를 포함한 UI 요소는 사실 그 레이블을 그대로 읽어주므로 꼭 넣어줄 필요는 없을 수 있다.
- accessibilityIdentifier는 런타임 중에 해당 요소를 String값으로 찾아낼 때 쓰인다. UI Test를 진행할 때, 해당 요소를 찾아낼 때 매우 편해진다.
- Label(요소에 대한 설명) Identifier(프로그램 내부적으로 찾아가는 용도) Hint(디스크립션)
- 이미지의 경우, 기본적으로는 접근성 요소가 아니기 때문에 `isAccessibilityElement`를 true로 해 주어야 한다.

### Dynamic Type
- 글씨 크기를 접근성에 의거 조절할 수 있는 폰트.
- systemFont 말고, title이나 caption 등의 프리셋 폰트, Text Style을 사용하면 구현할 수 있다.
    - 그리고 나서 `adjustsFontForContentSizeCategory` 프로퍼티를 true로 해 주면 된다.
    - 꼭 Text Style을 써야 하는 건 또 아니다.
``` swift
@IBOutlet weak var someLabel: UILabel! 

let customFont = UIFont(name: "Helvetica Neue", size: 50)! 
someLabel.font = UIFontMetrics.default.scaledFont(for: customFont)
```

- 다이나믹 폰트의 크기에 따라 스택 뷰의 배열 상태를 변경해줄 수도 있다. 다 보기 좋으라고 하는 거지
``` swift
// MARK: - Accessibility 
extension MainTableViewCell { 
	private func configureStackViewAccessibility() { 
		if traitCollection.preferredContentSizeCategory < .accessibilityLarge {
			 stackView.axis = .horizontal stackView.alignment = .center 
		} else {
			 stackView.axis = .vertical stackView.alignment = .leading 
		} 
	} 
}
```

- 사실 오토 레이아웃이랑 충돌할 가능성이 높다.
- 그래서 최대한 상수로 크기를 주지 않는 지향성이 필요하다..

### Contrast
- 색의 대비. 충분히 대비되지 않는 색이면 보기 어려울 수도 있으니까..
- 물론 이것도 기획과 개발 디자인의 합의가 필요하다.
- HIG에서는 텍스트의 경우 17포인트 이하는 4.5대 1, 18포일트 이상은 3대 1, 볼드의 경우 3대 1 이상의 대비를 유지하도록 규정하고 있다.
- Accessibility Inspector의 Color Contrast Calculator가 있다. 정답이라기보단 참고!

- 이미지의 경우 Assets에서 이미지를 선택한 뒤 High Contrast를 체크해 주면 새로운 고대비 이미지를 위한 공간이 생긴다. 그러면 이제 미리 준비해 둔 고대비 이미지를 애셋에 넣어 주면 된다.
- 라이트 모드, 다크 모드에 따라 대비를 다르게 정해줄 수 있다. 이미지의 Appearance 항목 참고

### 그 외의 접근성 지원
- 지금까지는 시각 장애인을 기준으로 한 접근성 지원이었는데, 앱은 특성상 시각에 의존하기 때문.
- 다른 것들은 뭐가 있을까?

- 저데이터 모드
    - 데이터가 모자란 곳이나 모자란 요금제(...)일 경우, 최대한 아낄 수 있다.
    - URLRequest 객체를 만들 때 꼭 필요하지 않거나 데이터 소모량이 많을 것 같은 부분에 allowsConstrainedNetworkAccess 프로퍼티를 false로 설정한다. 이렇게 하면 저데이터 모드에서 해당 부분은 로드하지 않는다.
    - 이 경우 왜 로드가 안 됐어? 를 반환하기 위한 부분에서 던지는 에러는 URLError.networkUnavailableReason == .constrained로 하면 된다.
    - 이외의 방법
        - 이미지 요청 시 저해상도로 요청
        - 플레이스 홀더 이미지 사용
        - 유저가 명시적으로 요청하지 않은 네트워크를 제한함
        - prefetch(데이터를 미리 가져오는 동작)을 비활성화함

- 스위치 컨트롤
    - 기기에 외부 기기를 연결해 사용을 돕는 기능
    - 보이스오버에 많이 의존하므로, 보이스오버를 잘 지원하면 된다.
    - 단! 몇 가지 고려해볼 사항은 있다.
        - 위험한 동작에 대해 고려해야 한다. 예를 들면 손 하나 까딱했을 뿐인데 데이터가 삭제된다던지.
        - 앱을 제어할 충분한 시간이 고려되어야 한다.
        - 화면 속 요소가 너무 많다면 그룹화해 원하는 것에 쉽게 접근할 수 있게 해야 한다.
        - 보안에 취약할 수도 있다는 점을 고려해야 한다.

- 그 외의 그 외
    - 손쉬운 사용 설정에서 여러 접근성들을 경험해보자
    - HIG도 잘 읽어보자

### UITest(드디어)
- UI가 의도대로 잘 동작하는지 보는 테스트.
- 유닛 테스트와 크게 다르지 않다. 결과값을 비교해주면 되는 것.
- 그런데! 생각보다 푸대접을 받는 경우가 있다.
    - 작은 변화에도 취약하다. UI 컴포넌트나 앱 사용 플로우가 조금이라도 달라지면 처음부터 다시 짜야 할 수도 있다.
    - 실제 앱 유즈 케이스와는 차이가 있을 수도 있다.
- 그리고 이런 단점들을 AccessibilityIdentifier의 존재로 보완할 수 있다~

- 대표적으로 UI 테스트에 사용되는 것들
    - XCUIApplication: 실행 중인 앱의 테스트 도구에 액세스할 수 있는 클래스
        - 테스트하기 위한 UI 요소를 찾는 클래스이기도 하다
        - 사용자가 조작하는 것처럼 앱을 조작할 수 있다
        - 테스트하는 앱의 프록시이기 때문에, 실제 앱과 무관하게 동작한다. 그래서 앱의 시작과 종료를 명시적으로 제어할 수 있다.
        - 테스트를 시작하면 항상 새 프로세스가 시작된다.
            - launch()
            - terminate()
            - wait()
            - state
        - 테스트 요소를 찾기 위한 시작점의 기능을 하기도 한다
    - XCUIElement
        - 테스트할 앱의 UI 엘레멘트의 프록시
    - XCUIElementQuery
        - 원하는 UI 요소를 구체적으로 찾는 API
    - 이외에는 Unit Test에 쓰이는 것과 동일한 것이 많다

- sut 대신 app을 쓰고, XCUIApplication 객체를 할당한다.
    - launch()를 통해 실행하고, UI 요소를 찾는다.
    - UI 요소를 찾을 때는 종류와 Identifier를 찾아서 사용한다.
    - tap() 같은 메서드를 사용할 수 있다.