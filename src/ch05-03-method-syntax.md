## 메서드 구문

*메서드*는 함수와 유사합니다. `fn` 키워드와 이름으로 선언되며, 매개변수와 반환 값을 가질 수 있으며, 메서드가 어디에서 호출될 때 실행되는 코드를 포함합니다. 함수와 달리 메서드는 구조체(또는 [Chapter 6][enums]<!-- ignore -->에서 다룰 
enum 또는 [Chapter 17][trait-objects]<!-- ignore -->에서 다룰 trait 객체)의 맥락에서 정의되며, 첫 번째 매개변수는 항상 `self`로, 메서드가 호출되는 구조체의 인스턴스를 나타냅니다.

### 메서드 정의

`Rectangle` 인스턴스를 매개변수로 받는 `area` 함수를 변경하여 `Rectangle` 구조체에 정의된 `area` 메서드로 바꾸겠습니다. Listing 5-13을 참조하세요.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-13/src/main.rs}}
```

Listing 5-13: `Rectangle` 구조체에 `area` 메서드 정의

`Rectangle`의 맥락 내에서 함수를 정의하려면 `impl` 블록(구현 블록)을 `Rectangle`에 대해 시작합니다. 이 `impl` 블록 내의 모든 내용은 `Rectangle` 유형과 관련이 있습니다. 그런 다음 `impl` 괄호 안에 `area` 함수를 옮기고 첫 번째(이 경우 유일한) 매개변수를 `self`로 변경합니다.
`main`에서 `area` 함수를 호출하고 `rect1`을 인수로 전달했던 경우, `Rectangle` 인스턴스에 `area` 메서드를 호출하기 위해 *메서드 구문*을 사용할 수 있습니다. 메서드 구문은 인스턴스 뒤에 오며, 점(.)을 사용하여 메서드 이름, 괄호(), 그리고 모든 인수를 추가합니다.

`area`의 시그니처에서 `&self`를 사용하는 이유는 `rectangle: &Rectangle`을 사용했던 것과 같습니다. `&self`는 실제로 `self: &Self`의 약자입니다. `impl` 블록 내에서 `Self` 유형은 `impl` 블록이 정의된 유형의 별칭입니다. 메서드는 첫 번째 매개변수로 `self`라는 이름을 가진 `Self` 유형의 매개변수를 가져야 하므로 Rust는 첫 번째 매개변수 위치에서 `self`라는 이름으로 이를 축약합니다. `self`의 약자를 사용할 때는 `&`를 사용하여 메서드가 `Self` 인스턴스를 대여하는지 여부를 나타내야 합니다. 메서드는 `self`를 소유하거나, `self`를 불변으로 대여하거나, `self`를 가변으로 대여할 수 있습니다.

`&self`를 선택한 이유는 `rectangle: &Rectangle`을 사용했던 것과 동일합니다. 즉, 소유권을 가져오지 않고 구조체 내의 데이터만 읽고 싶기 때문입니다. 메서드가 호출된 인스턴스를 변형하는 작업을 수행해야 하는 경우 `&mut self`를 첫 번째 매개변수로 사용합니다.

`self`만을 사용하여 메서드가 인스턴스의 소유권을 가져오는 것은 드물며, 일반적으로 메서드가 `self`를 다른 것으로 변환하고 호출자는 변환 후 원래 인스턴스를 사용하지 못하도록 하려는 경우에 사용됩니다.

메서드를 함수 대신 사용하는 주요 이유는 메서드 구문을 제공하고 `self` 유형을 매개변수의 모든 시그니처에 반복적으로 작성할 필요가 없기 때문입니다. 또한, 유형의 인스턴스와 관련된 모든 작업을 하나의 `impl` 블록에 넣어 미래 사용자가 코드 라이브러리에서 `Rectangle`의 기능을 찾는 데 필요한 시간을 줄입니다.

구조체의 필드와 동일한 이름을 가진 메서드를 정의할 수도 있습니다. 예를 들어, `Rectangle`에 `width`라는 이름의 메서드를 정의할 수 있습니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-06-method-field-interaction/src/main.rs:here}}
```

여기서는 `width` 메서드가 구조체의 필드 값과 동일한지 여부를 나타내는 `true`를 반환하도록 정의했습니다.

인스턴스의 `width` 필드가 0보다 크고 `false`이면 값이
`0`입니다. 동일한 이름의 메서드 내에서 필드를 사용할 수 있습니다. `main`에서 `rect1.width`에 괄호를 붙이면 Rust는 `width` 메서드를 의미한다는 것을 알고 있습니다. 괄호를 사용하지 않으면 Rust는 `width` 필드를 의미한다는 것을 알고 있습니다.

