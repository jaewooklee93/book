## 고급 트레이트

Chapter 10의 [“트레이트: 공통
동작 정의”][traits-defining-shared-behavior]<!-- ignore --> 섹션에서 처음 트레이트를 다루었지만, 더
심층적인 내용은 살펴보지 않았습니다. Rust에 대해 더 많이 알게 되었으므로, 이제
구체적인 내용을 살펴볼 수 있습니다.

### 트레이트 정의에서 플레이스홀더 유형 지정: 연관된 유형

*연관된 유형*은 트레이트와 유형 플레이스홀더를 연결하여 트레이트
메서드 정의가 이 플레이스홀더 유형을 서명에 사용할 수 있도록 합니다.
트레이트를 구현하는 사람은 특정 구현에 사용될 구체적인 유형을 지정합니다.
이렇게 하면 트레이트가 특정 유형이 무엇인지 알지 못해도 트레이트를
정의할 수 있습니다.

이 책의 나머지 부분에서 설명한 기능과 마찬가지로, 이 장에서 설명한 대부분의
고급 기능은 드물게 필요합니다. 연관된 유형은 중간 정도로 사용됩니다.
즉, 책의 나머지 부분에서 설명한 기능보다 더 자주 사용되지만, 이 장에서
설명한 다른 기능들보다 덜 자주 사용됩니다.

`Iterator` 트레이트와 같은 연관된 유형을 가진 트레이트의 한 예는 표준 라이브러리에서 제공됩니다.
연관된 유형은 `Item`이라고 명명되며, `Iterator` 트레이트를 구현하는 유형이
반복하는 값의 유형을 나타냅니다. `Iterator` 트레이트의 정의는 19-12번
리스트에 나와 있습니다.

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-12/src/lib.rs}}
```

19-12번 리스트: `Iterator` 트레이트의 정의
(연관된 유형 `Item`을 포함)


`Item` 유형은 플레이스홀더이며, `next` 메서드의 정의는 `Option<Self::Item>` 유형을 반환한다는 것을 보여줍니다.
`Iterator` 트레이트의 구현자는 `Item`에 대한 구체적인 유형을 지정하고, `next` 메서드는 그 구체적인 유형의 `Option`을 반환합니다.

연관된 유형은 유형 매개변수와 유사한 개념처럼 보일 수 있습니다. 유형 매개변수는 함수를 정의할 때 특정 유형을 처리할 수 있는지 알 수 없도록 합니다.
두 개념의 차이를 이해하기 위해 `Counter`라는 유형에 대한 `Iterator` 트레이트의 구현을 살펴보겠습니다. 이 구현은 `Item` 유형이 `u32`임을 지정합니다.

Filename: src/lib.rs

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-22-iterator-on-counter/src/lib.rs:ch19}}
```

