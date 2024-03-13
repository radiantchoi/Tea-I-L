### 식(Expressions)
- Rust는 Expression-based Language! 대부분의 것이 값을 반환한다.
- 한 점으로 모이므로, 중첩 로직을 사용할 수 있다.
``` rust
let number = 3

// if를 변수 할당에 넣기
let is_less_than_five = if number < 5 {
    true
} else {
    false
};

// 어쨌든 true나 false를 반환하니까
let is_more_than_five = number > 5;

// match도 가능하다.
let quote = match number {
    1 => "Hello!",
    _ => "Goodbye"
};

// 상기 연산을 중첩시키는 것도 가능하다
enum Menu {
    Burger,
    Fries,
    Drink,
}

struct Order {
    menu: Menu,
    quantity: i32,
}

let order = Order {
    menu: Menu::Drink,
    quantity: 3,
};

// 다만 중첩을 남용하진 말자
let order_placed = match order.menu {
    Menu::Drink => {
        if order.quantity < 5 {
            true
        } else {
            false
        }
    },
    _ => true,
}

// 데모
enum Permission {
    Admin,
    Manager,
    User,
    Guest,
}

fn main() {
    let permission = Permission::Guest;
    let access_availability = match permission {
        Access::Admin => true,
        _ => false,
    };
}
```
### 실습 10
``` rust
fn main() {
    let number = 99;
    let is_bigger_than_100 = if number > 100 {
        true
    } else {
        false
    };

    // 혹은
    let is_more_than_100 = number > 100;
    
    matcher(is_bigger_than_100);
}

fn matcher(result: bool) {
    match result {
        true => println!("It's big"),
        false => println!("It's small"),
    }
}
```
### 중급 메모리 개념
- 메모리는 0과 1로만 이루어진 비트를 통해 이진으로 저장된다. 비트는 가장 작은 단위.
- 사실 컴퓨터는 바이트(8비트) 작업에 최적화되어 있다. 바이트는 8개의 일렬로 늘어선 비트..
- 메모리에는 주소가 있다. 데이터의 위치를 특정하는 데 쓰이며, 안에 담긴 데이터가 바뀔 뿐 주소는 항상 같다.
    - 주소를 직접 사용하는 대신 변수를 이용한다.
- 메모리 오프셋은 특정 주소에 있는 항목의 위치를 특정하는 데 사용한다.
    - 0부터 시작하며, 원래 주소로부터 몇 바이트 떨어져 있는지 측정한다.
    - 직접 사용하는 대신 인덱스를 쓰면, 컴파일러가 자동으로 계산해 준다.
### 소유권
- 코드의 성능을 향상시키고, 다양한 상황에서 올바르게 컴파일되도록 한다.
- 프로그램은 메모리를 추적해야 한다. 그렇지 않으면 메모리 릭이 발생.
- Rust에서 메모리를 관리하는 방법은 ownership(소유권).
    - 메모리의 소유자가 정리할 의무가 있음을 의미한다.
    - 메모리는 대여나 이동이 가능하다.
``` rust
enum Light {
    Bright,
    Dull,
}

fn display_light(light: Light) {
    match light {
        Light::Bright => println!("Bright"),
        Light::Dull => println!("Dull"),
    }
    // 함수 호출이 끝나면 이동한 소유권 light를 삭제한다.
}

fn main() {
    // light 자료(의 메모리)의 소유권은 main 함수에 있다.
    let light = Light::Dull;
    // display_light 함수로 light의 소유권을 이동한다.
    display_light(light);
    // 이 함수 호출이 종료되는 순간 위에 정의한 light 변수는 삭제된다(!!!).
    // 이미 소유권이 다른 함수로 이동했고, 함수가 종료되는 순간 삭제되었기 때문.

    // 그래서 한 번 더 쓰려고 하면 에러가 발생한다.
    display_light(light);
}
```
- 변수가 처음 만들어진 곳이 그 소유권을 가진다.
- 이 변수를 파라미터로 활용한 함수가 있다면, 기본적으로 소유권이 "이동"한다.
- 이를 회피하고 싶다고?
``` rust
enum Light {
    Bright,
    Dull,
}

// & 기호로 대여임을 나타낸다. 이거 완전 포인터
// 사실 그냥 포인터 맞다. "참조" 라는 표현이 여러 번 등장한다. 
fn display_light(light: &Light) {
    match light {
        Light::Bright => println!("Bright"),
        Light::Dull => println!("Dull"),
    }
    // 함수 호출이 끝나도 대여받은 light의 소유권이 없기 때문에, 삭제되지 않는다.
}

fn main() {
    // light 자료(의 메모리)의 소유권은 main 함수에 있다.
    let light = Light::Dull;
    // display_light 함수로 light의 소유권을 이동하지 않고 대여한다.
    display_light(&light);
    // 이 함수 호출이 종료되는 순간 위에 정의한 light 변수가 더 이상 삭제되지 않는다.

    // 그래서 더 쓸 수 있다.
    display_light(&light);
}
// 다만 main 함수가 끝나면 삭제된다. 소유권을 가진 함수가 종료되었기 때문.
```
- 메모리는 함수 블럭(스코프)이 끝나면 자동으로 해제된다.
- 이런 낯선 짓을 하는 이유는 효율성과 메모리 관리를 위해서.
    - 데이터 크기가 굉장히 컸다면, (메모리)이동에는 엄청난 자원이 소모되었을 거다.
    - 대여를 하면 메모리가 있는 곳만 알려주므로.. 진짜 그냥 포인터잖아요.
