# Meet SwiftData
https://developer.apple.com/wwdc23/10187

## SwiftData?
- 데이터 모델링/관리 프레임워크
- SwiftUI처럼 코드에만 집중하면서도 Swift 매크로를 이용해 API를 만든다.
    - 이 부분에서는, CloudKit, Widget과도 역시 함께 잘 잘동한다.

## Using the Model Macro
- SwiftData 모델은 일반적인 Swift 코드와 같다.
- SwiftData를 사용하는 앱의 Single Source of Truth이다.
```Swift
import SwiftData

// 단지 이 애트리뷰트만 달아 준다면!
@Model
class Trip {
    var name: String
    var destination: String
    var endDate: Date
    var startDate: Date
 
    var bucketList: [BucketListItem]? = []
    var livingAccommodation: LivingAccommodation?
}
```
- 값 타입 프로퍼티로부터 자동으로 애트리뷰트를 추론해낸다.
  - 기본 타입, `enum`, `struct`, `Codable`, ...
- 참조형 타입으로부터 관계를 추론해 낸다.
  - 모델 타입과 모델 타입의 관게를 본다.
  - 컬렉션 사이에 링크를 추가할 수도 있다.
- 붙는 것에 따라 차이가 있다.
  - `@Model`: 타입에 대한 저장 프로퍼티를 모두 수정할 수 있다.
  - `@Attribute`: 유일 제약조건을 설정할 수 있다.
  - `@Relationship`: 인버스 선택을 통제하고 삭제 전파 규칙을 지정할 수 있다.
    - 이를 통해 모델 간 링크의 동작을 바꿀 수 있다.
  - `@Transient`: 특정 프로퍼티를 포함하지 않게 할 수 있다.
```Swift
@Model
class Trip {
    @Attribute(.unique) var name: String
    var destination: String
    var endDate: Date
    var startDate: Date
    
    // 이 앱에서 삭제된 모든 관련 버킷 리스트 항목을 SwiftData로 하여금 삭제하게 한다.
    @Relationship(.cascade) var bucketList: [BucketListItem]? = []
    var livingAccommodation: LivingAccommodation?
}
```

## Working with your Data
- `ModelContainer`, `ModelContext`

### `ModelContainer`
- 모델 타입에 백엔드를 제공한다.
- 커스텀 설정 구성이나 마이그레이션 옵션으로 커스텀할 수 있다.
- 저장하고 싶은 모델 타입을 지정하기만 하면 모델 컨테이너를 생성할 수 있다.
```Swift
// Initialize with only a schema
let container = try ModelContainer([Trip.self, LivingAccommodation.self])

// Initialize with configurations
let container = try ModelContainer(
    for: [Trip.self, LivingAccommodation.self],
    configurations: ModelConfiguration(url: URL("path"))
) 
```

### `ModelContext`
- 모델에 생기는 모든 변화를 관찰한다.
- 업데이트 관찰, 모델 가져오기, 모델 변경사항 저장, 변경사항 취소 등 여러 기능을 할 수 있다.
```Swift
import SwiftUI

@main
struct TripsApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(
            for: [Trip.self, LivingAccommodation.self]
        )
    }
}

struct ContextView : View {
    // 뷰에서는 이렇게 사용할 수 있다.
    @Environment(\.modelContext) private var context
}

struct MyContainer {
    // container 프로퍼티가 있다고 가정하고
    let context = container.mainContext
    let context = ModelContext(container)
}
```
- Context가 있다면 데이터를 가져올 준비가 된 것!
- `Predicate`, `FetchDescriptor`, `SortDescriptor`와 함께 써 보자!

### `Predicate`
- `NSPredicate`를 대체한다.
- 타입 체크가 완전히 이루어지고, 텍스트 파싱 대신 `#Predicate` 매크로를 쓴다.
- KeyPath와 함께 이용하는데, Xcode 자동완성과 함께 생산성의 증대도 노릴 수 있다.
```Swift
let today = Date()
// Predicate(서술자)를 이용해 데이터 조회에 조건을 걸 수 있다.
let tripPredicate = #Predicate<Trip> { 
    $0.destination == "New York" &&
    $0.name.contains("birthday") &&
    $0.startDate > today
}
```

