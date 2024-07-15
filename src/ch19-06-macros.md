## 매크로

이 책에서 `println!`과 같은 매크로를 사용해 왔지만, 매크로가 무엇이며 어떻게 작동하는지 완전히 탐구하지 않았습니다. *매크로*라는 용어는 Rust에서의 기능 가족을 나타냅니다. `macro_rules!`를 사용하는 *선언적* 매크로와 세 가지 유형의 *순차적* 매크로입니다.

* `#[derive]` 매크로를 사용하여 구조체와 열거형에 `derive` 속성이 사용될 때 추가되는 코드를 지정합니다.
* 어떤 항목에도 사용할 수 있는 사용자 정의 속성 매크로
* 함수 호출과 유사하게 보이지만 인자로 지정된 토큰에 작동하는 함수형 매크로

각각에 대해 차례로 설명하겠지만, 먼저 함수가 이미 있음에도 불구하고 매크로가 필요한 이유를 살펴보겠습니다.

### 매크로와 함수의 차이

기본적으로 매크로는 다른 코드를 작성하는 코드를 작성하는 방법입니다. 이것이 *메타 프로그래밍*이라고 합니다. 부록 C에서는 `derive` 속성에 대해 설명합니다. 이 속성은 여러 트레이트의 구현을 자동으로 생성합니다. 이 책에서 `println!`와 `vec!` 매크로를 사용해 왔습니다. 이러한 모든 매크로는 더 많은 코드로 확장되어 수동으로 작성한 코드보다 많습니다.

메타 프로그래밍은 작성해야 하는 코드의 양을 줄이고 유지 관리하기 쉽게 하는 데 유용합니다. 이는 함수의 역할 중 하나이기도 합니다. 그러나 매크로는 함수가 가지지 않는 추가적인 힘을 가지고 있습니다.

함수 서식은 함수가 가지는 인자의 수와 유형을 선언해야 합니다. 반면 매크로는 변수 개수의 인자를 받을 수 있습니다. `println!(\"hello\")`와 같이 하나의 인자를 사용하거나 `println!(\"hello {}\", name)`와 같이 두 개의 인자를 사용할 수 있습니다. 또한 매크로는 컴파일러가 코드의 의미를 해석하기 전에 확장되므로 매크로는 특정 유형에 대해 트레이트를 구현할 수 있습니다. 함수는 실행 시간에 호출되기 때문에 트레이트는 컴파일 시간에 구현되어야 하기 때문에 할 수 없습니다.

함수 대신 매크로를 구현하는 단점은 매크로 정의가 함수 정의보다 복잡하다는 것입니다. 왜냐하면 Rust 코드를 작성하는 Rust 코드를 작성하고 있기 때문입니다. 이러한 간접성으로 인해 매크로 정의는 일반적으로 함수 정의보다 읽기, 이해하고 유지 관리하기 어렵습니다.

매크로와 함수의 또 다른 중요한 차이점은 매크로를 정의하거나 파일에서 호출하기 전에 범위 내에 가져와야 한다는 것입니다. 반면 함수는 어디에서든 정의하고 어디에서든 호출할 수 있습니다.

### `macro_rules!`를 사용한 선언적 매크로: 일반 메타 프로그래밍

Rust에서 가장 널리 사용되는 매크로 유형은 *선언적 매크로*입니다. 이것들은 때때로 \u201c매크로 by example\u201d, \u201c`macro_rules!` 매크로\u201d 또는 단순히 \u201c매크로\u201d라고도 합니다. 핵심은 선언적 매크로가 Rust `match` 표현식과 유사한 것을 작성할 수 있다는 것입니다. 6장에서 설명했듯이 `match` 표현식은 값을 비교하는 제어 구조이며, 표현식의 결과 값을 패턴과 비교한 다음 일치하는 패턴에 연결된 코드를 실행합니다. 매크로 또한 패턴과 비교하는 값을 받습니다. 이 값은 매크로에 전달된 문자열 Rust 코드입니다. 패턴은 해당 문자열 Rust 코드의 구조와 비교되며, 일치하는 패턴에 연결된 코드는 매크로에 전달된 코드를 대체합니다. 이 모든 작업은 컴파일 중에 수행됩니다.

