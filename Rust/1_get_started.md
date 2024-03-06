### 설치
IDE는 VSCode에서 사용
``` shell
# 설치
brew install rustup-init

rustup-init

# 쉘을 껐다 켜자

rustup toolchain install stable
```

``` shell
# 간단히 테스트해 보자
# 먼저 원하는 디렉토리로 이동한 후
cargo init hello
# hello라는 디렉토리 안에 기본적인 소스 코드가 만들어진다

# 실행하기
# hello 내부로 이동해서
cargo run
```
### 타입
물론 모든 타입은 바이너리 데이터로 표현된다. 메모리에 저장되는 것은 바이너리 데이터.

**기본 타입**
- Boolean
- Integer
- Double / Float
- Character (작은따옴표)
- String (큰따옴표)
### 변수
데이터를 임시 메모리에 할당하는 방법.
Immutable: 상수, Mutable: 변수
당연히 Immutable이 성능상 더 좋다. 변경사항을 체크하지 않아도 되기 때문.
``` rust
// 세미콜론을 쓰고, snake case를 쓰는 언어.
let two = 2;
let my_half = 0.5;
let your_half = my_half;
let mut my_name = "Gordon" // 변수는 이렇게 정의한다
let mut letter = 'a'
```
### 함수
기능의 캡슐화. 선택적으로 데이터를 받아들이고 반환하며, 가독성을 올린다.
"호출" 함으로써 실행된다.
``` rust
// 당연히 선점되어 있는 키워드로는 함수 심볼을 못 만든다
// 파라미터 타입을 주는 거는 Swift와 스타일이 똑같다 - 커스텀 타입도 가능
// 인수의 i32는, 짐작되겠지만, 32비트 정수를 나타낸다.
fn add(a: i32, b: i32) -> i32 {
	a + b
}

let gross = add(3, 4);
```
### println 매크로
매크로라고 부르네..? 매크로를 만들 수도 있는 모양. 이 강의에서는 다루지 않는다.
"유용한 디버깅 도구"
``` rust
let life = 42;
// 매크로는 호출할 때 ! 마크를 붙인다고 한다.
println!("Hello");
// {:?} 기호는 토큰 안에서(중괄호) 디버그 모드에서(콜론 물음표) 값을 포시함을 말한다.
// 여러 번 쓰면 이 토큰을 반복할 때도 있다.
println!("{:?} {:?}", life, life);
```

### if문을 활용한 제어 흐름
if, else if, else
``` rust
let a = 99;
if a > 99 {
    println!("Big Enough!");
} else {
    println!("Still Small.");
}

// 아래 두 개는 결과는 같긴 하지만, 확인하는 순서가 다르다.
if a > 99 {
    if a > 200 {
    	println!("Too Big!");
    } else {
    	println!("Big Enough.");
    }
} else {
    println("Too small..");
}

if a > 200 {
    println!("Too Big!");
} else if a > 99 {
    println!("Big Enough.");
} else {
    println("Too small..");
}
```
가급적이면 항상 else에 해당하는 과정을 명시하자.
### 반복문
loop, while 키워드가 있다.
``` rust
// loop를 while true처럼 사용하네
let mut a = 0;
loop {
    if a == 5 {
    	break;
    }

    println!("{:?}", a);
    a = a + 1;
}

// 당연히 rust의 while도 break로 빠져나올 수 있다.
let mut b = 0;
while b != 5 {
    println!("{:?}", b);
    b = b + 1;
}
```
### 주석
지금까지.. 잘 썼죠..?
다만 역시 주석이 꼭 필요한 코드보다는 주석 없이도 잘 읽히는 코드가 좋다. 클린 코드!
