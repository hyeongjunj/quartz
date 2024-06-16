## Primitive Datatypes
### Slices vs. Arrays

- **Arrays** (`[T; N]`) have a fixed size and store their elements inline.
- **Slices** (`&[T]`) are dynamically-sized view into a sequence of `T` and cannot exist independently; they must always be borrowed from an array or another slice.

### Tuple
```rust
let cat = ("String1", 3.5);
let (a,b) = cat;
```

indexing 은 `cat.0`, `cat.1` 이런 식으로 함. 배열이랑 다르다.

---
## Vector and Iterator

```rust
fn vec_loop(mut v: Vec<i32>) -> Vec<i32> {
    for element in v.iter_mut() {
        *element *= 2;
    }
    v
}

fn vec_map(v: &Vec<i32>) -> Vec<i32> {
    v.iter().map(|element| {
        *element * 2
    }).collect()
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_vec_loop() {
        let v: Vec<i32> = (1..).filter(|x| x % 2 == 0).take(5).collect();
        let ans = vec_loop(v.clone());

        assert_eq!(ans, v.iter().map(|x| x * 2).collect::<Vec<i32>>());
    }

    #[test]
    fn test_vec_map() {
        let v: Vec<i32> = (1..).filter(|x| x % 2 == 0).take(5).collect();
        let ans = vec_map(&v);

        assert_eq!(ans, v.iter().map(|x| x * 2).collect::<Vec<i32>>());
    }
}
```

```rust
    v.iter().map(|&element| {
        element * 2
    }).collect()
```

이렇게 해도 됨. Vector element 의 참조자와 &element 둘이 pattern matching 을 하는 형태라고 보면 될 듯 하다. (맞나?)

 `iter()` 는 원소들의 참조자를 반환하고 변경 불가하지만, `iter_mut()` 은 `&mut T` 를 반환해줘서 변경 가능한 것을 볼 수 있다.

Rust에서 `Vec<T>` 타입의 벡터를 `clone()`할 때, 참조에 대해 `clone()`을 호출하는 방법과 직접 벡터에 `clone()`을 호출하는 방법 모두 가능하고, 이 두 방식의 동작은 동일하다.(아래 코드 참고) 
이는 `clone()` 메서드가 `Clone` 트레잇을 구현하는 모든 타입에 대해 제공되며, `&T` (T의 참조)에 대해서도 `Clone` 트레잇이 자동으로 구현되기 때문이다. (자동으로 구현된다는 말이 뭐지?)

```rust
fn main() {
    let v = vec![1, 2, 3];
    let v_cloned = v.clone(); // v를 직접 클론
    let v_cloned = (&v).clone(); // v의 참조를 클론
    println!("Original: {:?}, Cloned: {:?}", v, v_cloned);
}
```

---
## Enum

```rust
#[derive(Debug)]
enum Message {
    Move {x: u32, y: u32},
    Echo(String),
    ChangeColor(i32, i32, i32),
    Quit,
}

impl Message {
    fn call(&self) {
        println!("{:?}", self);
    }
}

fn main() {
    let messages = [
        Message::Move { x: 10, y: 30 },
        Message::Echo(String::from("hello world")),
        Message::ChangeColor(200, 255, 255),
        Message::Quit,
    ];

    for message in messages {
        message.call();
    }
}
```

> [!question] Question
>  `for message in messages` 이거랑  `for message in &messages` 이거 둘다 `call()` 메소드를 호출 가능한 이유는?

---
## String

String 과 as_str() (a.k.a. 문자열 슬라이스 &str)
일단 러스트에서 문자열 리터럴을 이미 `&str`  타입이다.
`let var = "String...";` 
이럴 때, `var` 변수의 타입은 `&str` 가 된다. 

### Deref Coercion
```rust
fn main() {
    let word = String::from("green"); // Try not changing this line :)
    if is_a_color_word(&word) {
...
}

fn is_a_color_word(attempt: &str) -> bool {
    attempt == "green" || attempt == "blue" || attempt == "red"
}
```

위 코드에서처럼 Rust 는 특이하게 String 타입을 &str 타입의 매개변수로 넘길 수 있는데 Rust의 **자동 참조 및 해제 (deref coercion)** 기능 덕분이다. 

Rust에서 `String` 타입은 `Deref` 트레잇을 구현한다. 이 트레잇의 구현은 `String`을 그 내용을 가리키는 `&str` 슬라이스로 변환하는 `deref` 메소드를 제공한다. `Deref` 트레잇을 구현하는 모든 타입은 필요할 때 컴파일러에 의해 자동으로 참조될 수 있다. 
위 코드를 예시로 보면, `main` 함수에서 `&word` 표현은 `&String` 타입이지만, `is_a_color_word` 함수에 전달될 때, Rust 컴파일러가 `&String`을 `&str`로 자동 변환한다. 

> [!question] Question
> Deref trait 이 구현되어 있으면 저렇게 자동으로 & 참조 연산을 할 때 Deref 트레잇이 호출되는가?

### String and Char

```rust
fn main() {
	let s = String::from("hello");
	let last_char = s.chars().last().unwrap(); // 'o'를 반환     
	println!("Last character: {}", last_char); }`
```
- `chars()` 메서드는 주어진 `String`에서 유니코드 스칼라 값들을 순회할 수 있는 반복자(iterator)를 반환한다. 이 반복자는 문자열의 각 `char`를 순차적으로 반환한다.
- 유니코드 스칼라 값은 UTF-8 인코딩된 문자열에서 유효한 문자 단위로, 각각은 하나 이상의 바이트로 구성된다.

- `last()` 메서드는 반복자의 마지막 요소를 반환한다. 이 메서드는 `Option` 타입을 반환하는데, 이는 반복자가 비어있을 경우 `None`을 반환하기 때문.
- 반복자가 비어있지 않은 경우, 즉 문자열에 최소 한 개의 문자가 있는 경우에는 `Some(char)`가 반환된다.

- `unwrap()` 메서드는 `Option` 타입에서 값을 추출한다. 만약 `Option`이 `Some`이면 그 내부의 값을 반환하고, `None`인 경우에는 패닉(프로그램 에러로 인한 종료)을 발생시킨다.

## HashMap

```rust
    let mut scores: HashMap<String, Team> = HashMap::new();

    for r in results.lines() {
        let v: Vec<&str> = r.split(',').collect();
        let team_1_name = v[0].to_string();
        let team_1_score: u8 = v[2].parse().unwrap();
        let team_2_name = v[1].to_string();
        let team_2_score: u8 = v[3].parse().unwrap();

        match scores.get_mut(&team_1_name) {
            Some(&mut v) => { // ERROR!!!!!!!!!!!!!!!!!!!!
                v.goals_scored += team_1_score;
                v.goals_conceded += team_2_score;
            },
            None => {
                scores.insert(team_1_name, Team{goals_scored: team_1_score,
                                                goals_conceded: team_2_score});
            },
        }
        match scores.get_mut(&team_2_name) {
            Some(v) => { // This is Ok.
                v.goals_scored += team_2_score;
                v.goals_conceded += team_1_score;
            },
            None => {
                scores.insert(team_2_name, Team{goals_scored: team_2_score,
                                                goals_conceded: team_1_score});
            },
        }
    }
```

>[!question] Question
> Why `Some(&mut v)` not working? I thought I can be pattern matched.



