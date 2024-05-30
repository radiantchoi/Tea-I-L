# Swift Continuation
https://medium.com/@itsuki.enjoy/swift-continuation-and-convert-completion-handlers-callback-into-async-async-throws-3895c76980c1

- 오래된 라이브러리 중에는 아직도 completion handler를 통해 작성된 게 많다.
- `continuations`를 사용하면 이러한 함수를 `async` 혹은 `async throws` 함수로 바꿀 수 있다!
- 덤으로 이걸 잘못 쓰면 일어날 수 있는 에러도 알아보자.

## `withCheckedContinuation`
- 동기 코드와 비동기 코드의 가교가 되는 `CheckedContinuation`.
- 예시 코드로 함께 보자.
``` swift
// completion을 사용하는 비동기(시뮬레이션) 함수
func fetchResult(for number: Int, completion: @escaping (Int) -> Void) {
    sleep(1)
    completion(number)
}

// 간단한 뷰
import SwiftUI

struct ContinuationDemo: View {
    @State var result: Int? = nil
    
    var body: some View {
        VStack {
            if let result {
                Text("result: \(result)")
            } else {
                Text("Loading...")
            }
        }
        .foregroundStyle(.white)
        .padding()
        .background(
            RoundedRectangle(cornerRadius: 16)
                .fill(.black)
        )
        .task {
            fetchResult(for: 8) { result in
                DispatchQueue.main.async {
                    self.result = result
                }                    
            }
        }
    }
}
```
- 자 이제, `fetchResult` 함수를 `async` 함수로 바꿔서 사용해 보자.
``` swift
func fetchResultAsync(for number: Int) async -> Int {
    let result = await withCheckedContinuation { continuation in 
        fetchResult(for: number) { result in 
            continuation.resume(returning: result)                         
        }
    }
    
    return result
}

// 뷰의 task 부분에서는
.task {
    let result = await fetchResultAsync(for: 8)
    DispatchQueue.main.async {
        self.result = result
    }
}
```
- 이렇게만 봐서는 `async`로 바꾸는 게 뭐 그리 중요하냐? 느낄 수 있다.
- 하지만 이런 간단한 경우가 아니라 URL 요청을 핸들링한다면?
    - 실패해서 에러가 뜨는 경우
    - 성공했더라도 응답 코드가 200인지 체크해야 하고
    - 응답 코드가 200이더라도 원하는 데이터가 떨어졌는지 체크해야 하고...
    - 이 경우에는 `async` 함수를 사용함으로써 각각의 단계를 더 가독성 좋게 컨트롤할 수 있다.
- 정말 다행히도 `URLSession`에는 이미 `async` 버전 함수가 있지만, 그렇지 않은 것도 많으니까!

## `withCheckedThrowingContinuation`
- 에러를 던지고 싶다면!!
- 위의 예시와 거의 같은데, `resume(throwing:)`을 써서 에러를 던질 수 있다는 것이 다르다.
- 위의 함수를 다시 짜 보자.
``` swift
enum NumberError: Error {
    case notEven
}

func fetchResultAsync(for number: Int) async throws -> Int {
    let result = try await withCheckedThrowingContinuation { continuation in 
        fetchResult(for: number) { result in 
            if number % 2 != 0 {
                continuation.resume(throwing: NumberError.notEven)
            } else {
                continuation.resume(returning: result)
            }
        }
    }
    
    return result
}
```
- 이거 하면서 `Generic parameter ‘T’ could not be inferred` 에러가 뜬다면?
    - `result` 반환값의 타입을 명시해 주면 된다.
- 사용할 때는 `do-catch`를 사용하면 된다.
``` swift
import SwiftUI

struct ContinuationDemo: View {
    @State var result: Int? = nil
    @State var error: NumberError? = nil
    
    var body: some View {
        VStack {
            if let result {
                Text("result: \(result)")
            } else if let error {
                Text("error: \(error)")
            } else {
                Text("Loading...")
            }
        }
        .foregroundStyle(.white)
        .padding()
        .background(
            RoundedRectangle(cornerRadius: 16)
                .fill(.black)
        )
        .task {
            do {
                let result = try await fetchResultAsync(for: 7)
                DispatchQueue.main.async {
                    self.result = result
                }
            } catch let error {
                if let error = error as? NumberError {
                    DispatchQueue.main.async {
                        self.error = error
                    }
                }
            }
        }
    }
}
```

## TASK CONTINUATION MISUSE 에러를 조심해
- 만약 completion handler를 여러 번 쓰는 함수라면?
``` swift
func fetchResult(for number: Int, completion: @escaping (Int) -> Void) {
    sleep(2)
    for i in 0..<number {
        completion(i)
    }
}
```
- `swift tried to resume its continuation more than once` 에러가 나게 된다.
- 이를 방지하기 위해 `nillableContinuation`을 만들어 보자!!
    - 한 번 resume 동작을 수행한 후 nil로 만들어주는 타입이다.
``` swift
func fetchResultAsync(for number: Int) async throws -> Int {
    let result: Int = try await withCheckedThrowingContinuation { continuation in
        var nillableContinuation: CheckedContinuation<Int, any Error>? = continuation
                                                                 
        fetchResult(for: number) { result in
            if result % 2 != 0 {
                nillableContinuation?.resume(throwing: NumberError.notEven)
                nillableContinuation = nil
            } else {
                nillableContinuation?.resume(returning: result)
                nillableContinuation = nil
            }
        }
    }
    
    return result
}
```
- `nillableContinuation` 프로퍼티를 정의할 때 타입이 뭔지 모르겠다면 쓸 수 있는 트릭!
    - nillableContinuation 정의할 때 nil을 꽂으면 에러를 통해 타입이 나온다!!