이 구문은 유형 매개변수를 사용하여 `Iterator` 트레이트를 정의하는 것과 유사해 보입니다. 그렇다면 19-13번 리스트와 같이 유형 매개변수를 사용하여 `Iterator` 트레이트를 정의하지 않을 수 있을까요?

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-13/src/lib.rs}}
```

19-13번 리스트: 유형 매개변수를 사용하여 `Iterator` 트레이트를 정의하는 가상 예시

하지만 유형 매개변수를 사용하면, 각 구현에서 유형을 지정해야 합니다. `Iterator<String>`을 `Counter`에 대해 구현하거나 다른 유형을 구현할 수 있기 때문입니다. 즉, `Iterator` 트레이트에 유형 매개변수가 있으면 `Counter`에 대해 여러 개의 `Iterator` 구현이 있을 수 있습니다. `next` 메서드를 `Counter`에서 호출할 때는 어떤 `Iterator` 구현을 사용할지 유형 지정을 통해 알려야 합니다.

연관된 유형을 사용하면 유형 지정이 필요하지 않습니다. 19-12번 리스트와 같이 연관된 유형을 사용하여 트레이트를 정의하면, `Item` 유형에 대한 구체적인 유형을 한 번만 지정할 수 있습니다. `Counter`에서 `next` 메서드를 호출할 때는 어떤 유형을 사용할지 지정할 필요가 없습니다.

연관된 유형은 또한 트레이트 계약의 일부가 됩니다. 트레이트를 구현하는 사람은 트레이트의 플레이스홀더 유형에 대응하는 유형을 제공해야 합니다. 연관된 유형은 종종 사용되는 방식을 설명하는 이름을 가지며, API 문서에서 연관된 유형을 설명하는 것은 좋은 관행입니다.

### 기본 유형 매개변수 및 연산자 오버로딩

일반적인 타입 매개변수를 사용할 때, 일반적인 타입에 대한 기본 구체적인 타입을 지정할 수 있습니다. 이를 통해 트레이트를 구현하는 사람들이 기본 타입이 적용되는 경우 구체적인 타입을 지정할 필요가 없습니다. 기본 타입을 지정하려면 `<PlaceholderType=ConcreteType>` 문법을 사용하여 일반적인 타입을 선언합니다.

이러한 기술이 유용하게 사용되는 좋은 예는 *연산자 오버로딩*입니다. 연산자 오버로딩은 특정 상황에서 연산자(예: `+`)의 동작을 사용자 정의하는 것을 의미합니다.

Rust는 새로운 연산자를 만들거나 임의의 연산자를 오버로드할 수 없습니다. 그러나 `std::ops`에 나열된 연산자와 해당하는 트레이트를 오버로드하여 연산자와 관련된 트레이트를 구현함으로써 오버로드할 수 있습니다. 예를 들어, 19-14번 목록에서 `+` 연산자를 두 `Point` 인스턴스를 더하는 연산자로 오버로드하는 방법을 보여줍니다. 이를 위해 `Point` 구조체에 `Add` 트레이트를 구현합니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-14/src/main.rs}}
```

Listing 19-14: `Point` 인스턴스에 대한 `+` 연산자 오버로드를 위한 `Add` 트레이트 구현

`add` 메서드는 두 `Point` 인스턴스의 `x` 값과 두 `Point` 인스턴스의 `y` 값을 더하여 새로운 `Point`를 생성합니다. `Add` 트레이트는 `Output`이라는 이름의 연관된 타입을 가지고 있으며, `add` 메서드에서 반환되는 타입을 결정합니다.

이 코드에서 기본 일반적인 타입은 `Add` 트레이트 내에 있습니다. 정의는 다음과 같습니다.

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

이 코드는 일반적으로 익숙한 코드입니다. 메서드와 연관된 타입을 가진 트레이트입니다. 새로운 부분은 `Rhs=Self`입니다. 이 문법은 *기본 타입 매개변수*라고 합니다. `Rhs` 일반적인 타입 매개변수(즉, "오른쪽 측면"의 약자)는 `add` 메서드의 `rhs` 매개변수의 타입을 정의합니다. `Add` 트레이트를 구현할 때 `Rhs`에 대한 구체적인 타입을 지정하지 않으면 `Rhs`의 타입은 `Self`로 기본값이 설정됩니다. `Self`는 구현 중인 `Add` 트레이트의 타입입니다.

`Point`에 대해 `Add`를 구현할 때는 `Rhs`에 기본값을 사용했습니다. 왜냐하면 두 `Point` 인스턴스를 더하고 싶었기 때문입니다. `Rhs` 타입을 기본값 대신 사용하여 사용자 정의하는 `Add` 트레이트 구현의 예를 살펴보겠습니다.

`Millimeters`와 `Meters`라는 두 개의 구조체가 있습니다. 이 구조체는 다른 단위의 값을 저장합니다. 이러한 기존 타입을 다른 구조체로 감싸는 것을 *새로운 타입 패턴*이라고 합니다. 이 패턴에 대한 자세한 내용은 [“Using the Newtype Pattern to Implement External Traits on External Types”][newtype]<!-- ignore --> 섹션에서 설명합니다. `Millimeters`와 `Meters`를 더하고 `Add` 트레이트를 구현하여 `Millimeters`를 `Meters`로 변환하는 방법을 살펴보겠습니다. `Add` 트레이트를 `Millimeters`에 구현하여 `Meters`를 `Rhs`로 지정하면 다음과 같습니다.

