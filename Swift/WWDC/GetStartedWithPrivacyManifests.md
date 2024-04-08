# Get started with privacy manifests
https://developer.apple.com/wwdc23/10060
- 앱 추적 투명성과 통합된 새로운 개인정보 취급 개요표.
- 흔히 앱 스토어에서 다운받을 때 밑에 있는 카드 같은 그것!
- 또한 팝업을 통해 관리를 할 수 있게 됐다.
- 특히 타사 SDK에 의존하는 경우 유용할 것이다.

## Privacy Manifest
- PrivacyInfo.xcprivacy 파일을 이용해서 SDK에도 Privacy Manifest를 포함할 수 있다.
- 소스 코드 있는 그 폴더로 가서 만들자!
- plist 파일로, 수집하는 데이터 타입과 사용 방법, 사용자와의 연결 여부, 추적에 어떻게 사용되는지 등을 기입해 둔다.

## Privacy Report
- 앱 스토어에 제출할 앱을 빌드할 때, Xcode는 매니페스트를 싹 모으고, 요약하는 리포트를 작성한다.
- Xcode Organizer에서 Generate Privacy Report를 생성하면 된다.
- pdf 파일로 뽑혀 나온다.
- WWDC22 Create your Privacy Nutrition Labels 참조

## Tracking Domains
- 앱의 네트워크 연결을 더 쉽게 이해하고 제어할 수 있다.
- 의도치 않게 사용자의 활동을 추적할 수 있다! 서드 파티 라이브러리를 사용하는 경우 그렇다.
- 그래서 이번에 나온 Privacy manifest에는 추적을 선언한 도메인이 자동으로 포함된다.
- 유저가 추적을 허용하지 않으면 자동으로 연결을 끊는다.
- WWDC22 Explore App Tracking Transparency 참조
- 트래킹을 하는 기능과 아닌 기능을 구분해서 도메인을 부여하고 호스팅하는 것도 방법이다.

## Required Reason APIs
- 트래킹은 되지만 핑거프린팅은 절대 안 된다.
    - 핑거프린팅은 기기의 신호를 이용해 기기 혹은 사용자를 식별하는 것이다.
- 기존 API 중 핑거프린팅에 오용될 가능성이 있지만 잘 쓰면 좋은 것들을 위한 새로운 카테고리가 상술한 제목.
- 승인된 몇몇 사유에 한해서만 해당 API를 사용할 수 있게 바뀐다.
- 또한 해당 API에서 반환된 데이터는 다른 용도로 사용할 수 없다.
- 이러한 종류의 API에 의존하는 서드 파티 SDK라면, 마찬가지로 개인정보 매니페스트를 만들어서 제공해야 한다.
- 이외에 개인정보에 특히 큰 영향을 끼치는 SDK를 Privacy-Impacting SDK로 구분했고, 이것의 목록과 업데이트는 애플 개발자 문서에 게시되어 있다.
    - 이러한 SDK가 포함된 앱은 개인정보 매니페스트와 함께 SDK 사본을 포함해야 한다.
    - Xcode 15는 SDK 서명과 검증을 돕는다. 그러니까 서명된 SDK를 이용하는 게 가장 좋다.

## 개발자가 할 일? (오만하기 짝이 없음)
- 앱 개발자라면
    - 서드 파티 SDK 개발자에게 매니페스트 만들 것을 요청한다.
    - Xcode 개인정보 리포트를 활용한다.
- SDK 개발자라면
    - 서명과 매니페스트를 채택한다.
- 공통적으로 추적 도메인 및 필수 사유 API 사용 이유를 자신의 앱의 PrivacyInfo.xcprivacy에 기록한다.
