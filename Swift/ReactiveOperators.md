# Intro
- RxSwift, Combine. 둘 다 iOS 개발 과정에서 사용되는 반응형 프레임워크!
- 자주 쓰이는 오퍼레이터들은 어떻게 다른 표현법을 가지고 있는지 알아본다.
- 오늘 도움을 줄 배열들.
```Swift
var firstNumbers = [1, 2, 3, 4]
var secondNumbers = [5, 6, 7, 8]
var firstLetters = ["a", "b", "c", "d"]
var secondLetters = ["e", "f", "g", "h"]

let disposeBag = DisposeBag()
var cancellables = Set<AnyCancellable>()
```

# Combine Operators

## Merge
- 두 개의 스트림을 하나로 합한다.
  - 이 때, 두 스트림의 반환 타입은 같아야 한다.
```Swift
// RxSwift
let leading = Observable.from(firstNumbers)
let trailing = Observable.from(secondNumbers)

Observable.merge(leading, trailing)
    .subscribe(onNext: { print($0) }
    .dispose(in: disposeBag)

// Combine
let leading = firstNumbers.publisher
let trailing = secondNumbers.publisher

leading
    .merge(with: trailing)
    .sink { print($0) }
    .store(in: &cancellables)

// 결과값으로는 1 2 3 4 5 6 7 8 이 순서대로 프린트된다.
```

## zip
- 두 스트림에서 발신된 값을 튜플로 묶어서 발신한다.
- 이 때, 순서가 일치하는 신호만 발신한다.
- 한편 묶어 주는 두 스트림의 반환 타입은 같지 않아도 된다.
```Swift
// RxSwift
let leading = Observable.from(firstNumbers)
let trailing = Observable.from(firstLetters)

Observable.zip(leading, trailing)
    .subscribe(onNext: { print($0) }
    .dispose(in: disposeBag)

// Combine
let leading = firstNumbers.publisher
let trailing = firstLetters.publisher

leading
    .zip(trailing)
    .sink { print($0) }
    .store(in: &cancellables)

// 출력값은 모두 (1, "a") (2, "b") (3, "c") (4, "d")
```
- 물론 `map` 등의 오퍼레이터를 사용해서 형태를 변환할 수도 있다.

## CombineLatest
- `zip`과 유사한데, 꼭 순서가 일치할 필요는 없다.
- 더 최근에 발신된 스트림에 의거 값이 변동된다.
```Swift
// 위와 똑같은 이벤트 스트림을 쓴다고 했을 때
// RxSwift
Observable.combineLatest(leading, trailing)
    .subscribe(onNext: { print($0) }
    .dispose(in: disposeBag)
    
// Combine
leading
    .combineLatest(trailing)
    .sink { print($0) }
    .store(in: &cancellables)

// 출력은 (1, "a") (2, "a") (3, "a") (4, "a") (4, "b") (4, "c") (4, "d")
```

## 앞에 끼워넣기
- 스트림이 발신하는 값 앞에 특정한 값을 끼워넣는다.
```Swift
// RxSwift
let numbersObserver = Observer.from(firstNumbers)

numbersObserver
    .startWith(0)
    .subscribe(onNext: { print($0) }
    .dispose(in: disposeBag)

// Combine
let numbersPublisher = firstNumbers.publisher

numbersPublisher
    .prepend(0)
    .sink { print($0) }
    .store(in: &cancellables)

// 0 1 2 3 4
```
- 초기값/기본값 제공 등에 유용하게 쓸 수 있다.

## 순서대로 합치기
- `merge`처럼 스트림을 합치긴 합치는데, 순서가 있게 합친다.
```Swift
// RxSwift
let leading = Observable.from(firstNumbers)
let trailing = Observable.from(secondNumbers)

Observable.concat(leading, trailing)
    .subscribe(onNext: { print($0) }
    .dispose(in: disposeBag)

// Combine
let leading = firstNumbers.publisher
let trailing = secondNumbers.publisher

leading
    .append(trailing)
    .sink { print($0) }
    .store(in: &cancellables)

// 결과값으로는 1 2 3 4 5 6 7 8이 순서대로 프린트된다.
// merge와 비슷해 보일 수 있는데, 먼저 등장한 스트림이 "끝나고" 다음 스트림이 발신된다.
// 따라서 여러 스트림에서 실시간으로 값이 들어올 때 유용하다.
```
- 여담으로, Combine 프레임워크는 애플에서 배열처럼 다루고자 하는 경향이 있다.
  - 시간에 따른 이벤트의 나열이다, 라고 생각하는 것.
  - 그래서 `append`라는 오퍼레이터를 사용한 것인지도.

