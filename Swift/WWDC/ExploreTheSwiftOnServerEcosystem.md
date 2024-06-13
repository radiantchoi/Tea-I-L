# Explore the Swift on Server Ecosystem
- 제발 서버 사이드에서도 Swift 쓸만하게 해 주세요 다른 언어까지 하고 싶지 않단 말이예요

## Meet Swift on Server
- C언어 수준의 퍼포먼스를 가지며, 동시에 메모리 발자국을 덜 남긴다.
    - 이는 GC 대신 ARC 사용을 하기 떄문에 가능한 것.
    - 요즘 클라우드 서비스는 자원 사용량의 예측 가능성과 빠른 시동 시간이 중요한데, 이에 부합한다.
- 표현력이 좋고 안전한 언어이다.
    - 많은 버그를 컴파일 타임에 인식해 방지한다.
    - 성숙하고 믿을 만 한 시스템을 배포할 수 있다.
    - 강타입, 옵셔널, 메모리 안전성 등의 특징은 크래시나 보안 취약점에 덜 노출되게 한다.
- 비동기 로드를 견뎌야 하는 클라우드 서버에 적합한 퍼스트-파티 비동기 지원이 있다.
- 그러니 Swift로 서버를 쓰는 것은 좋은 선택! 이라는데?
- 아이클라우드, 앱스토어, 셰어플레이 등의 서버적인 기능에 Swift가 실제로 쓰인다고 한다.
- Swift Server Workgroup이라는, 오래 된 서버 사이드 스위프트 그룹이 있다.

