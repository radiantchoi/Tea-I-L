# TCA와 Tuist 위젯 개괄

기준 버전은 TCA 1.9
교보재는 tcaexamples, beatmaster
# 주요 개념
## `Store`
- view에서 `@Bindable` 래퍼를 통해 묶어서 가지고 있는 프로퍼티.
    - 바인더블을 쓰지 않으면, withTransaction을 사용해야 한다.
``` swift
struct MyView: View {
	// iOS 17 이상일 때만 이 애트리뷰트를 사용한다
	@Bindable var store: StoreOf<MyFeature>
	// iOS 16 이하일 때는 이 애트리뷰트를 붙여야 한다
	@Perception.Bindable var previousStore: StoreOf<MyFeature>

	var body: some View {
		// iOS 17 이후에는 그냥 SwiftUI 뷰 쓰듯이 쓰되
		// 필요할 때 Store 프로퍼티에 접근하면 된다.

		// iOS 16 이전
		withTransaction(previousStore) {
			//...
		}
	}
}
```
- StoreOf 객체 자체를 개발자가 만들 필요는 없고, `Reducer`로 작성된 Feature를 associated type으로 넣어 주면 된다.
## `Reducer`
- 기존에 뷰 모델에서 하던 `@Published` 프로퍼티, 메서드를 통한 액션 등을 여기서 정의한다고 보면 된다.
- 뷰 트랜지션 동작도 액션으로 취급한다(!!!)
``` swift
@Reducer
struct MyFeature {
	@ObservableState
	struct State: Equatable {
		// 기존에 @Published로 정의했던, 뷰에 바인딩할 프로퍼티를 정의한다.
		// 이것이 StoreOf로 바인딩된다.
		var message: String = ""

		// TCA Tutorial 중 Standup에서
		// 뷰 간의 데이터 교환을 위한 IdentifiedArray 사용 예시가 있다.
		var auth: IdentifiedArrayOf<AuthModel> = []
		var authModel: AuthModel
		var webResponseData: String?

		// 참고. TCA의 이전 버전에서는 UI 액션 바인딩을 프로퍼티에서 따로 했어야 했다.
		@BindingState var alertColor: Color
	}

	// 기존에 메서드로 정의했던, Action을 정의한다.
	// BindableAction을 사용하는 경우는, 퍼스트 파티 UI 액션에 대응해야 할 때.
	enum Action: BindableAction {
		// 이제 UI 액션은 이쪽으로 받는다
		// 유지해야 하는 UI 이벤트가 있을 경우 사용한다.
		// 이 케이스 자체로는 UI 입력을 "받을 수 있게 뚫어주는" 역할만 한다.
		case binding(BindingAction<State>)

		// 서로 다른 액션의 경우, 각각의 액션을 정의해서 따로 처리한다.

		case textFieldTapped
		case textFieldChanged(value: String)
		case hello(message: String)

		// 기본적으로는 웹 요청과 응답 액션이 분리되어야 한다.
		case webRequest(quote: String)
		case webResponse(data: String)
	}

	// SwiftUI와의 일체감을 위해, body 프로퍼티를 사용한다.
	var body: some ReducerOf<Self> {
		// BindableAction을 채용한 경우 필요한 메서드
		BindingReducer()
	
		// 각각의 액션에 대한 처리를 한다
		Reduce { state, action in 
			switch action {
			case binding():
				return .none
			case textFieldTapped:
				print("Text field tapped!")
				return .none
			case textFieldChanged(value: String)
				state.message = value
				return .none
			case hello:
				print("Hello!")
				return .none
			default:
				return .none
			// 비동기 관련 부분은 TCA Tutorials - Counter 보고 보충하기
		}
	}
}
```
### `body.onChange`
- 앱이 받는/수행하는 액션이 바뀌었을 경우 호출된다.
- 액션을 수행할 때마다 불려온다고 봐도 된다.
``` swift
@Reducer
struct MyFeature {
	var body: some ReducerOf<Self> {
		// ...
	}
	.onChange(of: /.hello) {
		Reduce { state, action in 
			// hello가 불러와졌을 때 수행할 일종의 후처리 "액션"을 넣는다
			// 실제 사용시에는 유저가 액션을 할 때마다 db에 연동한다던지, 하는 동작을 할 수 있다.
			state.message = "Hello!"
			return .none
		}
	}
}
```
# 실제
### 비동기적 액션 (심화)
- Swift Concurrency를 활용해 이루어진다
- 보내고 받는 액션을 수행할 때, Reducer가 `@Dependency` 키 프로퍼티를 활용해 UseCase에 의존하게 함으로써 자세한 로직은 유즈케이스에 분리해둘 수 있다.
```swift
struct MyFeatureUsingUseCase {
	// 예를 들면 이렇게 미리 정의되어 있는 dismiss 액션을 활용할 수도 있다.
	// 내비게이션 스택에서 알려드림
	@Dependency(/.dismiss) var dismiss

	// 바깥의 객체에 연결할 때 쓸 수 있다.
	// UseCase도 이 경우 struct로 작성해야 한다..?!
	// Repository도 @Observable 클래스로 정의해 주자.
	@Dependency(MyUseCase.self) var myUseCase
}
```
## 내비게이션 스택 관리
- Stack과 Destination이 있다.
- Stack의 경우 피처(리듀서) 간의 이동에 있어 필요하다.
```swift
@Reducer
strcut RootFeature {
	// 루트 뷰의 State는 각기 다른 피처 간의 데이터 교환소 역할을 한다.
	@ObservableState
	struct State: Equatable {
		// 내비게이션 스택을 관리하기 위한 프로퍼티
		// 내부적으로 배열로 이루어져 있다.
		var path: StackState<Path.State> = .init()

		// ...
	}

	// 하나의 내비게이션 스택을 공유하는 기능들에서 필요한
	@Reducer
	enum Path {
		case auth(AuthFeature)
		case profile(ProfileFeature)
	}

	// 각각의 기능 중점적인 부분보다는 루트 뷰에 기대할 법 한 액션을 수행한다.
	enum Action {
		// Path를 위한 액션이 필요하다
		case Path(StackAction<Path.State, Path.Action>)
	}

	var body: some ReducerOf<Self> {
		// 각각의 path를 선택했을 때 화면을 전환한다던지
		// 루트에서 "액션"으로 취급되는 것에 대한 처리를 해 준다.
		
	}
	// 각각의 path 케이스에 대해 자기들과 이름이 같은 액션이랑 묶어 준다.
	// 루트 뷰에서 이 케이스에 대해 수행할 액션을 해당 케이스와 묶어 주는 작업.
	// 자세한 예시는 BeatMaster를 보자.
	.forEach(/.path, action: /.path)
}
```
- 응용하면 탭 뷰도 이런 식으로 루트 피처를 활용해 작성할 수 있겠다.
- 한편 Destination은 피처 안에서 피처와 관련된 기능 간의 전환을 할 때 사용된다.
    - `.ifLet` 오퍼레이터와 함께 사용한다!