# Filtering Operators

## filter
- 흔히 아는 고차함수의 그 필터가 맞다.. 일정한 기준에 따라 발신값을 거른다.
- 전반적으로 시간이라는 요소가 개입된 배열이라고 생각하면 이해하기 쉽다.
```Swift
// RxSwift
let numbersObserver = Observer.from(firstNumbers)

numbersObserver
    .filter { $0 % 2 == 0 }
    .subscribe(onNext: { print($0) }
    .dispose(in: disposeBag)

// Combine
let numbersPublisher = firstNumbers.publisher

numbersPublisher
    .filter { $0 % 2 == 0 }
    .sink { print($0) }
    .store(in: &cancellables)
   
// 2 4
```

## 무시하기
- 발신되는 값들을 무시하고 스트림을 완료시켜버린다.
```Swift
// RxSwift
let numbersObserver = Observer.from(firstNumbers)

numbersObserver
    .ignoreElements()
    .subscribe(onNext: { print($0) }
    .dispose(in: disposeBag)

// Combine
let numbersPublisher = firstNumbers.publisher

numbersPublisher
    .ignoreOutput()
    .sink { print($0) }
    .store(in: &cancellables)

// 두 스트림 모두 값을 발신하지 않는다. 
```

## 앞의 몇 개 건너뛰기
- 말 그대로! 발신되는 값 중 앞의 몇 개를 건너뛴다.
```Swift
// RxSwift
let numbersObserver = Observer.from(firstNumbers)

numbersObserver
    .skip(2)
    .subscribe(onNext: { print($0) }
    .dispose(in: disposeBag)

// Combine
let numbersPublisher = firstNumbers.publisher

numbersPublisher
    .drop(2)
    .sink { print($0) }
    .store(in: &cancellables)

// 3 4
```

### 특정 조건이 만족되지 않을 때까지 건너뛰기
- 갯수를 정하는 게 아니고 조건을 정하는 것.
- 특정 조건을 만족하지 않는 이벤트가 나올 때까지 건너뛴다. 
```Swift
// RxSwift
let numbersObserver = Observer.from(firstNumbers)

numbersObserver
    .skipWhile { $0 % 2 == 1 }
    .subscribe(onNext: { print($0) }
    .dispose(in: disposeBag)

// Combine
let numbersPublisher = firstNumbers.publisher

numbersPublisher
    .drop(while: { $0 % 2 == 1 })
    .sink { print($0) }
    .store(in: &cancellables)

// 2 3 4
```
- Combine 상에서는 함수 심볼은 달라도 이름은 같기 때문에, 좀 더 일관성 있게 쓸 수 있는 것 같기도.

### 이벤트가 수신될 때까지 건너뛰기
- 다른 스트림으로부터 이벤트가 수신될 때까지 건너뛴다.
```Swift
// RxSwift
let numbersObserver = Observer.from(firstNumbers)
let triggerObserver = Observable<Void>.Just(())

numbersObserver
    .skipUntil(triggerObserver)
    .subscribe(onNext: { print($0) }
    .dispose(in: disposeBag)

// Combine
let numbersPublisher = firstNumbers.publisher
let triggerPublisher = Just(())

numbersPublisher
    .drop(untilOutputFrom: triggerPublisher)
    .sink { print($0) }
    .store(in: &cancellables)

// 1 이벤트가 지나가고 트리거를 걸었다는 임의의 전제 하에
// 2 3 4
```

## 앞의 몇 개만 받기
- 스킵/드랍 계열과 반대라고 생각하면 된다.
```Swift
// RxSwift
let numbersObserver = Observer.from(firstNumbers)

numbersObserver
    .take(2)
    .subscribe(onNext: { print($0) }
    .dispose(in: disposeBag)

// Combine
let numbersPublisher = firstNumbers.publisher

numbersPublisher
    .prefix(2)
    .sink { print($0) }
    .store(in: &cancellables)

// 1 2
```

### 특정 조건이 만족될 동안만 받기
```Swift
// RxSwift
let numbersObserver = Observer.from(firstNumbers)

numbersObserver
    .takeWhile { $0 % 2 == 1 }
    .subscribe(onNext: { print($0) }
    .dispose(in: disposeBag)

// Combine
let numbersPublisher = firstNumbers.publisher

numbersPublisher
    .prefix(while: { $0 % 2 == 1 })
    .sink { print($0) }
    .store(in: &cancellables)

// 1
```

