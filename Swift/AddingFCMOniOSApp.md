# Complete guide to apply FCM on iOS App ver.202405
- FCM: Firebase Cloud Messaging.
    - 애플 푸시 알림 서비스, APNS를 위해서는 해당 서비스를 이용하는 애플리케이션의 전용 서버가 필요한데, 이를 대신 수행해 준다.
    - 간편하다보니 푸시에 한해서는 전용 서버가 있는 앱도 FCM을 사용하는 경우가 많다.
- 워낙 많이 바뀌다보니.. 지금 시점을 기준으로 한 글이라고 보면 편하다.

## Apple Developer 세팅
- 순서상 애플 개발자 페이지에서 앱 세팅을 먼저 해 주는 것이 편하다.
- 먼저 Certificates, Identifiers & Profiles에서 내 앱을 위한 Identifier를 하나 만든다.
    - App IDs를 만들고, App을 선택하고, 설명과 앱 번들 아이디를 넣은 다음, 하단에서 Push Notifications를 선택한다.
        - 여기서 잠깐! 앱 번들 아이디는 (다들 아시겠지만) 리버스 도메인 형식으로 되어 있으며, 엑코에서 확인 가능하다.
        - com.어쩌구.저쩌구
- 다음으로는 Keys에 들어가서 새로 생성하고, 시키는 대로 이름과 APNs 활성화를 하고 레지스터하면 된다.
    - Key는 한 사람당 두 개씩만 가질 수 있으니 주의!
    - Key를 생성하면 .p8 파일을 다운받을 수 있는데, 주의사항에 써 있듯 한 번 다운받으면 다시 다운이 불가능하다!
    - 파일도 잘 받아서 보관해 두고, 팀 아이디도 잘 보관해 두자. Firebase 세팅에 필요하다.

## Google Console 세팅
- Firebase 콘솔 상단 Firebase 프로젝트 상단 배너에서 "프로젝트 추가"를 클릭한다.
- 시키는 대로 프로젝트 이름 짓고, 구글 애널리틱스도 하고 싶으면 하고. 업무에서도 유용한 툴이니까!
- 프로젝트를 만들었다면 좌상단 기어 버튼을 눌러서 프로젝트 설정으로 들어간다.
- 프로젝트에 "내 앱"을 추가한다. 이 경우는 iOS+ 버튼을 누르면 된다.
    - 1단계에서 시키는 대로 이름을 넣는다. 이름만 넣어도 일단 만들어진다.
    - 2단계에서 다운받은 GoogleService-Info.plist를 그림에 써 있는 위치, 앱의 루트 폴더에 넣는다. 친절함!
    - Swift Package Manager를 활용해 Firebase SDK 의존성을 설치한다.
        - 이거는 Xcode 세팅이긴 한데, 시키기도 하고 어차피 이따가 더 세팅할거니까 그냥 지금 켜자.
        - File - Add Package Dependencies에서 추가할 수 있다.
    - init 코드를 제공한다. 아까 켜 둔 Xcode에서 이어서 작업하자. 어차피 5단계는 완료! 따위의 것이다.

## Xcode 세팅
- 코드를 치기 전, Target 정보에서 Frameworks, Libraries and Embedded Contents에 필요한 것들이 잘 의존되어 있나 확인하자.
    - FirebaseAnalytics, FirebaseMessaging 라이브러리는 적어도 의존해야 한다.
- UIKit이라면 AppDelegate, SwiftUI라면 App에다가 어댑터를 만들어서 초기화 코드를 짜 준다.
``` swift
// UIKit
@main
class AppDelegate: UIResponder, UIApplicationDelegate {
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        if #available(iOS 12.0, *) {
            // 퍼스트 파티 프레임워크 UNUserNotificationCenter에서 알림 설정 권한을 요청해 받아 온다.
            UNUserNotificationCenter.current().requestAuthorization(
                // deprecate되었지만, 옵션에서는 여전히 .alert가 사용된다.
                options: [.alert, .sound, .badge, .providesAppNotificationSettings], completionHandler: { didAllow,Error in
            })
        } else {
            // 아직도 iOS 12 미만에서 사용하는 흑우 없제?
            UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .sound, .badge], completionHandler: {didAllow,Error in
                print(didAllow)
            })
        }
        // UNUserNotificationCenter도 Delegate가 필요하다. 알림 수신 및 향후 액션을 지원하려면.
        UNUserNotificationCenter.current().delegate = self
        
        // Firebase의 대리자로서 AppDelegate 설정 및 리모트 알림 수신 준비
        FirebaseApp.configure()
        Messaging.messaging().delegate = self
        
        application.registerForRemoteNotifications()
        
        return true
    }
}

extension AppDelegate: UNUserNotificationCenterDelegate, MessagingDelegate {
    // 최초 통신시 기기의 고유한 토큰을 요청하여 받아 온다. 이 토큰을 토대로 알림을 받는다.
    public func messaging(_ messaging: Messaging, didReceiveRegistrationToken fcmToken: String?) {
        let firebaseToken = fcmToken ?? ""
        print("firebase token: \(firebaseToken)")
    }
    
    // 알림이 왔을 때 어떤 것을 표시할지에 관한 설정이다.
    // 기존에 쓰던 .alert는 Deprecated되었다.
    // banner는 알림 배너(위쪽), list는 알림 센터, badge는 앱 뱃지, sound는 알림음을 표시할 것임을 각각 나타낸다.
    public func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        completionHandler([.banner, .list, .badge, .sound])
    }
    
    // 알림을 가지고 뭘 할지! 잠시 후 더 자세히 알아본다.
    public func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
        completionHandler()
    }
}
```