- 다만 Best Practice라고 장담은 못 하고, 이런 방법이 있다~ 정도 참고!!
# 번외: Tuist 프로젝트에서 Widget 만들기 (다이나믹 아일랜드 포함)
- product를 .app이 아니고 .appExtension으로 해야 한다.
- bundle id는 "WidgetExtension"으로 해야 한다 (예외없음)
- source와 resource 폴더 또한 WidgetExtension/Sources/** 이런 식으로 해야 한다.
    - 리소스 폴더는 사실 필요없을 수도 있는데, 에러를 뱉을 수도 있기 때문에 돌려 보고 판단할 것.
- 특이사항으로, dependencies에서 .sdk 형식으로 WidgetKit, SwiftUI를 의존시켜 줘야 한다... 고 한다.
- 그리고 makeAppModule 등 메서드 안에서 타겟을 정할 때, 직접 dependency로서 target으로 추가해줄 수 있다. Project에서 추가하면 안 된다.
    - 후술하겠지만 watch 타겟도 마찬가지!
# 번외: Tuist 프로젝트에서 Watch 타겟 만들기
- 플랫폼을 watchOS로 해야 하고, product는 watch2App, watch2Extension으로 각각 해야 한다.
- 앱과 익스텐션을 모두 정의해서 타겟으로 만든 다음, 위젯을 앱에 추가하듯 추가해줘야 한다.
- 앱은 뷰, 익스텐션은 그 이외라고 생각하면 편한.. 데 안 편해.
- sdk로 watchkit, watchconnectivity를 넣어 줘야 한다.