### 이벤트가 수신되기 전까지만 받기
```Swift
// RxSwift
let numbersObserver = Observer.from(firstNumbers)
let triggerObserver = Observable<Void>.Just(())

numbersObserver
    .takeUntil(triggerObserver)
    .subscribe(onNext: { print($0) }
    .dispose(in: disposeBag)

// Combine
let numbersPublisher = firstNumbers.publisher
let triggerPublisher = Just(())

numbersPublisher
    .prefix(untilOutputFrom: triggerPublisher)
    .sink { print($0) }
    .store(in: &cancellables)

// 1 이벤트가 지나가고 트리거를 걸었다는 임의의 전제 하에
// 1
```

## 특정 인덱스의 값만 발신하기
- 이런 오퍼레이터가 있다는 건 역시 반응형 프로그래밍의 이벤트 스트림은 시간이 개입된 배열과 비슷하다.. 라는 것이겠다.

```Swift
// RxSwift
let numbersObserver = Observer.from(firstNumbers)

numbersObserver
    .elementAt(2)
    .subscribe(onNext: { print($0) }
    .dispose(in: disposeBag)

// Combine
let numbersPublisher = firstNumbers.publisher

numbersPublisher
    .output(at: 2)
    .sink { print($0) }
    .store(in: &cancellables)

// 3
```
- 묘한 차이가 있는데, RxSwift에서는 인덱스를 벗어나는 값을 요청받으면 에러를 발신하나, Combine에서는 빈 시퀀스를 반환한다.
  - 그래서 사실 상기 코드에서 RxSwift의 onError 처리도 해 주는 것이 맞긴 하다.

# Transforming Operators

## map
- 익숙한 그 map!
```Swift
// 위와 똑같은 이벤트 스트림을 쓴다고 했을 때
// RxSwift
Observable.combineLatest(leading, trailing)
    .map { "\($0)\($1)" }
    .subscribe(onNext: { print($0) }
    .dispose(in: disposeBag)
    
// Combine
leading
    .combineLatest(trailing)
    .map { "\($0)\($1)" }
    .sink { print($0) }
    .store(in: &cancellables)

// 출력은 1a 2a 3a 4a 4b 4c 4d
```

## 결과를 배열로 모아서 발신하기
- 드디어! 진짜 배열이 되는 순간이다!
```Swift
// RxSwift
let numbersObserver = Observer.from(firstNumbers)

numbersObserver
    .toArray()
    .subscribe(onNext: { print($0) }
    .dispose(in: disposeBag)

// Combine
let numbersPublisher = firstNumbers.publisher

numbersPublisher
    .collect()
    .sink { print($0) }
    .store(in: &cancellables)

// [1, 2, 3, 4]
```

## flatMap
- 마찬가지로 원래 쓰던 플랫맵과 비슷한데..
- 간단히 설명하면 "시퀀스의 각 요소(원소)를 다른 시퀀스로 변환하고, 변환되어 반환된 이 시퀀스들을 하나의 시퀀스로 합치는 것".
- 시퀀스란 옵저버블, 혹은 퍼블리셔를 말한다.
  - 원래 플랫맵도 반환값이 배열인 트랜스폼 함수를 배열에 맵으로 먹인다던지, 옵셔널로 그렇게 한다던지 할 때 이중 래퍼가 등장하지 않게 하는 거니까.
```Swift
// RxSwift
let numbersObserver = Observer.from(firstNumbers)

numbersObserver
    .flatMap { number -> Observable<String> in
        return Observable.of("\(number)A", "\(number)B", "\(number)C")
    }
    .subscribe(onNext: { print($0) }
    .dispose(in: disposeBag)

// Combine
let numbersPublisher = firstNumbers.publisher

numbersPublisher
    .flatMap { number -> AnyPublisher<String, Never> in
        return ["A", "B", "C"].publisher
            .map { "\(number)\($0)" }
            .eraseToAnyPublisher()
    }
    .sink { print($0) }
    .store(in: &cancellables)

// 1A, 1B, 1C, 2A, 2B, 2C, 3A, 3B, 3C, 4A, 4B, 4C 
```

