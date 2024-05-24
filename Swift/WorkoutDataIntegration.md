# Getting data from HealthKit (and CoreMotion)
- HealthKit에서 데이터를 받아오는 방법!
- HealthKit을 쓰지 않고 만보기를 구현하는 방법(ft. CoreMotion)!

## Aim
- 만보기 기능에 사용할 데이터 가져오기
    - 거리, 시간, 페이스, 칼로리, 걸음 수
- 덤으로, 심박 수를 받아와서 검색해주는 기능 구현을 위해서도 조금 공부해 두기

## Pre-Code Setting

### Apple Developer Console
- Certificates, Identifiers & Profiles 탭에서 사용하고자 하는 앱의 ID를 선택한다.
- 여러 가지 권한들 중 HealthKit 체크박스에 체크하고 Save함으로써 활성화시킨다.
    - 만약 Tuist(3.x) 등을 활용해 수동으로 인증서 관리를 하고 있다면, Provisioning Profile을 다시 발급받아야 한다.

### Xcode
- 프로젝트 세팅에서, 해당 데이터를 이용하고자 하는 타겟(위젯, 앱)을 선택한다.
- Signing & Capability 탭에서 좌상단의 + 버튼을 누른 다음 HealthKit을 추가한다.
- 다음으로는 Info.plist에 권한 요청 팝업에 들어갈 안내를 추가해야 한다.
    - HealthKit
        - Privacy - Health Share Usage Description (NSHealthShareUsageDescription)
        - Privacy - Health Update Usage Description (NSHealthShareUpdateUsageDescription)
    - CoreMotion
        - Privacy - Motion Usage Description(NSMotionUsageDescription)

## Wait, What is the difference?
- HealthKit은 사용자의 건강 및 피트니스 데이터에 접근하는 것 자체를 중점으로 둔다.
    - 하술한 번외 메뉴에서 볼 수 있듯 매우 다양한 데이터를 제공한다.
    - 1년 전의 데이터도 접근 가능할 정도로 데이터를 오래 보관한다.
    - 그런데 실시간으로 동작시키려면, 지속적으로 쿼리문을 보내기 따위의 방법을 써야 한다.
- CoreMotion은 iOS 장치의 하드웨어에서 일어나는 모션과 환경 데이터를 관리한다.
    - 운동.. 보다는 모션에 초점이 맞추어져 있다. 보행은 "모션"의 하나로 관리되는 느낌.
    - 7일간의 데이터 정도만 보관한다.
    - 대신, Serial DispatchQueue를 활용해 데이터를 실시간으로 가져오기 용이하다.
- 구현 목표를 고려했을 때
    - CoreMotion 기능을 위주로 실시간 업데이트가 필요한 만보기 기능을 구현
        - 걸음 수, 거리
    - HealthKit 기능과 CoreMotion을 활용해 대시보드 기능 구현
        - 걸음 수, 운동 거리, 운동 시간, 소모 칼로리
        - 페이스는 CoreMotion에서만 제공한다.

## Coding
- 먼저 초기 세팅을 해 보자.
``` swift
import CoreMotion
import Foundation
import HealthKit

final class WorkoutManager {
    static let shared = WorkoutManager()
    
    // 걷기와 달리기까지도 커버해야 할 때 필요하다
    private let motionActivityManager = CMMotionActivityManager()
    // 걷기는 만보기 객체를 쓸 수 있다
    private let pedometer = CMPedometer()
    // HealthKit 데이터는 여기로 얻어와야만 한다
    private let healthStore = HKHealthStore()
    
    /// HealthKit을 활용해 얻고자 하는 데이터
    /// 거리, 시간, 칼로리, 걸음수 포함
    /// 페이스는 CoreMotion을 통해 계산
    private let desiredHealthData = Set([
        HKQuantityType(.distanceWalkingRunning),
        HKQuantityType(.appleExerciseTime),
        HKQuantityType(.activeEnergyBurned),
        HKQuantityType(.stepCount),
    ])
    
    // ...
}
```
- `desiredHealthData`의 경우, `HKObjectType.quantityType(forIdentifier:)` 메서드로 되어 있는 예제가 많다.
- https://developer.apple.com/documentation/healthkit/hkobjecttype/1615298-quantitytype
- 그러나 기한이 명확하지 않지만, 먼 미래에 deprecated될 거라고 겁을 준다. (내부적으로는 10만 버전 후 정도..)
    - 그래서 대체재로 HKQuantityType을 사용할 수 있다.
    - https://developer.apple.com/documentation/healthkit/hkquantitytype/3778608-init
    - 반환값이 옵셔널도 아니라서 훨씬 좋음