Filename: src/lib.rs

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-15/src/lib.rs}}
```

Listing 19-15: `Millimeters`에 대한 `Add` 트레이트 구현으로 `Millimeters`를 `Meters`로 더하기

`Millimeters`와 `Meters`를 더하려면 `impl Add<Meters>`를 지정하여 `Rhs` 타입 매개변수를 지정합니다. 기본값 대신 사용자 정의된 값을 사용합니다.

기본 타입 매개변수를 두 가지 주요 방법으로 사용합니다.

* 기존 코드를 변경하지 않고 타입을 확장하는 데 사용합니다.
* 대부분의 사용자가 필요로 하지 않는 특정 경우에 사용자 정의를 허용하는 데 사용합니다.

표준 라이브러리의 `Add` 트레이트는 두 번째 목적의 예입니다. 일반적으로 유사한 타입을 더하지만, `Add` 트레이트는 그 이상의 가능성을 제공합니다. `Add` 트레이트 정의에서 기본 타입 매개변수를 사용하면 대부분의 경우 추가 매개변수를 지정할 필요가 없어 코드를 간소화할 수 있습니다. 즉, 일부 구현 코드가 필요하지 않아 트레이트를 사용하기 쉬워집니다.



첫 번째 목적은 두 번째와 유사하지만 역순입니다. 기존 형식에 유형 매개변수를 추가하려면 기존 구현 코드를 깨뜨리지 않고 형식의 기능을 확장할 수 있도록 기본값을 제공할 수 있습니다.

### 완전한 자격명칭을 사용한 풀이: 동일한 이름의 메서드를 호출하는 경우

Rust에서는 다른 형식의 메서드와 동일한 이름을 가진 형식을 가질 수 있으며, 한 유형에 두 형식 모두를 구현하는 것도 허용됩니다. 또한 유형에 직접 형식의 메서드와 동일한 이름을 가진 메서드를 구현하는 것도 가능합니다.

동일한 이름을 가진 메서드를 호출할 때는 Rust에 어떤 메서드를 사용하고 싶은지 알려야 합니다. 19-16번 목록에서 두 개의 형식 `Pilot` 와 `Wizard` 를 정의했는데, 두 형식 모두 `fly` 메서드를 가지고 있습니다. 이제 `Human` 유형에 두 형식 모두를 구현했고, `Human` 유형에 직접 `fly` 메서드가 구현되어 있습니다. 각 `fly` 메서드는 다른 작업을 수행합니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-16/src/main.rs:here}}
```

19-16번 목록: 두 개의 형식이 `fly` 메서드를 가지고 있으며 `Human` 유형에 구현되었고, `Human` 유형에 직접 `fly` 메서드가 구현되어 있습니다

`Human` 인스턴스에 `fly` 를 호출하면 컴파일러는 유형에 직접 구현된 메서드를 호출하는 것을 기본으로 합니다. 19-17번 목록을 참조하세요.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-17/src/main.rs:here}}
```

19-17번 목록: `Human` 인스턴스에 `fly` 를 호출하는 경우

이 코드를 실행하면 `*waving arms furiously*` 가 출력되어 Rust가 `Human` 유형에 직접 구현된 `fly` 메서드를 호출했다는 것을 보여줍니다.

`Pilot` 형식 또는 `Wizard` 형식의 `fly` 메서드를 호출하려면 형식 이름을 명시적으로 사용하여 어떤 `fly` 메서드를 호출하고 싶은지 명확히 해야 합니다. 19-18번 목록이 이러한 문법을 보여줍니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-18/src/main.rs:here}}
```

19-18번 목록: 어떤 형식의 `fly` 메서드를 호출하고 싶은지 명시적으로 지정하는 경우

형식 이름을 메서드 이름 앞에 명시하면 Rust가 어떤 `fly` 메서드를 호출해야 하는지 명확해집니다. `Human::fly(&person)` 을 사용할 수도 있지만, 유형을 명확히 구분할 필요가 없는 경우에는 좀 더 길게 작성해야 합니다.

이 코드를 실행하면 다음과 같은 출력이 나타납니다.

```console
{{#include ../listings/ch19-advanced-features/listing-19-18/output.txt}}
```

`fly` 메서드가 `self` 매개변수를 받기 때문에 두 개의 유형이 하나의 형식을 구현하는 경우 Rust는 `self` 유형에 따라 어떤 형식의 구현을 사용해야 하는지 알 수 있습니다.

