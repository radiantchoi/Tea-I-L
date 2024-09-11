# Swift 6 핥아보기

## Swift 6까지의 여정
- 점진적인 변화를 천명했다.
- 최종 목표는 동시성 지원과 메모리 소유권 모델의 개선.
1. Swift 5.1: Opaque Type(`some View`), 암시적 `return`(`return` 표현의 생략)
2.  Swift 5.2: 함수로서의 Key Path(`user.map(\.id)`), 코드 분석 개선
3.  Swift 5.3: 다중 패턴 에러 캐치, 다중 후행 클로저, SPM 개선
4.  Swift 5.4: Multiple variadic parameter(다중 다수 파라미터..?), Result Builder
5.  Swift 5.5: `async/await`, `actor`, `Sendable`, `CGFloat`와 `Double`의 혼용 등 다수의 업데이트
6.  Swift 5.6: Type Placeholder, Existential `any`, SPM 플러그인 추가
7.  Swift 5.7: 옵셔널 언래핑 축약 표현, Swift RegEx
8.  Swift 5.8: Result Builder 내부의 변수에 대한 제한 완화, 암시적 `self` 강화, `backDeployed` 도입
9.  Swift 5.9: Value and Type parameter Pack, `if`와 `switch` 직접 변수에 할당, `DiscardingTaskGroup`
10.  Swift 5.10: 데이터 레이스 의심되는 구멍 막기 - 글로벌 변수에 대한 엄격한 동시성 적용, 독립된 기본값
- 간략한 예시 코드로 보자면..!
``` swift
import Foundation
import SwiftUI

// Task 즉 async/await 컨텍스트에서 조정 가능한 글로벌 변수 (5.10)
nonisolated(unsafe) var globalNumber = 1

struct Person {
    let id: String
    let name: String
    let age: Int
    let birthDate: String
    
    // 다중 다수 파라미터 (5.4)
    static func makeList(
        names: String...,
        ages: Int...,
        birthDates: String...
    ) -> [Person] {
        guard names.count == ages.count && 
              names.count == birthDates.count else {
            return [] 
        }
        
        var result: [Person] = []
        
        for (index, name) in names.enumerated() {
            result.append(
                Person(
                    id: UUID().uuidString,
                    name: name,
                    age: ages[index],
                    birthDates: birthDates[index]
                )
            )
        }
        
        return result
    }
    
    // Value and Type parameter Pack (5.9)
    static func makePack<each String, each Int, each String>(
        name: repeat each String,
        age: repeat each Int,
        birthDate: repeat each String
    ) -> (repeat Person) {
        return (
            repeat Person(
                id: UUID().uuidString, 
                name: each name,
                age: each age,
                birthDate: each birthDate
            )
        )
    }
}

// SwiftUI View는 @ViewBuilder로 이루어지며, 
// 이는 내부적으로 @ResultBuilder를 사용한다. (5.4)
struct ContentView: View {
    @State private var people: [Person] = []
    // Type Placeholder (5.6)
    @State private var username: _? = ""
    
    private var identifiers: [String] {
        // 함수로서의 Key Path (5.2)
        people.map(\.id)
    }

    // Opaque Type (5.1)
    var body: some View {
        // 다중 후행 클로저 (5.3)
        Button {
            fetchName()
            people = makeExampleList()
        } label: {
            Text("Get username data")
        }
        
        List(people, id: \.self) { identifier in
            Text(identifier)
        }
        
        Button {
            printName()
        } label: {
            Text("Who is it?")
        }
    }
    
    func fetchName() {
        // 다중 에러 캐치 (5.3)
        // async/await 및 Task (5.5)
        Task {
            do {
                try await randomlyPopName()
            } catch DomainError.recoverable(let message),
                    DomainError.small(let message) {
                print(message)
                username = "Under maintenance"
            } catch DomainError.surprise(let message) {
                print(message)
                username = nil
            }
        }
    }
    
    func randomlyPopName() async throws -> String {
        let randomNumber = Int.random(in: 1...10)
        let result = randomNumber % 2 == 0
        
        if result {
            return "Gordon"
        } else if result % 3 == 0 {
            throw DomainError.recoverable("Good for you.")
        } else if result < 4 {
            throw DomainError.small("Too small.")
        } else {
            throw DomainError.surprise("Peekaboo!")
        }
    }
    
    // 암시적 return (5.1)
    func makeExampleList() -> [Person] {
        Person.makeList(
            names: "Gordon", "Greg",
            ages: 30, 28,
            birthDates: "930913", "960220"
        )
    }
    
    func printName() {
        // 옵셔널 언래핑 축약 (5.7)
        // if/switch의 변수 할당 (5.9)
        let displayingName = if let username {
            "Mr./Mrs. \(username)"
        } else {
            "Mr. Guy Folks"
        }
        
        print(displayingName)
    }
    
    // 이 코드 상에서는 불러오는 곳이 없는 함수
    // 오로지 하술한 Swift Regex를 보여드리기 위해,,
    func printBirthDate(for person: Person) {
        guard let birthDate = checkBirthDate(person.birthDate) else {
            print("Dunno")
            return
        }
        
        print(birthDate)
    }
    
    private func checkBirthDate(_ birthDate: String) -> String? {
        birthDate.containsMonth ? birthDate : nil
    }
}

extension String {
    var isSixDigitNumber: Bool {
        // Swift Regex (5.7)
        let digits = /[0-9]{6}/
        return self.contains(digits) && self.count == 6 ? true : false
    }
    
    var containsMonth: Bool {
        guard self.isSixDigitNumber else {
            return false
        }
        
        let startingIndex = self.index(self.startIndex, offsetBy: 4)
        let endingIndex = self.index(self.startIndex, offsetBy: 5)
        let month = String(self[startingIndex...endingIndex])
        
        if ("01"..."12") ~= month {
            return true
        } else {
            return false
        }
    }
}

enum DomainError: Error {
    case recoverable(message: String)
    case small(message: String)
    case surprise(message: String)
}
```
- 그리고 이를 기반으로 Swift 6가 등장하게 된다..!!