## Build a Service
- 이벤트 서비스를 만들어 보자! 이벤트 목록 제공, 새 이벤트 생성의 두 가지 기능을 가진다.
- 오늘은 VSCode로 짜 볼 거다. 끼얏호우~
- OpenAPI와 Vapor를 사용한다.
- 세션에선 원형을 미리 만들어 왔는데, 처음부터 만들어보고 싶다면 [여기](https://www.swift.org/getting-started/vapor-web-server/) 참조
```Swift
// swift-tools-version:5.9
import PackageDescription

let package = Package(
  name: "EventService",
  platforms: [.macOS(.v14)],
  dependencies: [
    .package(
      url: "https://github.com/apple/swift-openapi-generator",
      from: "1.2.1"
    ),
    .package(
      url: "https://github.com/apple/swift-openapi-runtime",
      from: "1.4.0"
    ),
    .package(
      url: "https://github.com/vapor/vapor",
      from: "4.99.2"
    ),
    .package(
      url: "https://github.com/swift-server/swift-openapi-vapor",
      from: "1.0.1"
    ),
  ],
  targets: [
    .target(
      name: "EventAPI",
      dependencies: [
        .product(
          name: "OpenAPIRuntime",
          package: "swift-openapi-runtime"
        ),
      ],
      plugins: [
        .plugin(
          name: "OpenAPIGenerator",
          package: "swift-openapi-generator"
        )
      ]
    ),
    .executableTarget(
      name: "EventService",
      dependencies: [
        "EventAPI",
        .product(
          name: "OpenAPIRuntime",
          package: "swift-openapi-runtime"
        ),
        .product(
          name: "OpenAPIVapor",
          package: "swift-openapi-vapor"
        ),
        .product(
          name: "Vapor",
          package: "vapor"
        ),
      ]
    ),
  ]
)
```
- 타겟이 두 개인데, EventAPI 타겟 그리고 실행 가능한 EventService 타겟이 있다.
- Swift OpenAPI Generator를 통해 우리의 서비스를 YAML로 문서화하고, 서버와 클라를 위한 코드를 만들 수 있다.
- 이를 위해서는 [WWDC23 Meet Swift OpenAPI Generator 세션](https://youtu.be/US5Blei-5Kg?si=EE4-6rE19CZpWx87)을 참고! 
- 일단 문서를 보자.
```yaml
openapi: "3.1.0"
info:
  title: "EventService"
  version: "1.0.0"
servers:
  - url: "https://localhost:8080/api"
    description: "Example service deployment."
paths:
  /events:
    get:
      operationId: "listEvents"
      responses:
        "200":
          description: "A success response with all events."
          content:
            application/json:
              schema:
                type: "array"
                items:
                  $ref: "#/components/schemas/Event"
    post:
      operationId: "createEvent"
      requestBody:
        description: "The event to create."
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Event'
      responses:
        '201':
          description: "A success indicating the event was created."
        '400':
          description: "A failure indicating the event wasn't created."
components:
  schemas:
    Event:
      type: "object"
      description: "An event."
      properties:
        name:
          type: "string"
          description: "The event's name."
        date:
          type: "string"
          format: "date"
          description: "The day of the event."
        attendee:
          type: "string"
          description: "The name of the person attending the event."
      required:
        - "name"
        - "date"
        - "attendee"
```
- 돌발 퀴즈! 이것만 보고 구조 파악해보기!
- 다음으로 서버 애플리케이션을 만드는데, 이것도 모바일 앱처럼 `@main`을 시작점으로 삼는다. 아래는 그 원형.
```Swift
import OpenAPIRuntime
import OpenAPIVapor
import Vapor
import EventAPI

@main
struct Service {
  static func main() async throws {
    // 먼저 서버 애플리케이션을 만들고
    let application = try await Vapor.Application.make()
    // OpenAPI 전송자를 만들고
    let transport = VaporTransport(routesBuilder: application)

    // 서비스 인스턴스 만들기
    let service = Service()
    try service.registerHandlers(
      on: transport,
      serverURL: URL(string: "/api")!
    )

    // 그리고 실행! HTTP 서버를 시작하고, 들어오는 요청을 받는다.
    try await application.execute()
  }
}

extension Service: APIProtocol {
  // 일단 초기 상태라 하드코딩 값을 내보낸다.
  func listEvents(
    _ input: Operations.listEvents.Input
  ) async throws -> Operations.listEvents.Output {
    let events: [Components.Schemas.Event] = [
      .init(name: "Server-Side Swift Conference", date: "26.09.2024", attendee: "Gus"),
      .init(name: "Oktoberfest", date: "21.09.2024", attendee: "Werner"),
    ]

    return .ok(.init(body: .json(events)))
  }

  func createEvent(
    _ input: Operations.createEvent.Input
  ) async throws -> Operations.createEvent.Output {
    return .undocumented(statusCode: 501, .init())
  }
}
```
- 하지만 우리가 만들고 싶은 건 동적으로 이벤트를 추가할 수 있는 앱이지 하드코딩 값을 내보내는 앱이 아니다.
- 많은 데이터베이스 사용 선택지가 있다만, 오늘은 PostgreSQL을 쓴다.
- [PostgresNIO](https://github.com/vapor/postgres-nio)는 애플과 Vapor가 함께 관리하는 오픈 소스 DB이다.
- 비동기 작업을 구현하기 위한 `PostgresClient`! 이걸 가지고 뭘 할거냐면
  - `PostgresNIO` 의존성을 가져오고
  - 데이터베이스 쿼리를 위해 `PostgresClient`를 쓰고
  - `createEvent` 메서드 만들기!
- 일단 의존성을 추가하자.
```Swift
// Package.swift

// swift-tools-version:5.9
import PackageDescription

let package = Package(
  name: "EventService",
  platforms: [.macOS(.v14)],
  dependencies: [
    // 요 네 개는 아까부터 있던 패키지들
    .package(
      url: "https://github.com/apple/swift-openapi-generator",
      from: "1.2.1"
    ),
    .package(
      url: "https://github.com/apple/swift-openapi-runtime",
      from: "1.4.0"
    ),
    .package(
      url: "https://github.com/vapor/vapor",
      from: "4.99.2"
    ),
    .package(
      url: "https://github.com/swift-server/swift-openapi-vapor",
      from: "1.0.1"
    ),
    // 여기 추가된 PostgresNIO
    .package(
      url: "https://github.com/vapor/postgres-nio",
      from: "1.19.1"
    ),
  ],
  targets: [
    .target(
      name: "EventAPI",
      dependencies: [
        .product(
          name: "OpenAPIRuntime",
          package: "swift-openapi-runtime"
        ),
      ],
      plugins: [
        .plugin(
          name: "OpenAPIGenerator",
          package: "swift-openapi-generator"
        )
      ]
    ),
    .executableTarget(
      name: "EventService",
      dependencies: [
        "EventAPI",
        .product(
          name: "OpenAPIRuntime",
          package: "swift-openapi-runtime"
        ),
        .product(
          name: "OpenAPIVapor",
          package: "swift-openapi-vapor"
        ),
        .product(
          name: "Vapor",
          package: "vapor"
        ),
        // 실행 가능한 서버 애플리케이션에 추가한 PostgresNIO
        .product(
            name: "PostgresNIO",
          package: "postgres-nio"
        ),
      ]
    ),
  ]
)
```
- 그리고 `PostgresNIO`를 활용해 실제 DB를 연동한다.
```Swift
import OpenAPIRuntime
import OpenAPIVapor
import Vapor
import EventAPI
import PostgresNIO

@main
struct Service {
  // 준비해둔 DB와 연결하기 위해 PostgresClient가 필요하다.
  let postgresClient: PostgresClient
  
  static func main() async throws {
    let application = try await Vapor.Application.make()
    let transport = VaporTransport(routesBuilder: application)

    let postgresClient = PostgresClient(
      configuration: .init(
        host: "localhost",
        username: "postgres",
        password: nil,
        database: nil,
        tls: .disable
      )
    )
    let service = Service(postgresClient: postgresClient)
    try service.registerHandlers(
      on: transport,
      serverURL: URL(string: "/api")!
    )

    try await withThrowingDiscardingTaskGroup { group in
      group.addTask {
        // 이쪽 클라이언트도 run 해줘야 작동한다. Vapor 애플리케이션과는 독립적으로, 동시성을 가지고 실행되어야 하므로 Task에 넣는다.
        await postgresClient.run()
      }

      group.addTask {
        try await application.execute()
      }
    }
  }
}

extension Service: APIProtocol {
  func listEvents(
    _ input: Operations.listEvents.Input
  ) async throws -> Operations.listEvents.Output {
    // listEvents 메서드 안에서 쿼리를 넣어 준다.
    // 반환값은 AsyncSequence로, DB에서 자동으로 프리-페칭을 함으로써, 성능 향상을 꾀한다.
    let rows = try await self.postgresClient.query("SELECT name, date, attendee FROM events")

    // 각각의 row를 순회하고, 필드들을 디코딩한 다음, 이벤트로서 결과값 배열에 넣는다.
    var events = [Components.Schemas.Event]()
    for try await (name, date, attendee) in rows.decode((String, String, String).self) {
      events.append(.init(name: name, date: date, attendee: attendee))
    }

    return .ok(.init(body: .json(events)))
  }

  func createEvent(
    _ input: Operations.createEvent.Input
  ) async throws -> Operations.createEvent.Output {
    return .undocumented(statusCode: 501, .init())
  }
```
- 앗 근데 지금은 DB가 빈 것 같지? 그래서 createEvent 메서드를 만들어줘야 한다.
```Swift
func createEvent(
  _ input: Operations.createEvent.Input
) async throws -> Operations.createEvent.Output {
  // 인풋이 json인지 먼저 점검하고
  switch input.body {
  case .json(let event):
  
    // SQL Injection에 취약한 거 아니냐고?
    // 이거 사실 String 아니고, 파라미터가 있는 쿼리로 자동으로 변환된다. 이는 Swift String Interpolation의 힘을 빌린 것.
    try await self.postgresClient.query(
      """
      INSERT INTO events (name, date, attendee)
      VALUES (\(event.name), \(event.date), \(event.attendee))
      """
    )
    
    // 쿼리를 통해 자료로 변환해서 반환한다.
    return .created(.init())
  }
}
```
- 한편 오류가 뜰 때를 위해 로깅, 측정 그리고 추적을 쓸 수 있다.
  - 로깅은 각각의 상황에 대한 자세한 디스크립션을 명시할 수 있다.
  - 측정은 서비스의 상태에 대한 고수준의 오버뷰를 제공한다.
  - 하나의 요청이 서비스의 어떤 경로를 탔는지를 위해 추적을 쓸 수 있다.
```Swift
func listEvents(
  _ input: Operations.listEvents.Input
) async throws -> Operations.listEvents.Output {
  // 처리 시작했을 때 로그 찍기
  let logger = Logger(label: "ListEvents")
  logger.info("Handling request", metadata: ["operation": "\(Operations.listEvents.id)"])

  // 얼마나 많은 요청을 수행했는지 측정
  Counter(label: "list.events.counter").increment()

  // withSpan - 추적을 통해 데이터베이스에 들어온 요청 하나하나를 볼 수 있다.
  return try await withSpan("database query") { span in
    let rows = try await postgresClient.query("SELECT name, date, attendee FROM events")
    return try await .ok(.init(body: .json(decodeEvents(rows))))
  }
}
```
- 이 세 가지에 대해 자세히 알고 싶다면 [WWDC23 - Beyond the basics of structured concurrency](https://www.youtube.com/watch?v=Ce0GsaRCMew) 참조!
- 이 세 가지를 지원하는 여러 가지 백엔드가 있다.
- 이것들을 쓰려면 부트스트랩을 잘 해 줘야 한다. 이벤트를 하나라도 놓치면 안 되니까..
```Swift
@main
struct Service {
    static func main() async throws {
        LoggingSystem.bootstrap(StreamLogHandler.standardError)
        
        let registry = PrometheusCollectorRegistry()
        registry.bootstrap(PrometheusMetricsFactory(registry: registry)
        
        let otelTracer = OTel.Tracer(...)
        InstrumentationSystem.bootstrap(otelTracer)
    }
}
```
- 이제 에러를 캐치해보자.
```Swift
// Package.swift
// swift-tools-version:5.9
import PackageDescription

let package = Package(
  name: "EventService",
  platforms: [.macOS(.v14)],
  dependencies: [
    .package(
      url: "https://github.com/apple/swift-openapi-generator",
      from: "1.2.1"
    ),
    .package(
      url: "https://github.com/apple/swift-openapi-runtime",
      from: "1.4.0"
    ),
    .package(
      url: "https://github.com/vapor/vapor",
      from: "4.99.2"
    ),
    .package(
      url: "https://github.com/swift-server/swift-openapi-vapor",
      from: "1.0.1"
    ),
    .package(
      url: "https://github.com/vapor/postgres-nio",
      from: "1.19.1"
    ),
    // 로거 의존성 더하기
    .package(
        url: "https://github.com/apple/swift-log",
        from: "1.5.4"
    ),
  ],
  targets: [
    .target(
      name: "EventAPI",
      dependencies: [
        .product(
          name: "OpenAPIRuntime",
          package: "swift-openapi-runtime"
        ),
      ],
      plugins: [
        .plugin(
          name: "OpenAPIGenerator",
          package: "swift-openapi-generator"
        )
      ]
    ),
    .executableTarget(
      name: "EventService",
      dependencies: [
        "EventAPI",
        .product(
          name: "OpenAPIRuntime",
          package: "swift-openapi-runtime"
        ),
        .product(
          name: "OpenAPIVapor",
          package: "swift-openapi-vapor"
        ),
        .product(
          name: "Vapor",
          package: "vapor"
        ),
        .product(
            name: "PostgresNIO",
          package: "postgres-nio"
        ),
        // 로거 의존성 더하기
        .product(
            name: "Logging",
            package: "swift-log"
        ),
      ]
    ),
  ]
)
```
```Swift
func createEvent(
  _ input: Operations.createEvent.Input
) async throws -> Operations.createEvent.Output {
  switch input.body {
  case .json(let event):
    do {
      try await self.postgresClient.query(
        """
        INSERT INTO events (name, date, attendee)
        VALUES (\(event.name), \(event.date), \(event.attendee))
        """
      )
      return .created(.init())
    } catch let error as PSQLError {
      // 에러 핸들링 자체는 통상적인 Swift 에러 핸들링과 비슷하다
      let logger = Logger(label: "CreateEvent")

      if let message = error.serverInfo?[.message] {
        logger.info(
          "Failed to create event",
          metadata: ["error.message": "\(message)"]
        )
      }
      
      return .badRequest(.init())
    }
  }
}
```

## Explore the Ecosystem
- 위에서 말한 것은 이 생태계의 일부일 뿐!
- https://www.swift.org/packages/server.html
- Swift Server Workgroup의 Incubation process를 체크해보는 것도 좋다.