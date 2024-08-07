### 3
함수 키워드는 fun
main() 함수는 엔트리 함수

### 4
var는 변수, val은 상수
변수명은 시작 글자는 유니코드 문자나 밑줄이어야 한다. 숫자는 안 된다.

### 5
Double, Int, String이 있다.
타입 추론이 강력해서, 스트링과 인트를 더하면 일단 결과값을 스트링으로 추론한다.
더블과 인트도 그런 식으로 더한다.
``` kotlin
val example = "Gordon" + 9.13
println(example) // Gordon9.13
```
단! 한 번 추론되어 정의된 변수/상수는 다른 타입의 값을 할당할 수 없게 된다.
근데 연산자를 쓰려면 자바에 정의되어 있는 Sequence? 그런 게 필요한데 그걸 어떻게 해야 되는지 모르겠다..
Character가 아니라 Char로 쓴다.
큰따옴표 세 개를 쓰고 스트링을 입력하는 것을 raw string이라고 하며, 여러 줄을 써도 되고 이스케이핑 문자를 안 써도 된다.

### 6
함수 이름, 함수 파라미터, 함수 반환 타입을 합쳐 함수 시그니처라고 한다 - 파라미터 이름은 빼고
함수를 불러올 때 파라미터 레이블을 붙여 주지 않는다. 그냥 해당 위치에 값을 넣을 뿐.
함수를 명령어 모음이라고 생각할 수도 있고, 치환 가능한 값을 넣었을 때 결과를 내는 식이라고 생각할 수도 있다.
Swift의 Void는 여기서는 Unit이다.
함수 컨벤션은 대략 이렇다.
``` kotlin
// 블록 본문 함수 - 보통 아는 그거
fun sum(first: Int, second: Int): Int {
	return first + second
}

// 식 본문 함수 - 함수가 단일 식으로만 이루어져 있다면 이런 식으로도 표현할 수 있다
// 식 본문 함수의 경우 반환 타입을 명시하지 않아도 추론한다
fun sum2(first: Int, second: Int) = first + second

// 파라미터 레이블을 명시하지 않으며, 함수 안에서만 그 시그니처를 사용한다
sum(1, 2) // 3
sum2(3, 4) // 7
```

### 7
if문 컨벤션은 다음과 같다.
``` kotlin
var x = 0

// if문에는 괄호를 붙인다
// 뒤에 콜론이나 중괄호 등은 넣지 않는다
// 따라서 이렇게 삼항 연산자 같은 느낌으로 쓸 수도 있다
val biggerThanZero = if (x > 0) true else false
println(biggerThanZero) // false
// !로 부정할 수 있는 것도 똑같다.
println(!biggerThanZero) // true

x = 2
println(biggerThanZero) // true

// 여기서도 return에서 스코프가 종료되는 것은 비슷하다
// 당연히 else 뒤에 return시켜 줘도 된다
fun booleanFunction(exp: Boolean): Boolean {
	if (exp)
		return true

	return false
}

// if문도 하나의 식이라, 이런 식으로 써도 된다
fun booleanFormulaFunction(exp: Boolean) = if (exp) true else false

```

### 8
스위프트에서는 문자열 보간법을 위해 백슬래시 소괄호를 사용했다만, 코틀린에서는 $를 사용한다.
``` kotlin
fun sayHello(name: String): Unit {
	println("Hello, $name!")
} 

/* 
$ 뒤에 오는 것이 식별자로 인식된 게 없다면 
아무 일도 일어나지 않고 그냥 달러 기호와 따라오는 스트링이 표시된다.
*/
```
String간의 덧셈도 당연히 가능하다.
이스케이핑 문자와, 아까 말한 raw string 큰따옴표 등도 스위프트와 마찬가지 감각이다.
${} 안에 식을 넣으면 그 식을 평가(계산..?)한다.
``` kotlin
fun main(): Unit {
	val condition = true
	println("${if (condition) 'Good condition!' else 'Bad condition...'}")
	// Good condition!

	val x = 11
	println("$x + 3 = ${x + 3}") // 11 + 3 = 14
}
```

