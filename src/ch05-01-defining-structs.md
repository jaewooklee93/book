## 구조체 정의 및 인스턴스 생성

구조체는 튜플과 유사하며, 튜플은 [“튜플 타입”][튜플]<!--
ignore --> 섹션에서 논의되었습니다. 둘 다 관련된 여러 값을 저장합니다. 튜플과 마찬가지로 구조체의 조각은 다른 유형일 수 있습니다. 튜플과 달리 구조체에서는 각 데이터 조각에 이름을 지정하므로 값의 의미가 명확합니다. 이러한 이름 추가는 구조체가 튜플보다 더 유연하다는 것을 의미합니다. 데이터의 순서에 의존하지 않고 인스턴스의 값을 지정하거나 액세스할 수 있습니다.

구조체를 정의하려면 `struct` 키워드를 입력하고 전체 구조체의 이름을 지정합니다. 구조체의 이름은 그룹화되는 데이터 조각의 중요성을 설명해야 합니다. 그런 다음 중괄호 안에서 데이터 조각의 이름과 유형을 정의합니다. 이러한 데이터 조각을 *필드*라고 합니다. 예를 들어, 사용자 계정에 대한 정보를 저장하는 구조체를 보여주는 5-1번 표를 참조하십시오.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-01/src/main.rs:here}}
```

5-1번 표: `User` 구조체 정의

구조체를 정의한 후에 사용하려면 해당 구조체의 *인스턴스*를 생성합니다. 인스턴스를 생성하려면 각 필드에 대한 구체적인 값을 지정합니다. 인스턴스를 생성하려면 구조체의 이름을 지정하고 필드의 이름과 값이 있는 *키:값* 쌍을 포함하는 중괄호를 추가합니다. 필드를 구조체에서 선언한 순서대로 지정할 필요는 없습니다. 즉, 구조체 정의는 유형의 일반적인 템플릿과 같으며, 인스턴스는 특정 데이터를 채워 유형의 값을 생성합니다. 예를 들어, 5-2번 표와 같이 특정 사용자를 선언할 수 있습니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-02/src/main.rs:here}}
```

5-2번 표: `User` 구조체의 인스턴스 생성

구조체에서 특정 값을 가져오려면 점 표기법을 사용합니다. 예를 들어, 사용자의 이메일 주소에 액세스하려면 `user1.email`을 사용합니다. 인스턴스가 변경 가능하면 점 표기법을 사용하여 특정 필드에 값을 할당하여 값을 변경할 수 있습니다. 5-3번 표는 변경 가능한 `User` 인스턴스의 `email` 필드의 값을 변경하는 방법을 보여줍니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-03/src/main.rs:here}}
```

5-3번 표: `User` 인스턴스의 `email` 필드의 값을 변경하는 방법

참고로 전체 인스턴스가 변경 가능해야 합니다. Rust는 특정 필드만 변경 가능하도록 허용하지 않습니다. 어떤 표현과 마찬가지로, 함수 몸체의 마지막 표현으로 새로운 구조체 인스턴스를 생성하여 암시적으로 해당 새로운 인스턴스를 반환할 수 있습니다.

5-4번 표는 `build_user` 함수를 보여줍니다. 이 함수는 주어진 이메일 주소와 사용자 이름을 사용하여 `User` 인스턴스를 반환합니다. `active` 필드는 `true` 값을 받고, `sign_in_count` 필드는 `1` 값을 받습니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-04/src/main.rs:here}}
```

5-4번 표: `build_user` 함수는 이메일 주소와 사용자 이름을 받아 `User` 인스턴스를 반환합니다.

함수 매개변수에 구조체 필드 이름과 동일한 이름을 지정하는 것은 의미가 있습니다. 그러나 구조체 필드 이름과 변수를 반복하는 것은 조금 지루합니다. 구조체에 더 많은 필드가 있으면 각 이름을 반복하는 것은 더욱 번거로워집니다. 다행히도 편리한 약자를 사용할 수 있습니다! 

<!-- Old heading. Do not remove or links may break. -->
## 필드 초기화 약속 사용하기