그러나 `self` 매개변수가 없는 연관 함수는 Rust가 유형을 알 수 없습니다. 여러 유형이나 형식이 동일한 함수 이름을 정의하는 경우 Rust는 완전한 자격명칭을 사용해야 합니다. 예를 들어 19-19번 목록에서 동물 보호소를 위한 형식을 만들고 모든 강아지 새끼에게 이름을 `Spot` 으로 지정합니다. `Animal` 형식은 연관 함수 `baby_name` 을 가지고 있습니다. `Animal` 형식은 `Dog` 구조체에 구현되며, `Dog` 구조체에도 직접 연관 함수 `baby_name` 이 정의되어 있습니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings
listings/ch19-advanced-features/listing-19-19/src/main.rs}}
```

19-19번 목록: 연관 함수를 가진 형식과 직접 연관 함수를 정의한 구조체

특정 이름을 가진 연관 함수를 구현하는 타입

우리는 `Dog`에서 정의된 동일한 이름을 가진 연관 함수를 구현하여 강아지의 이름을 모두 Spot으로 지정하는 코드를 구현합니다. `Dog` 타입은 또한 모든 동물이 가지는 특징을 설명하는 `Animal` 트레이트를 구현합니다. 강아지 새끼는 강아지라고 불리며, 이는 `Animal` 트레이트에서 `Dog`에 대한 `baby_name` 함수의 연관 함수 구현에서 표현됩니다. 

`main`에서 `Dog::baby_name` 함수를 호출하면 `Dog`에 직접 정의된 연관 함수가 호출됩니다. 이 코드는 다음과 같은 출력을 생성합니다.

```console
{{#include ../listings/ch19-advanced-features/listing-19-19/output.txt}}
```

이 출력은 원하는 결과가 아닙니다. `Animal` 트레이트에서 구현한 `baby_name` 함수를 호출하려고 하므로 코드가 `강아지 새끼는 강아지라고 합니다.`를 출력해야 합니다. 19-18번 목록에서 사용한 트레이트 이름을 지정하는 기술은 여기서는 도움이 되지 않습니다. 19-20번 목록의 `main`을 변경하면 컴파일 오류가 발생합니다.

Filename: src/main.rs

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-20/src/main.rs:here}}
```

Listing 19-20: `Animal` 트레이트의 `baby_name` 함수를 호출하려고 하지만 Rust은 어떤 구현을 사용해야 할지 알 수 없습니다.

`Animal::baby_name`에는 `self` 매개변수가 없으며, `Animal` 트레이트를 구현하는 다른 유형도 있을 수 있기 때문에 Rust은 어떤 `Animal::baby_name` 구현을 원하는지 알 수 없습니다. 다음과 같은 컴파일 오류를 받게 됩니다.

```console
{{#include ../listings/ch19-advanced-features/listing-19-20/output.txt}}
```

`Animal` 트레이트의 `baby_name` 함수를 `Dog`에 대한 구현으로 사용하려면 Rust에게 명확하게 알려야 합니다. 즉, 다른 유형에 대한 `Animal` 구현이 아닌 `Dog`에 대한 구현을 사용해야 합니다. 이를 위해서는 완전한 이름을 사용해야 합니다. 19-21번 목록은 완전한 이름을 사용하는 방법을 보여줍니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-21/src/main.rs:here}}
```

Listing 19-21: `Dog`에 대한 `Animal` 트레이트의 `baby_name` 함수를 호출하는 데 완전한 이름을 사용합니다.

이 코드는 원하는 결과를 출력합니다.

```console
{{#include ../listings/ch19-advanced-features/listing-19-21/output.txt}}
```

일반적으로 완전한 이름은 다음과 같이 정의됩니다.

```rust,ignore
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

`receiver`가 없는 연관 함수의 경우에는 `receiver`가 없을 뿐, 나머지 인수만 있을 것입니다. 완전한 이름을 사용할 수 있는 모든 곳에서 함수나 메서드를 호출할 수 있습니다. 그러나 Rust이 프로그램의 다른 정보에서 알 수 있는 부분은 생략할 수 있습니다. Rust이 알 수 있는 부분을 생략하고, Rust이 어떤 구현을 사용해야 할지 알 수 없는 경우에만 더 자세한 이름을 사용해야 합니다.

### 상위 트레이트를 사용하여 다른 트레이트의 기능을 요구하는 방법

때때로 트레이트 정의가 다른 트레이트에 의존하는 경우가 있습니다. 특정 유형이 첫 번째 트레이트를 구현하려면, 그 유형이 두 번째 트레이트를 구현해야 한다는 것을 요구할 수 있습니다. 이렇게 하면 트레이트 정의가 두 번째 트레이트의 연관 항목을 사용할 수 있습니다. 트레이트 정의가 의존하는 트레이트를 *상위 트레이트*라고 합니다.

예를 들어, `OutlinePrint` 트레이트를 만들고 싶다고 가정해 보겠습니다. 이 트레이트는 `Animal` 트레이트의 기능을 사용해야 합니다. `Animal` 트레이트를 구현하는 유형이 `OutlinePrint` 트레이트를 구현하려면, `Animal` 트레이트의 연관 함수를 사용할 수 있어야 합니다. 이를 위해 `OutlinePrint` 트레이트를 `Animal` 트레이트의 상위 트레이트로 정의할 수 있습니다.

```rust
trait Animal { 
    fn baby_name(&self) -> String; 
}

trait OutlinePrint: Animal { 
    fn outline_print(&self) -> String; 
}

struct Dog; 

impl Animal for Dog { 
    fn baby_name(&self) -> String { 
        String::from("puppy") 
    } 
}

impl OutlinePrint for Dog { 
    fn outline_print(&self) -> String { 
        String::from("Dog outline") 
    } 
}

fn main() { 
    let dog = Dog; 
    println!("A baby dog is called a {}", dog.outline_print()); 
}
```

이 코드는 `OutlinePrint` 트레이트가 `Animal` 트레이트의 기능을 사용하여 `outline_print` 메서드를 정의하는 방법을 보여줍니다. `OutlinePrint` 트레이트는 `Animal` 트레이트를 상위 트레이트로 정의하여 `baby_name` 연관 함수를 사용할 수 있습니다. 이는 트레이트를 정의할 때 다른 트레이트의 기능을 요구하는 방법을 보여줍니다.
 `outline_print` 메서드를 정의하여 주어진 값을 별로 둘러싸여 출력하는 방식으로 포맷합니다. 즉, `Display` 트레이트를 구현하여 `(x, y)`로 표현되는 `Point` 구조체가 주어졌을 때, `outline_print`를 `x`가 1이고 `y`가 3인 `Point` 인스턴스에 호출하면 다음과 같은 출력을 가져야 합니다.

```text
**********
*        *
* (1, 3) *
*        *
**********
```

`outline_print` 메서드 구현에서 `Display` 트레이트의 기능을 사용하려면 `OutlinePrint` 트레이트가 `Display`를 구현하는 유형에만 작동하도록 지정해야 합니다. 트레이트 정의에서 `OutlinePrint: Display`를 지정하여 이를 수행할 수 있습니다. 이 기술은 트레이트에 트레이트 제약을 추가하는 것과 유사합니다. 19-22번 목록은 `OutlinePrint` 트레이트의 구현을 보여줍니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-22/src/main.rs:here}}
```

Listing 19-22: `OutlinePrint` 트레이트를 구현하는데 `Display` 트레이트의 기능이 필요합니다

`OutlinePrint`가 `Display` 트레이트를 요구하기 때문에 `to_string` 함수를 사용할 수 있습니다. `to_string` 함수는 `Display` 트레이트를 구현하는 모든 유형에 대해 자동으로 구현됩니다. `Display` 트레이트 이름 뒤에 콜론과 `Display` 트레이트를 추가하지 않고 `to_string` 함수를 사용하려고 하면 `&Self` 유형의 현재 범위에서 `to_string`이라는 이름의 메서드가 없다는 오류가 발생합니다.

`Display` 트레이트를 구현하지 않은 유형인 `Point` 구조체에 `OutlinePrint`를 구현하려고 시도해 보겠습니다.

Filename: src/main.rs

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-02-impl-outlineprint-for-point/src/main.rs:here}}
```