- 그래서 이동과 대여를 적절히 사용하면 성능을 굉장히 올릴 수 있다.
### 실습 11
``` rust
struct GroceryItem {
    id: i32,
    quantity: i32,
}

fn main() {
    let item = GroceryItem {
        id: 0,
        quantity: 3
    };

    display_id(&item);
    display_quantity(&item);
}

fn display_id(item: &GroceryItem) {
    println!("id: {:?}", item.id);
}

fn display_quantity(item: &GroceryItem) {
    println!("quantity: {:?}", item.quantity);
}
```
### 기능 구현 - Impl
- 열거형이나 구조체가 기능을 구현하게 한다.
- 코드의 구성과 프로그램 가독성을 높인다.
``` rust
struct Temperature {
    degrees_f: f64,
}

// 사실 이 함수는 온도를 다룰 때만 의미가 있지
fn show_temp(temp: Temperature) {
    println!("{:?} degrees F", temp.degrees_f);
}

// 그래서 impl 안에 옮겨 줬다
impl Temperature {
    fn show_temp(temp: Temperature) {
        println!("{:?} degrees F", temp.degrees_f);
    }
    // 파라미터도 바꿔 준다. self에 대한 참조가 된다.
    // 많은 언어에서 함수 심볼에 self를 명시하는 경우가 있던는데..
    fn show_temp(&self) {
        println!("{:?} degrees F", self.degrees_f);
    }

    // Self를 반환하는 것.
    // self는 자기 타입의 인스턴스가 있어야 한다.
    // Self는 타입 이름을 참조하거나 인스턴스가 아직 없을 때 쓴다.
    // 타입 이름이 바뀌었을 때도 여기까지 바꿀 필요 없다는 장점이 있다.
    fn freezing() -> Self {
        Self { degrees_f: 32.0 }
    }
}

fn main() {
    let hot = Temperature { degrees_f: 99.9 };
    show_temp(hot);

    // 사용할 때는 이런 식으로 사용한다
    Temperature::show_temp(hot);
    // self로 바꾼 후에는
    hot.show_temp();

    let cold = Tempearature::freezing();
    cold.show_temp();
}
```
- self를 참조하는 함수는 .으로 접근, 아니라면 ::로 접근.
### 실습 12
``` rust
fn main() {
    let my_box = ShippingBox::new(3, 3, 3, 27.0, Color::Blue);
    my_box.print_characteristics();
}

struct ShippingBox {
    dimensions: Dimensions,
    weight: f64,
    color: Color,
}

impl ShippingBox {
    // Rust 컨벤션에 의하면, 새로운 인스턴스를 만드는 함수는 new라고 이름붙여야 한다.
    fn new(width: i32, height: i32, depth: i32, weight: f64, color: Color) -> Self {
        Self {
            dimensions: Dimensions {
                width: width,
                height: height,
                depth: depth,
            },
            weight: weight,
            color: color,
        }
    }
    
    fn print_characteristics(&self) {
        self.dimensions.print();
        println!("weight: {:?}", self.weight);
        self.color.print();
    }
}

struct Dimensions {
    width: i32,
    height: i32,
    depth: i32,
}

impl Dimensions {
    fn print($self) {
        println!("width: {:?}", self.width);
        println!("height: {:?}", self.height);
        println!("depth: {:?}", self.depth);
    }
}

enum Color {
    Red,
    Green,
    Blue,
}

impl Color {
    fn print(&self) {
        match self.color {
            Color::Red => println!("color: Red"),
            Color::Green => println!("color: Green"),
            Color::Blue => println!("color: Blue"),
        };
    }
}
```
