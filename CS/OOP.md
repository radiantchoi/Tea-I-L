# 객체 지향 프로그래밍
## 용어
### 클래스 (Class)
- classification에서 유래한 단어. 객체 지향 프로그래밍에서, 특정 객체를 생성하기 위해 변수와 메서드를 정의해 둔 일종의 틀, 템플릿이다.
- 객체 지향 프로그래밍에서는 모든 데이터를 객체로 취급하며, 이러한 객체를 중심으로 프로그래밍한다. 
  - 클래스는 이 객체를 만들어내기 위한 틀 또는 설계도로 설명될 수 있다.
- 클래스는 상속의 개념이 있고, 해당 클래스의 전부 또는 일부 기능을 이 클래스를 상속받는 클래스, 즉 서브클래스에게 계승할 수 있다. 
  - 물론 서브클래스도 자신만의 프로퍼티와 메서드를 갖는 것이 가능하다. 이를 클래스 계층이라 부른다.
### 객체 (Object)
- 객체 지향 프로그래밍의 중심이 되는 요소. 보통 현실의 사물에 많이 비유된다.
- 변수, 자료구조, 함수, 메서드가 될 수 있다.
- 객체 지향 프로그래밍에서, 객체는 클래스의 인스턴스이다.
- 또한 객체는 상태와 행동을 가진다. 알고 있는 것(상태)과 할 수 있는 것(행동)을 각각 변수와 메서드로 나타낼 수 있어야 하는 것이다.
    - 그래서 객체화를 시키려면 “알아야 한다". 의미 있는 값을 프로퍼티로 만들고, 프로퍼티를 통해 알아야 할 행동들을 메서드로 구현한다.
### 인스턴스 (Instance)
- 일반적으로는 실행되고 있는 프로세스나 클래스의 현재 생성된 객체를 말한다.
- 객체 지향 프로그래밍에서는, 할당된 물리 메모리의 일부를 가리키기도 한다.
- 클래스에서 정의한 것을 토대로 실제 메모리에 할당된 것이다. 프로그램에서 사용되는 데이터 혹은 식별자(이 경우엔 변수 이름이라던지)에 의해 참조된다.
- 객체가 메모리에 할당되어 실제로 사용될 때 인스턴스라고 불린다. 인스턴스는 결국 객체의 부분집합인 셈.
## 객체지향의 구현
### 프로퍼티 (Property)
- 객체가 가지고 있는 의미있는 값.
- 보통 클래스 내에 상수나 변수를 정의함으로써 구현된다.
- 하나의 객체를 하나의 데이터 튜플로 볼 때, 변수명은 프로퍼티 키, 할당된 값은 프로퍼티 값으로 볼 수 있다.
### 메서드 (Method)
- 객체 내에 함수로써 구현되어 있는 동작.
- JS에서는 프로퍼티의 타입이 함수일 경우 구분을 위해 메서드라고 부른다고 한다.
  - 하지만 Swift에서는 함수가 First Citizen이기 때문에, 클로저(함수) 타입을 가진 프로퍼티가 있을 수 있다.
  - `var completion: Result<String, Error> -> Void`
### 캡슐화 (Encapsulation)
- 객체의 프로퍼티와 메서드를 하나로 묶고, 실제 구현 내용 중 일부를 감추어 은닉한다. 그래서 은닉화와 자주 붙어 나오는 것.
- 클래스로 만드는 것 자체가, 캡슐화의 일종이 될 수 있다.
- 이 과정에서 디미터의 법칙에 유의할 필요가 있다. 
  - 프로그래밍 언어 측면에서 간단히 말하자면, 너무 깊이 파고들지 말라는 것이다.
``` swift
    let man = Human()
    man.brain.isEmpty() // 너무 깊이 파고듬
    man.isIdiot() // 좋은 예시
```
- 은닉화를 구현하는 방법은 접근 제한자이다. 
- Swift에는 5가지 수준의 접근 제한자가 있다.
    - private, fileprivate, internal, public, open
### 상속 (Inheritance)
- 객체 간의 관계를 구축하는 방법이다.
- 기존 클래스로부터 일부 또는 전부의 동작을 계승받을 수 있다.
- 상속받은 클래스는 서브클래스, 상속하는 클래스는 수퍼클래스라고 한다.
- 리스코프 치환 원칙에 주의하자. 자식 클래스의 오버라이드된 메서드는, 부모 클래스의 해당 메서드가 할 수 있는 일을 모두 할 수 있어야 한다.
    - Swift의 super 키워드가 등장하는 이유.
