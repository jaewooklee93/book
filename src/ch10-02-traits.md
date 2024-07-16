## 추상적인 동작 정의: 트레이트

*트레이트*는 특정 유형이 가지는 기능과 다른 유형과 공유할 수 있는 기능을 정의합니다. 트레이트를 사용하여 추상적인 방식으로 공통된 동작을 정의할 수 있습니다. *트레이트 경계*를 사용하여 일반 유형이 특정 동작을 가진 유형이 될 수 있도록 지정할 수 있습니다.

> 참고: 트레이트는 다른 언어에서 *인터페이스*라고 불리는 기능과 유사하지만, 몇 가지 차이점이 있습니다.

### 트레이트 정의

유형의 동작은 해당 유형에 대해 호출할 수 있는 메서드로 구성됩니다. 다른 유형이 동일한 동작을 공유하는 경우, 모든 유형에서 동일한 메서드를 호출할 수 있습니다. 트레이트 정의는 특정 목적을 달성하기 위해 필요한 동작 세트를 정의하는 데 사용되는 메서드 서명을 함께 그룹화하는 방법입니다.

예를 들어, 다양한 종류와 양의 텍스트를 저장하는 여러 구조체가 있다고 가정해 보겠습니다. 특정 위치에 기고된 뉴스 기사를 저장하는 `NewsArticle` 구조체와 최대 280자를 포함할 수 있는 `Tweet` 구조체는 새로운 트윗, 리트윗 또는 다른 트윗에 대한 답장인지 여부를 나타내는 메타데이터를 포함합니다.

`aggregator`라는 이름의 미디어 애그리게이터 라이브러리 크레이트에서 `NewsArticle` 또는 `Tweet` 인스턴스에 저장된 데이터의 요약을 표시할 수 있는 도구를 만들고 싶습니다. 이를 위해 각 유형에서 요약을 요청하고 `summarize` 메서드를 호출하여 요약을 가져오려고 합니다. 10-12번 목록은 이 동작을 표현하는 공개 `Summary` 트레이트의 정의를 보여줍니다.

Filename: src/lib.rs

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-12/src/lib.rs}}
```

Listing 10-12: `Summary` 트레이트의 정의는 `summarize` 메서드를 제공합니다

여기서 `trait` 키워드를 사용하여 트레이트를 선언하고 트레이트의 이름은 `Summary`입니다. 또한 이 트레이트를 `pub`로 선언하여 이 크레이트에 의존하는 크레이트에서도 이 트레이트를 사용할 수 있도록 합니다. 괄호 안에는 트레이트를 구현하는 유형의 동작을 설명하는 메서드 서명을 선언합니다. 이 경우 `fn summarize(&self) -> String`입니다.

메서드 서명 뒤에는 괄호 안에 구현을 제공하는 대신 세미콜론을 사용합니다. 이 트레이트를 구현하는 유형은 `summarize` 메서드의 몸체에 대한 고유한 동작을 제공해야 합니다. 컴파일러는 `Summary` 트레이트를 가진 모든 유형이 정확히 이 서명으로 `summarize` 메서드를 정의해야 함을 강제합니다.

트레이트에는 몸체에 여러 메서드가 있을 수 있습니다. 메서드 서명은 한 줄에 나열되며 각 줄은 세미콜론으로 끝납니다.

### 유형에 트레이트 구현

이제 `Summary` 트레이트의 메서드 서명을 정의했으므로, 미디어 애그리게이터에서 `NewsArticle`과 `Tweet` 유형에 트레이트를 구현할 수 있습니다. 10-13번 목록은 헤드라인, 저자, 위치를 사용하여 `summarize`를 구현하는 `NewsArticle` 구조체에 대한 `Summary` 트레이트의 구현을 보여줍니다. `Tweet` 구조체의 경우, 트윗 내용이 이미 280자로 제한된다고 가정하여 사용자 이름을 따라 트윗 전체 내용을 `summarize`로 정의합니다.

Filename: src/lib.rs

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-13/src/lib.rs:here}}
```

Listing 10-13: `NewsArticle`과 `Tweet` 유형에 `Summary` 트레이트를 구현합니다

유형에 트레이트를 구현하는 것은 일반 메서드를 구현하는 것과 유사합니다. 차이점은 `impl` 뒤에 구현하려는 트레이트 이름을 넣고 `for` 키워드를 사용하여 구현하려는 유형의 이름을 지정하는 것입니다. `impl` 블록 내에서 트레이트 정의가 정의한 메서드 서명을 넣습니다. 각 서명 뒤에 세미콜론 대신 괄호를 사용하여 메서드 몸체를 채웁니다. 이 몸체는 특정 유형에 대한 트레이트 메서드의 구체적인 동작을 나타냅니다.

