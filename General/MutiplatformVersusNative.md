# Android & iOS native vs. Flutter vs. Compose Multiplatform
https://medium.com/@jacobras/android-ios-native-vs-flutter-vs-compose-multiplatform-7ef3d5ec2a56
- Flutter와 KMP, 그리고 각각의 네이티브 코드베이스로 인한 결과물의 크기와 속도를 비교해보자!

## 예제 앱
- 화면 하나에, api 통신을 통해 고양이 사진을 가져오는 앱!
- 컴포즈, SwiftUI 등 선언형 UI를 사용
- 레포지토리는 여기 - https://github.com/jacobras/flutter-vs-native-vs-kmp

## 빌드 결과물(앱)의 크기
- 안드로이드 환경에서
    - 네이티브: 1.463메가바이트
    - KMP/Compose: 네이티브와 동일.
        - 단지 iOS 타겟이 추가되는 건데, 이는 안드로이드 빌드 과정에서 코드에 포함되지는 않는다.
    - 플러터: 네이티브의 5배 정도 용량

- iOS 환경에서
    - 네이티브: 1.7메가바이트
    - KMP/Compose: 1.5 버전 기준으로 32메가(!!!) Skia 번들링 때문에 그런 것 같다고 했다.
        - 코틀린 멀티플랫폼은 안드로이드로 빌드하고 그것을 기반으로 iOS에 쏴 준다.
        - 근데 iOS를 위한 Skia도 따로 빌드를 해야 된다.
    - 플러터: 25.4메가
        - KMP와 마찬가지 이슈가 있긴 한데, 용량은 더 작다.

## 초기 실행 시간
- 안드로이드 환경에서 측정할 때는 Macrobenchmark를 사용해 구글 픽셀 4a에서 측정했다.
- 플러터는 네이티브와 KMP에 비해 54% 정도 초기 로드가 느렸다.
    - 안드로이드 네이티브는 400ms 초반 정도, 플러터는 600ms 정도의 시간이 걸렸다.
    - 플러터 엔진을 실행해야 돼서 그런듯?
- iOS 환경에서는 네이티브에 비해 멀티플랫폼과 플러터 모두 12% 정도만 느렸다.
    - iOS 네이티브는 1400ms, 컴포즈 멀티플랫폼과 플러터는 비슷하게 1600ms보다 약간 적은 정도.