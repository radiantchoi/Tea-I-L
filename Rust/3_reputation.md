### loop
loop은 while true가 맞다
``` rust
fn main() {
    let mut countdown = 5
    
    loop {
        // 안녕로봇
        println!("Hello!");
        println!("{:?}", countdown);

        countdown = countdown - 1;
        // 중단을 위해서는 반드시 break가 필요하다
        if countdown == 0 {
            break;
        }
    }

    println!("LAUNCH");
}
```

### 실습 5
``` rust
// 1부터 4까지 출력하기
fn main() {
    let mut countup = 1;
    
    loop {
        println!("{:?}", countup);
        // += 연산자도 작동하는구나~
        countup += 1;
        if countup == 5 {
            break;
        }
    }
}
```

### while
``` rust
fn main() {
    let mut n = 1

    while n <= 3 {
        println!("{:?}", n)
        n += 1
    }
}
```

### 실습 6
``` rust
fn main() {
    let mut countdown = 5;
    while countdown > 0 {
        println!("{:?}", countdown);
        countdown -= 1;
    }
    
    println!("Done!");
}
```

