# Dive Deeper into SwiftData
https://developer.apple.com/wwdc23/10196
- Resources에서 다운로드받을 수 있는 여행 샘플 앱을 활용한다.

### `@Model`
- SwiftData를 활용하면 실행 취소, 응용 프로그램 전환 등의 상황에서 자동 저장을 하는 등의 표준 플랫폼 기능을 쉽게 구현할 수 있다.
- Class, Struct 등과 함께 동작하도록 애시당초 구현되었으며, 이는 `@Model` 매크로로 구현된다.
```Swift
@Model
final class Trip {
    var destination: String?
    var end_date: Date?
    var name: String?
    var start_date: Date?
  
    @Relationship(.cascade)
    var bucketListItem: [BucketListItem] = [BucketListItem]()
  
    @Relationship(.cascade)
    var livingAccommodation: LivingAccommodation?
}
```
- 앱에서 쓰이는 다른 객체에 대한 참조도 있음에 주의!
- SwiftData를 활용해 최소한의 코드만으로 이 모델이 지속하고 싶은 모델임을 알렸고, 다른 참조 타입과 어떻게 동작해야 하는지 서술했다.
- 자동으로 구조를 추론하는 기능도 있다.
- 데이터 저장 방식도 정확히 서술할 수 있다. Model Your Schema 세션 참조
- 이 클래스가 이제 맡게 되는 역할은...
    - 응용 프로그램의 객체 그래프인, 스키마를 서술
    - 코드를 작성할 수 있는 (DB와 프로그램 사이의) 인터페이스
- 이런 점에서 `@Model` 매크로는 SwiftData를 사용하는 앱의 중심 - 접촉 중심점 - 이다.

## 지속성(persistence) 설정하기

### `ModelContainer`
- 스키마를 소모해 모델 클래스의 인스턴스를 보유하는 데이터베이스를 만든다.
- 코드 상에서 모델 클래스의 인스턴스를 다룰 때는 `ModelContext`에 연결된다.
- 데이터가 기기에서 저장되고 지속되는 방식에 대해 서술한다.
  - 말하자면 스키마와 스키마의 지속성을 잇는 다리 정도?
- 객체가 어떻게 저장되는지에 대해서도 묘사한다.
  - 메모리에 저장할 것인가? 디스크에 저장할 것인가?
- 버저닝, 마이그레이션, 그래프 분리 등 관리 측면에서도 사용된다.
```Swift
// ModelContainer initialized with just Trip
let container = try ModelContainer(for: Trip.self)

// SwiftData infers related model classes as well
let container = try ModelContainer(
    for: [
        Trip.self, 
        BucketListItem.self, 
        LivingAccommodation.self
    ]
)
```
- `ModelContainer`는 이렇게 타입만 받아도, 나머지 스키마를 추론해서 처리해 준다.
- 다양하고 강력한 init이 있고, `ModelConfiguration` 클래스를 사용해 구성을 복잡하게 가져갈 수도 있다.

### `ModelConfiguration`
- 스키마의 지속성에 대해 서술한다.
- 데이터의 저장 위치를 제어한다.
  - 임시 데이터는 메모리, 영구 데이터는 디스크, 이런 식으로.