- 다음으로는 각각의 데이터에 대해 사용자로부터 권한을 승인받아야 한다.
``` swift
extension WorkoutManager {
    func checkCoreLocationAuthStatus() {
        // CoreMotion의 경우 앱 단에서 권한을 처리하며, 따로 모든 것에 대해 권한을 받을 필요는 없다
        if CMMotionActivityManager.authorizationStatus() != .authorized {
            VividLogger.apiLog(.critical, "Core Motion is not authorized")
        } else if !CMMotionActivityManager.isActivityAvailable() {
            VividLogger.apiLog(.warning, "Activity tracking is not available")
        } else {
            VividLogger.apiLog(.info, "Core Motion available")
        }
    }
    
    func checkHealthKitAuthStatus() {
        if !unauthorizedHealthData.isEmpty {
            requestHealthKitAuth(for: unauthorizedHealthData)
        }
    }
    
    func requestHealthKitAuth(for desired: Set<HKQuantityType>) {
        healthStore.requestAuthorization(toShare: nil, read: desired) { succeeded, error in
            guard error == nil else {
                VividLogger.apiLog(.critical, "HealthKit auth error occured")
                return
            }
            
            if succeeded {
                VividLogger.apiLog(.info, "HealthKit permission granted")
            } else {
                VividLogger.apiLog(.critical, "HealthKit permission denied")
            }
        }
    }
}
```
- Singleton 객체기 때문에, 앱 실행시 객체가 로드되고 각각의 권한에 대한 검증을 수행한다.
- 만약 수락이 되어 있지 않다면?
    - HealthKit은 modal을 띄워서 각각에 대한 권한을 수령한다.
    - CoreMotion은 alert를 띄워서, 모션 인식 전체에 대한 권한을 수령한다.