매크로를 정의하려면 `macro_rules!` 구문을 사용합니다. `macro_rules!`를 사용하는 방법을 이해하기 위해 `vec!` 매크로의 정의를 살펴보겠습니다. 8장에서는 `vec!` 매크로를 사용하여 특정 값을 가진 새로운 벡터를 만드는 방법을 설명했습니다. 예를 들어, 다음 매크로는 세 개의 정수를 포함하는 새로운 벡터를 만듭니다.

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

`vec!` 매크로를 사용하여 두 개의 정수 또는 다섯 개의 문자열 슬라이스를 포함하는 벡터를 만들 수도 있습니다. 함수를 사용하면 이 작업을 수행할 수 없습니다. 왜냐하면 인자의 수와 유형을 미리 알 수 없기 때문입니다.

19-28번 목록은 `vec!` 매크로의 간략하게 정의된 버전을 보여줍니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-28/src/lib.rs}}
```

<span class=\"caption\">19-28번 목록: `vec!` 매크로 정의의 간략한 버전</span>

> 주의: 표준 라이브러리에서 `vec!` 매크로의 실제 정의는 더 복잡합니다.
```rust,ignore
> 메모리 할당을 미리 정확한 양으로 할당하는 코드를 포함합니다. 이 코드는 예제를 간소화하기 위해 여기에서는 포함하지 않습니다.

`#[macro_export]` 어노테이션은 해당 매크로가 정의된 crate가 스코프로 가져올 때 사용 가능하도록 지정합니다. 이 어노테이션이 없으면 매크로를 스코프로 가져올 수 없습니다.

그런 다음 `macro_rules!`로 매크로 정의를 시작하고 매크로 이름(물론 `!` 기호 없이)을 씁니다. 이 경우 `vec`라는 이름은 매크로 정의의 몸체를 나타내는 중괄호로 뒤따릅니다.

`vec!` 몸체의 구조는 `match` 표현의 구조와 유사합니다. 여기서는 `( $( $x:expr ),* )` 패턴과 `=>` 그리고 이 패턴과 연결된 코드 블록이 있는 하나의 팔이 있습니다. 패턴이 일치하면 관련된 코드 블록이 생성됩니다. 이 매크로에서 유일한 패턴이므로 일치하는 유일한 방법이 있습니다. 다른 패턴은 오류를 발생시킵니다. 더 복잡한 매크로는 여러 개의 팔을 가질 것입니다.

매크로 정의에서 유효한 패턴 문법은 제18장에서 다룬 패턴 문법과 다릅니다. 매크로 패턴은 Rust 코드 구조와 일치하는 반면, 값과 일치하는 패턴을 사용합니다. 19-28번 목록의 패턴 조각이 무엇을 의미하는지 살펴보겠습니다. 전체 매크로 패턴 문법은 [Rust 참조][ref]를 참조하십시오.

첫째, 괄호를 사용하여 전체 패턴을 묶습니다. `$`를 사용하여 매크로 시스템에서 패턴과 일치하는 Rust 코드를 담을 변수를 선언합니다. `$` 기호는 이것이 정규 Rust 변수가 아니라 매크로 변수임을 명확히 합니다. 다음은 패턴 내의 코드와 일치하는 값을 캡처하는 괄호 세트입니다. `$()` 내부에는 `$x:expr`가 있으며, 이는 모든 Rust 표현식과 일치하고 표현식에 `$x`라는 이름을 부여합니다.

