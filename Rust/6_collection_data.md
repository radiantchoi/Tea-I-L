### Vector
- 여러 조각의 데이터를 저장할 수 있는 저장소. 이 조각들의 타입은 모두 같아야 한다.
- 정보 목록에 사용된다.
  - 식료품 목록이 있다면 이걸 벡터에 저장한다는 거.
- 더하고, 빼고, 순회할 수 있다.
```Rust
// 벡터 매크로라고 한다.
let numbers = vec![1, 2, 3];

// 변수로 만들고 싶으면
let mut numbers = Vec::new();
numbers.push(1);
numbers.push(2);
numbers.pop(); // 2
numbers.len(); // 1
let one = numbers[0];

numbers.push(2);
numbers.push(3);

for num in numbers {
    println!("{:?}", num);
}
```
- 사실상 배열이 아닌가..

### 실습 13
```Rust
fn main() {
    let numbers = vec![10, 20, 30, 40];

    // for문의 생성 과정에서도 소유권이 이전된다. 그래서 대여해야 한다.
    for number in &numbers {
        match number {
            30 => println!("thirty"),
            _ => println!("{:?}", number),
        }
    }

    println!("length: {:?}", numbers.len());
}
```

### Strings
- 그래 내가 이거 컬렉션일 줄 알았다.
- Rust의 String에는 여러 가지가 있는데, 대표적으로 두 가지 꼽아보자면
  - String: 소유 데이터 타입. `struct`에 넣으려면 반드시 이걸 써야 한다. "아직까지는"
  - &str: 문자열 슬라이스에 대한 참조. 함수에 문자열에 관한 정보를 제공할 떄 좋다.
```Rust
fn print_it(data: &str) {
    println!("{:?}", data);
}

fn main() {
    // 일반적인 큰따옴표를 사용하는 경우는 자동으로 참조로 생성된다.
    print_it("a string:");
    
    // 소유 스트링을 만들고 싶으면
    let owned = "owned".to_owned;
    let pwned = String::from("pwned");
    print_it(&owned);
}
```
- 왜 `struct`에서 쓸 때 `String`을 써야만 하느냐?
  - 스코프의 종료시 자신의 메모리를 정리(드랍)할 책임이 있는데, 참조는 자기가 멋대로 정리할 수 없기 떄문.
```Rust
// 아래 코드는 컴파일 에러가 난다
struct Employee {
    name: $str,
}

fn main() {
    let name = "James";
    let employee = Employee { name: name };
}

// 이렇게 바꿔주면 된다
struct Employee {
    name: String,
}

fn main() {
    let name = "James".to_owned;
    let employee = Employee { name: name };
}
```

```Rust
struct LineItem {
    name: String,
    count: i32,
}

fn main() {
    let receipt = vec![
        LineItem {
            name: "cereal".to_owned(),
            count: 1,
        },
        LineItem {
            name: String::from("fruit"),
            count: 3,
        },
    ];
    
    for item in receipt {
        // 차용타입을 쓰려면 이렇게 참조해줘야 한다.
        print_name(&item.name);
        println!("count: {:?}", item.count);
    };
}

fn print_name(name: &str) {
    println!("name: {:?}", name);
}
```

### 실습 14
```Rust
struct Person {
    name: String,
    age: i32,
    favorite_color: String
}

fn main() {
    let people = vec![
        Person {
            name: "Gordon".to_owned(),
            age: 10,
            favorite_color: String::from("Blue")
        },
        Person {
            name: "Greg".to_owned(),
            age: 7,
            favorite_color: String::from("Green")
        },
        Person {
            name: "Therese".to_owned(),
            age: 34,
            favorite_color: String::from("Green")
        },
    ];

    for person in people {
        if person.age <= 10 {
            print_info(&person.name);
            println!("{:?}", person.age);
            print_info(&person.favorite_color);
        }
    }
}

fn print_info(info: &str) {
    println!("{}", info)
}
```