### 9
정수를 넣으면 Int 타입을 추론한다.
자릿수를 표시할 때 언더바를 사용할 수 있다.
더하기, 빼기, 곱하기, 나누기, 나머지 연산은 스위프트와 동일하다.
나눗셈 나머지는 버린다.
연산자의 연산 순서는 수학 산술 연산 순서를 따른다.
Double은 큰 부동소수점 수.
계산 식의 앞부분에 Double이 나오면, 뒤에서 Int를 더해도 타입이 Double로 추론된다.
Int.MAX_VALUE로 미리 최댓값이 스태틱하게 정해져 있다. 32비트 공간 크기다.
이거보다 큰 정수를 저장하려면 Long 타입을 써야 한다. 숫자 뒤에 L을 붙이면 적용된다. 아니면 타입 명시를 하던가.
Long이 포함되어 있을 경우 그 위치에 상관없이 결과값이 Long으로 추론된다.

### 10
and와 or는 스위프트와 똑같다. && ||
괄호가 없다면 and를 먼저 평가하고 이후 or를 평가한다.

### 11
while 컨벤션은 이렇다.
``` kotlin
var x = 0
fun threshold(x: Int) = x < 100

// 조건이 안 맞으면 while은 그냥 패스해버린다
// 참고. += -= 이런 것들의 규칙은 스위프트와 똑같다
// 오 근데 1씩 더하기 빼기에 한해서 ++ --도 작동한다!
while (threshold(x)) {
	println(x)
	x += 1
}

// do-while은 그래도 한 번은 실행된다
do {
	println(x)
	x++
} while (threshold(x))

```

### 12
for문 컨벤션이다.
``` kotlin
fun divideAndPrint(count: Int) {
	// 각각의 범위를 나타내는 구문은 Progression이라는 타입으로 판정된다.
	// 스위프트에서는 ClosedRange<T> / Range<T> where T: Comparable 였던가?
	// IntProgression, CharProgression, ... 등 여러 가지가 있다.
	// ComparableRange를 커스텀하게 생성할 수도 있다. 
		// 이 경우 CloseRange<T>와 같은 모양을 여기도 가지게 된다!
		// 물론 Comparable한 객체에 대해서만 가능한 동작이다.
	for (i in 0..count) {
		print("$i") // 0123...count
	}

	// until은 정수에만 쓸 수 있다. 문자일 때는 못 쓴다.
	// kotlin 1.8부터 ..< 연산자를 써도 된다.
	for (j in 0 until count) {
		print("$j") // 0123...(count-1)
	}

	for (k in count downTo 0) {
		print("$k") // (count)(count-1)...0
	}

	for (l in 0..count step 2) {
		print("$l") // 02468...(count or count-1)
	}

	// CharRange / CharProgression
	for (c in 'a'...'z') {
		print($c) // abcde...z
	}

	// 스트링 인덱싱은 파이썬처럼 된다
	// 문자는 아스키 코드에 해당하는 숫자 값으로 저장되어, 1을 더하면 그 코드의 문자가 나온다
	val name = "Gordon"
	for (d in 0..name.lastIndex) {
		print(s[i]) // Gordon
		print(s[i] + 1) // Hpsepo
	}
}

// Kotlin Range는 Progression의 특수한 형태로, step 같은 걸 적용할 수 없다. 1로 고정!
```

### 13
in 키워드는 주어진 값이 주어진 범위 안에 들어 있는지 체크한다.
``` kotlin
val letters = 'a'...'z'
println('c' in letters) // true

val name = "Gordon"
println('c' in name) // false
println('c' !in name) // true
```
참고로 코틀린은 문자를 사전식으로 비교한다.

### 14
문은 효과를 발생시키지만 결과를 내놓지 않고, 식은 항상 결과를 내놓는다.
A statement changes state, an expression expresses
단적으로, for"문"은 그 자체로 결과값을 만들어내지 않고, 그래서 다른 변수에 대입할 수 없다.
식은 값을 만들어내기 때문에 그 자체를 다른 변수로 사용하거나 다른 식의 일부분으로 사용할 수 있다.
if도 식을 만들어내기 때문에 그 결과를 변수에 대입할 수 있다.
``` kotlin
fun main() {
	val unitOne = println(2) // 42
	println(unitOne) // kotlin.Unit

	val result = if (1 < 2)
		val a = 11 // 로컬 변수 - 이 스코프를 나가면 해제!
		a + 42
	else 42

	println(result) // 53
}

// 번외: i++도 식이다!
// i++ ++i i-- --i가 다 다르다. 연산 전의 값을 주느냐, 연산 후의 값을 주느냐의 차이.
var x = 10
println(x++) // 10
println(++x) // 12
println(x--) // 12
println(--x) // 10
```