- 사용자가 선택한 특정 파일 URL을 쓰거나, 응용 프로그램의 권한을 사용해 자동으로 URL을 생성한다.
- 읽기 전용 모드로 불러오는 동작도 가능하다.
- 하나 이상의 CloudKit 컨테이너를 쓰는 프로그램이라면, 스키마 `ModelConfiguration`의 일부로 지정할 수 있다.
```Swift
// 사용할 유형이 모두 포함된 전체 스키마
let fullSchema = Schema([
    Trip.self,
    BucketListItem.self,
    LivingAccommodations.self,
    Person.self,
    Address.self
])

// 구성을 선언한다. 상술한 모델 중 몇몇을 포함한다.
// 저장에 사용할 파일의 URL을 지정할 수 있다.
// CloudKit에 동기화할 때 사용할 컨테이너 id도 입력해 준다.
let trips = ModelConfiguration(
    schema: Schema([
        Trip.self,
        BucketListItem.self,
        LivingAccommodations.self
    ]),
    url: URL(filePath: "/path/to/trip.store"),
    cloudKitContainerIdentifier: "com.example.trips"
)

// 사람에 대한 설정은 따로 저장한다. Trips 그래프에서 데이터를 분리할 수 있다.
let people = ModelConfiguration(
    schema: Schema([Person.self, Address.self]),
    url: URL(filePath: "/path/to/people.store"),
    cloudKitContainerIdentifier: "com.example.people"
) 

// 상술한 스키마와 구성을 결합해 ModelContainer를 설정한다.
let container = try ModelContainer(for: fullSchema, trips, people)
```
- 앱의 복잡한 지속성 요구 사항을 간단히 서술할 수 있다.
- 수동으로 컨테이너를 인스턴스화하는 것에 더해 SwiftUI 앱이라면 `.modelContainer` 모디파이어로 원하는 컨테이너를 생성할 수도 있다.
```Swift
// ModelContainer는 앱 내의 View, Scene을 막론하고 추가할 수 있다.
// 간단한 것도, 강력하고 복잡한 것도 모두 가능하다.
@main
struct TripsApp: App {
    let fullSchema = Schema([
        Trip.self, 
        BucketListItem.self,
        LivingAccommodations.self,
        Person.self, 
        Address.self
    ])
  
    let trips = ModelConfiguration(
        schema: Schema([
            Trip.self,
            BucketListItem.self,
            LivingAccommodations.self
        ]),
        url: URL(filePath: "/path/to/trip.store"),
        cloudKitContainerIdentifier: "com.example.trips"
    )
  
    let people = ModelConfiguration(
        schema: Schema([
            Person.self, 
            Address.self
        ]),
        url: URL(filePath: "/path/to/people.store"),
        cloudKitContainerIdentifier: "com.example.people"
    )
  
    let container = try ModelContainer(for: fullSchema, trips, people)
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(container)
    }
}
```

## 변경사항 추적하고 지속하기

### `ModelContext`
- Model과 ModelContext는 UI 작성이나 모델 운용시 가장 자주 쓰이는 개념이다.
- `ModelContainer`를 통해 이것들에 관련된 변경사항을 추적한다.
```Swift
@main
struct TripsApp: App {
   var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Trip.self)
    }
}
```
- `.modelContainer`를 사용하면, Environment에 새로운 modelContext 키를 mainContext에 바인딩한다.
  - mainContext는 특수한 MainActor 모델 컨텍스트로, 씬이나 뷰에서 ModelObject와 함께 작동한다.
- 사용할 때는..?
```Swift
struct ContentView: View {
    @Query var trips: [Trip]
    // environment에 key로 들어가 있는 것을 keyPath로 끄집어 내어 사용할 수 있다.
    @Environment(\.modelContext) var modelContext
  
    var body: some View {
        NavigationStack (path: $path) {
            List(selection: $selection) {
                ForEach(trips) { trip in
                    TripListItem(trip: trip)
                        .swipeActions(edge: .trailing) {
                            Button(role: .destructive) {
                                // 저 환경 변수로 인해 삭제 액션을 수행할 수 있는 것
                                modelContext.delete(trip)
                            } label: {
                                Label("Delete", systemImage: "trash")
                            }
                        }
                }
                .onDelete(perform: deleteTrips(at:))
            }
        }
    }
}
```
- `ModelContext` 쓰기 쉬운 거 알겠는데 역할이 뭘까? 앱이 관리하는 데이터에 대한 뷰로 생각할 수 있다.
- 작업할 데이터는 사용 중인 `ModelContext`로 페칭된다.
  - 상기 앱의 경우, 각각의 여행 객체를 메인 컨텍스트로 가져온다.
- 변경사항이 발생하면 Context가 이를 추적하고 유지하다가 `context.save()`가 호출되면 멈춘다.
  - 여행 목록에서 여행을 삭제했더라도, `save()` 메서드를 통해 지속성이 부여되기 전까지는 삭제되지 않고 있다는 말..!
  - 저장이 호출되면, Context는 변경 사항을 Container에 지속시키고 상태를 지운다.
- 한편 Context가 계속 참조하는 객체는 사용이 끝날 때까지 거기 남는다. 

### `ModelContext`의 역할
- 사용 중인 객체 추적
- 변경사항을 컨테이너에 저장
- 롤백이나 리셋을 통해 캐싱된 상태 삭제
- 실행 취소나 자동 저장 등의 기능을 지원하기 좋다!
```Swift
// isUndoEnabled를 통해 추가 코드 없이 시스템 제스처를 사용해 undo 동작을 도입할 수 있다.
@main
struct TripsApp: App {
   @Environment(\.undoManager) var undoManager
   var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Trip.self, isUndoEnabled: true)
    }
}
```
- Context가 자동으로 실행 취소 및 복귀 동작을 등록한다.
- 기본 시스템 동작으로 또 제공하는 것은 자동 저장!
  - Context가 시스템 이벤트에 저장된다 - foreground, background에 접어들 때 등