`Display`가 필요하지만 구현되지 않았다는 오류가 발생합니다.

```console
{{#include ../listings/ch19-advanced-features/no-listing-02-impl-outlineprint-for-point/output.txt}}
```

`Point`에 `Display`를 구현하여 `OutlinePrint`가 요구하는 제약 조건을 충족하면 `OutlinePrint` 트레이트를 `Point`에 구현할 수 있습니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-03-impl-display-for-point/src/main.rs:here}}
```

그러면 `Point` 인스턴스에 `outline_print`를 호출하여 별로 둘러싸여 출력할 수 있습니다.

### 새로운 유형 패턴을 사용하여 외부 유형에 외부 트레이트를 구현하는 방법

Chapter 10의 [“Type에 트레이트를 구현하는 방법”][implementing-a-trait-on-a-type]<!-- ignore --> 섹션에서 언급했듯이, 트레이트 또는 유형이 우리의 crate에 국지적이거나, 우리는 유형에 트레이트를 직접 구현할 수 없습니다. 이 제약을 우회하는 방법은 *새로운 유형 패턴*을 사용하는 것입니다. 새로운 유형 패턴은 튜플 구조체를 사용하여 새로운 유형을 만들고, 이 튜플 구조체는 우리가 트레이트를 구현하려는 유형을 하나의 필드로 감싸는 것을 의미합니다. 그러면 래퍼 유형은 우리의 crate에 국지적이므로 우리는 래퍼 유형에 트레이트를 구현할 수 있습니다. *Newtype*는 Haskell 프로그래밍 언어에서 유래된 용어입니다. 이 패턴을 사용하는 경우 런타임 성능 손실이 없으며, 컴파일 시간에 래퍼 유형은 생략됩니다.

예를 들어, 우리가 `Vec<T>`에 `Display` 트레이트를 구현하고 싶다고 가정해 보겠습니다. 하지만 `Display` 트레이트와 `Vec<T>` 유형은 우리의 crate 외부에서 정의되어 있기 때문에 트레이트를 직접 구현할 수 없습니다. `Wrapper` 구조체를 만들어 `Display` 트레이트를 구현할 수 있습니다.
 `Vec<T>` 인스턴스를 갖는 타입이라면, `Display`를 `Wrapper`에 구현하고 `Vec<T>` 값을 사용할 수 있습니다. Listing 19-23과 같이 보여줍니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-23/src/main.rs}}
```