## Swift 6의 기능

### 호출부에서의 expression 매크로
- 함수 파라미터로 이제 유저가 커스텀하게 만든 매크로를 넣을 수 있다.
``` swift
@freestanding(expression)
macro MyFileID<T: ExpressibleByStringLiteral>() -> T = ...

public func callSiteFile(_ file: String = #MyFileID) { file }

public func declarationSiteFile(_ file: String = (#MyFileID)) { file }

public func alsoDeclarationSiteFile(
    file: String = callSiteFile(#MyFileID)
) { file }

print(callSiteFile()) // print main.swift, the current file
print(declarationSiteFile()) // always prints MyLibrary.swift
print(alsoDeclarationSiteFile()) // always prints MyLibrary.swift
```

### Pack 순회
- 5.9에서 등장한 Pack을 순회할 수 있게 된다.
``` swift
func == <each Element: Equatable>(
    lhs: (repeat each Element),
    rhs: (repeat each Element)
) -> Bool {
  for (left, right) in repeat (each lhs, each rhs) {
    guard left == right else { return false }
  }
  
  return true
}
```
### `Equatable`, `Hashable`, `Comparable`한 튜플
- 튜플은 그간 프로토콜을 채택할 수 없어서, 많은 개발자가 차라리 `struct`를 써 왔다.
- 튜플은 이제 위의 세 가지 프로토콜에 부합한다. 물론! 튜플의 원소가 위 세 가지 프로토콜에 부합한다면.


### 타입이 있는 throw
- Swift의 에러는 언제나 `any Error`로 타입이 지워졌다.
- 이 때문에 에러 핸들링 코드는 범용성 있게 쓸 수 밖에 없었다.
- 이제 함수나 클로저가 정해진 에러 타입을 던질 수 있다.
``` swift
struct ContentView: View {
    // ...
    
    func randomlyPopName() async throws(DomainError) -> String {
        // ...
    }
}
```

### `count(where:)`
- 이제 `count` 메서드에 조건에 맞는 것만 세 주는 기능이 추가된다.
``` swift
// before
[1, 2, 3, 4, 5].filter { $0 % 2 == 0 }.count // 2

// after
[1, 2, 3, 4, 5].count(where: { $0 % 2 == 0}) // 2
```

### 128비트 정수 타입
- `Int128` 타입과 `UInt128` 타입이 추가된다.
- 당연히 더 큰 수를 다룰 수 있고, 암호화에도 도움이 된다.

## 결론
- SwiftUI, Swift Regex, SwiftData, RegexBuilder, Algorithms..
- Swift 업그레이드를 따라가면서 얻을 수 있는 장점은 이렇게 많다.
    - 특히 옵젝씨에서 갈아엎으려면 지금이 기회다.
- Swift 개발 지금바로 시작해

## 참조
* [Swift 6 New Features](https://blog.swiftify.com/whats-new-in-swift-6-e875ca675a28)