종종, 하지만 항상은 아닙니다. 필드와 동일한 이름의 메서드를 정의할 때, 그 메서드가 필드의 값만 반환하고 다른 작업을 하지 않도록 하려면 이러한 메서드를 *getter*라고 합니다. Rust는 구조체 필드에 대해 다른 언어처럼 자동으로 getter를 구현하지 않습니다. getter는 필드를 private로 만들고 메서드를 public으로 만들 수 있기 때문에 유용합니다. 이를 통해 유형의 public API의 일부로서 해당 필드에 대한 읽기 전용 액세스를 가능하게 할 수 있습니다. [Chapter 7][public]<!-- ignore -->에서 public과 private가 무엇이며 필드 또는 메서드를 public 또는 private로 지정하는 방법에 대해 설명합니다.

> ### `->` 연산자는 어디에 있습니까?
>
> C와 C++에서는 메서드를 호출하는 데 두 가지 다른 연산자가 사용됩니다. 객체에 직접 메서드를 호출하는 경우에는 `.`를 사용하고, 객체의 포인터에 메서드를 호출하고 포인터를 해제해야 하는 경우에는 `->`를 사용합니다. 즉, `object`가 포인터라면 `object->something()`는 `(*object).something()`과 유사합니다.
>
> Rust에는 `->` 연산자와 같은 것이 없습니다. 대신 Rust는 *자동 참조 및 해제*라는 기능이 있습니다.
>
> 메서드 호출은 Rust에서 이러한 동작이 일어나는 몇 가지 장소 중 하나입니다.
>
> 이렇게 작동하는 방법은 다음과 같습니다. `object.something()`으로 메서드를 호출하면 Rust는 자동으로 `&`, `&mut` 또는 `*`를 추가하여 `object`가 메서드의 시그니처와 일치하도록 합니다. 즉, 다음은 동일합니다.
>
> <!-- CAN'T EXTRACT SEE BUG https://github.com/rust-lang/mdBook/issues/1127 -->
> ```rust
> # #[derive(Debug,Copy,Clone)]
> # struct Point {
> #     x: f64,
> #     y: f64,
> # }
> #
> # impl Point {
> #    fn distance(&self, other: &Point) -> f64 {
> #        let x_squared = f64::powi(other.x - self.x, 2);
> #        let y_squared = f64::powi(other.y - self.y, 2);
> #
> #        f64::sqrt(x_squared + y_squared)
> #    }
> # }
> # let p1 = Point { x: 0.0, y: 0.0 };
> # let p2 = Point { x: 5.0, y: 6.5 };
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> 첫 번째 코드가 훨씬 깔끔합니다. 이러한 자동 참조 동작은 메서드가 명확한 수신자를 가지고 있기 때문입니다. 수신자와 메서드 이름을 알면 Rust는 메서드가 읽기 (`&self`), 변경하기 (`&mut self`) 또는 소비하기 (`self`)를 수행하는지 명확하게 파악할 수 있습니다. Rust가 메서드 수신자에 대한 대출을 암묵적으로 만드는 것은 소유권을 실제로 사용하기 쉽게 만드는 데 큰 역할을 합니다.

### 더 많은 매개변수를 가진 메서드

`Rectangle` 구조체에 `can_hold` 메서드를 정의하여 `Rectangle` 인스턴스가 다른 `Rectangle` 인스턴스를 완전히 포함할 수 있는지 여부를 확인하는 방법을 연습해 보겠습니다. 즉, `can_hold` 메서드를 정의한 후 `Listing 5-14`와 같이 프로그램을 작성할 수 있어야 합니다.

Filename: src/main.rs

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-14/src/main.rs}}
```

Listing 5-14: 아직 정의되지 않은 `can_hold` 메서드를 사용

예상 출력은 두 차원의 크기가 모두 작기 때문에 다음과 같습니다.
 `rect2`는 `rect1`의 치수보다 작지만, `rect3`은 `rect1`보다 넓습니다:

```text
rect1이 rect2를 포함할 수 있나요? true
rect1이 rect3를 포함할 수 있나요? false
```