Listing 19-23: `Vec<String>` 주변에 `Wrapper` 타입을 생성하여 `Display`를 구현

`Display` 구현은 `self.0`을 사용하여 내부 `Vec<T>`에 액세스합니다. `Wrapper`는 튜플 구조체이고 `Vec<T>`는 튜플에서 인덱스 0에 있는 항목이기 때문입니다. 그런 다음 `Wrapper`에 대한 `Display` 트레이트의 기능을 사용할 수 있습니다.

이 기술을 사용하는 단점은 `Wrapper`가 새로운 타입이기 때문에 갖고 있는 값의 메서드를 가지지 않기 때문입니다. `Wrapper`에 `Vec<T>`의 모든 메서드를 직접 구현해야 하며, 이러한 메서드가 `self.0`에 위임되어 `Wrapper`를 `Vec<T>`와 동일하게 처리할 수 있도록 해야 합니다. 내부 타입의 모든 메서드를 가진 새로운 타입을 원한다면, `Wrapper`에 `Deref` 트레이트(Chapter 15의 "Smart Pointers Like Regular References with the `Deref` Trait" 섹션에서 논의됨)를 구현하여 내부 타입을 반환하는 것이 해결책입니다. `Wrapper` 타입이 내부 타입의 모든 메서드를 가지지 않기를 원한다면(예: `Wrapper` 타입의 동작을 제한하려면) 원하는 메서드만 수동으로 구현해야 합니다.

이 새로운 타입 패턴은 트레이트가 관여하지 않더라도 유용합니다. 집중을 바꾸고 Rust의 타입 시스템과의 상호 작용에 대한 몇 가지 고급 방법을 살펴보겠습니다.

[newtype]: ch19-03-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types
[implementing-a-trait-on-a-type]:
ch10-02-traits.html#implementing-a-trait-on-a-type
[traits-defining-shared-behavior]:
ch10-02-traits.html#traits-defining-shared-behavior
[smart-pointer-deref]: ch15-02-deref.html#treating-smart-pointers-like-regular-references-with-the-deref-trait
[tuple-structs]: ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types