```Swift
// 단 수동으로 ModelContext를 만들었다면 자동 저장은 비활성화된다.
@main
struct TripsApp: App {
   var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Trip.self, isAutosaveEnabled: false)
    }
}
```

## 코드에서 모델을 다룰 때 SwiftData 활용하기
- UI에서만 데이터를 다루는 것은 아니지! 하기 작업은 모두 모델 객체를 집합이나 그래프 형태로 동반한다.
  - 백그라운드 큐에서 데이터 다루기
  - 원격 서버나 다른 영구 지속 메커니즘과 동기화
  - Batch 프로세싱
- 이 과정에서 `ModelContext`의 fetch 메서드로 작업할 객체 집합을 많이들 가져 온다.
```Swift
let context = self.newSwiftContext(from: Trip.self)
var trips = try context.fetch(FetchDescriptor<Trip>())
```

### `FetchDescriptor`
- 복잡한 쿼리를 손쉽게 만들 수 있다.
```Swift
let context = self.newSwiftContext(from: Trip.self)
let hotelNames = ["First", "Second", "Third"]

// predicate는 일종의 조건문
// 이걸 Swift 코드 조건문으로 작성할 수 있다는 것이 강점.
var predicate = #Predicate<Trip> { trip in
    trip.livingAccommodations.filter {
        hotelNames.contains($0.placeName)
    }.count > 0
}

// 조건문 작성시에는 Swift 모델을 썼지만, 이에 기반해 쿼리를 반환하고 지속해줄 수 있다.
var descriptor = FetchDescriptor(predicate: predicate)
var trips = try context.fetch(descriptor)
```
- `FetchDescriptor` 이외에 `SortDescriptor`도 있다.
  - 이들은 Result 타입 제네릭을 활용해 결과 타입을 만들고 사용 가능한 모델을 컴파일러에게 알려 준다.
  - offset, limit, faulting, prefetching 등의 옵션이 있다.
- 이 모든 기능이 `ModelContext`의 `enumerate` 함수에서 결합된다는 거!
```Swift
let predicate = #Predicate<Trip> { trip in
    trip.bucketListItem.filter {
        $0.hasReservation == false
    }.count > 0
}

let descriptor = FetchDescriptor(predicate: predicate)
descriptor.sortBy = [SortDescriptor(\.start_date)]

context.enumerate(descriptor) { trip in
    // Remind me to make reservations for trip
}
```
- 한편 커스터마이징이 가능한 부분도 있다.
```Swift
let predicate = #Predicate<Trip> { trip in
    trip.bucketListItem.filter {
        $0.hasReservation == false
    }.count > 0
}

let descriptor = FetchDescriptor(predicate: predicate)
descriptor.sortBy = [SortDescriptor(\.start_date)]

// 기본값은 흔히 쓰이는 값으로 돼 있지만, 커스텀할 수 있다.
// 예를 들어 메모리 사용 증가를 감안하고 I/O 작업량을 줄이려면 배치 사이즈를 늘릴 수 있다.
// 무거운(용량이 큰) 객체 그래프라면 배치 사이즈를 줄일 수도 있다 당연히.
context.enumerate(
    descriptor,
    batchSize: 10000
) { trip in
    // Remind me to make reservations for trip
}
```
- 기본적으로 데이터 변형에 대한 방지(guard)처리가 돼 있지만, 이것도 변경할 수 있다.
```Swift
let predicate = #Predicate<Trip> { trip in
    trip.bucketListItem.filter {
        $0.hasReservation == false
    }.count > 0
}

let descriptor = FetchDescriptor(predicate: predicate)
descriptor.sortBy = [SortDescriptor(\.start_date)]

// 규모가 큰 traverse에서 발생하는 성능 문제는 대개 enumeration중 Context에 갇힌 mutation 때문이다.
// allowEscapingMutations로 이 mutation이 의도된 것임을 enumerate 함수에 알릴 수 있다.
// 설정하지 않는다면, enumeration을 수행하는 ModelContext가 dirty 상태로 발견됐을 때
// enumerate 함수에서 예외를 발생시켜 이미 순회한 객체의 해제를 막는다. 
context.enumerate(
    descriptor,
    batchSize: 500,
    allowEscapingMutations: true
) { trip in
    // Remind me to make reservations for trip
}
```