Listing 5-4에서 보여진 것처럼, 매개변수 이름과 구조체 필드 이름이 정확히 동일하기 때문에 `build_user` 함수를 재작성하여 `username` 과 `email` 과 같은 반복을 없앨 수 있습니다. Listing 5-5와 같이 *필드 초기화 약속* 문법을 사용하여 동일한 동작을 수행할 수 있습니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-05/src/main.rs:here}}
```

Listing 5-5: 필드 초기화 약속을 사용하는 `build_user` 함수. `username` 과 `email` 매개변수는 구조체 필드와 이름이 같기 때문에 `email: email` 처럼 반복적으로 쓰지 않아도 됩니다.

여기서는 `User` 구조체의 새로운 인스턴스를 생성하고 있습니다. `User` 구조체에는 `email`이라는 필드가 있습니다. `build_user` 함수의 `email` 매개변수의 값을 `email` 필드에 할당하고 싶습니다. `email` 필드와 `email` 매개변수가 같은 이름이므로 `email: email` 대신 `email`만 쓰면 됩니다.

## 다른 인스턴스에서 구조체 인스턴스 생성하기

구조체의 인스턴스를 생성할 때, 기존 인스턴스의 대부분의 값을 사용하고 몇 가지 값만 변경하는 경우가 많습니다. 이를 위해 *구조체 업데이트 문법*을 사용할 수 있습니다.

먼저 Listing 5-6에서 `user2`라는 새로운 `User` 인스턴스를 정상적으로 생성하는 방법을 보여줍니다. `email` 값을 새로 설정하지만, Listing 5-2에서 생성한 `user1`에서 사용한 나머지 값은 동일합니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-06/src/main.rs:here}}
```

Listing 5-6: `user1`에서 대부분의 값을 사용하여 새로운 `User` 인스턴스 `user2`를 생성합니다.

구조체 업데이트 문법을 사용하면 Listing 5-7과 같이 코드를 줄일 수 있습니다. `..`는 나머지 필드에 대해 기존 인스턴스의 값을 사용하라는 의미입니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-07/src/main.rs:here}}
```

Listing 5-7: 구조체 업데이트 문법을 사용하여 `User` 인스턴스 `user2`를 생성합니다. `email` 값은 변경하지만 `user1`의 나머지 값은 유지됩니다.

Listing 5-7의 코드는 `user2`라는 새로운 인스턴스를 생성하고 `email` 값은 다르지만 `username`, `active`, `sign_in_count` 필드는 `user1`에서 가져옵니다. `..user1`은 나머지 필드에 대해 기존 인스턴스의 값을 사용하라는 의미이며, 구조체 정의 순서와 관계없이 원하는 필드만 지정할 수 있습니다.

구조체 업데이트 문법은 `=`를 사용하여 값을 할당하는 방식으로 작동합니다. 이는 데이터가 이동하기 때문입니다. 이 예제에서는 `user2`를 생성한 후 `user1`을 전체로 사용할 수 없습니다. `User` 구조체의 `username` 필드에 있는 `String`은 `user2`로 이동되었기 때문입니다. `user2`에 `email`과 `username`에 대한 새로운 `String` 값을 지정하고 `user1`의 `active`와 `sign_in_count` 값만 사용했다면, `user1`은 `user2`를 생성한 후에도 유효하게 남아있을 것입니다. `active`와 `sign_in_count`는 `Copy` 트레이트를 구현하는 유형이므로, 이전 섹션에서 설명한 것처럼 데이터가 복사되는 방식으로 동작합니다. "


```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-05/src/main.rs:here}}
```

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-06/src/main.rs:here}}
```

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-07/src/main.rs:here}}
```데이터: 복사”][복사]<!-- 무시 --> 섹션이 적용될 것입니다.

### 명명된 필드가 없는 튜플 구조체를 사용하여 다른 유형을 만들기

Rust는 튜플과 유사한 구조체를 지원합니다. 이를 *튜플 구조체*라고 합니다.
튜플 구조체는 구조체 이름이 제공하는 추가 의미를 가지지만, 필드에 이름이
없습니다. 오히려 필드의 유형만 있습니다. 튜플 구조체는 전체 튜플에 이름을
주고 다른 튜플과 구분된 유형으로 만들고 싶을 때 유용하며, 일반 구조체에서
각 필드를 명명하는 것이 verbose하거나 중복될 때 유용합니다.

튜플 구조체를 정의하려면 `struct` 키워드와 구조체 이름을 따르고 튜플의 유형을
입력합니다. 예를 들어, `Color` 및 `Point`라는 두 개의 튜플 구조체를 정의하고 사용하는
예를 들어 보겠습니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-01-tuple-structs/src/main.rs}}
```