`$()` 다음의 쉼표는 문자 쉼표 구분자가 패턴 내의 코드 뒤에 옵션으로 나타날 수 있음을 나타냅니다. `*`는 앞서 있는 것의 0개 또는 여러 개를 일치하는 패턴임을 나타냅니다.

`vec![1, 2, 3];`와 같이 매크로를 호출하면 `$x` 패턴은 `1`, `2`, `3` 세 개의 표현식과 세 번 일치합니다.

이제 이 팔과 연결된 코드 패턴을 살펴보겠습니다. `$()*` 내부의 `temp_vec.push()`는 패턴이 `$()`에서 0개 또는 여러 번 일치하는 경우마다 생성됩니다. `$x`는 일치하는 각 표현식으로 대체됩니다. `vec![1, 2, 3];`와 같이 매크로를 호출하면 이 매크로 호출을 대체하는 생성된 코드는 다음과 같습니다.

```rust,ignore
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

매크로를 정의하여 임의의 수의 인자를 받고, 지정된 요소를 포함하는 벡터를 생성하는 코드를 생성했습니다.

매크로를 작성하는 방법에 대한 자세한 내용은 온라인 문서나 [Daniel Keep가 시작하여 Lukas Wirth가 계속 작성한 `The Little Book of Rust Macros`](https://www.rust-lang.org/learn/the-little-book-of-rust-macros)와 같은 다른 리소스를 참조하십시오.

### 속성에서 코드를 생성하는 절차적 매크로

두 번째 형태의 매크로는 *절차적 매크로*로, 함수와 유사하게 작동합니다. 절차적 매크로는 코드를 입력으로 받고, 해당 코드를 처리하여 출력으로 코드를 생성합니다. 선언적 매크로가 패턴과 일치하는 코드를 다른 코드로 대체하는 반면, 절차적 매크로는 코드를 입력으로 받고, 해당 코드를 처리하여 출력으로 코드를 생성합니다. 세 가지 유형의 절차적 매크로는 사용자 정의 derive, 속성과 함수와 유사하며, 모두 유사한 방식으로 작동합니다.

절차적 매크로를 만들 때 정의는 특수한 crate 유형을 가진 자신의 crate에 있어야 합니다. 이는 복잡한 기술적 이유로, 앞으로는 해결할 예정입니다. 19-29번 목록에서 `some_attribute`는 특정 매크로 유형을 사용하는 데 사용되는 빈자리입니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,ignore
use proc_macro;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

## 19.6. 매크로 정의하기

<span class=\"caption\">Listing 19-29: 절차적 매크로 정의 예시</span>

절차적 매크로를 정의하는 함수는 `TokenStream`을 입력으로 받고 `TokenStream`을 출력으로 반환합니다. `TokenStream` 유형은 Rust에 포함된 `proc_macro` crate에서 정의되며, 토큰의 순서를 나타냅니다. 이것이 매크로의 핵심입니다. 매크로가 작동하는 소스 코드가 입력 `TokenStream`을 구성하고, 매크로가 생성하는 코드가 출력 `TokenStream`입니다. 함수에는 또한 어떤 종류의 절차적 매크로를 만들고 있는지 지정하는 속성이 있습니다. 같은 crate에서 여러 종류의 절차적 매크로를 가질 수 있습니다.

다음은 절차적 매크로의 종류를 살펴보겠습니다. 먼저 사용자 정의 `derive` 매크로를 살펴보고, 다른 형태의 차이점을 설명하겠습니다.

### 사용자 정의 `derive` 매크로 작성하기

`hello_macro`라는 이름의 crate를 만들고 `HelloMacro`라는 트레이트와 `hello_macro`라는 이름의 연관 함수를 정의합니다. 사용자가 자신의 유형에 대해 `#[derive(HelloMacro)]`를 추가하여 `HelloMacro` 트레이트의 기본 구현을 얻도록 하기 위해 `HelloMacro` 트레이트를 구현하도록 사용자에게 요구하지 않습니다. 기본 구현은 `Hello, Macro! My name is TypeName!`를 출력합니다. 여기서 `TypeName`은 이 트레이트가 정의된 유형의 이름입니다. 즉, 다른 프로그래머가 Listing 19-30과 같이 코드를 작성할 수 있도록 하는 crate를 작성할 것입니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-30/src/main.rs}}
```

<span class=\"caption\">Listing 19-30: 우리 crate를 사용하는 사용자가 매크로를 사용할 수 있도록 할 수 있는 코드</span>

이 코드는 `Hello, Macro! My name is Pancakes!`를 출력합니다. 작업이 완료되면.
첫 번째 단계는 `hello_macro`이라는 이름의 새로운 라이브러리 crate를 만드는 것입니다.

```console
$ cargo new hello_macro --lib
```

다음으로 `HelloMacro` 트레이트와 연관 함수를 정의합니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-20-impl-hellomacro-for-pancakes/hello_macro/src/lib.rs}}
```

