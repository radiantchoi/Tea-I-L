# Router Pattern과 Coordinator Pattern 간단정리

## Router Pattern
- 간단히 말하면, 요청하는 부분과 요청을 처리하는 부분을 분리하는 패턴.
- 예를 들면, UI상에서 받아들이는 클라이언트 요청은 그 자체로 어떤 효과를 일으키지 않고, 적절한 핸들러로 이관되어 수행된다.

### Router Pattern을 활용한 네트워크 통신
```Swift
enum APIRouter {
    case getUsers
    case createUser(name: String, email: String)
    case updateUser(id: Int, name: String, email: String)
    // other API endpoint cases...
    
    var baseURL: URL {
        return URL(string: "https://api.example.com")!
    }
    
    var path: String {
        switch self {
        case .getUsers:
            return "/users"
        case .createUser:
            return "/users"
        case .updateUser(let id, _, _):
            return "/users/\(id)"
        }
    }
    
    var method: String {
        switch self {
        case .getUsers:
            return "GET"
        case .createUser:
            return "POST"
        case .updateUser:
            return "PUT"
        }
    }
    
    var parameters: [String: Any]? {
        switch self {
        case .getUsers:
            return nil
        case .createUser(let name, let email):
            return ["name": name, "email": email]
        case .updateUser(_, let name, let email):
            return ["name": name, "email": email]
        }
    }
    
    func asURLRequest() throws -> URLRequest {
        let url = baseURL.appendingPathComponent(path)
        var request = URLRequest(url: url)
        request.httpMethod = method
        
        if let parameters = parameters {
            request.httpBody = try JSONSerialization.data(withJSONObject: parameters, options: [])
        }
        
        return request
    }
}

// 사용 예시
let router = APIRouter.getUsers
let request = try? router.asURLRequest()

// URLSession을 사용하여 네트워크 요청 수행
URLSession.shared.dataTask(with: request!) { (data, response, error) in
    // 응답 처리
}.resume()
```
- 사용자가 직접 이런저런 정보를 신경써서 입력하지 않고, 적절히 필요하다 싶은 정보만 넘겨 주면 알아서 라우터가 처리해 준다.
  - 기초적인 통신에서는 하다 못해 URL을 파라미터에 넣어 줘야 하는 경우도 있음을 생각하면, 적절한 처리.
- 어디서 많이 봤다는 생각이 들 수도 있다. Swift 생태계 안에서는 Moya가 바로 이러한 형태의 네트워크 요청 추상화를 수행한다!
- 실제로 라우터 패턴의 장점 중 캡슐화가 포함되어 있다.
  - 요청 처리 로직의 집중화
  - 각 요청별 처리 로직의 모듈화
  - 개발의 유연성 - 새로운 요청 유형 쉽게 추가 가능
  - URL 매핑의 용이함

## Coordinator Pattern
- iOS 앱에서, 내비게이션 흐름을 관리하기 위한 패턴.
- 내비게이션 컨트롤러를 예로 들면, 푸시나 프레젠트 등의 화면 전환을 수행하기 위해서는 뷰가 다음 뷰에 대해 "알고" 있어야 한다. 뭔가.. 클린하지 않아..!
- 그래서 오직 화면 전환만을 담당하는 Coordinator 객체를 둠으로써 화면 전환을 전담시킨 것이다.
- 요청을 직접 수행하지 않고 적절한 핸들러가 수행하게 한다. 이것은 본질적으로 라우터 패턴과 유사하다!

### Coordinator Pattern의 실제
- Coordinator 자체는 프로토콜로 정의된다.
```Swift
protocol Coordinator: AnyObject {
    func start()
}
```
- 다음으로 앱 안의 모든 코디네이터의 어머니이자 근본인 `AppCoordinator`(가칭)이 있다.
- `AppCoordinator`는 앞으로 벌어질 화면 전환 로직에 대해 다 알고 있다. 어떤 뷰에서 뭘 누르면 어느 뷰로 넘어가는지.
  - 정확히 말하면, 상위 코디네이터가 하위 코디네이터의 화면 전환 로직에 대해 알고 있는 것이다.
- `AppCoordinator` 하에서, 전환되는 화면의 코디네이터가 존재할 수 있다. `TabBarCoordinator`라던지.
- 그래서 종국에는 코디네이터 간에 의존 관계 나무와도 같은 것을 가진 형태가 된다.
```Swift
// start뿐만 아니라 화면 전환 과정에서 필요한 여러 로직을 적용할 수 있다.
// 화면이 종료되었을 때는 내비게이션 컨트롤러 안의 모든 뷰 컨트롤러를 없앨 수도 있고,
// 언어에 따라 적용하는 자식 뷰를 다르게 할 수도 있다.
// window 프로퍼티가 아닌 navigation controller 프로퍼티를 가질 수도 있다.
// 필요하다면 화면 간의 분기 처리도 할 수 있다!

class AppCoordinator: Coordinator {
    let window: UIWindow
    
    init(window: UIWindow) {
        self.window = window
    }
    
    func start() {
        let homeCoordinator = HomeCoordinator(window: window)
        homeCoordinator.start()
    }
}

class HomeCoordinator: Coordinator {
    let window: UIWindow
    
    init(window: UIWindow) {
        self.window = window
    }
    
    func start() {
        let homeVC = HomeViewController()
        let navigationController = UINavigationController(rootViewController: homeVC)
        window.rootViewController = navigationController
        window.makeKeyAndVisible()
    }
    
    func showDetail() {
        let detailCoordinator = DetailCoordinator(navigationController: window.rootViewController as! UINavigationController)
        detailCoordinator.start()
    }
}

class DetailCoordinator: Coordinator {
    let navigationController: UINavigationController
    
    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }
    
    func start() {
        let detailVC = DetailViewController()
        navigationController.pushViewController(detailVC, animated: true)
    }
}
```
- 라우터건 코디네이터건, 난잡하게 흩어지기 쉬운 "동작"을 추상화하여 한 곳에서 집중적으로 관리한다는 공통점이 있다는 점 다시 짚고 넘어간다.