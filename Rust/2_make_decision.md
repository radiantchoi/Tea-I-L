### 실습 3a
if else를 활용해 보자.
boolean이 아니고 bool이다.
bool은 true인지 아닌지 굳이 쓰지 않아도 좋다.
``` rust
fn main() {
    let mut status = true;
    determine(status);
}

fn determine(status: bool) {
    if status {
    	println!("Hello");
    } else {
    	println!("Goodbye");
    }
}
```
### 실습 3b
``` rust
// if-else if-else까지 사용하기
fn main() {
    let number = 10;
    determine_number(number);
}

fn determine_number(n: i32) {
    if n > 5 {
    	println!(">5");
    } else if n == 5 {
    	println!("==5");
    } else {
    	println!("<5");
    }
}
```
### Match
Match 사용하기.
if-else와 같다만, exhaustive해야 한다. 모든 옵션 고려하기. Swift의 switch와 비슷하다.
``` rust
fn main() {
    let status = true;

    // fat arrow라고 표현하는 것이 너무 재밌다
    match status {
    	// 각각의 케이스가 끝날 땐 세미콜론 대신 반점을 사용한다.
    	true => println!("True!"),
    	false => println!("False!"),
    }

    let number = 5; 

    match number {
    	1 => println!("One"),
    	2 => println!("Two"),
    	// default case는 이렇게 표현한다
    	// 케이스가 추가되면 컴파일러가 "야 이건 어쩔거야" 라고 알려 준다.
    	_ => println!("elsewhere"),
    }
}
```
단일 변수로 작업할 때는 if-else보다 match가 좋다. 모든 가능성을 생각하기 더 쉽기 때문.
### 실습 4a
``` rust
fn main() {
    let status = true;
    // 여기 세미콜론을 안 붙여도 컴파일은 되긴 하던데, 마지막 문장이라 그렇다.
    determine(status);
} 

fn determine(status: bool) {
    match status {
        true => println!("it's true!"),
        false => println!("it's false!"),
    }
}
```
### 실습 4b
처음 생각한 코드
``` rust
fn main() {
    let number = 1;
    let result = determine(number);
    println!(result);
}

// str은 컴파일 타임에 크기가 정해져 있지 않다.
// 하지만 rust의 함수 반환값은 static한 사이즈를 가진 자료형이어야 한다.
fn determine(number: i32) -> str {
    match number {
    	1 => return "one",
    	2 => return "two",
    	3 => return "three",
    	_ => return "other",
    }
}
```
타입에 관한 것은 이후에 나올 것이므로, 급하게 가지 않기로 한다.
``` rust
fn main() {
    let number = 1;
    determine(number);
}

fn determine(number: i32) {
    match number {
    	1 => println!("one"),
    	2 => println!("two"),
    	3 => println!("three"),
    	_ => println!("other"),
    }
}
```