참고로 `black`과 `origin` 값은 서로 다른 유형입니다. 왜냐하면 그들은 서로 다른
튜플 구조체의 인스턴스이기 때문입니다. 정의한 각 구조체는 자신의 유형이
됩니다. 예를 들어, `Color` 유형의 매개변수를 받는 함수는 `Point`를
인수로 받을 수 없습니다. 튜플 구조체 인스턴스는 튜플과 유사하게
구조화하여 개별 부분으로 분해할 수 있으며, `.`를 사용하여 인덱스를
따라 개별 값에 액세스할 수 있습니다.

### 필드가 없는 단위 구조체

구조체가 필드가 없는 경우도 있습니다! 이를 *단위 구조체*라고 합니다.
왜냐하면 `()`에서 언급한 단위 유형과 유사하게 동작하기 때문입니다.
단위 구조체는 특정 유형에 대한 행동을 구현해야 하지만, 구조체 자체에
저장할 데이터가 없을 때 유용합니다. 10장에서 트레이트에 대해 논의할 것입니다.
`AlwaysEqual`이라는 단위 구조체를 선언하고 인스턴스화하는 예를 보겠습니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-04-unit-like-structs/src/main.rs}}
```

`AlwaysEqual`을 정의하려면 `struct` 키워드, 원하는 이름, 그리고 세미콜론을
사용합니다. 괄호나 중괄호가 필요하지 않습니다! 그런 다음 `subject` 변수에
`AlwaysEqual`의 인스턴스를 얻을 수 있습니다. 정의한 이름을 사용하여
괄호나 중괄호 없이 합니다. 나중에 이 유형에 대해 행동을 구현할 것이라고
가정해 보겠습니다. 모든 `AlwaysEqual` 인스턴스가 다른 유형의 모든
인스턴스와 항상 같도록 구현합니다. 예를 들어 테스트를 위해 알려진
결과를 가져오는 것입니다. 구현하려는 행동을 위해 데이터가 필요하지 않습니다!
10장에서 트레이트를 정의하고 어떤 유형에도 구현하는 방법을 알아볼 것입니다.

> ### 구조체 데이터 소유권
>
> 5-1 부록에서 나열된 `User` 구조체 정의에서 `String` 유형을 소유된
> 유형으로 사용했습니다. `&str` 문자열 슬라이스 유형 대신 이를 의도적으로
> 선택한 이유는 각 구조체 인스턴스가 모든 데이터를 소유하고 구조체
> 자체가 유효한 동안 데이터가 유효해야 하기 때문입니다.
>
> 구조체가 다른 것에 의해 소유된 데이터에 대한 참조를 저장할 수도
> 있지만, 이를 위해서는 *라이프타임*이라는 Rust 기능이 필요합니다.
> 라이프타임은 구조체가 유효한 동안 참조하는 데이터가 유효하도록
> 보장합니다. 예를 들어 라이프타임을 지정하지 않고 구조체에 참조를
> 저장하려고 하면 다음과 같이 작동하지 않습니다.
>
> Filename: src/main.rs
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore,does_not_compile
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
>
> fn main() {
```
>     let user1 = User {
>         active: true,
>         username: \"someusername123\",
>         email: \"someone@example.com\",
>         sign_in_count: 1,
>     };
> }
> ```
>
> 컴파일러는 타임 라이프 스펙이 필요하다고 알려줄 것입니다.
>
> ```console
> $ cargo run
>    Compiling structs v0.1.0 (file:///projects/structs)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
>
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
>
> For more information about this error, try `rustc --explain E0106`.
> error: could not compile `structs` (bin \"structs\") due to 2 previous errors
> ```
>
> 10장에서 이러한 오류를 해결하는 방법에 대해 알아보겠지만, 지금은 `String`과 같은 소유된 유형을 사용하여 `&str`과 같은 참조를 저장하는 오류를 해결할 것입니다.

<!-- manual-regeneration
for the error above
after running update-rustc.sh:
pbcopy < listings/ch05-using-structs-to-structure-related-data/no-listing-02-reference-in-struct/output.txt
paste above
add `> ` before every line -->

[tuples]: ch03-02-data-types.html#the-tuple-type
[move]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move
[copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
