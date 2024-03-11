# 라이브러리나 프레임워크 만들기

## Cocoapods
먼저 코코아팟 명령어부터
``` shell
pod install # podfile을 기반으로 라이브러리를 페칭해서 워크스페이스 생성
pod update # podfile을 참고하지 않고 팟을 최신 버전으로 업데이트 / 개별 팟도 업데이트 가능
pod outdated # 그간 업데이트된 팟들의 목록을 표시
pod repo update # 모든 팟의 podspec 파일을 업데이트
```
### 생성
``` shell
pod lib create {라이브러리 이름}
```
실행해서 먼저 지정한 경로에 라이브러리 작업 파일을 만든다.
이후 .podspec 파일에 들어가서 해당 라이브러리에 관한 정보를 입력한다.
- `name`: 라이브러리 이름
- `version`: 배포 버전
- `license`: 오픈소스 라이선스 정보
- `homepage`: 홈페이지 주소로 주로 Github 저장소의 메인 페이지를 사용
- `author`: 라이브러리 만든이의 이름과 이메일
- `summary`: 라이브러리에 대한 간단한 설명
- `source`: 라이브러리 소스가 위치해있는 원격 저장소 주소
- `source_files`: 소스 파일이 위치한 디렉토리 주소
- `frameworks`: 사용한 프레임워크
깃헙에 들어가서 해당 이름으로 되어 있는 레포를 만든다. 이걸 만들고 나서 올려야 하기 때문에.. 
    podspec 파일이 잘 올라갔는지 확인하자.
### 개발
워크스페이스를 열어보면 (예시 앱을 선택했다는 가정 하에) 예시 앱 프로젝트와 Pods 프로젝트가 있다.
Pods에서 ReplaceMe라고 쓰여 있는 파일이 있는 공간이 실제 라이브러리 코드를 개발하는 공간.
접근제어자에 주의하자! 라이브러리나 프레임워크에서 사용자가 사용하기를 원하는 부분은 public과 open 중 하나로 선언되어야 한다.
### 배포
podspec을 다시 확인하자.
``` shell
pod spec lint
```
다양한 오류를 검출해준다.
    예시로 나온 건 스위프트 버전을 안 적었잖아! / Xcode 설정에 커맨드 라인 툴이 없잖아(simctl)! / 저장소에는 버전 태그가 없잖아! 오류 정도
그 다음엔 코코아팟 계정을 만들어야 한다.
``` shell
# 새로운 컴퓨터에서 작업할 때마다 해 줘야 하는 작업
pod trunk register {이메일주소} {이름}
```
마참내 배포를 할 수가 있다.
``` shell
pod trunk push {라이브러리 이름}.podspec
```

## Carthage
간단 명령어
``` shell
carthage update # Cartfile을 기반으로 모든 의존성 갱신하고 빌드. 마찬가지로 각각도 가능
carthage bootstrap # Cartfile.resolved를 기반으로 모든 의존성 갱신하고 빌드
carthage build # 갱신 안 하고 모든 의존성을 빌드
carthage outdated # 새로운 버전이 있는 항목들 출력
```
### 생성 및 개발
Xcode에서 New Project - iOS - Framework로 만들어야 한다.
처음 만들었을 땐 헤더 파일이랑 info.plist 파일밖에 없다.
프로덕트 이름으로 이름지어져 있는 폴더 하에 소스 코드를 넣으면 된다.
Manage Schemes에 들어가 Shared 항목을 체크한다. 체크가 되어 있더라도 한 번 더 껐다 켜 줘야 한다.
Scheme을 Generic iOS Device로 설정하고 잘 빌드되나 확인한다.
### 배포
그냥 카르타고 지원한다는 배지만 리드미에 달아 주고 원격 저장소에 푸시하면 된다.
``` markdown
[![Carthage Compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)
```
물론 앞에서 코코아팟 할 때 원격 저장소를 만들어서 그런 거니까, 만약 안 했다면 레포를 만드는 걸 잊지 말자.
### 사용
이게 좀 웃긴데, Cartfile을 프로젝트 디렉토리에 만들어줘야 한다.
vi를 통해 Cartfile을 만들고, 내용을 넣어 준다.
``` vi
github "{레포주인}/{레포이름}"
```
이걸 하고 나서 `carthage update`해 준다.
이후 Xcode 프로젝트 내에서 Frameworks, Libraries and Embedded content 메뉴로 간다.
Add Other... -> add files한 다음, Carthage - build - iOS - {이름}.framework 파일을 선택하자.
## SPM
명령어는 딱히?
### 생성 및 개발
Podfile, Cartfile 같은 역할을 하는 것이 바로 Package.swift 파일.
기존의 라이브러리 파일에 Package.swift 파일만 추가해주면 쓸 수 있다. 앞서 카르타고 라이브러리 만들 때 썼던 프레임워크 프로젝트에 추가해 줘볼까나
    단, xcodeproj 파일이 있는 바로 그 폴더에 추가해야 한다.
``` swift
// swift-tools-version:5.1

import PackageDescription

let package: Package(
    name: "MyLibrary",
    platforms: [
        .iOS(.v15)
    ],
    products: [
        .library(
            name: "MyLibrary",
            targets: ["MyLibrary"]
        )
    ],
    targets: [
        .target(
            name: "MyLibrary",
            path: "MyLibrary/Sources"
        )
    ],
    swiftLanguageVersions: [
        .v5
    ]
)
```
맨 위에 주석으로 달아 둔 스위프트 툴 버전은 PackageDescription의 버전을 말하며, 꼭 써 주어야 한다.
각 항목은..
    name: 이름(...).
    platforms: 지원 플랫폼. iOS, macOS, tvOS, watchOS를 쓸 수 있다.
    products: library와 executable이 있다. 후자의 경우 사용자가 실행할 수 있는 뭔가를 제공한다.
    targets: 동일한 패키지 안에서 의존성이 지정된 것들을 나타낸다. 소스 코드라던지?
    swiftLanguageVersions: 지원하는 Swift 언어 버전을 나타낸다.
### 배포
리드미에 SPM 지원 배지 달고 푸시하면 된다...
``` markdown
[![SwiftPM](https://img.shields.io/badge/SPM-supported-DE5C43.svg?style=flat)](https://swift.org/package-manager/)
```
머 이렇게 배포할 때 버전 정도 바꿔 주면 괜찮을지도
### 사용
Local과 Remote 방식이 있다.
로컬 방식의 경우, Package.swift가 들어 있는 폴더를 드래그 앤 드랍해서 프로젝트 파일 목록에 갖다 놓으면 추가가 된다..
    이후 Xcode 프로젝트 내에서 Frameworks, Libraries and Embedded content 메뉴로 간다.
    카르타고와 달리 Add Other까지 갈 건 없고, 프레임워크 목록에 MyLibrary가 나타날 것이다. 그걸 선택하면 된다.
리모트 방식은 맨날 하던 그 방식. File - Swift Packages - Add Package Dependency를 선택하고 깃헙 레포 주소 치고 버전 설정 하고 페칭해오면 된다.
    리모트 방식은 프로젝트 내에서 설정해줄 건 없지만, 로컬과 달리 버전 정보는 필요하다.

