# Meet Swift Testing
https://developer.apple.com/wwdc24/10179

- 신뢰도는 소프트웨어 퀄리티에 중요한 부분이며, 자동화된 테스트는 이를 보장한다.
- Swift Testing은 Swift 코드를 테스트하기 위한 오픈 소스 패키지이다.
- Concurrency나 Macro같은 Swift 최신 문법에 대응한다.

## Building Blocks
- 들어가기 앞서! 테스트를 수행하기 위해서는 테스트를 위한 타겟을 새로 추가해야만 한다.
  - File - New - Target에서 Unit Testing Bundle을 선택
- 엑코16에서는 테스팅 프레임워크를 선택할 수 있고, Swift Testing이 기본으로 설정되어 있다.

- 테스트의 단위인 블럭. 함수, expectation, traits, suite 네 가지가 있다.
```Swift
import Testing

@Test func videoMetadata() {
    // ...
}
```
- `@Test` 애트리뷰트는 첫 번째 블럭이다. 이 함수는 테스트 함수요! 라는 뜻.
- 전역 함수, 타입 함수, 비동기, 스로잉, 메인액터.. 다 될 수 있다.
- 그럼 이제 안에 뭔가 넣어봐야겠지.
```Swift
import Testing
@testable import DestinationVideo

@Test func videoMetadata() {
    let video = Video(fileName: "By the Lake.mov")
    let expectedMetadata = Metadata(duration: .seconds(90))
    #expect(video.metadata == expectedMetadata)
}
```
- `@testable import`는 여전히 필요하구나.
- `#expect` 매크로를 활용해 값을 캡쳐하고 미리 정의해 둔 예상값과 비교한다.
- 테스트 실패 문구를 클릭하면, 어디서 어떻게 틀렸는지 자세히 볼 수 있다!
- `#expect` 매크로는 안이 `bool`이기만 하면 뭐든 넣을 수 있다.
- 한편 테스트가 실패했을 때 빨리 탈출하려면 `#require` 매크로를 사용할 수 있다.
``` Swift
let method = try #require(paymentMethods.first)
#expect(method.isDefault)
```
- `@Test` 애트리뷰트 안에, 테스트 이름을 지정할 수도 있다.
```Swift
@Test("Check video metadata") func videoMetadata() {
    let video = Video(fileName: "By the Lake.mov")
    let expectedMetadata = Metadata(duration: .seconds(90))
    #expect(video.metadata == expectedMetadata)
}
```
- Trait라는 또 하나의 블럭을 통해 다양한 정보를 제공할 수 있다.
- 테스트 추가 정보, 언제 돌려야 하는지, 어떻게 동작하는지 등을 조작할 수 있다.
```Swift
@Test("이름") // 표시 이름 정하기
@Test(.bug("example.com/...", "Title") // 버그 트래커의 이슈
@Test(.tags(.critical)) // 태그 달기
@Test(.enabled(if: server.isOnline)) // 런타임 조건 달기
@Test(.disabled("Broken")) // 테스트가 실패 혹은 비정상 종료되었을 때를 위한 설명
@Test({내용}) @available(macOS 15, *)
@Test(timeLimit(.minutes(3)))
@Suite(.serialized) // 병렬 차리 없이 하나씩 여러 테스트 돌리기
```

```Swift
@Test func rating() async throws {
    let video = Video(fileName: "By the Lake.mov")
    #expect(video.contentRating == "G")
}
```
- 위에 있던 테스트와 묶으면?
```Swift
struct VideoTests {
    let video = Video(fileName: "By the Lake.mov")

    @Test("Check video metadata") func videoMetadata() {
        let expectedMetadata = Metadata(duration: .seconds(90))
        #expect(video.metadata == expectedMetadata)
    }

    @Test func rating() async throws {
        #expect(video.contentRating == "G")
    }

}
```
- 이제 첫 번째 `Suite`를 만들었다! 테스트 함수를 갖고 있는 인스턴스는 암시적으로 스위트로 인식된다.
- 기존에 sut 설정하듯이 공통적으로 쓰이는 프로퍼티를 분리할 수도 있다.

## Common Workflows

### Test in Conditions
- 특정 환경에서만 되는 테스트.
- enabled, disabled, bug, @available, ...
```Swift
@Test(.enabled(if: AppFeatures.isCommentingEnabled))
func videoCommenting() {
    // ...
}

@Test(.disabled("Due to a known crash"))
func example() {
    // ...
}

// 당연히 한 번에 여러 개도 가능
@Test(.disabled("Due to a known crash"),
      .bug("example.org/bugs/1234", "Program crashes at <symbol>"))
func example() {
    // ...
}
```