- 이후 각각의 함수에 대해 요청을 수행하는 로직을 작성한다.
- 완성된 전체 코드는 다음과 같다.
``` swift
extension AppDelegate {
    // 이 메서드를 AppDelegate didFinishLaunchingWithOptions에서 불러와서 체크한다.
    private func singletonChecker() {
        let starting = Date().days(before: 7) ?? Date()
        
        WorkoutManager.shared.requestDistance(startingDate: starting, period: .week) {
            VividLogger.apiLog(.info, $0)
        }
        
        WorkoutManager.shared.requestTime(startingDate: starting, period: .week) {
            VividLogger.apiLog(.info, $0)
        }
        
        WorkoutManager.shared.requestEnergyBurnt(startingDate: starting, period: .week) {
            VividLogger.apiLog(.info, $0)
        }
        
        WorkoutManager.shared.requestPace(startingDate: starting, period: .week) {
            VividLogger.apiLog(.info, $0)
        }
        
        WorkoutManager.shared.requestSteps(startingDate: starting, period: .week) {
            VividLogger.apiLog(.info, $0)
        }
    }
}

// WorkoutManager.swift
import CoreMotion
import Foundation
import HealthKit

final class WorkoutManager {
    enum Period {
        case day
        case week
        case month(days: Int)
        case year(isLeapYear: Bool)
        
        var toDays: Int {
            switch self {
            case .day:
                1
            case .week:
                7
            case .month(let days):
                days
            case .year(let isLeapYear):
                isLeapYear ? 366 : 365
            }
        }
        
        var toSeconds: Int {
            toDays * 86400
        }
    }
    
    @Published var steps = 0
    
    static let shared = WorkoutManager()
    
    private let motionActivityManager = CMMotionActivityManager()
    private let pedometer = CMPedometer()
    private let healthStore = HKHealthStore()
    
    /// HealthKit을 활용해 얻고자 하는 데이터입니다.
    /// 거리, 시간, 칼로리, 걸음수가 포함됩니다.
    /// 페이스는 CoreMotion을 통해 계산합니다.
    private let desiredHealthData = Set([
        HKQuantityType(.distanceWalkingRunning),
        HKQuantityType(.appleExerciseTime),
        HKQuantityType(.activeEnergyBurned),
        HKQuantityType(.stepCount),
    ])
    private var unauthorizedHealthData: Set<HKQuantityType> {
        desiredHealthData.filter {
            healthStore.authorizationStatus(for: $0) != .sharingAuthorized
        }
    }
    
    private init() {
        checkCoreLocationAuthStatus()
        checkHealthKitAuthStatus()
        
        setTimer()
    }
    
    func checkCoreLocationAuthStatus() {
        // CoreMotion의 경우 앱 단에서 권한을 처리하며, 따로 모든 것에 대해 권한을 받을 필요는 없습니다.
        // TODO: 에러 핸들링
        if CMMotionActivityManager.authorizationStatus() != .authorized {
            VividLogger.apiLog(.critical, "Core Motion is not authorized")
        } else if !CMMotionActivityManager.isActivityAvailable() {
            VividLogger.apiLog(.warning, "Activity tracking is not available")
        } else {
            VividLogger.apiLog(.info, "Core Motion available")
        }
    }
    
    func checkHealthKitAuthStatus() {
        if !unauthorizedHealthData.isEmpty {
            requestHealthKitAuth(for: unauthorizedHealthData)
        }
    }
    
    func requestHealthKitAuth(for desired: Set<HKQuantityType>) {
        healthStore.requestAuthorization(toShare: nil, read: desired) { succeeded, error in
            guard error == nil else {
                VividLogger.apiLog(.critical, "HealthKit auth error occured")
                return
            }
            
            if succeeded {
                // TODO: 에러 핸들링
                VividLogger.apiLog(.info, "HealthKit permission granted")
            } else {
                VividLogger.apiLog(.critical, "HealthKit permission denied")
            }
        }
    }
    
    func setTimer() {
        Timer.scheduledTimer(timeInterval: 3,
                             target: self,
                             selector: #selector(updateDailyCurrentSteps),
                             userInfo: nil,
                             repeats: true)
    }
    
    /// 주기적으로 현재 걸음 수를 업데이트하는 메서드입니다.
    @objc func updateDailyCurrentSteps() {
        let today = Date()
        guard let startDate = Calendar.current.date(bySettingHour: 0, minute: 0, second: 0, of: Date()) else {
            return
        }
        
        pedometer.queryPedometerData(from: startDate, to: today) { data, error in
            guard error == nil else {
                VividLogger.apiLog(.critical, error)
                return
            }
            
            if let steps = data?.numberOfSteps {
                DispatchQueue.main.async { [weak self] in
                    // TODO: 나중에는 completion으로 빼서 다른 객체에 적용시키든, 스텝을 구독시키든 해야 할 듯
                    self?.steps = Int(truncating: steps)
                }
            }
        }
    }
    
    /// 정해진 날짜로부터 정해진 기간 동안의 운동 거리를 구하는 메서드입니다.
    func requestDistance(startingDate: Date, period: Period, completion: @escaping (Double?) -> Void) {
        let distanceType = HKQuantityType(.distanceWalkingRunning)
        
        executeHealthKitQuery(
            quantityType: distanceType,
            startingDate: startingDate,
            period: period,
            unit: .meter(),
            completion: completion
        )
    }
    
    // TODO: 검증 필요
    /// 정해진 날짜로부터 정해진 기간 동안의 운동 시간을 구하는 메서드입니다.
    func requestTime(startingDate: Date, period: Period, completion: @escaping (Double?) -> Void) {
        let timeType = HKQuantityType(.appleExerciseTime)
        
        executeHealthKitQuery(
            quantityType: timeType,
            startingDate: startingDate,
            period: period,
            unit: .second(),
            completion: completion
        )
    }
    
    /// 정해진 날짜부터 정해진 기간 동안의 운동 페이스를 구하는 메서드입니다.
    func requestPace(startingDate: Date, period: Period, completion: @escaping (Double?) -> Void) {
        let expectedDate = startingDate.days(after: period.toDays)
        let endingDate = min(expectedDate ?? Date(), Date())
        
        pedometer.queryPedometerData(from: startingDate, to: endingDate) { data, error in
            guard error == nil else {
                VividLogger.apiLog(.critical, error)
                completion(nil)
                return
            }
            
            if let pace = data?.averageActivePace {
                DispatchQueue.main.async {
                    completion(Double(truncating: pace) / Double(period.toDays))
                }
            }
        }
    }
    
    /// 정해진 날짜로부터 정해진 기간 동안의 칼로리 소모량을 구하는 메서드입니다.
    func requestEnergyBurnt(startingDate: Date, period: Period, completion: @escaping (Double?) -> Void) {
        let calorieType = HKQuantityType(.activeEnergyBurned)
        
        executeHealthKitQuery(
            quantityType: calorieType,
            startingDate: startingDate,
            period: period,
            unit: .kilocalorie(),
            completion: completion
        )
    }
    
    /// 정해진 날짜로부터 정해진 기간 동안의 걸음 수를 구하는 메서드입니다.
    func requestSteps(startingDate: Date, period: Period, completion: @escaping (Double?) -> Void) {
        let stepsType = HKQuantityType(.stepCount)
        
        executeHealthKitQuery(
            quantityType: stepsType,
            startingDate: startingDate,
            period: period,
            unit: .count(),
            completion: completion
        )
    }
    
    /// 직접적으로 HealthKit 관련 계산을 수행하는 메서드입니다.
    /// 현재는 정해진 기간 동안 정해진 지표의 평균을 반환합니다.
    /// unit 파라미터의 경우 가져오고자 하는 지표마다 사용 단위가 다릅니다. HKUnit -> Jump to definiition을 통해 확인해주세요.
    private func executeHealthKitQuery(
        quantityType: HKQuantityType,
        startingDate: Date,
        period: Period,
        unit: HKUnit,
        completion: @escaping (Double?) -> Void) {
            let expectedDate = startingDate.days(after: period.toDays)
            let endingDate = min(expectedDate ?? Date(), Date())
            
            let predicate = HKQuery.predicateForSamples(withStart: startingDate, end: endingDate, options: .strictStartDate)
            let query = HKStatisticsQuery(
                quantityType: quantityType,
                quantitySamplePredicate: predicate,
                options: .cumulativeSum) { _, result, error in
                    guard let result,
                          let sum = result.sumQuantity() else {
                        VividLogger.apiLog(.warning, "No sum data")
                        completion(nil)
                        return
                    }
                    
                    DispatchQueue.main.async {
                        completion(sum.doubleValue(for: unit) / Double(period.toDays))
                    }
                }
            
            healthStore.execute(query)
    }
}

```
- 전반적으로, HealthKit/CoreMotion이라는 이름의 데이터베이스에서 원하는 조건의 데이터를 가져오는 것에 가까운 동작이 이루어진다.