### 다형성 (Polymorphism)
- 한 마디로 먼저 말하면, 같은 모양의 코드가 다른 역할을 하는 것.
- 프로그래밍 언어의 각 요소들이 다양한 타입에 속하는 것이 허가되는 성질. 하나의 타입에 여러 객체를 대입할 수 있는 성질이라고도 할 수 있다.
- 다형성을 활용하면 기능의 확장을 꾀할 수 있고, 객체를 변경해야 할 때 타입 변경 없이 객체 주입만으로 수정이 일어나게 할 수 있다. 
  - 한편 상속을 사용하면 중복되는 코드까지도 제거할 수 있다.
- 메서드 오버로딩과 메서드 오버라이딩이 대표적인 다형성 구현의 예시이다.
- 오버로딩은, 하나의 메서드의 파라미터 심볼을 다르게 해 여러 종류의 타입에서 결과적으로 같은 기능을 하도록 하기 위한 작업이다.
  - 생성자 오버로딩이나 연산자 오버로딩도 꽤나 자주 사용되는 방법.
  - 제네릭과는 다른 개념. 제네릭은 특정 타입을 만족하는 모든 타입에서 하나의 메서드를 쓰는 방법이다.
- 오버라이딩은 상위 클래스의 메서드를 하위 클래스에서 재정의하는 방법이다. 따라서 상속의 개념이 추가!
## 객체와 클래스
### 추상화와 구체화
- 먼저 추상화는 어떤 양상, 세부 사항, 구조를 좀 더 명확하게 이해하기 위해 특정 절차나 물체를 의도적으로 생략하거나 감춤으로써 덜 복잡하게 하는 방법이다.
- 일반화를 통해 이루어질 수 있다.
    - 일반화는, 구체적인 사물들 간의 공통점은 취하고 차이점은 버리는 것이다.
    - 중요한 부분을 강조하기 위해 불필요한 세부 사항을 제거한다.
- 추상화의 목적은 복잡한 것을 이해하기 쉬운 수준으로 단순화하는 것이다.
- 개념을 추상화하기 위해서 우리가 사용하는 것이 바로 “타입"!
- 구체화는 추상화의 반대로써, 좀 더 분명히 객체의 정체성을 나타내야 할 때 차이점과 세부 사항 위주로 덧대 주는 것을 말하는 것이다.
- 객체나 인스턴스를 클래스 코드로 나타냄으로써 일반화되고 추상화할 수 있다. 
- 반면 클래스 코드를 사용해 객체, 인스턴스를 생성했다면 구체화와 개별화가 될 것이다.
### Classification
- 간단히 말하자면, 지식을 정리하는 방법이다.
- 객체 지향 설계에서는, 여러 요소들간의 공통점을 찾는 것이 classification의 척도가 된다.
- 이 과정에서 핵심이 되는 요소와 동작이 필요하다.
- 잠깐, 위에서 class는 classification에서 유래했다고 했다?!
- 객체 지향 설계에서 클래스를 만드는 것은, 곧 현실의 물체/관념을 추상화해서 분류하는 classification 작업을 한다고 볼 수 있다.
### 간접 참조 (Indirection)
- 컴퓨터 공학에서 "간접 참조"는 값 자체보다 컨테이너, 연결, 별명(alias)등을 사용해서 우회해서 참조하도록 하는 방식을 말한다.
  - 언뜻 보면 추상화와 비슷해 보이지만, 다르다.
- 추상화와 가장 큰 차이는, 구체적인가 아닌가라고 볼 수 있다. 추상화된 클래스는 말 그대로 구체적이지 않다.
- C에서는 pointer of pointer로 구현할 수 있다.
- 참조에 의한 문제가 발생할 수 있기 때문에 쓰이는 방법이기도 하다. 
  - 가를 나가 참조하고, 다 = 나, 라 = 나라고 할당했을 때, 다와 라에는 사실 가에 대한 직접 참조 주소가 들어가게 된다. 
  - 나의 값은 가를 가리키는 주소값이니까! 그래서 나가 마를 참조하게 바꾸고 나면, 다와 라도 마를 참조하게 되는 것이다.