이제 `hello_macro` 트레이트를 구현하여 원하는 기능을 달성할 수 있습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-20-impl-hellomacro-for-pancakes/pancakes/src/main.rs}}
```

그러나 각 유형을 `hello_macro`와 함께 사용하려면 트레이트를 구현하는 데 필요한 구현 블록을 직접 작성해야 합니다. 이 작업을 수행하지 않도록 하려고 합니다.

또한, 현재 Rust는 반사 기능을 제공하지 않으므로 컴파일 시점에 유형의 이름을 찾을 수 없습니다. 컴파일 시점에 코드를 생성하는 매크로가 필요합니다.

다음 단계는 절차적 매크로를 정의하는 것입니다. 현재까지 절차적 매크로는 자신의 crate에 있어야 합니다. 이 제약 조건은 언젠가 해제될 수 있습니다. crate와 매크로 crate를 구조화하는 관습은 다음과 같습니다. `foo`라는 이름의 crate의 경우, 사용자 정의 `derive` 절차적 매크로 crate는 `foo_derive`로 불립니다. `hello_macro` crate 내부에 `hello_macro_derive`라는 이름의 새로운 crate를 시작해 보겠습니다.

```console
$ cargo new hello_macro_derive --lib
```

두 crate는 밀접하게 관련되어 있으므로 `hello_macro` crate의 디렉토리 내에 절차적 매크로 crate를 만듭니다. `hello_macro`에서 트레이트 정의를 변경하면 `hello_macro_derive`에서 매크로의 구현도 변경해야 합니다. 두 crate는 별도로 게시되어야 하며, 이러한 crate를 사용하는 프로그래머는 두 crate 모두를 의존성으로 추가하고 두 crate 모두를 범위 내에 가져와야 합니다. 대신, 매크로를 정의하는 crate가 `hello_macro` crate의 일부로 포함될 수 있습니다. 이렇게 하면 두 crate를 관리하는 것이 더 쉬워집니다. 그러나 이렇게 하면 매크로를 사용하는 프로그래머가 `hello_macro` crate의 모든 부분을 가져와야 합니다. 이는 필요하지 않은 부분을 가져오는 것을 의미할 수 있습니다. 이러한 이유로, 절차적 매크로를 정의하는 crate를 별도로 유지하는 것이 일반적입니다.

`hello_macro_derive` crate의 `src/lib.rs` 파일을 다음과 같이 수정합니다.

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-20-impl-hellomacro-for-pancakes/hello_macro_derive/src/lib.rs}}
```