``` swift
// SwiftUI
// 어댑터 빼고는 UIKit 코드를 복사-붙여넣기 해도 된다.
@main
struct SomethingUsefulSwiftUIApp: App {
    @UIApplicationDelegateAdaptor(AppDelegate.self) var appDelegate
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

// 나머지는 거의 위의 코드를 복붙해야 하는데, 하나 추가해야 하는 것이 있다.
// AppDelegate는 이미 정의했다고 가정하고
extension AppDelegate {
    func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
            Messaging.messaging().apnsToken = deviceToken
    }
}
// 어댑터를 통해 AppDelegate의 기능을 대리해주기 때문이다.
// 단 @main은 빼고! @main은 앱의 시작점을 나타내며, 하늘 아래엔 두 개의 태양이 존재할 수 없다.
```

## 테스트 메시지 전송

### Firebase 자체 제공
- 다시 Firebase 콘솔로 돌아와서, 모든 제품 - Cloud Messaging을 추가한다.
- 이후 프로젝트 바로가기 섹션의 Messaging으로 와서, 상단 배너의 첫 번째 캠페인 만들기를 클릭한다.
- 선택지에서 Firebase 알림 메시지를 선택한다. 우리는 이게 필요한 거니까.
- 제목이랑 텍스트만 입력하고 "테스트 메시지 전송"을 누른다.
- 누르기 전에! 앱을 실기기에서 빌드하면 상기 코드대로라면 fcm token이 올 것이다. 이를 테스트 메시지 전송 팝업에서 FCM 등록 토큰 추가 항목에 넣고 추가한다.
- 테스트 버튼을 누르면 응답이 온다!

### Postman
- Postman에서 테스트하기 위해서는 몇 가지 사전 준비가 필요하다.
- 먼저 didReceiveRegistrationToken 메서드를 구현한 다음, fcmToken 값을 기록해 둔다.
    - 이 메서드는 MessagingDelegate를 채택한 다음 구현한다.
    - 일정 주기마다 갱신되므로, 만약 유효하지 않다면 다시 앱을 빌드해서 실행시킨 후 가져올 것!
- 다음으로 Postman에서 POST 요청을 준비한다.
    - url은 https://fcm.googleapis.com/fcm/send
    - Header의 Authorization 키에 key={서버 키} 값을 할당한다.
        - 서버 키는 Firebase 콘솔 -> 프로젝트 설정 -> 클라우드 메시징 -> Cloud Messaging API(기존) 탭에서 받아올 수 있다. 만약 사용 설정이 되어 있지 않다면 사용하도록 해 준다.
        - 만약 헤더에 자동으로 들어가 있지 않다면, Content-Type 키에 application/json 값도 할당한다.
    - Body를 raw JSON으로 세팅해 주고 아래와 같은 값을 입력한다.
``` json
{
    "to":"{아까 받아온 fcm 토큰}",
    "notification":{
        "title":"{푸시 메시지 제목}",
        "body":"{푸시 메시지 내용}"
    },
    "data":{
        "linkId": "607",
        "info": "3",
        "type": "16"
    }
}
```
- 이러면 이제 메시지가 온다.

## 메시지 해석, 파싱 및 적용
- 위와 같은 메시지를 날렸을 때, didReceive 메서드에서 받는 userInfo의 형식은 다음과 같다.
```
[
	"google.c.fid": "{일단 신경쓰지 않아도 되는 값}",
	"google.c.sender.id": "{일단 신경쓰지 않아도 되는 값}",
	"gcm.message_id": "{일단 신경쓰지 않아도 되는 값}",
	"linkId": "607",
	"info": "3",
	"type": "16",
	"google.c.a.e": "{일단 신경쓰지 않아도 되는 값}",
	"aps": {
		alert = {
			body = "{푸시 메세지 내용}";
			title = {푸시 메세지 제목};
		};
	}
]
```
- notification 필드에 있는 내용이 자동으로 aps 키의 값으로 들어간다.
- 이외에 linkId, info, type 등 원하는 정보를 전송할 수 있다.
- 어떤 원하는 정보를 전송할지는, 푸시 메시지의 data 항목에서 정하는 Key에 따라 결정된다.
    - 위의 예시에서 data 키 안에 들어 있는 linkId, info, type 등의 키가 그대로 userInfo 딕셔너리에서 키로 작동함을 볼 수 있다. 
- data 항목에서 어떤 데이터가 왔느냐에 따라 이제 분기 처리를 할 수 있는 것.
