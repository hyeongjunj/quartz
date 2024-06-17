## Closure 기본 개념

Rust, python 등 많은 언어에 들어있는 개념이다.
[참고자료]https://tibetsandfox.tistory.com/9\
https://velog.io/@gamjagoon/Rust-%ED%81%B4%EB%A1%9C%EC%A0%80closure

위 글에 따르면, 아래 조건들을 만족시키는 함수는 python 에서 자동으로 클로저로 등록이 된다고 한다. 

1. 어떤 함수의 내부 함수일 것
2. 그 내부 함수가 외부 함수의 변수를 참조할 것
3. 외부 함수가 내부 함수를 리턴할 것

간단하게 클로저가 뭔지 보여주는 예제 코드
```python
def hello(msg):
	message = "Hi, " + msg
	
	def say():
		print(message)

	return say
```

`say()` 함수는 위 조건들을 모두 만족시키고 있으므로 클로져라 볼 수 있다.
그럼 이걸 뭐에다 써먹냐?

```python
f = hello("Fox")
f()
```

이렇게 하면 2번째 라인에서 `"Hi, Fox"` 가 실행된다. 주목할 점은 `hello()` 함수의 호출이 종료되었음에도 hello() 함수 내부의 `message` 변수가 출력되는 것을 확인할 수 있다.

## 러스트에서의 Closure

참고로 러스트 공식 가이드북에서는 클로저 소개 chapter 에서 처음에는 그냥 익명 함수에 대한 것만 늘어놓다가 정작 변수 캡쳐에 대해서는 후반부에 설명해서 맘에 안든다..

아무튼 러스트에서도 클로저의 기본 개념은 동일하다. 아래 예제에서 `equal_to_x` 가 클로져인데 `main` 에서 정의된 변수인 `x` 를 쓰고 있다.

```rust
fn main() { 
	let x = 4;
	let equal_to_x = |z| z == x;
	let y = 4; 
	assert!(equal_to_x(y)); 
}
```

그런데 러스트는 변수에 소유권이라는 개념이 붙는다. 클로저는 외부 함수의 변수를 사용하기 때문에 외부 함수에서 그 변수에 대한 소유권을 어떻게 클로저에 넘겨주느냐를 생각해 봐야 한다.

1. 불변으로 빌려온다. (읽기만 하고 아무 짓도 안한다.)
	`Fn` trait 
	 위 예제 equal_to_x 가 바로 이 케이스
1. 클로저가 그 변수를 바꿀 수 있게 해 준다.
	`FnMut` trait
	 아래 my_closure 에 예시를 들어놓았다.
1. 클로저에게 소유권을 아예 넘겨버릴 것이다.
	`FnOnce` trait

```rust
fn main() {
    let mut x = 10;
    let mut my_closure = |v| {
        x = x+1;
        v + x};

    println!("v: {}", my_closure(3));
    println!("v: {}", my_closure(3));
    println!("v: {}", my_closure(3));
```

클로저가 외부변수를 바꿀 수 있도록 하기 위해서는 일단 그 외부 변수가 당연히 `mut` 이어야 하고, 추가적으로 closure 자체도  `mut` 선언이 필요하다.

## Closure Examples

``` Rust
fn main() {
    let mut x = 10;

    let mut my_closure =  move |v: &mut i32| {
        x = x+1;
        *v = *v+1;
        *v + x};

    let mut _arg = 2;

    my_closure(&mut _arg);
    
    println!("outer x: {}", x);
    println!("v      : {}", _arg);
}
```

>[!question] 저기서 move 를 빼면 어떻게 될까?

>[!question] closure 의 argument 를 아래와 같이 줄 때의 차이점들은? 
	1. `|v|`
		1. v data type 에 Copy trait 이 구현되어 있는 경우
			그냥 카피해버림.
		2. v data type 에 Copy trait 이 구현되어 있지 않는 경우
			소유권 이전. 이후에는 이 변수를 쓸 수 없음
	1. `|&v|`
		immutable reference 로 넘겨짐. read only!
		 주의할 점은 얘네는 패턴 매칭을 사용한다는 것.
	 3. `|v: &i32|` 
	1. `|v: &mut i32|`
		값 변경 가능하다!

## Closure and Trait Example

러스트에서 map 함수를 통해 closure 가 어떻게 이용되는지 좀 더 자세히 살펴보자.
```rust
let numbers = vec![1, 2, 3];
let squares: Vec<_> = numbers.iter().map(|&x| x * x).collect();
```

iterator 의 trait 인 map 의 시그니쳐는 다음과 같다.
```rust
fn map<B, F>(self, f: F) -> Map<Self, F> 
where
    Self: Sized,
    F: FnMut(Self::Item) -> B,
```

여기서 자세히 봐야 할 부분은 where 의 `F: FnMut(Self::Item) -> B` 이 부분인데, F 가 FnMut trait 을 구현해야 함을 나타낸다. 다시 위 예제로 돌아가보면, `let squares: Vec<_> = numbers.iter().map(|&x| x * x).collect();` 여기서 map 안에서 전달되는 클로저는 FnMut trait 이라는 점이다. 

여기서 아니 그럼 아래처럼 Fn 으로 정의된 트레잇은 못쓰는거냐 하고 따질 수 있는데, 결론부터 말하자면 가능하다. FnMut 은 FnOnce 를 상속받고 Fn 은 FnMut 를 상속받는 상속의 관계가 있기 때문에 Fn 트레잇을 구현하는 클로저는 자동적으로 FnMut 과 FnOnce 의 요구사항도 만족하게 되는 것이다.

```Rust
let twice: Vec<_> = numbers.iter().map(|x| x * 2).collect();
```

그러니 아래와 같은 짓을 하려고 하면,
```
let : Vec<_> = numbers.iter().map(move |x| x * 2).collect();
```

?????????????? 되나?