### Test with Common Characteristics
- tag 트레잇을 통해 태그를 달면, 테스트 디버깅 화면에서 태그별로 묶을 수 있다!!
```Swift
@Test(.tags(.formatting)) func rating() async throws {
    #expect(video.contentRating == "G")
}

@Suite(.tags(.formatting))
struct MetadataPresentation {
    let video = Video(fileName: "By the Lake.mov")

    @Test func rating() async throws {
        #expect(video.contentRating == "G")
    }

    @Test func formattedDuration() async throws {
        let videoLibrary = try await VideoLibrary()
        let video = try #require(await videoLibrary.video(named: "By the Lake"))
        #expect(video.formattedDuration == "0m 19s")
    }
}
```
- 스위트에 태그를 달면 아래의 테스트 함수에 전부 그 태그가 상속된다.
- 태그별로 묶어서 테스트하거나, 리포트에서 거르거나, 할 수 있다.
- 태그를 붙이는 것이 권장된다 이 프레임워크에서는. 하지만! 그래도 가장 상황에 맞는 트레잇을 사용할 것.

### Tests with Different Arguments
- 당연히! 보일러플레이트가 반복되는 것보다 매개변수를 바꿔가면서 하나의 함수로 제네릭하게 테스트하는 게 좋겠지.
- parameterized testing.
```Swift
// before
struct VideoContinentsTests {

    @Test func mentionsFor_A_Beach() async throws {
        let videoLibrary = try await VideoLibrary()
        let video = try #require(await videoLibrary.video(named: "A Beach"))
        #expect(!video.mentionedContinents.isEmpty)
        #expect(video.mentionedContinents.count <= 3)
    }

    @Test func mentionsFor_By_the_Lake() async throws {
        let videoLibrary = try await VideoLibrary()
        let video = try #require(await videoLibrary.video(named: "By the Lake"))
        #expect(!video.mentionedContinents.isEmpty)
        #expect(video.mentionedContinents.count <= 3)
    }

    @Test func mentionsFor_Camping_in_the_Woods() async throws {
        let videoLibrary = try await VideoLibrary()
        let video = try #require(await videoLibrary.video(named: "Camping in the Woods"))
        #expect(!video.mentionedContinents.isEmpty)
        #expect(video.mentionedContinents.count <= 3)
    }

    // ...and more, similar test functions
}

// after - 이렇게 만들어도, 마치 분리된 테스트를 하는 것처럼 해 준다.
// 당연히 여기에 트레잇을 더 붙이는 것도 가능
struct VideoContinentsTests {

    @Test("Number of mentioned continents", arguments: [
        "A Beach",
        "By the Lake",
        "Camping in the Woods",
        "The Rolling Hills",
        "Ocean Breeze",
        "Patagonia Lake",
        "Scotland Coast",
        "China Paddy Field",
    ])
    func mentionedContinentCounts(videoName: String) async throws {
        let videoLibrary = try await VideoLibrary()
        let video = try #require(await videoLibrary.video(named: videoName))
        #expect(!video.mentionedContinents.isEmpty)
        #expect(video.mentionedContinents.count <= 3)
    }

}
```
- 패러메트라이즈 테스팅은 for문을 돌리는 것과 컨셉은 비슷하다만, 각각을 독립적으로 돌린다는 데에서 차이가 있다.
- 게다가 병렬로 돌려서 더 빠르기도 하다.

## Swift Testing and XCTest
- 이미 써 둔 테스트를 갈아엎으라면 억장이 무너지겠지.
- 근데 머 컨셉은 비슷하다. 그런데 Swift Testing은 XCTest에 비해
  - 이름 제한도 없고
  - 함수도 더 다양한 종류를 쓸 수 있고
  - 트레잇이 있고
  - 실기기에서도 할 수 있다.
- `XCTest` 상에서는 여러 다른 assert 함수를 통해 비교했지만, `#expect` 매크로를 사용하면 쉽게 이 모든 것을 대체할 수 있다.
- 테스트가 실패했을 때 더 이상 진행하지 않기 위해서도, `XCTest`에서는 `self.continueAfterFailure = false`를 따로 해 줘야만 한다.
- 더 다양한 타입으로 테스팅 객체가 존재할 수 있다는 거!
- teardown도 따로 해 줄 필요가 없다는 거!
- 테스트 그룹도 해 줄 수 있다는 거!
- (대충 더 좋다는 뜻)
- 그러나 `XCTest`를 유지해야 할 떄도 있다.
  - UI 오토메이션 `XCUIApplication`
  - 퍼포먼스 테스트 `XCTMetric`
  - Obj-C로만 쓰여있는 코드인 경우

## Open Source
- https://github.com/apple/swift-testing
- 오픈 쏘오스! 애플 운영체제나 리눅스, 윈도우에서도 다 돌아가고, 동일한 코드베이스를 지닌다.
- VSCode Swift Extension 깔면 작됭된다는 것이 고무적인듯.
- 터미널에 `demo@Demo MyLibrary %` 상에서 `swift test` 치면, 예를 들어 콘솔에서 아래 테스트를 실행해 준다.
```Swift
import Testing
@testable import MyLibrary

@Test func example() throws {
    #expect("abc" == "abc")
}

@Test func failingExample() throws {
    #expect(123 == 456)
}
```

