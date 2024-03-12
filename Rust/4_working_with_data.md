### Enumeration
- 타입-세이프하게 데이터를 쓸 수 있게 해 준다.
- 하나의 데이터 조각을 가지고 있는 타입.
- 각각의 케이스는 "variant"라고 부른다.
- 컴파일러에게 프로그램의 정보를 제공하므로 더 안정적인 프로그래밍이 가능하다.
``` rust
enum Direction {
    North,
    South,
    East,
    West
}

// enum을 쓸 경우, 아래와 같은 방법으로 match해 줄 수 있다.
// match이므로 모든 가능한 값을 확인해야 한다. 하지 않으면 non-exhaustive patterns 에러가 뜬다.
// Swift였으면 Direction.north로 썼을 것이지만, Rust에서는 Direction::North로 쓴다.
// 각각의 variant는 대문자로 써야 하는 건지 궁금하다.
fn which_direction(heading: Direction) {
    match heading {
        Direction::North => "north",
        Direction::South => "south",
        Direction::East => "east",
        Direction::West => "west",
    }
}
```
- enum은 한 번에 하나의 variant 상태만 가질 수 있다.
### 실습 7
``` rust
enum Color {
    Red,
    Green,
    Blue,
}

fn main() {
    let color = Color::Red;
    spoid(color);
}

fn spoid(color: Color) {
    match color {
        Color::Red => println!("The color is Red."),
        Color::Green => println!("The color is Green."),
        Color::Blue => println!("The color is Blue."),
    }
}
```
- 재미있는 부분인데, 사용하지 않은 열거형 variant에 대해 컴파일러가 warning을 뱉는다.
    - Swift 코드를 짤 때 "노란 줄 오류"라고 부르는 것.
- 더 완벽한 코드를 위해 컴파일러가 도와준다는 것이 체감이 됐다.
### Structure
- 여러 데이터 조각을 포함하는 데이터 타입.
- 각각의 데이터 조각은 필드라고 부른다. 유사한 데이터끼리 묶을 수 있기 때문.
    - 다분히 근본적인 설명이지만, 사실 Swift 구조체의 프로퍼티와 같다.
``` rust
struct Box {
    depth: i32,
    width: i32,
    height: i32,
}

// 모든 값을 넣어 줌으로써 init한다. 중괄호를 사용한다.
// 옵셔널이 나중에 나올지 모르겠지만...
let my_box = Box {
    depth: 2,
    width: 3,
    height: 4,
}

// 필드 접근은 Swift 프로퍼티 접근과 유사하다.
let width = my_box.width;
println!("the box is {:?} units wide", width);
```
### 실습 8
``` rust
// f32는 Float, f64는 Double에 대응한다.
struct Drink {
    flavor: Flavor,
    ounces: f64,
}
  
enum Flavor {
    Peach,
    Grape,
}
  
fn main() {
    let drink = Drink {
        flavor: Flavor::Peach,
        ounces: 3.0,
    };

    drink_info(drink);

    let beverage = Drink {
        flavor: Flavor::Grape,
        ounces: 4.5,
    };

    drink_info(beverage);
}

fn drink_info(drink: Drink) {
    match drink.flavor {
        Flavor::Peach => println!("Peach flavor"),
        Flavor::Grape => println!("Grape flavor"),
    };
    
    println!("oz: {:?}", drink.ounces);
}
```
### Tuples
- 하나의 "레코드" 유형.
- 익명으로 데이터를 저장하며, 튜플의 필드에 이름을 붙일 필요는 없다.
- 데이터 쌍을 반환할 때 유용하며, 쉽게 해체될 수 있다.
``` rust
fn three_numbers() -> (i32, i32, i32) {
    (1, 2, 3)
}

// 이건 파이썬에서 하던 방법인데?
let numbers = three_numbers();
let (x, y, z) = three_numbers();
println!("{:?} {:?}", x, numbers.0);
```
- 튜플을 사용하면 다른 타입의 데이터도 묶을 수 있다. Swift와 비슷하다.
- 다만 2-3개가 넘는 필드를 쓸 거면 튜플 대신 구조체를 쓰는 것을 권장!
### 실습 9
``` rust
fn main() {
    let (latitude, longitude) = current_location();
    println!("{:?}", latitude);
    println!("{:?}", longitude);
    
    if longitude > 5 {
        println!("Higher than 5");
    } else if longitude < 5 {
        println!("Lower than 5");
    } else {
        println!("Exactly at 5");
    }
}

fn current_location() -> (i32, i32) {
    // 임의의 좌표값
    (132, 37)
}
```