### `FetchDescriptor` / `SortDescriptor`
- 가져오고자 하는 것이 무엇인지 결정했을 경우, 얘를 활용해서 DB에서 튜플들을 가져오자.
```Swift
// 상단의 Predicate를 그대로 사용할 경우
let descriptor = FetchDescriptor<Trip>(predicate: tripPredicate)

let trips = try context.fetch(descriptor) 

// 한편 자료를 정렬된 채로 가져오고 싶을 경우
let descriptor = FetchDescriptor<Trip>(
    sortBy: SortDescriptor(\Trip.name),
    predicate: tripPredicate
)

let trips = try context.fetch(descriptor)
```
- 관련 객체를 지정해 미리 가져왁너 미지정 변경 사항을 제외해 결과 카운트를 제외하는 등 다른 것도 할 수 있다.
- `ModelContext`를 활용해 이러한 CRUD를 쉽게 할 수 있다.
```Swift
var myTrip = Trip(name: "Birthday Trip", destination: "New York")

// Insert a new trip
context.insert(myTrip)

// Delete an existing trip
context.delete(myTrip)

// Manually save changes to the context
try context.save() 
```
- @Model 오퍼레이터와 함께라면, 평상시 프로퍼티를 바꾸듯 저장된 데이터 프로퍼티를 바꿀 수 있다.

##  Using SwiftData with SwiftUI
- 단언컨대 SwiftData를 시작하기 가장 쉬운 방법.
- 데이터 스토어 구성 옵션 변경, 실행 취소 자동저장 토글 등의 액션이 가능하다.
- `Environment`로 사용한다.
```Swift
import SwiftUI

struct ContentView: View  {
    @Query(sort: \.startDate, order: .reverse) var trips: [Trip]
    @Environment(\.modelContext) var modelContext
    
    var body: some View {
       NavigationStack() {
          List {
             ForEach(trips) { trip in 
                 // ...
             }
          }
       }
    }
}
```
- `Observable` 기능을 자동으로 (당연히) 지원한다.

## Where to go from here?
- https://developer.apple.com/wwdc23/10195 (스키마 만들기)
- https://developer.apple.com/wwdc23/10196 (더 뜯어보기)
- https://developer.apple.com/wwdc23/10189 (Core Data에서 마이그레이션하기)

## (DB가 처음인 사람을 위한) 용어집

### Model
- 데이터베이스에서 테이블의 개념과 유사하다.
- 속성(프로퍼티/컬럼)들을 가진 각각의 자료(인스턴스/튜플)를 표현한다.

### Attribute vs Transient
- Attribute는 데이터베이스 상의 튜플이 가지고 있는 프로퍼티이다.
  - 예를 들면 유저 ID, 이름, 이메일, ...
- Transient는 데이터베이스 컬럼과 매핑되지 않는 임시 프로퍼티이다.
  - 예를 들면 암호화된 비밀번호
  - 그래서 DB 조회 대상이 아니다. SwiftData에서는 "무시"한다고 표현했다.

### Predicate
- 관계형 데이터베이스에서 데이터를 검색하거나 필터링할 때 사용되는 조건식.
- 아래 두 문장은 같은 뜻이다.
- `destination = "New York" AND name LIKE '%birthday%'` 부분이 Predicate에 해당한다.
``` SQL
SELECT * FROM Trip WHERE destination = "New York" AND name LIKE '%birthday%'
```
```Swift
let tripPredicate = #Predicate<Trip> { 
    $0.destination == "New York" &&
    $0.name.contains("birthday")
}
```

### Query
- 상술한 것이 바로 SELECT를 활용한 쿼리. 어쨌든 SELECT해 오라고 "명령"한 것이다.
- 데이터베이스에 정보를 요청하는 것.
- SwiftData에서 가져온 타입에 `@Query` 애트리뷰트가 붙은 이유는, 자료의 변경을 동적으로 감지하고 실제 DB에 반영하기 때문.
  - 이는 DB에서 CRUD 쿼리를 작성하는 것과 비슷한다.