이 매크로는 `#[derive(HelloMacro)]` 어트리뷰트를 사용하여 트레이트를 구현하는 유형을 인식하고, 해당 유형의 이름을 사용하여 `hello_macro` 함수를 생성합니다. 이제 `hello_macro` crate를 사용하는 프로그래머는 `hello_macro` 함수를 사용할 수 있습니다. 이 함수는 `Hello, Macro! My name is TypeName!`를 출력합니다. 이제 `hello_macro` crate를 사용하여 `HelloMacro` 트레이트를 구현하는 유형을 정의할 수 있습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-20-impl-hellomacro-for-pancakes/pancakes/src/main.rs}}
```

이 코드는 `Hello, Macro! My name is Pancakes!`를 출력합니다. 이제 `hello_macro` crate를 사용하여 `HelloMacro` 트레이트를 구현하는 유형을 정의할 수 있습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-20-impl-hellomacro-for-pancakes/pancakes/src/main.rs}}
```

이 코드는 `Hello, Macro! My name is Pancakes!`를 출력합니다. 이제 `hello_macro` crate를 사용하여 `HelloMacro` 트레이트를 구현하는 유형을 정의할 수 있습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-20-impl-hellomacro-for-pancakes/pancakes/src/main.rs}}
```

이 코드는 `Hello, Macro! My name is Pancakes!`를 출력합니다. 이제 `hello_macro` crate를 사용하여 `HelloMacro` 트레이트를 구현하는 유형을 정의할 수 있습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-20-impl-hellomacro-for-pancakes/pancakes/src/main.rs}}
```

