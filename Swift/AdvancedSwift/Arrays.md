# Built-in Collections
- 프로그래밍 언어에서, 데이터의 묶음(Collection)은 굉장히 중요하다.
- 여러 가지 다른 컨테이너를 제공하는 것이 좋다.
- Swift 또한 여러 가지 Collection을 제공한다!

# Arrays (배열)
- 순서를 가지고 있는 원소(Element)들의 컨테이너이다.
  - 이 원소들은 전부 같은 타입을 가진다.
  - 각각의 원소에 대해 Random Access 기능을 지원한다.
```Swift
let fibo = [0, 1, 1, 2, 3, 5, 8]
```

## Arrays and Mutability
- 배열의 내용을 바꾸고 싶다면, 위에서 `let`이 아니라 `var`로 선언해야 한다. 당연한 얘기인가?
- `append` 등의 메서드를 활용해 이제 바꿀 수 있다.
```Swift
var fibo = [0, 1, 1, 2, 3, 5]
fibo.append(8)
fibo.append(contentsOf: [13, 21])

print(fibo) // [0, 1, 1, 2, 3, 5, 8, 13, 21]
```
- `var`과 `let`을 구분함으로써 얻는 몇 가지 이점이 있다.
- 먼저 `let`으로 선언한 것은 불변함이 보장된다. 코드를 읽을 때도 도움이 된다!
  - 단 이것은 value semantics가 적용될 때, 즉 원소가 값 타입일 때에 한정된다.
  - 참조 타입 원소가 들어 있다면 배열이 `let`으로 선언되었다 하더라도 그 원소는 바뀔 수 있다.
- 배열은 다른 stdlib 컬렉션 타입들처럼, value semantics를 따른다.
  - `=` 연산자로 로컬 복사본을 가져올 수 있고, 복사본에 가하는 연산이 원본에 영향을 미치지 않는다.
```Swift
var a = [1, 2, 3]
var b = a
b.append(4)

print(a) // [1, 2, 3]
print(b) // [1, 2, 3, 4]
```
- Obj-C 포함 다른 많은 언어는 Reference Semantics 보유 언어인 경우가 많다. JavaScript의 예를 들자면..
```Javascript
const a = [1, 2, 3];
const b = a;
b.push(4);

// 분명 b에만 원소를 추가했지만...
console.log(a); // [1, 2, 3, 4]
console.log(b); // [1, 2, 3, 4]

// 이를 회피하기 위해 똑같은 배열을 따로 만들어 주어야 한다..
c = [1, 2, 3];
d = c.slice();
d.push(5);

console.log(c); // [1, 2, 3]
console.log(d); // [1, 2, 3, 5]
```
- Reference semantics 배열의 경우 이런 특징을 까먹으면 오류가 나기 쉽다.
- 하지만 Swift 언어의 경우 배열에 value semantics를 적용시킴으로써 이를 막는다.
- 이 때문에 메모리/퍼포먼스 문제가 발생할 수 있지만, Swift 내부적으로 Copy on Write 방법을 사용함으로써 이를 방지한다.
  - 이는 Swift 안의 모든 Collection 타입에 적용되어 있다.
  - 위의 예시의 경우 a, b 두 배열은 b의 값이 바뀌기 전까지는 같은 메모리를 공유한다.

## Array Indexing
- Swift 배열은 isEmpty, count, 그리고 subscripting을 통해 특정 인덱스의 원소에 직접 접근하는 방법 등을 지원한다.
  - subscript 방법을 통해 원소를 얻고자 할 경우, 배열의 크기보다 크지 않은 값을 사용해야 함에 유의!
- Swift 배열을 통해 다양한 작업을 할 수 있다.
```Swift
var a = [1, 2, 3, 4, 5]

for x in array { //... }

a.dropLast(2) // [1, 2, 3]

for (i, number) in a.enumerated() { //... }

for (index, element) in zip(a.indices, a) { //... }

if let idx = a.firstIndex { $0 % 2 == 0 } { //... }

a.map { $0 * 2 } // [2, 4, 6, 8, 10]

a.filter { $0 % 2 == 1 } // [1, 3, 5]
```
- 위의 여러 방법들을 보면, 전통적 C 스타일로 인덱스를 직접 활용해서 뒤지는 것은 Swift 스타일은 아님을 알 수 있다.
- 하지만 가끔은 그런 작업도 필요한 법..! 
- 이런 경우 Swift 차원에서는 조회하기 위해 넣은 index 번호가 유효한지 딱히 검증하진 않는다.
  - 만약 배열의 크기보다 index 번호가 크다면? "Index out of range" (펑)
```Swift
var a = [1, 2, 3]

// first와 last는 optional을 반환한다. 빈 배열일 수도 있으니까!
a.first // Optional(1)
a.last // Optional(3)

// removeLast 메서드는 빈 배열에서 불러 오면 프로그램 크래쉬를 발생시킨다
a.removeLast() // 3

// popLast 메서드는 빈 배열에서 불러 오면 nil을 반환한다
a.popLast() // Optional(2)
```