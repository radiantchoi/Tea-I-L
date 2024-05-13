# OSLog 활용해서 잘 로깅하기
https://medium.com/@kodekarim/xcode-15s-hidden-treasure-logging-like-a-pro-2024-6fc993d65ccb
https://developer.apple.com/documentation/os/logger
- `Logger`에서 Xcode 상에서 하단부 콘솔 창에 찍히는 로그를 작성할 수 있다. 또한 색깔을 부여할 수 있다는 사실!
```Swift
import OSLog

extension Logger {
    private static var subsystem = Bundle.main.bundleIdentifier!

    static let viewCycle = Logger(subsystem: subsystem, category: "viewcycle")
    static let api = Logger(subsystem: subsystem, category: "API")
}
```
- 상기 코드를 통해 뷰 라이프 사이클, 혹은 분석이나 통신 관련 로그를 눈에 띄게 쓸 수 있다!
- `Logger`에 붙는 메서드는 여러 가지가 있다.
```Swift
// 회색
Logger.viewCycle.notice("Notice example") // 기본 로그 타입

// 파란색
Logger.viewCycle.info("Info example") // 정보 발신

// 회색
Logger.viewCycle.debug("Debug example") // Debug 타겟에서 print할 만 한 문구
Logger.viewCycle.trace("Trace example") // 추적할 것이 있는 경우

// 노란색
Logger.viewCycle.warning("Warning example") // 주의해야 할 부분. 소위 "노란 줄"
Logger.viewCycle.error("Error example") // warning보다 좀 더 중대한 "노란 에러"

// 빨간색
Logger.viewCycle.fault("Fault example") // 여기서부턴 아예 틀린 것이다. 컴파일러에서 떴으면 실행이 안 될 만 한 에러.
Logger.viewCycle.critical("Critical example") // 치명적인 에러 상황에 대해 기록할 때 사용
```
- 실제 콘솔 상에서 찍어봤을 때는 잘 안 보이기도 하던데, 유니코드 이모지나 Description 등을 덧붙여 새로운 로깅 라이브러리를 만들어 보자.
- 무턱대고 `print`만 찍기보다, 색상으로 구분해 한 눈에 알 수 있는 에러 로깅을 하기 위해 알아 두자.