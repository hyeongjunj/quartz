## 참조와 패턴 매칭, 그리고 &...

https://users.rust-lang.org/t/automatic-dereferencing/53828/2

### Reference and Dereference ( + Auto dereference)

아래 코드는 전부 어떤 변수를 Call-by-reference 로 넘겨서 값을 변경시키는 예제이다. & 연산자는 C 와 유사하게 참조를 나타내며, * 연산자는 역참조를 나타낸다.

그런데, 아래 예제에서 보다시피 Point 구조체와 Vector 원소 바꾸는 곳에서
`*v.x = 3` 이나 `*v[0] += 100` 이렇게 역참조를 하지 않는다.

그 이유는 Rust 의 automatic-dereferencing 기능 때문인데, `.` 연산자나 `[]` 연산자는 연산을 수행하면서 자동으로 dereferencing 을 해준다. 따라서 역참조를 한 번 더 하게 되면 컴파일 에러가 난다.

```rust
#[derive(Debug, Clone)]
struct Point {
    x: i32,
    y: i32,
}

fn add_one(v: &mut i32) {
    if *v == 2 {
        println!("this is 2!");
    }
    *v += 1;
}

fn add_point(v: &mut Point) {
    v.x = 3;
    v.y = 4;
}

fn add_vec(v: &mut Vec<i32>) {
    v[0] += 100;
}

fn main() {
    let var = &2;
    let mut two = 2;
    let mut point = Point {x:1, y:2};

    add_one(&mut two);
    add_point(&mut point);

    let mut myVec = vec![1, 2, 3];
    for i in &mut myVec {
        *i += 1;
    }
    add_vec(&mut myVec);

    println!("{}", two);
    println!("{}", point.x);
    println!("{:?}", myVec);

    let ref mut test = two;
    add_one(test);
    println!("{}", test);
}
```

- Output
```
this is 2!
3
3
[102, 3, 4]
4
```
### Pattern Matching

Rust 의 가장 강력한 기능 중 하나라고 알려져 있는 pattern matching 을 할 때 `&` 연산자를 써야 할 경우가 있다. 아래 예시를 보자.
```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn main() {
    let coin = Coin::Quarter(UsState::Alabama);
    let value = match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    };

    // try to take ownership
    let var = coin;
}
```

`match coin` 에서 coin 이 가지고 있던 ownership 이 이동했다. 따라서 `let var = coin` 은 coin 이 더이상 ownership 이 없으므로 실행될 수 없다. 이 문제는 `match &coin` 이렇게 바꿔줌으로써 해결될 수 있다.

### `match &x` 와 `|&x|`
`match &x`와 클로저에서 사용하는 `|&x|`가 어떻게 다른지에 대해 알아보자.

#### `match &x`

- `match &x`에서 `&x`는 `x` 변수의 참조를 생성한다. 이렇게 참조를 사용하는 이유는 `match` 문에서 `x`의 값을 비교하면서 `x`의 소유권을 유지하기 위해서인데, 이 경우 패턴 매칭에 사용되는 각 케이스는 `x`의 참조와 일치시켜야 한다.
- 예를 들어, `match &x`는 `x`의 값을 복사하거나 소유권을 이동시키지 않고 `x`에 대해 패턴 매칭을 수행한다. `match &x` 뒤에 오는 패턴들은 모두 참조(`&`)에 맞추어 정의되어야 한다.

#### `|&x|`

- 클로저에서 `|&x|`는 클로저의 매개변수 `x`가 참조로 전달될 때, 이 참조를 자동으로 역참조하여 값을 직접적으로 `x`에 바인딩한다. 이는 패턴 매칭을 사용하여 클로저의 인자로부터 값을 추출하는 과정이다.
- 이 구문은 `x`의 참조를 받고, 그 참조를 역참조하여 값을 복사하여 클로저 내의 `x`에 저장한다. 클로저 내부에서는 `x`를 참조가 아닌 실제 값으로 사용할 수 있게 된다.

### 차이점

- `match &x`는 `x`의 참조를 사용하여 패턴 매칭을 수행하며, 각 케이스는 `x`의 참조를 필요로 함.
- `|&x|`는 클로저가 `x`의 참조를 받아 직접적으로 값을 사용하기 위해 자동 역참조를 수행하는 구문
