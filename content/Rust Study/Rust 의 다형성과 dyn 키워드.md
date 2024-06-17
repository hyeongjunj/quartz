```Rust
trait Drawable {
    fn draw(&self) -> f32;
}

struct Circle {
    radius: f32,
}

struct Square {
    x: f32,
    y: f32,
}

impl Drawable for Circle {
    fn draw(&self) -> f32 {
        let r = self.radius;
        r*r
    }
}

impl Drawable for Square {
    fn draw(&self) -> f32 {
        let size = self.x * self.y;
        size
    }
}

fn main() {
    let mut shapes: Vec<Box<Drawable>> = vec![];
    shapes.push(Box::new(Circle {radius: 1.25}));
    shapes.push(Box::new(Square {x: 3.0, y: 4.0}));

    for shape in shapes {
        let size = shape.draw();
        println!("size: {}", size);
    }
}
```

이 예시 코드에는 Drawable 이라는 트레잇이 정의되어 있고 Circle 과 Square 를 나타내는 구조체가 선언되어 있다. 각 구조체마다 알맞는 사이즈를 계산해주는 draw 를 구현하였다.

위 코드를 컴파일 해보면, 
```Bash
➜  practice git:(master) ✗ cargo run
   Compiling practice v0.1.0 (/Users/hyeongjun/work/test02/practice)
error[E0782]: trait objects must include the `dyn` keyword
  --> src/main.rs:29:29
   |
29 |     let mut shapes: Vec<Box<Drawable>> = vec![];
   |                             ^^^^^^^^
   |
help: add `dyn` keyword before this trait
   |
29 |     let mut shapes: Vec<Box<dyn Drawable>> = vec![];
   |                             +++

For more information about this error, try `rustc --explain E0782`.
error: could not compile `practice` (bin "practice") due to 1 previous error
```

이런 `dyn` 키워드를 붙이라는 에러가 나온다. 