## 번외. HealthKit을 통해 얻어올 수 있는 데이터의 종류 (feat. ChatGPT)
https://developer.apple.com/documentation/healthkit

### 1. Body Measurements (신체 측정 데이터)
- 체중 (Body Mass)
- 신장 (Height)
- 체질량지수 (Body Mass Index)
- 체지방율 (Body Fat Percentage)

### 2. Fitness (피트니스 데이터)
- 걸음 수 (Step Count)
- 이동 거리 (Distance Walking/Running)
- 소모 칼로리 (Active Energy Burned)
- 운동 시간 (Exercise Time)
- 계단 오르기 (Flights Climbed)

### 3. Vital Signs (생명 징후)
- 심박수 (Heart Rate)
- 혈압 (Blood Pressure)
- 산소포화도 (Oxygen Saturation)
- 호흡수 (Respiratory Rate)
- 체온 (Body Temperature)

### 4. Nutrition (영양 데이터)
- 칼로리 섭취 (Dietary Energy Consumed)
- 탄수화물 (Carbohydrates)
- 지방 (Fat Total)
- 단백질 (Protein)
- 물 섭취량 (Water)

### 5. Sleep (수면 데이터)
- 수면 분석 (Sleep Analysis)

### 6. Reproductive Health (생식 건강)
- 생리 주기 (Menstrual Flow)
- 배란 검사 결과 (Ovulation Test Result)
- 자궁 경부 점액 품질 (Cervical Mucus Quality)
- 기초 체온 (Basal Body Temperature)
- 성교 (Sexual Activity)

### 7. Mindfulness (마음챙김)
- 마음챙김 세션 (Mindful Session)

### 8. Other (기타 데이터)
- 청각 건강 (Hearing Health)
- 메디컬 ID (Medical ID)
- 임상 기록 (Clinical Records)

## 번외. CoreMotion을 통해 얻어올 수 있는 데이터의 종류 (ft. ChatGPT)
https://developer.apple.com/documentation/coremotion

### 1. Pedometer Data (보행 데이터)
- Steps Count (걸음 수): 사용자가 걸은 총 걸음 수.
- Distance (거리): 사용자가 걸은 총 거리.
- Floors Ascended (오른 층 수): 사용자가 오른 층의 수.
- Floors Descended (내려간 층 수): 사용자가 내려간 층의 수.
- Cadence (보행 속도): 걸음의 속도.
- Pace (페이스): 분당 걸음 수.

### 2. Accelerometer Data (가속도 데이터)
- Acceleration (가속도): 장치의 x, y, z 축에 대한 가속도 값.

### 3. Gyroscope Data (자이로스코프 데이터)
- Rotation Rate (회전율): 장치의 x, y, z 축에 대한 회전율 값.

### 4. Magnetometer Data (자력계 데이터)
- Magnetic Field (자기장): 장치 주변의 x, y, z 축에 대한 자기장 값.

### 5. Device Motion Data (장치 모션 데이터)
- Attitude (자세): 장치의 롤, 피치, 요 값.
- Rotation Rate (회전율): 자이로스코프 데이터를 기반으로 한 회전율.
- Gravity (중력): 장치에 작용하는 중력 벡터.
- User Acceleration (사용자 가속도): 장치의 가속도에서 중력을 뺀 값.
- Magnetic Field (자기장): 장치 주변의 자기장 값.