이제 라이브러리가 `NewsArticle`과 `Tweet`에 `Summary` 트레이트를 구현했으므로, 크레이트의 사용자는 `NewsArticle`과 `Tweet` 인스턴스에 트레이트 메서드를 호출하여 요약을 가져올 수 있습니다.
 `NewsArticle`과 `Tweet`를 동일한 방식으로 일반 메서드를 호출합니다. 유일한 차이점은 사용자가 트레이트와 유형을 모두 스코프에 가져와야 한다는 것입니다. `aggregator` 라이브러리 크레이트를 사용하는 이진 크레이트의 예시는 다음과 같습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-01-calling-trait-method/src/main.rs}}
```

이 코드는 `1 new tweet: horse_ebooks: of course, as you probably already know, people`를 출력합니다.

`aggregator` 크레이트에 의존하는 다른 크레이트도 `Summary` 트레이트를 스코프에 가져와 자신의 유형에 대해 `Summary`를 구현할 수 있습니다. 주의해야 할 한 가지 제한 사항은 트레이트 또는 유형, 또는 그 두 가지가 모두 우리 크레이트에 국지적이어야만 특정 유형에 트레이트를 구현할 수 있다는 것입니다. 예를 들어, `aggregator` 크레이트의 기능으로서 `Tweet`와 같은 사용자 정의 유형에 `Display`와 같은 표준 라이브러리 트레이트를 구현할 수 있습니다. `aggregator` 크레이트에서 `Summary` 트레이트와 `Vec<T>`를 구현할 수도 있습니다. 

그러나 외부 트레이트를 외부 유형에 구현할 수 없습니다. 예를 들어, `aggregator` 크레이트 내에서 `Display` 트레이트를 `Vec<T>`에 구현할 수 없습니다. `Display`와 `Vec<T>`는 모두 표준 라이브러리에 정의되어 `aggregator` 크레이트에 국지적이지 않기 때문입니다. 이 제한 사항은 *일관성*이라는 특성의 일부이며, *유아 규칙*이라고도 불리며, 부모 유형이 존재하지 않기 때문에 이렇게 불립니다. 이 규칙은 다른 사람의 코드가 우리 코드를 망가뜨리지 못하고 그 반대도 되지 않도록 보장합니다. 규칙이 없으면 두 개의 크레이트가 동일한 유형에 대해 동일한 트레이트를 구현할 수 있으며, Rust은 어떤 구현을 사용해야 할지 알 수 없습니다.

### 기본 구현

때때로 트레이트의 모든 메서드에 대해 구현을 요구하지 않고 일부 또는 모든 메서드에 대해 기본 동작을 제공하는 것이 유용합니다. 그런 다음 특정 유형에 트레이트를 구현할 때 각 메서드의 기본 동작을 유지하거나 재정의할 수 있습니다.

10-14번 목록에서 `Summary` 트레이트의 `summarize` 메서드에 기본 문자열을 지정하여 10-12번 목록에서만 메서드 서명을 정의했던 것과는 다릅니다.

Filename: src/lib.rs

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-14/src/lib.rs:here}}
```

Listing 10-14: 기본 구현을 가진 `Summary` 트레이트를 정의하는 것

`NewsArticle` 인스턴스를 요약하기 위해 기본 구현을 사용하려면 `impl Summary for NewsArticle {}`와 같이 빈 `impl` 블록을 지정합니다.

기본 구현을 만들었더라도 `Tweet`에 대한 `Summary` 트레이트의 구현은 10-13번 목록에서 변경할 필요가 없습니다. 이유는 기본 구현을 재정의하는 문법이 기본 구현이 없는 트레이트 메서드를 구현하는 문법과 동일하기 때문입니다.

기본 구현은 다른 트레이트 메서드를 호출할 수 있습니다. 다른 트레이트 메서드가 기본 구현이 없더라도 그렇습니다. 이렇게 하면 트레이트가 많은 유용한 기능을 제공하고 구현자는 그 중 일부만 지정해야 합니다. 예를 들어, `summarize_author` 메서드가 필수적인 구현을 가진 기본 구현을 가진 `Summary` 트레이트를 정의할 수 있습니다. `summarize` 메서드는 `summarize_author` 메서드를 호출하는 기본 구현을 가질 수 있습니다.

```rust,noplayground
## 추상형 인터페이스 (Traits)

{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:here}}
```

이 버전의 `Summary`를 사용하려면 `summarize_author`를 정의해야 합니다.

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:impl}}
```

`summarize_author`를 정의한 후 `Tweet` 구조체의 인스턴스에 `summarize`를 호출할 수 있습니다. 그리고 우리가 정의한 `summarize_author`의 정의를 사용하여 `summarize`의 기본 구현이 호출됩니다. `summarize_author`를 구현했기 때문에 `Summary` 추상형 인터페이스는 `summarize` 메서드의 동작을 제공했으며, 추가 코드를 작성할 필요가 없습니다. 다음은 그 모습입니다.

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/main.rs:here}}
```

