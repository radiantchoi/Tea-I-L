# Model your schema with SwiftData
https://developer.apple.com/wwdc23/10195
- SampleTrips 예제 프로젝트를 사용한다.
- 언제나처럼, SwiftData를 추가하는 것은 간단하다.
```Swift
import SwiftUI
import SwiftData

@Model
final class Trip {
    var name: String
    var destination: String
    var start_date: Date
    var end_date: Date
    
    var bucketList: [BucketListItem]? = []
    var livingAccommodation: LivingAccommodation?
} 
```
- 사실 SwiftData의 장점은 별도의 외부 파일 형식 없이 코드의 추가만으로 데이터베이스를 연동할 수 있다는 것.
    - (물론 내부적으로 SQLite인 건 다 알고 있다)
- `@Model` 매크로는 Trip 클래스를 PersistentModel과 일치시키고 기술적 스키마를 생성한다.
  - Persistent는 대충 영구적이라는 뜻.

## 스키마 매크로 사용하기

### `@Attribute`
- 위처럼 정의하면 각각의 여행에 이름을 딱히 부여하지 않았으므로 같은 이름의 여행끼리는 충돌한다.
```Swift
@Model 
final class Trip {
    @Attribute(.unique) var name: String
    var destination: String
    var start_date: Date
    var end_date: Date
    
    var bucketList: [BucketListItem]? = []
    var livingAccommodation: LivingAccommodation?
}
```
- 이렇게 하면 이름이 유일성을 띄게 되고, 이름이 중복되는 데이터가 들어오면 업데이트되는 식으로 동작한다.
  - Upsert라고 한다. 기존 데이터와 충돌하는 Insert는 Update로 바뀐다는 것.
- `.unique` 제한은 기본 타입에 대해 적용된다. 숫자, 문자, UUID, ...
- 한편 스키마 이름을 바꾸고 싶은데, db 안에서는 유지했으면 좋겠다면?
```Swift
@Model 
final class Trip {
    @Attribute(.unique) var name: String
    var destination: String
    @Attribute(originalName: "start_date") var startDate: Date
    @Attribute(originalName: "end_date") var endDate: Date
    
    var bucketList: [BucketListItem]? = []
    var livingAccommodation: LivingAccommodation?
}
```
- 데이터 손실을 막는 효과도 있다. 스키마 이름을 무턱대고 바꾸면, 새로운 프로퍼티가 생긴 것으로 인식하기 떄문.
  - 새로운 프로퍼티만 있고, 있어야 할 프로퍼티가 없다면, 불러올 수 없는 데이터가 되겠죠..
- 이러한 특성 덕분에 이 애트리뷰트는 간단한 마이그레이션을 돕기도 한다.
- `@Attribute` 매크로는 이 외에도 대용량 데이터를 외부에 저장하거나, 트랜스폼 가능성을 지원하거나, 할 수 있다.

### `@Relationship`
- 위 모델에서 새 버킷 리스트나 숙소를 여행에 추가하면, SwiftData가 암시적으로 모델 간의 역관계(Inverse)를 발견하고 대신 설정해 준다.
- 기본 삭제 규칙을 적용해, 여행이 삭제될 때 버킷 리스트와 숙소를 null로 만든다.
- 그런데 그냥 같이 삭제됐으면 좋겠는데..
```Swift 
@Model 
final class Trip {
    @Attribute(.unique) var name: String
    var destination: String
    @Attribute(originalName: "start_date") var startDate: Date
    @Attribute(originalName: "end_date") var endDate: Date
    
    @Relationship(.cascade)
    var bucketList: [BucketListItem]? = []
  
    @Relationship(.cascade)
    var livingAccommodation: LivingAccommodation?
}
```
- Cascade 규칙을 적용함으로써 같이 삭제되게 했다!
  - cascade 규칙은 외래 키에 관해 적용되는 규칙이다.
  - 기본 동작상, 부모 테이블에 있는 튜플이 삭제되면 해당 튜플을 참조하는 자식 엔티티의 튜플에서 해당 키값을 null로 만든다.
    - 이 경우 고아 레코드가 발생할 우려가 있다.
  - 하지만 cascade 옵션을 적용하면 해당 튜플을 참조하는 자식 엔티티의 튜플도 삭제된다!
    - 참조 무결성을 유지하는 것을 돕지만, 예기치 못한 데이터의 삭제가 있을 수 있으므로 주의해야 한다.
- `@Relationship` 매크로의 기능은 다양하다.
  - originalName 수정자를 포함한다던지
  - to-many 관계에서 `toMany` 제한을 활용해 최소 최대 갯수를 제한할 수도 있다.

