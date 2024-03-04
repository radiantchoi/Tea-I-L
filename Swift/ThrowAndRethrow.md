## Throw vs Rethrow

throws, rethrows, @escaping 그리고 @nonescaping
함수의 반환 타입에 throws를 붙이면 예정된 반환 타입 뿐만 아니라 에러를 "반환"할 수 있게 된다.

``` swift
enum CustomError: Error {
    case runtimeError(String)
}

func addHello(to name: String) throws -> String {
    // 여기다가 정책적 검증 로직을 넣으면 되겠군
    guard !name.isEmpty else { 
        throw CustomError.runtimeError("공백은 허용되지 않습니다.") 
    }

    return "Hello, \(name)!"
}

func helloBuilder(
    name: String, 
    process: (String) throws -> (String)
) rethrows {
    let incompleteHello = try process(name)
    print(incompleteHello)
}
```

rethrows 키워드를 붙이는 데에는 두 가지 조건이 있다.
- 인수로 받는 클로저가 throws한다.
- 한편 함수 본체는 딱히 throws하지 않는다.
이 경우는 helloBuilder가 해당한다. 한편 helloBuilder가 오류를 던진다면 아예 throws로 표시한다.

``` swift
func helloBuilder(
    name: String, 
    process: (String) throws -> (String)
) throws {
    let incompleteHello = try process(name)

    if name == "AAA" {
        throw CustomError.runtimeError("올바른 이름을 입력하세요.")
    }

    print(incompleteHello)
}
```

그런데 위의 예시에서 throw하지 않게 바꾸면? 그런데 그래도 rethrows 말고 throws로 쓸 수 있다! 엥?

``` swift
func helloBuilder(
    name: String, 
    process: (String) throws -> (String)
) throws {
    let incompleteHello = try process(name)

    if name == "AAA" {
        print("올바른 이름을 입력하세요.")
    }

    print(incompleteHello)
}
```

[애플 문서](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/declarations/#Rethrowing-Functions-and-Methods)를 보면..
- 파라미터가 throws 함수여야 rethrows를 쓸 수 있다.
- do-catch를 쓰고, catch 안에서 throw하는 것까지는 된다.
- throwing method는 rethrowing method의 프로토콜 요구조건을 만족시킬 수 없다. 하지만, rethrowing method는 throwing method를 오버라이드할 수 있다. 

그러니까, rethrowing method는 throwing method의 부분집합으로 봐도 된다는 얘기다.
실제로 애플 함수 중에서도 rethrows를 쓰는 것이 있다! 바로 filter.

``` swift
func filter(_ isIncluded: (Self.Element) throws -> Bool) rethrows -> [Self.Element] { 
    // ... 
}

// 그래서 이게 왜 중요하냐?
var nums = [1, 2, 3, 4]

let goodNumbers = nums.filter { $0 != 4 }
let goodNumbersWithError = try nums.filter { num in
    if num == 4 { throw CustomError.runtimeError("Why 4?") }

    return num != 4
}
// rethrowing method를 throwing method처럼 쓸 수 있기 때문이다!
```

### Reference
https://stevenpcurtis.medium.com/throw-or-rethrow-that-error-31ce0912de26
