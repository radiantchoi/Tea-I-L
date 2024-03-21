# Build an app with SwiftData
https://developer.apple.com/wwdc23/10154
- 플래시 카드 앱을 만들면서 설명이 이루어진다.
- 관련 Starter 프로젝트를 다운로드받아 함께 코드를 쓰면서 배울 수 있다.

## 알아볼 것
- SwiftData 모델
- 모델의 쿼리화
- 모델 생성, 모델 업데이트, UI와 바인딩
- Document-Based Apps


## SwiftData 모델
```Swift
import SwiftData

// 모델로 적용하기 전
final class Card {
    @Published var front: String
    @Published var back: String
    var creationDate: Date

    init(front: String, back: String, creationDate: Date = .now) {
        self.front = front
        self.back = back
        self.creationDate = creationDate
    }
}

// 모델을 적용한 후
@Model
final class Card {
    var front: String
    var back: String
    var creationDate: Date

    init(front: String, back: String, creationDate: Date = .now) {
        self.front = front
        self.back = back
        self.creationDate = creationDate
    }
}

// 뷰 안에서
struct CardEditorView: View {
    // 모델을 적용하기 전
    @ObservedObject var card: Card
    
    // 모델을 적용한 후 - Observable을 도입하기 위해 Bindable을 달아 준다.
    @Bindable var card: Card
    
    // ...
}
```
- `@Observable`과 `@Bindable` 프로퍼티 래퍼는 더 적은 코드로 앱의 데이터 흐름을 자연스럽게 만든다.
- 해당 프로퍼티 수정시 자동으로 업데이트된다.
- 모델의 가변 상태와 UI 요소를 쉽게 바인딩할 수 있다.
- Discover Observation with SwiftUI 세션 참조

## 모델의 쿼리화와 UI 반영
```Swift
import SwiftData
import SwiftUI

struct ContentView: View {
    // before
    @State private var cards: [Card]
    
    // after
    @Query private var cards: [Card]
}
```
- `@Query`는 SwiftData에서 관리하는 데이터를 뷰에 보이고 싶을 때 사용한다.
  - `@State`와 비슷한 역할을 하는데, 모델에 일어나는 변화를 뷰에 반영하고자 할 때 특히 그렇다.
```Swift
extension ContentView {
    @Query(sort: \.created) private var cards: [Card]
}
```
- 정렬, 순서 지정, 필터링, 변경시 애니메이팅까지 지원한다.
- 쿼리는 뷰의 `ModelContext`를 데이터 소스로 사용한다.
  - `ModelContainer`에서 가져오자. SwiftData를 사용하는 앱이라면 무조건 하나씩은 있는 그것.
```Swift
struct FlashCardApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Card.self)
    }
}

@MainActor
let previewContainer: ModelContainer = {
    do {
        let container = try Modelcontainer(
            for: Card.self, ModelConfiguration(inMemory: true)
        )
        
        for card in SampleDeck.contents {
            container.mainContext.inserd(object: card)
        }
        
        return container
    } catch {
        fatalError("Failed to create container")
    }
}

// 프리뷰를 위해 샘플 덱을 따로 제공
#Preview {
    ContentView()
        .frame(minWidth: 500, minHeight: 500)
        .modelContainer(previewContainer)
}
```
- 물론 윈도우/뷰 단위에서 분리해서 여러 개의 컨테이너를 사용하는 것도 가능하다.

## 생성과 업데이트
- 뷰의 모델 컨텍스트에 접근해야 한다.
```Swift 
@Environment(\.modelContext) private var modelContext
```
- 모든 뷰는 하나의 컨텍스트를 가진다.
- 새로운 모델을 삽입할 때?
```Swift
let newCard = Card(front: "Sample Front", back: "Sample Back")
modelContext.insert(object: newCard)
// modelContext.save()는 안 써도 된다. Swiftdata가 자동으로 저장해 준다.
// 다만 SwiftData 스토리지를 공유/전송하기 전에는 확인이 필요하다. 이럴 때 써 주자.
```
- 이렇게 하면 새 창을 열었을 때 동일한 데이터베이스에서 불러온다.
- 하지만 문서 베이스를 다르게 가져가고 싶다면?
- 혹은 Freeform, 미리보기, Keynote, Numbers, Pages 등의 Document based App에서 사용하고 싶다면?

## Document-Based Apps
```Swift
@main
struct SwiftDataFlashCardSample: App {
    var body: some Scene {
        #if os(iOS) || os(macOS)
        DocumentGroup(editing: Card.self, contentType: .flashCards) {
            ContentView()
        }
        #else
        WindowGroup {
            ContentView()
                .modelContainer(for: Card.self)
        }
        #endif
    }
}

extension UTType {
    // Info.plist에도 동일한 형식으로 정의해야 한다고 한다.
    // 타겟 설정 - exported type modifiers를 활용
    static var flashCards = UTType(exportedAs: "com.example.flashCards")
}
```
- `ContentType`은 SwiftData Document App을 위한 맞춤형 컨텐츠 타입이다.
- 문서 컨텍스트에서 컨텐츠 타입은 이진 파일 포맷이랑 비슷한 취급이라고 생각하면 된다. 고정된 이진 구조를 가진다.
- 또 다른 유형의 문서인 패키지는, 고정된 디렉토리 구조를 가진 디렉토리이다.
- SwiftData 문서는 패키지다!
  - 일부 프로퍼티를 externalStorage 속성으로 마킹하면, 외부에 저장된 모든 아이템이 문서 패키지의 일부가 된다.
- 문서 기반 프로그램은, 고유한 확장자를 가진 문서를 만들 수 있는 기능인거다 그냥!
  - 이 경우 .sampledeck 확장자를 가진 파일이 생기게 된다.