### flatMapLatest
- 플랫맵과 비슷한데, 가장 최근에 변경사항이 있는 시퀀스만 구독해 반영한다.
```Swift
// RxSwift
let observable = Observable<Int>.interval(.seconds(1), scheduler: MainScheduler.instance)
    .take(3)

observable
    .flatMapLatest { number -> Observable<String> in
        return Observable<String>.interval(.milliseconds(500), scheduler: MainScheduler.instance)
            .map { "\(number)-\($0)" }
            .take(2)
    }
    .subscribe(onNext: { print($0) })
    .disposed(by: disposeBag)
    
/* 
출력
0-0
0-1
1-0
1-1
2-0
2-1
*/

// Combine
let publisher = PassthroughSubject<Int, Never>()

publisher
    .flatMapLatest { number -> AnyPublisher<String, Never> in
        return ["A", "B", "C"].publisher
            .map { "\(number)-\($0)" }
            .eraseToAnyPublisher()
    }
    .sink { print($0) }
    .store(in: &cancellables)

publisher.send(1)
publisher.send(2)
publisher.send(3)

/*
출력
1-A
1-B
1-C
2-A
2-B
2-C
3-A
3-B
3-C
*/

// 여기서 발신이 컴퓨터의 연산 속도를 넘어설 정도로 빠르게 됐다면, 1-C 이런 건 잘릴 수도 있다.
```

# Error Handling Operators

## 에러 캐치해서 핸들링하기
- 퍼블리셔를 발신하는 와중에 에러가 생겼다면, 에러 이벤트를 발신해야 한다. 그 핸들링을 담당하는 부분이다.
- 에러 이벤트를 발신하면 스트림은 종료된다. 항상 욕보는 부분.
```Swift
// RxSwift
let observable = Observable<Int>.create { observer in
    observer.onNext(1)
    observer.onNext(2)
    observer.onError(NSError(domain: "", code: 0, userInfo: nil))
    observer.onNext(3)
    observer.onCompleted()
    return Disposables.create()
}

observable
    .catchError { error in
        return Observable.just(0)
    }
    .subscribe(
        onNext: { print($0) },
        onError: { print("Error: \($0)") },
        onCompleted: { print("Completed") }
    )
    .disposed(by: disposeBag)
    
/*
출력
1
2
0
Completed
*/

// Combine
struct CustomError: Error {}

let publisher = PassthroughSubject<Int, NSError>()

publisher
    .mapError { _ in CustomError() }
    .sink(
        receiveCompletion: { completion in
            switch completion {
            case .finished:
                print("Finished")
            case .failure(let error):
                print("Error: \(error)")
            }
        },
        receiveValue: { print($0) }
    )
    .store(in: &cancellables)

publisher.send(1)
publisher.send(2)
// 강제로 에러를 꽂아넣을 수 있는 게 흥미롭다
publisher.send(completion: .failure(NSError(domain: "", code: 0, userInfo: nil)))

/*
출력
1
2
Error: CustomError()
*/
```
- `catchError`는 에러의 처리, `mapError`는 해결에 좀 더 중점을 둔 뉘앙스가 있다.
  - 컴바인에서는 에러는 받는 부분에서 핸들링해라, 이런 뜻인지.

## 재시도하기
- 지정한 횟수만큼 재시도한다.
- 만일 지정한 횟수만큼 재시도한 후에도 같은 에러가 발생한다면, 그 에러를 그대로 발신한다.
  - 당연히 스트림이 종료되겠지?
```Swift
// RxSwift
let observable = Observable<Int>.create { observer in
    var count = 0
    
    observer.onNext(1)
    observer.onNext(2)
    
    if count < 2 {
        count += 1
        observer.onError(NSError(domain: "", code: 0, userInfo: nil))
    } else {
        observer.onNext(3)
        observer.onCompleted()
    }
    
    return Disposables.create()
}

observable
    .retry(2)
    .subscribe(
        onNext: { print($0) },
        onError: { print("Error: \($0)") },
        onCompleted: { print("Completed") }
    )
    .disposed(by: disposeBag)
    
/*
출력
1
2
1
2
1
2
3
Completed
*/

// Combine
struct CustomError: Error {}

let publisher = PassthroughSubject<Int, Error>()

publisher
    .retry(2)
    .sink(
        receiveCompletion: { completion in
            switch completion {
            case .finished:
                print("Finished")
            case .failure(let error):
                print("Error: \(error)")
            }
        },
        receiveValue: { print($0) }
    )
    .store(in: &cancellables)

publisher.send(1)
publisher.send(2)
publisher.send(completion: .failure(CustomError()))
publisher.send(3)

/*
출력
1
2
1
2
1
2
Error: CustomError()
*/
```
- 무한 재시도를 막기 위해 적절한 횟수를 선정하는 것이 중요하다 할 수 있다.