- 이 때 간접 참조를 활용해 나.value에 가의 값을 할당해 두었다가, 그 값만 바꾸면, 다른 값들도 똑같이 바뀌게 된다.
### 추상화와 일반화
- 객체를 활용해 프로그래밍하다 보면, 중복되는 부분을 발견할 수 있다.
- 중복되는 부분은 추상화를 통해 분리할 수 있고, 상속으로 적용할 수 있다.
``` swift
// before
class Dog {
    func move() { //... } 
}

class Cat {
    func move() { //... }
}

// after
class Animal {
    func move() { //... }
}

class Dog: Animal { 
    // 별도로 move 함수를 구현할 필요가 없다
}

class Cat: Animal {
    // 필요하다면 override할 수 있다.
    override func move() {
        // 리스코프 치환 원칙의 준수
        super.move()
        // ...
    }
}
```
## 객체지향 5원칙
### 단일 책임 원칙 (SRP: Single Responsibility Principle)
- 클래스는 단 하나의 책임을 가져야 하며 클래스를 변경하는 이유는 단 하나의 이유이어야 한다.
- 이를 지키지 않으면, 한 책임의 변경에 의해 다른 책임과 관련된 코드에 영향을 미칠 수 있다. 
  - 이렇게 되면 유지보수가 매우 비효율적
### 개방 폐쇄 원칙 (OCP: Open-Closed Principle)
- 확장(기능 추가)에는 열려 있어야 하고 변경(기능 변경 등)에는 닫혀 있어야 한다.
- 연쇄적인 수정을 막기 위함.
- 확장은 코드 상에서 `extension`으로 표현된다.
  - 상속을 통한 확장은 수직적 확장, `extension`과 프로토콜을 활용한 확장은 수평적 확장.
### 리스코프 치환 원칙 (LSP: Liscov Substitution Principle)
- 상위 타입의 객체를 하위 타입의 객체로 치환해도 상위 타입을 사용하는 프로그램은 정상적으로 동작해야 한다.
- Swift에서는 `super` 키워드를 사용함으로써 준수할 수 있다.
- 상속관계에서는 꼭 일반화 관계 (`IS-A`) 가 성립해야 한다는 의미 (일관성 있는 관계인지)
- 상속관계가 아닌 클래스들을 상속관계로 설정하면, 이 원칙이 위배됨 (재사용 목적으로 사용하는 경우)
### 인터페이스 분리 원칙 (ISP: Interface Segregation Principle)
- 인터페이스는 그 인터페이스를 사용하는 클라이언트를 기준으로 분리해야 한다.
- 무슨 말이냐면, 큰 인터페이스를 작은 인터페이스들로 분리하여, 사용자가 이용할 인터페이스를 취사선택할 수 있게 해야 한다는 말이다. 
- 클라이언트가 사용하지 않는 인터페이스에 변경이 발생하더라도 영향을 받지 않도록 만들어야 하는 것이 핵심이다.
- 한 클래스는 자신이 사용하지 않는 인터페이스는 구현하지 않아야 한다.
  - 따라서 하나의 범용적인 인터페이스보다는 차라리 여러 개의 세부적인 (구체적인) 인터페이스가 나을 수 있다.
### 의존성 역전 원칙 (DIP: Dependency Inversion Principle)
- 의존 관계를 맺을 때, 변하기 쉬운 것 (구체적인 것) 보다는 변하기 어려운 것 (추상적인 것)에 의존해야 함
- 즉 고수준 모듈은 저수준 모듈의 구현에 의존해서는 안 됨
  - 저수준 모듈이 고수준 모듈에서 정의한 추상 타입에 의존해야 함
  - 저수준 모듈이 변경되어도 고수준 모듈은 변경이 필요없는 형태가 이상적
- Swift에서는 프로토콜을 사용해 이를 실현해줄 수 있다. 
  - 프로토콜을 통해 하위 모듈을 추상화해 주고, 그 프로토콜을 상위 모듈의 인스턴스로 넣어주면 된다.
``` swift 
protocol Mammal {
    var legs: Int { get }
}

struct Dog: Mammal {
    let legs: Int
}

// 좀 더 범용적으로 쓰이는 "우리" 타입에서, 안에 있는 동물은 "개"에 의존하기보다 "포유류"에 의존해야 한다.
struct Barn {
    let animal: Mammal
    
    init(animal: Mammal) {
        self.animal = animal
    }
}
```