# Lottie, 0 to 1

## Lottie?
- 한 마디로 작은 용량으로 부드러운 애니메이션을 앱에 넣어 줄 수 있는 도구

## Lottie 적용하기

### Lottie Animation 초기 설정
- Lottie 애니메이션의 경우 json 파일로 이루어진다.
- 애프터 이펙트나 피그마 등을 통해 만들 수 있는 듯 하다.
- 파일 준비는 개발자가 할 일은 아니다! 디자이너에게 부탁하거나, 다운받거나 하자.
  - https://lottiefiles.com/featured
- Lottie JSON 파일을 구했다면, 프로젝트 내 임의의 폴더에 추가해주면 된다.
- 한편 CocoaPods, SPM 등의 도구를 통해 Lottie 의존성을 추가하는 것도 잊지 말자.
  - https://github.com/airbnb/lottie-ios

### UIKit의 경우
```Swift
import UIKit

import Lottie

final class ViewController: BaseViewController {
    // 대부분의 튜토리얼에서 AnimationView를 쓰라고 되어 있지만, 이름이 LottieAnimationView로 바뀌었다.
    // "Wave.json" 파일을 활용했다.
    private lazy var lottieAnimatedView: LottieAnimationView = {
        $0.loopMode = .loop
        $0.animationSpeed = 2
        $0.backgroundColor = .clear
        
        return $0
    }(LottieAnimationView(name: "Wave"))
    
    override func viewDidLoad() {
        super.viewDidLoad()
    }

    override func setupSubviews() {
        view.addSubview(lottieAnimatedView)
        
        lottieAnimatedView.play()
    }
    
    override func setupConstraints() {
        lottieAnimatedView.translatesAutoresizingMaskIntoConstraints = false
        
        lottieAnimatedView.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
        lottieAnimatedView.centerYAnchor.constraint(equalTo: view.centerYAnchor).isActive = true
    }
}
```

### SwiftUI의 경우
```Swift
import SwiftUI

import Lottie

struct LottieAnimationView: View {
    var body: some View {
        // "RoundedCheckmark.json" 파일을 활용했다.
        LottieView(animation: .named("RoundedCheckmark"))
            .looping()
            .animationSpeed(2)
    }
}
```

## 활용
- 상기 예시는 반복되는 애니메이션을 보여줄 수 있다는 것만 실험해본 것!
- 여러 가지 메서드, 트레잇 등을 통해 다양하게 응용할 수 있다.
- https://airbnb.io/lottie/#/ios

## References
- https://hongssup.tistory.com/160
- https://sunrinnote.tistory.com/186
- https://velog.io/@lawn/iOS-SwiftUI-Lottie-%EC%99%84%EB%B2%BD-%EC%A0%95%EB%B3%B5-Lawn