이 코드는 `1 new tweet: (Read more from @horse_ebooks...)`을 출력합니다.

같은 메서드의 기본 구현에서 재정의된 구현을 호출하는 것은 불가능합니다.

### 추상형 인터페이스를 매개변수로 사용하기

이제 추상형 인터페이스를 정의하고 구현하는 방법을 알고 있다면, 추상형 인터페이스를 사용하여 `Summary` 추상형 인터페이스를 구현한 `NewsArticle` 및 `Tweet` 유형에 대한 다양한 유형을 받는 함수를 정의하는 방법을 탐색해 볼 수 있습니다. Listing 10-13에서 구현한 `Summary` 추상형 인터페이스를 사용하여 `notify` 함수를 정의하여 `item` 매개변수의 `summarize` 메서드를 호출하는 함수를 정의합니다. 이때 `impl Trait` 문법을 사용하여 `Summary` 추상형 인터페이스를 구현하는 유형을 나타냅니다.

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-04-traits-as-parameters/src/lib.rs:here}}
```

`item` 매개변수에 대해 구체적인 유형 대신 `impl Trait` 키워드와 추상형 인터페이스 이름을 지정합니다. 이 매개변수는 `Summary` 추상형 인터페이스를 구현하는 모든 유형을 받습니다. `notify` 함수의 몸체에서 `Summary` 추상형 인터페이스에서 정의된 `summarize`와 같은 `item`에 대한 모든 메서드를 호출할 수 있습니다. `notify` 함수를 호출하고 `Summary` 추상형 인터페이스를 구현한 `NewsArticle` 또는 `Tweet`의 인스턴스를 전달할 수 있습니다. `String` 또는 `i32`와 같은 다른 유형으로 함수를 호출하려면 코드가 컴파일되지 않습니다. 이러한 유형은 `Summary` 추상형 인터페이스를 구현하지 않기 때문입니다.

<!-- Old headings. Do not remove or links may break. -->
<a id=\"fixing-the-largest-function-with-trait-bounds\"></a>

#### 추상형 인터페이스 경계 문법

`impl Trait` 문법은 간단한 경우에 유용하지만, 더 복잡한 경우를 표현하는 데 사용되는 더 긴 형태인 *추상형 인터페이스 경계*라는 더 긴 형태의 문법입니다. 다음과 같이 보입니다.

```rust,ignore
pub fn notify<T: Summary>(item: &T) {
    println!(\"Breaking news! {}\", item.summarize());
}
```

이 더 긴 형태는 이전 섹션의 예제와 동일하지만 더 상세합니다. 일반 유형 매개변수 선언 후 콜론과 각도 괄호 안에 추상형 인터페이스 경계를 표시합니다.

`impl Trait` 문법은 간단한 경우에 편리하고 코드를 더 간결하게 만드는 반면, 더 긴 추상형 인터페이스 경계 문법은 다른 경우에 더 복잡한 것을 표현할 수 있습니다. 예를 들어, `Summary` 추상형 인터페이스를 구현하는 두 개의 매개변수를 가질 수 있습니다. `impl Trait` 문법을 사용하면 다음과 같습니다.

```rust,ignore
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

이 함수는 `item1`과 `item2`가 다른 유형일 수 있도록 합니다. (두 유형이 모두 `Summary` 추상형 인터페이스를 구현하는 경우에만 가능합니다.) 두 매개변수가 같은 유형이어야 한다면 추상형 인터페이스 경계를 사용해야 합니다.

```rust,ignore
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

`item1`과 `item2` 매개변수의 유형으로 지정된 일반 유형 `T`는 함수를 제한하여 값의 구체적인 유형이 동일해야 함을 의미합니다.


## 추상적 데이터 유형: 트레이트

트레이트는 Rust에서 인터페이스와 유사한 개념입니다. 트레이트는 타입이 구현해야 하는 메서드나 방법을 정의합니다. 

**트레이트 정의**

트레이트를 정의하려면 `trait` 키워드를 사용합니다. 예를 들어, `Summary` 트레이트를 정의하는 코드는 다음과 같습니다.

```rust
trait Summary {
    fn summarize(&self) -> String;
}
```

이 트레이트는 `summarize` 메서드를 정의합니다. 이 메서드는 `&self`를 인자로 받고 `String`을 반환해야 합니다.

**트레이트 구현**

타입이 트레이트를 구현하려면 `impl` 키워드를 사용합니다. 예를 들어, `NewsArticle` 타입이 `Summary` 트레이트를 구현하는 코드는 다음과 같습니다.

```rust
struct NewsArticle {
    headline: String,
    location: String,
    author: String,
    content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}
```

이 코드는 `NewsArticle` 구조체가 `Summary` 트레이트를 구현한다는 것을 나타냅니다. `summarize` 메서드는 `NewsArticle` 구조체의 필드를 사용하여 요약 문자열을 생성합니다.

**트레이트 제약**

트레이트 제약은 일반적인 타입 파라미터에 적용되는 트레이트를 지정하는 것입니다. 예를 들어, `notify` 함수는 `Summary` 트레이트를 구현하는 타입을 인자로 받습니다.

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

이 코드는 `notify` 함수가 `Summary` 트레이트를 구현하는 타입을 인자로 받을 수 있다는 것을 나타냅니다.

**여러 트레이트 제약**

여러 트레이트 제약을 지정하려면 `+` 연산자를 사용합니다. 예를 들어, `notify` 함수는 `Summary`와 `Display` 트레이트를 구현하는 타입을 인자로 받습니다.

```rust
pub fn notify<T: Summary + Display>(item: &T) {
    println!("{}: {}", item.summarize(), item);
}
```

**`where` 절**

`where` 절은 트레이트 제약을 함수 정의에서 분리하여 읽기 쉽게 만듭니다.

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
    // ...
}
```

**트레이트를 사용하여 메서드를 조건적으로 구현**

트레이트 제약을 사용하여 일반적인 타입 파라미터에 대한 메서드를 조건적으로 구현할 수 있습니다.

```rust
struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Pair { x, y }
    }
}
```

이 코드는 `Pair` 구조체에 `new` 메서드를 정의합니다. 이 메서드는 `T` 타입 파라미터를 사용하여 `Pair` 구조체의 인스턴스를 생성합니다.

**결론**

트레이트는 Rust에서 타입의 행동을 정의하는 강력한 도구입니다. 트레이트를 사용하면 코드를 재사용하고 유지 관리하기 쉽게 만들 수 있습니다.


는 `impl` 블록의 타입에 대한 별칭입니다. 이 경우 `Pair<T>`입니다. 하지만 다음 `impl` 블록에서는 `Pair<T>`가 내부 타입 `T`가 비교 가능 (`PartialOrd`) 트레이트와 출력 가능 (`Display`) 트레이트를 구현한다면 `cmp_display` 메서드만 구현합니다.

Filename: src/lib.rs

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-15/src/lib.rs}}
```

Listing 10-15: 트레이트 경계 조건에 따라 일반적인 타입의 메서드를 조건적으로 구현

또한, 다른 트레이트를 구현하는 타입에 대해 트레이트를 조건적으로 구현할 수 있습니다. 트레이트 경계 조건을 충족하는 트레이트의 구현은 *일반 구현*이라고 하며 Rust 표준 라이브러리에서 널리 사용됩니다. 예를 들어, 표준 라이브러리는 `Display` 트레이트를 구현하는 모든 타입에 대해 `ToString` 트레이트를 구현합니다. 표준 라이브러리의 `impl` 블록은 다음과 같은 코드와 유사합니다.

```rust,ignore
impl<T: Display> ToString for T {
    // --snip--
}
```

표준 라이브러리가 이러한 일반 구현을 가지고 있기 때문에 `Display` 트레이트를 구현하는 모든 타입에 대해 `ToString` 트레이트에서 정의된 `to_string` 메서드를 호출할 수 있습니다. 예를 들어, `Display`를 구현하기 때문에 정수를 해당하는 `String` 값으로 변환할 수 있습니다.

```rust
let s = 3.to_string();
```

일반 구현은 트레이트의 설명서에서 "구현자" 섹션에 표시됩니다.

트레이트와 트레이트 경계는 일반적인 타입 매개변수를 사용하여 코드를 중복 없이 작성할 수 있도록 하면서 동시에 컴파일러에게 일반적인 타입이 특정 동작을 가져야 한다고 지정할 수 있도록 합니다. 컴파일러는 트레이트 경계 정보를 사용하여 코드에서 사용되는 모든 구체적인 타입이 올바른 동작을 제공하는지 확인할 수 있습니다. 동적으로 타입이 정의되는 언어에서는 메서드를 정의하지 않은 타입에 대해 메서드를 호출하면 실행 중에 오류가 발생합니다. 하지만 Rust는 이러한 오류를 컴파일 시간으로 이동하여 코드가 실행되기 전에 문제를 해결하도록 강요합니다. 또한, 컴파일 시간에 이미 확인했기 때문에 실행 시간에 동작을 확인하는 코드를 작성할 필요가 없습니다. 이렇게 하면 성능이 향상되지만 일반적인 유연성을 잃지 않습니다.

[using-trait-objects-that-allow-for-values-of-different-types]: ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[methods]: ch05-03-method-syntax.html#defining-methods