이 코드는 `Hello, Macro! My name is Pancakes!`를 출력합니다. 이제 `hello_macro` crate를 사용하여 `HelloMacro` 트레이트를 구현하는 유형을 정의할 수 있습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-20-impl-hellomacro-for-pancakes/pancakes/src/main.rs}}
```

이 코드는 `Hello, Macro! My name is Pancakes!`를 출력합니다. 이제 `hello_macro` `hello_macro` crate는 `hello_macro_derive`를 의존성으로 사용하고 절차적 맥락 코드를 다시 내보냅니다. 그러나 우리가 프로젝트를 구조화한 방식은 프로그래머가 `derive` 기능을 원하지 않더라도 `hello_macro`를 사용할 수 있게 합니다.

우리는 `hello_macro_derive` crate를 절차적 맥락 crate로 선언해야 합니다. `syn`과 `quote` crate의 기능도 필요하므로, 잠시 볼 예정인 `syn`과 `quote` crate를 의존성으로 추가해야 합니다. `hello_macro_derive`의 *Cargo.toml* 파일에 다음 내용을 추가하세요.

<span class=\"filename\">Filename: hello_macro_derive/Cargo.toml</span>

```toml
{{#include ../listings/ch19-advanced-features/listing-19-31/hello_macro/hello_macro_derive/Cargo.toml:6:12}}
```

`hello_macro_derive` crate의 *src/lib.rs* 파일에 Listing 19-31의 코드를 넣어 절차적 맥락을 정의하기 시작합니다. `impl_hello_macro` 함수의 정의가 없기 때문에 이 코드는 컴파일되지 않습니다.

<span class=\"filename\">Filename: hello_macro_derive/src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-31/hello_macro/hello_macro_derive/src/lib.rs}}
```

<span class=\"caption\">Listing 19-31: 대부분의 절차적 맥락 crate에서 Rust 코드를 처리하기 위해 필요한 코드</span>

`hello_macro_derive` 함수와 `impl_hello_macro` 함수로 코드를 분리했습니다. `hello_macro_derive` 함수는 `TokenStream`을 분석하고 `impl_hello_macro` 함수는 문법 트리의 변환을 담당합니다. 이렇게 하면 절차적 맥락을 작성하는 것이 더 편리해집니다. `hello_macro_derive` 함수의 코드는 거의 모든 절차적 맥락 crate에서 동일합니다. `impl_hello_macro` 함수의 코드는 절차적 맥락의 목적에 따라 다릅니다.

`proc_macro`, [`syn`], [`quote`] 세 개의 새로운 crate를 도입했습니다. `proc_macro` crate는 Rust에 포함되어 있으므로 *Cargo.toml* 파일에 추가할 필요가 없습니다. `proc_macro` crate는 컴파일러의 API로서 우리가 코드에서 Rust 코드를 읽고 조작할 수 있도록 합니다.

`syn` crate는 Rust 코드를 문자열에서 분석하여 우리가 작업할 수 있는 데이터 구조로 변환합니다. `quote` crate는 `syn` 데이터 구조를 다시 Rust 코드로 변환합니다. 이러한 crate는 우리가 처리하고 싶은 모든 종류의 Rust 코드를 분석하는 데 매우 간단하게 만들어줍니다. Rust 코드의 완전한 파서를 작성하는 것은 간단한 작업이 아닙니다.

`hello_macro_derive` 함수는 사용자가 라이브러리에 `#[derive(HelloMacro)]`를 유형에 지정할 때 호출됩니다. 이는 `hello_macro_derive` 함수를 `proc_macro_derive`로 표시하고 `HelloMacro`라는 이름을 지정했기 때문입니다. 이는 대부분의 절차적 맥락이 따르는 관례입니다.

`hello_macro_derive` 함수는 먼저 `input`을 `TokenStream`에서 데이터 구조로 변환합니다. 이때 `syn`이 사용됩니다. `syn`의 `parse` 함수는 `TokenStream`을 받아 분석된 Rust 코드를 나타내는 `DeriveInput` 구조를 반환합니다. Listing 19-32는 Listing 19-30의 코드에서 맥락의 속성을 가진 `struct Pancakes;` 문자열을 분석했을 때 얻는 `DeriveInput` 구조의 관련 부분을 보여줍니다.

```rust,ignore
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

<span class=\"caption\">Listing 19-32: Listing 19-30의 코드에서 맥락의 속성을 가진 `struct Pancakes;` 문자열을 분석했을 때 얻는 `DeriveInput` 인스턴스</span>

이 구조체의 필드는 우리가 파싱한 Rust 코드가 `Pancakes`라는 이름의 단위 구조체임을 보여줍니다. 이 구조체에는 Rust 코드를 설명하는 더 많은 필드가 있습니다. 자세한 내용은 [`syn` 문서의 `DeriveInput`][syn-docs]을 참조하십시오.

곧 `impl_hello_macro` 함수를 정의할 것입니다. 이 함수에서 우리가 포함하고자 하는 새로운 Rust 코드를 빌드할 것입니다. 하지만 그 전에 우리가 만들어내는 `derive` 매크로의 출력이 또한 `TokenStream`이라는 점에 유의하십시오. 반환된 `TokenStream`은 우리의 crate 사용자가 작성한 코드에 추가되므로, 그들이 자신의 crate를 컴파일할 때, 수정된 `TokenStream`에서 제공하는 추가 기능을 얻게 됩니다.

`unwrap`을 사용하여 `hello_macro_derive` 함수가 `syn::parse` 함수 호출에 실패하면 panic하도록 하는 것을 알았을 수 있습니다. 우리의 절차 매크로가 오류 발생 시 panic하도록 하는 것은 `proc_macro_derive` 함수가 절차 매크로 API를 준수하기 위해 `Result`가 아닌 `TokenStream`을 반환해야하기 때문입니다. 이 예제에서는 `unwrap`을 사용하여 간소화했지만, 실제 코드에서는 `panic!` 또는 `expect`를 사용하여 무엇이 잘못되었는지에 대한 더 구체적인 오류 메시지를 제공해야 합니다.

이제 `TokenStream`에서 `DeriveInput` 인스턴스로 해석된 Rust 코드를 처리하는 코드를 가지고 있으므로, Listing 19-33에 표시된 것처럼 `HelloMacro` 트레이트를 해당 코드에 적용하는 코드를 생성해 보겠습니다.

<span class=\"filename\">Filename: hello_macro_derive/src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-33/hello_macro/hello_macro_derive/src/lib.rs:here}}
```

<span class=\"caption\">Listing 19-33: `quote!` 매크로를 사용하여 `HelloMacro` 트레이트를 구현</span>

`ast.ident`를 사용하여 해당 구조체의 이름(identifier)을 나타내는 `Ident` 구조체 인스턴스를 얻습니다. Listing 19-32에서 보는 것처럼, Listing 19-30의 코드에 `impl_hello_macro` 함수를 실행하면 `ident` 필드가 `\"Pancakes\"` 값을 가진 `Ident` 인스턴스를 가져올 것입니다. 따라서 Listing 19-33의 `name` 변수에는 `\"Pancakes\"`라는 문자열을 포함하는 `Ident` 구조체 인스턴스가 있을 것입니다. Listing 19-30의 구조체의 이름입니다.

`quote!` 매크로는 반환하려는 Rust 코드를 정의할 수 있도록 합니다. 컴파일러는 `quote!` 매크로의 실행 결과를 직접적으로 기대하기 때문에, `TokenStream` 유형으로 변환해야 합니다. 이를 위해 `into` 메서드를 호출하여 이 중간 표현을 소비하고 필요한 `TokenStream` 유형의 값을 반환합니다.

`quote!` 매크로는 또한 매우 멋진 템플릿 메커니즘을 제공합니다. `#name`과 같이 입력할 수 있으며, `quote!`는 `name` 변수의 값으로 대체합니다. 정규 매크로와 유사한 방식으로 반복을 수행할 수도 있습니다. 자세한 내용은 [ `quote` crate 문서][quote-docs]를 참조하십시오.

우리의 절차 매크로는 사용자가 지정한 유형에 대한 `HelloMacro` 트레이트의 구현을 생성해야 합니다. 이를 `#name`을 사용하여 얻을 수 있습니다. 트레이트 구현에는 `hello_macro`라는 하나의 함수가 있으며, 함수 본문에는 제공하려는 기능이 있습니다. 즉, `Hello, Macro! My name is`를 출력하고 해당 코드에 지정된 유형의 이름을 출력합니다.

`stringify!` 매크로는 Rust에 내장되어 있습니다. `1 + 2`와 같은 Rust 표현을 입력받고 컴파일 시간에 해당 표현을 문자열 리터럴로 변환합니다. 예를 들어 `\"1 + 2\"`입니다. `format!` 또는 `println!`과는 다릅니다. 이 매크로는 표현을 평가한 후 결과를 `String`으로 변환합니다. `#name` 입력이 문자열 리터럴로 직접 출력될 수도 있으므로 `stringify!`를 사용합니다. `stringify!`를 사용하면 컴파일 시간에 `#name`을 문자열 리터럴로 변환하여 할당을 저장할 수 있습니다.

이제 `cargo build`를 실행하면 `hello_macro`와 `hello_macro_derive`에서 모두 성공적으로 완료되어야 합니다. 이제 Listing 19-30의 코드에 이 두 crate를 연결해 보겠습니다19-30번을 참조하여 절차적 맥락을 확인하세요! *projects* 디렉토리에 `cargo new pancakes`를 사용하여 새 이진 프로젝트를 만들어야 합니다. `pancakes` 크레이트의 *Cargo.toml*에 `hello_macro`와 `hello_macro_derive`를 의존성으로 추가해야 합니다. 만약 `hello_macro`와 `hello_macro_derive`를 [crates.io](https://crates.io/)에 게시한다면, 일반 의존성이 되지만, 게시하지 않는다면 다음과 같이 `path` 의존성으로 지정할 수 있습니다:

```toml
{{#include ../listings/ch19-advanced-features/no-listing-21-pancakes/pancakes/Cargo.toml:7:9}}
```

Listing 19-30의 코드를 *src/main.rs*에 넣고 `cargo run`을 실행하면 `Hello, Macro! My name is Pancakes!`가 출력됩니다. 절차적 맥락에서 `HelloMacro` 트레이트의 구현이 포함되었기 때문에 `pancakes` 크레이트가 트레이트를 구현할 필요가 없습니다. `#[derive(HelloMacro)]`가 트레이트 구현을 추가했습니다.

다음으로, 다른 유형의 절차적 맥락이 사용자 정의 `derive` 맥락과 어떻게 다른지 살펴보겠습니다.

### 속성과 유사한 맥락

속성과 유사한 맥락은 사용자 정의 `derive` 맥락과 유사하지만, `derive` 속성에 대한 코드를 생성하는 대신 새로운 속성을 만들 수 있습니다. 또한 더 유연합니다. `derive`는 구조체와 열거형에만 작동하지만, 함수와 같은 다른 항목에도 적용할 수 있습니다. `route`라는 이름의 속성이 웹 애플리케이션 프레임워크를 사용할 때 함수에 대한 애노테이션을 추가하는 예를 들어 보겠습니다.

```rust,ignore
#[route(GET, \"/\")]
fn index() {
```

이 `#[route]` 속성은 프레임워크에서 절차적 맥락으로 정의됩니다. 맥락 정의 함수의 서명은 다음과 같습니다.

```rust,ignore
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

여기서 두 개의 매개변수가 `TokenStream` 유형입니다. 첫 번째는 속성의 내용입니다. 즉, `GET, \"/\"` 부분입니다. 두 번째는 속성이 연결된 항목의 본문입니다. 이 경우 `fn index() {}`와 함수의 나머지 본문입니다.

그 외 속성과 유사한 맥락은 사용자 정의 `derive` 맥락과 동일하게 작동합니다. `proc-macro` 크레이트 유형을 가진 크레이트를 만들고 원하는 코드를 생성하는 함수를 구현합니다!

### 함수와 유사한 맥락

함수와 유사한 맥락은 함수 호출과 같은 모양의 맥락을 정의합니다. `macro_rules!` 맥락과 마찬가지로 더 유연합니다. 예를 들어, 알 수 없는 수의 인수를 받을 수 있습니다. 그러나 `macro_rules!` 맥락은 이전에 논의한 `match`와 같은 문법을 사용하여만 정의할 수 있습니다. 함수와 유사한 맥락은 `TokenStream` 매개변수를 받고 Rust 코드를 사용하여 해당 `TokenStream`을 조작하는 방식으로 정의됩니다. `sql!` 맥락이 다음과 같이 호출되는 예를 들어 보겠습니다.

```rust,ignore
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

이 맥락은 내부의 SQL 문을 분석하고 문법적으로 올바른지 확인합니다. 이는 `macro_rules!` 맥락이 할 수 있는 것보다 훨씬 복잡한 처리입니다. `sql!` 맥락은 다음과 같이 정의됩니다.

```rust,ignore
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

이 정의는 사용자 정의 `derive` 맥락의 서명과 유사합니다. `TokenStream` 내부에 있는 토큰을 받고 생성하려는 코드를 반환합니다.

## 요약

우와! 이제 책에서 논의한 여러 Rust 기능을 도구 상자에 넣었습니다. 이 기능들은 자주 사용하지는 않겠지만, 오류 메시지 제안이나 다른 사람들의 코드에서 발생할 때 인식할 수 있도록 했습니다. 이 장을 참조하여 해결책을 찾으세요.

다음으로, 책에서 논의한 모든 내용을 실제 프로젝트에 적용하여 마지막 프로젝트를 진행하겠습니다!

**참고:**

* [`syn`]: https://crates.io/crates/syn
* [`quote`]: https://crates.io/crates/quote
* [syn-docs]: https://docs.rs/syn/2.0/syn/struct.DeriveInput.html
* [quote-docs]: https://docs.rs/quote
* [decl]: #선언적-매크로-with-macro_rules-for-general-metaprogramming