`can_hold`이라는 메서드를 정의하고 싶으므로 `impl Rectangle` 블록 내에 있을 것입니다. 메서드 이름은 `can_hold`이며, 다른 `Rectangle`에 대한 불변 참조를 매개변수로 받습니다. 매개변수의 유형을 알 수 있는 방법은 메서드를 호출하는 코드를 살펴보는 것입니다:
`rect1.can_hold(&rect2)`는 `&rect2`를 전달하며, `Rectangle`의 인스턴스인 `rect2`에 대한 불변 참조입니다. 우리는 `rect2`만 읽기(쓰지 않기 때문에 변수 참조가 필요하지 않으며, `main` 함수가 `rect2`를 다시 사용할 수 있도록 소유권을 유지해야 함)이므로 불변 참조가 적절합니다. `can_hold`의 반환 값은 불리언이며, 구현은 `self`의 너비와 높이가 다른 `Rectangle`의 너비와 높이보다 크거나 같으면 확인합니다. Listing 5-13에서 가져온 `impl` 블록에 새로운 `can_hold` 메서드를 추가하는 Listing 5-15를 보여드리겠습니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-15/src/main.rs:here}}
```

Listing 5-15: `Rectangle`에 `can_hold` 메서드를 구현하는 것. 다른 `Rectangle` 인스턴스를 매개변수로 받습니다

Listing 5-14의 `main` 함수와 함께 이 코드를 실행하면 원하는 출력을 얻습니다. 메서드는 `self` 매개변수 다음에 추가할 수 있는 여러 매개변수를 받을 수 있으며, 이러한 매개변수는 함수의 매개변수와 동일하게 작동합니다.

### 연관 함수

`impl` 블록 내에서 정의된 모든 함수는 *연관 함수*라고 불립니다. 이는 `impl` 뒤에 명시된 유형과 관련되어 있기 때문입니다. `self`가 첫 번째 매개변수로 없는 연관 함수를 정의할 수 있습니다(따라서 메서드가 아닙니다) 왜냐하면 유형의 인스턴스가 필요하지 않기 때문입니다. 이미 `String` 유형에 정의된 `String::from` 함수와 같은 이러한 함수를 사용했습니다.

`self`가 없는 연관 함수는 구조체의 인스턴스를 반환하는 생성자를 위해 자주 사용됩니다. 이러한 함수는 종종 `new`로 지정되지만, `new`는 특별한 이름이 아니며 언어에 내장되어 있지 않습니다. 예를 들어, `square`라는 이름의 연관 함수를 제공하여 한 개의 치수 매개변수를 받고, 두 치수 모두에 같은 값을 사용하여 사각형 `Rectangle`을 만들 수 있도록 할 수 있습니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-03-associated-functions/src/main.rs:here}}
```

함수의 반환 유형과 함수 본문에서 `Self` 키워드는 `impl` 키워드 뒤에 나타나는 유형을 나타내는 별칭입니다. 이 경우 `Rectangle`입니다.

이 연관 함수를 호출하려면 구조체 이름과 `::` 구문을 사용합니다. `let sq = Rectangle::square(3);`는 예입니다. 이 함수는 구조체에 의해 네임스페이스가 정의됩니다. `::` 구문은 연관 함수와 모듈에서 생성된 네임스페이스 모두에 사용됩니다. 모듈에 대해서는 [Chapter 7][modules]에서 자세히 설명합니다.<!-- ignore -->

### 여러 `impl` 블록

각 구조체는 여러 `impl` 블록을 가질 수 있습니다. Listing 5-15는 Listing 5-16과 같습니다. Listing 5-16은 각 메서드를 자체 `impl` 블록에 넣은 것입니다.

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-16/src/main.rs:here}}
```

Listing 5-16: 여러 `impl` 블록을 사용하여 Listing 5-15를 다시 작성합니다

여기서는 이러한 메서드를 여러 `impl` 블록으로 분리하는 이유가 없습니다.
하지만 이것은 유효한 문법입니다. 10장에서 일반적인 유형과 트레이트에 대해 논의할 때 여러 개의 `impl` 블록이 유용한 경우를 살펴보겠습니다.

## 요약

구조체는 사용자 정의 유형을 만들어 자신의 영역에 의미를 부여할 수 있도록 합니다. 구조체를 사용하면 관련된 데이터 조각을 서로 연결하고 각 조각에 이름을 지어 코드를 명확하게 유지할 수 있습니다. `impl` 블록에서 유형과 관련된 함수를 정의할 수 있으며, 메서드는 구조체의 인스턴스가 가지는 동작을 지정하는 연관 함수의 일종입니다.

그러나 구조체만이 사용자 정의 유형을 만드는 유일한 방법은 아닙니다. Rust의 `enum` 기능으로 도구 상자에 또 다른 도구를 추가해 보겠습니다.

[enums]: ch06-00-enums.html
[trait-objects]: ch17-02-trait-objects.md
[public]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