### `@Transient`
- 여행 자료를 몇 번 참조했는지 알고 싶지만, DB에는 넣지 않고 싶을 경우!
```Swift
@Model 
final class Trip {
    @Attribute(.unique) var name: String
    var destination: String
    @Attribute(originalName: "start_date") var startDate: Date
    @Attribute(originalName: "end_date") var endDate: Date
    
    @Relationship(.cascade)
    var bucketList: [BucketListItem]? = []
  
    @Relationship(.cascade)
    var livingAccommodation: LivingAccommodation?

    @Transient
    var tripViews: Int = 0
}
```
- `@Transient` 매크로를 붙이면 값을 지속하지(Persist) 않을 수 있다.
- 단 기본값을 제공하는 것을 잊지 말자. 그래야 SwiftData에서 가져올 때 논리적이 값이 할당된다.

## 스키마 개선하기
- 출시할 때마다 앱이 업데이트를 해야 한다.
- 프로퍼티 추가나 제거 등의 스키마 변경이 일어나면, 마이그레인이 필요하다!
- 이를 위한 SwiftData의 두 가지 도구, `VersionedSchema`와 `SchemaMigrationPlan`!

### `VersionedSchema`
- 모델이 변경된 버전을 출시할 때마다 `VersionedSchema`를 정의함으로써 이전에 출시된 스키마를 캡슐화한다.
- 각각의 스키마 버전이 `VersionedSchema`로 정의되어 있어야, SwiftData가 버전 사이의 변활르 체크할 수 있다.
- `VersionedSchema`의 total oprdering을 사용해 `SchemaMigrationPlan`을 생성하면, 필요한 마이그레이션을 자동으로 수행한다.
- 두 가지 유형의 마이그레이션 스테이지가 제공된다.
  - 경량 마이그레이션은 다음 출시에 마이그레이션을 할 때 추가 코드가 필요하지는 않다.
    - 기존 프로퍼티에 `originalName`을 추가하거나 관계에서 삭제 규칙을 지정하는 것 등은 경량 마이그레이션에 적합하다.
  - 사용자 지정 마이그레이션 스테이지를 사용해야 할 때도 있다!
    - 여행의 이름을 고유 키값으로 두고 싶은데, 이는 중복 데이터 처리 등의 과정을 수반한다.
```Swift
// 각각의 모델을 VersionedSchema로 캡슐화
enum SampleTripsSchemaV1: VersionedSchema {
    static var models: [any PersistentModel.Type] {
        [Trip.self, BucketListItem.self, LivingAccommodation.self]
    }

    @Model
    final class Trip {
        var name: String
        var destination: String
        var start_date: Date
        var end_date: Date
    
        var bucketList: [BucketListItem]? = []
        var livingAccommodation: LivingAccommodation?
    }

    // Define the other models in this version...
}

enum SampleTripsSchemaV2: VersionedSchema {
    static var models: [any PersistentModel.Type] {
        [Trip.self, BucketListItem.self, LivingAccommodation.self]
    }

    @Model
    final class Trip {
        @Attribute(.unique) var name: String
        var destination: String
        var start_date: Date
        var end_date: Date
    
        var bucketList: [BucketListItem]? = []
        var livingAccommodation: LivingAccommodation?
    }

    // Define the other models in this version...
}

enum SampleTripsSchemaV3: VersionedSchema {
    static var models: [any PersistentModel.Type] {
        [Trip.self, BucketListItem.self, LivingAccommodation.self]
    }

    @Model
    final class Trip {
        @Attribute(.unique) var name: String
        var destination: String
        @Attribute(originalName: "start_date") var startDate: Date
        @Attribute(originalName: "end_date") var endDate: Date
    
        var bucketList: [BucketListItem]? = []
        var livingAccommodation: LivingAccommodation?
    }

    // Define the other models in this version...
}

// 마이그레이션 플랜 사용
enum SampleTripsMigrationPlan: SchemaMigrationPlan {
    static var schemas: [any VersionedSchema.Type] {
        [SampleTripsSchemaV1.self, SampleTripsSchemaV2.self, SampleTripsSchemaV3.self]
    }
    
    static var stages: [MigrationStage] {
        [migrateV1toV2, migrateV2toV3]
    }

    static let migrateV1toV2 = MigrationStage.custom(
        fromVersion: SampleTripsSchemaV1.self,
        toVersion: SampleTripsSchemaV2.self,
        willMigrate: { context in
            let trips = try? context.fetch(FetchDescriptor<SampleTripsSchemaV1.Trip>())
                      
            // De-duplicate Trip instances here...
                      
            try? context.save() 
        }, didMigrate: nil
    )
  
    static let migrateV2toV3 = MigrationStage.lightweight(
        fromVersion: SampleTripsSchemaV2.self,
        toVersion: SampleTripsSchemaV3.self
    )
}

// 앱 환경에 설정
struct TripsApp: App {
    let container = ModelContainer(
        for: Trip.self, 
        migrationPlan: SampleTripsMigrationPlan.self
    )
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(container)
    }
}
```
- 무엇이 경량 마이그레이션이고 무엇이 아닌지, 구분하는 것이 중요해 보인다.
- 데이터에 영향을 미치냐 아니냐가 기준이 될 수 있을 듯 하다.