## `Result`를 사용한 복구 가능한 오류

대부분의 오류는 프로그램을 완전히 중지시키는 정도로 심각하지 않습니다.
때때로 함수가 실패할 때는 쉽게 해석하고 대처할 수 있는 이유가 있습니다.
예를 들어 파일을 열려고 시도하고 파일이 존재하지 않아서 작업이 실패할 경우, 프로세스를 종료하는 대신 파일을 생성하는 것이 좋을 수 있습니다.

[“`Result`를 사용한 잠재적 실패 처리”][handle_failure]<!--
ignore --> 에서는 `Result` enum이 두 가지 변형, `Ok`과 `Err`로 정의됨을 기억하시죠?

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T`와 `E`는 일반 타입 매개변수입니다. 우리는 10장에서 일반 타입 매개변수에 대해 자세히 다룰 것입니다. 지금 당장 알아야 할 것은 `T`가 `Ok` 변형 내에서 성공적인 경우 반환될 값의 유형을 나타내고, `E`가 `Err` 변형 내에서 반환될 오류 유형을 나타낸다는 것입니다.
`Result`는 이러한 일반 타입 매개변수를 가지고 있기 때문에, 성공 값과 오류 값이 다를 수 있는 다양한 상황에서 `Result` 유형과 그 위에 정의된 함수를 사용할 수 있습니다.

`Result` 값을 반환하는 함수를 호출해 보겠습니다. 함수가 실패할 수 있기 때문입니다. 9-3번 목록에서 파일을 열려고 하는 함수를 살펴보겠습니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-03/src/main.rs}}
```

Listing 9-3: 파일 열기

`File::open`의 반환 유형은 `Result<T, E>`입니다. `T` 일반 매개변수는 `File::open`의 구현에 의해 성공 값의 유형인 `std::fs::File` (파일 핸들)로 채워집니다. `E` 유형은 오류 값에 사용됩니다. 이 유형은 `std::io::Error`입니다. 이 반환 유형은 `File::open` 호출이 성공하여 읽거나 쓰기 위해 사용할 수 있는 파일 핸들을 반환할 수도 있고, 실패할 수도 있습니다. 예를 들어 파일이 존재하지 않거나 파일에 액세스할 권한이 없을 수 있습니다. `File::open` 함수는 성공했는지 실패했는지 알려주는 방법이 필요하며 동시에 파일 핸들 또는 오류 정보를 제공해야 합니다. 이 정보는 `Result` enum이 전달하는 것입니다.

`File::open`이 성공하는 경우 `greeting_file_result` 변수의 값은 파일 핸들을 포함하는 `Ok` 인스턴스가 됩니다. 실패하는 경우 `greeting_file_result` 변수의 값은 발생한 오류 유형에 대한 정보를 포함하는 `Err` 인스턴스가 됩니다.

9-3번 목록의 코드에 추가하여 `File::open`이 반환하는 값에 따라 다른 작업을 수행해야 합니다. 9-4번 목록은 `match` 표현식을 사용하여 `Result` 변형을 처리하는 방법을 보여줍니다.

Filename: src/main.rs

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-04/src/main.rs}}
```

Listing 9-4: `Result` 변형을 처리하는 데 `match` 표현식을 사용하기

참고로 `Option` enum과 마찬가지로 `Result` enum과 그 변형은 prelude에 포함되어 있으므로 `match` 팔레트에서 `Result::`를 명시할 필요가 없습니다.

결과가 `Ok`인 경우 이 코드는 `Ok` 변형에서 내부 `file` 값을 반환하고, 그 파일 핸들 값을 `greeting_file` 변수에 할당합니다. `match` 이후에 파일 핸들을 사용하여 읽거나 쓸 수 있습니다.

`match`의 다른 팔레트는 `File::open`에서 `Err` 값을 받는 경우를 처리합니다. 이 예에서는 `panic!` 매크로를 호출하는 것을 선택했습니다. 현재 디렉토리에 *hello.txt*라는 파일이 없고 이 코드를 실행하면코드를 살펴보면 `panic!` 매크로에서 다음과 같은 출력을 볼 수 있습니다.

```console
{{#include ../listings/ch09-error-handling/listing-09-04/output.txt}}
```

보통 이 출력은 무엇이 잘못되었는지 정확하게 알려줍니다.

### 다양한 오류에 대한 매칭

Listing 9-4의 코드는 `File::open`이 실패한 이유와 상관없이 `panic!`합니다.
그러나 우리는 다양한 실패 이유에 대해 다른 조치를 취하고 싶습니다. 만약
`File::open`이 파일이 존재하지 않기 때문에 실패했다면, 파일을 생성하고 새 파일의 핸들을 반환하고 싶습니다. `File::open`이 다른 이유로 실패했다면—예를 들어 파일을 열 권한이 없기 때문—Listing 9-4에서와 같이 코드가 `panic!`하도록 하고 싶습니다. 이를 위해 Listing 9-5에 보여진 내부 `match` 표현식을 추가합니다.

Filename: src/main.rs

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-05/src/main.rs}}
```

Listing 9-5: 다양한 방식으로 다양한 종류의 오류를 처리하기

`File::open`이 `Err` 변형체 안에서 반환하는 값의 유형은 `io::Error`입니다. `io::Error`는 표준 라이브러리에서 제공되는 구조체입니다. 이 구조체에는 `kind` 메서드가 있으며, 이를 통해 `io::ErrorKind` 값을 가져올 수 있습니다. `io::ErrorKind`은 표준 라이브러리에서 제공되는 열거형으로, `io` 작업에서 발생할 수 있는 다양한 종류의 오류를 나타내는 변형체를 가지고 있습니다. 우리가 사용하고 싶은 변형체는 `ErrorKind::NotFound`로, 우리가 열려고 하는 파일이 아직 존재하지 않는다는 것을 나타냅니다. 따라서 `greeting_file_result`에 대해 `match`를 수행하지만, `error.kind()`에도 내부 `match`가 있습니다.

내부 `match`에서 확인하고 싶은 조건은 `error.kind()`가 반환하는 값이 `ErrorKind` 열거형의 `NotFound` 변형체인지 여부입니다. 그렇다면 `File::create`를 사용하여 파일을 생성하려고 합니다. 그러나 `File::create`도 실패할 수 있으므로 내부 `match` 표현식에 두 번째 팔이 필요합니다. 파일을 생성할 수 없을 때 다른 오류 메시지가 출력됩니다. 외부 `match`의 두 번째 팔은 동일하게 유지되므로 파일이 없는 오류 외에는 프로그램이 `panic!`합니다.

> #### `match`를 사용하지 않는 `Result<T, E>` 처리 방법
>
> 많이 `match`를 사용했죠! `match` 표현식은 매우 유용하지만 매우 기본적인 도구입니다. 13장에서 닫힌 함수를 배우게 되는데, 이는 `Result<T, E>`에 정의된 많은 메서드와 함께 사용됩니다. 이러한 메서드는 코드에서 `Result<T, E>` 값을 처리할 때 `match`를 사용하는 것보다 더 간결할 수 있습니다.
>
> 예를 들어, Listing 9-5와 동일한 논리를 닫힌 함수와 `unwrap_or_else` 메서드를 사용하여 작성하는 또 다른 방법을 보여드리겠습니다.
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore
> use std::fs::File;
> use std::io::ErrorKind;
>
> fn main() {
>     let greeting_file = File::open(\"hello.txt\").unwrap_or_else(|error| {
>         if error.kind() == ErrorKind::NotFound {
>             File::create(\"hello.txt\").unwrap_or_else(|error| {
>                 panic!(\"Problem creating the file: {error:?}\");
>             })
>         } else {
>             panic!(\"Problem opening the file: {error:?}\");
>         }
>     });
> }
> ```
>
> 이 코드는 Listing 9-5와 동일한 동작을 하지만 `match` 표현식이 없으며 읽기 쉽습니다. 13장을 읽은 후 이 예제를 다시 살펴보고 표준 라이브러리 문서에서 `unwrap_or_else` 메서드를 찾아보세요. 오류를 처리할 때 사용하는 `Result<T, E>`에 대한 많은 메서드가 깊이 있는 `match` 표현식을 간결하게 만들어줍니다.

#### 오류 발생 시 `unwrap` 및 `expect`를 사용한 단축키

`match`를 사용하는 것은 충분히 잘 작동하지만, 때로는 너무 verbose하고 의도를 명확하게 전달하지 못할 수 있습니다. `Result<T, E>` 유형은 다양하고 더 구체적인 작업을 수행하기 위해 정의된 많은 도우미 메서드를 제공합니다. `unwrap` 메서드는 Listing 9-4에서 작성한 `match` 표현식과 같은 단축 메서드입니다. `Result` 값이 `Ok` 변형이라면 `unwrap`은 `Ok` 안의 값을 반환합니다. `Result`가 `Err` 변형이라면 `unwrap`은 `panic!` 매크로를 호출합니다. `unwrap`이 작동하는 예시는 다음과 같습니다.

Filename: src/main.rs

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-04-unwrap/src/main.rs}}
```

*hello.txt* 파일이 없으면 이 코드를 실행하면 `unwrap` 메서드에서 호출하는 `panic!` 메시지로 오류 메시지가 표시됩니다.

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-04-unwrap
cargo run
relevant text를 복사하여 붙여넣기
-->

```text
thread 'main' panicked at src/main.rs:4:49:
called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: \"No such file or directory우선 함수의 반환형을 살펴보겠습니다. `Result<String, io::Error>`입니다. 이는 함수가 `Result<T, E>` 형식으로 값을 반환한다는 것을 의미하며, 일반 매개변수 `T`는 `String`으로, 일반 매개변수 `E`는 `io::Error`로 구체화되었습니다.

이 함수가 성공적으로 실행되면, 이 함수를 호출한 코드는 파일에서 읽은 `username`을 포함하는 `Ok` 값을 받습니다. 함수가 문제를 발생시키면, 호출 코드는 문제에 대한 자세한 정보를 포함하는 `io::Error` 인스턴스를 포함하는 `Err` 값을 받습니다. `io::Error`를 이 함수의 반환형으로 선택한 이유는 함수 본문에서 호출하는 두 가지 작업 중 하나가 실패할 수 있기 때문입니다. `File::open` 함수와 `read_to_string` 메서드입니다.

함수 본문은 `File::open` 함수를 호출하여 시작됩니다. `match`를 사용하여 `Result` 값을 처리합니다. Listing 9-4와 유사한 `match`입니다. `File::open`이 성공하면 패턴 변수 `file`에 있는 파일 핸들이 `username_file` 변수에 저장되고 함수가 계속 진행됩니다. `Err` 경우에는 `panic!` 대신 `return` 키워드를 사용하여 함수에서 일찍 종료하고 패턴 변수 `e`에 있는 `File::open`의 오류 값을 호출 코드로서 함수의 오류 값으로 전달합니다.

따라서 `username_file`에 파일 핸들이 있다면 함수는 `username` 변수에 새로운 `String`을 생성하고 `username_file`에 있는 파일 핸들을 사용하여 파일 내용을 `username`에 읽습니다. `read_to_string` 메서드 또한 `Result`를 반환합니다. `File::open`이 성공했더라도 실패할 수 있기 때문입니다. 따라서 `read_to_string` 메서드를 처리하기 위해 또 다른 `match`가 필요합니다. `read_to_string`이 성공하면 함수가 성공했으며, `username`에 있는 파일에서 읽은 username을 `Ok`로 감싸서 반환합니다. `read_to_string`이 실패하면 오류 값을 동일한 방식으로 반환하지만, `return`을 명시적으로 작성할 필요가 없습니다. 이는 함수의 마지막 표현식이기 때문입니다.

이 코드를 호출하는 코드는 `Ok` 값(username을 포함) 또는 `io::Error`를 포함하는 `Err` 값을 처리해야 합니다. 호출 코드가 `Err` 값을 받으면 `panic!`을 호출하여 프로그램을 종료하거나 기본 username을 사용하거나 파일 외 다른 곳에서 username을 검색할 수 있습니다. 호출 코드가 실제로 무엇을 시도하는지에 대한 충분한 정보가 없기 때문에, 성공 또는 오류 정보를 모두 상위로 전달하여 적절하게 처리하도록 합니다.

Rust에서 이러한 오류 전파 패턴은 매우 일반적이기 때문에 Rust는 이를 더 쉽게 만드는 물음표 연산자 `?`를 제공합니다.

#### 오류 전파를 위한 단축: 물음표 연산자 `?`

Listing 9-7은 Listing 9-6과 동일한 기능을 가진 `read_username_from_file` 함수의 구현을 보여줍니다. 하지만 이 구현은 `?` 연산자를 사용합니다.

Filename: src/main.rs

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-07/src/main.rs:here}}
```

Listing 9-7: `?` 연산자를 사용하여 오류를 호출 코드로 전달하는 함수

`Result` 값 뒤에 놓인 `?`는 Listing 9-6에서 `match` 표현식으로 처리한 것과 거의 동일한 방식으로 작동합니다. `Result` 값이 `Ok`이면 `Ok` 안의 값이 계속 진행됩니다. `Err`이면 함수가 종료되고 오류 값이 호출 코드로 전달됩니다.

`?` 연산자는 코드를 더 간결하게 만들어줍니다. `match` 표현식을 사용하는 것보다 훨씬 짧고 읽기 쉽습니다. 이러한 간결함은 오류 처리를 더 쉽게 만들고 코드를 더 가독성 있게 만듭니다.

이 표현에서 반환되며, 프로그램이 계속 실행됩니다. 값이 `Err`이면, 전체 함수에서 `Err`가 호출 코드로 전파되므로 마치 `return` 키워드를 사용했던 것처럼 반환됩니다.

Listing 9-6의 `match` 표현과 `?` 연산자의 차이점은 다음과 같습니다. `?` 연산자를 호출한 오류 값은 표준 라이브러리의 `From` 트레이트에서 정의된 `from` 함수를 통해 다른 유형의 값으로 변환됩니다. `?` 연산자가 `from` 함수를 호출할 때, 받은 오류 유형은 현재 함수의 반환 유형에 정의된 오류 유형으로 변환됩니다. 현재 함수가 실패할 수 있는 모든 방법을 나타내는 하나의 오류 유형을 반환하는 경우 유용합니다. 예를 들어, Listing 9-7의 `read_username_from_file` 함수를 `OurError`라는 이름의 사용자 정의 오류 유형으로 변경할 수 있습니다. `impl From<io::Error> for OurError`를 정의하여 `io::Error`에서 `OurError`를 생성하면 `read_username_from_file` 함수의 본문에서 `?` 연산자 호출은 `from`을 호출하여 오류 유형을 변환하는 데 필요한 추가 코드를 추가하지 않아도 됩니다.

Listing 9-7의 맥락에서 `File::open` 호출의 끝에 있는 `?` 연산자는 `username_file` 변수에 `Ok` 안의 값을 반환합니다. 오류가 발생하면 `?` 연산자는 전체 함수에서 일찍 반환되어 호출 코드에 `Err` 값을 전달합니다. `read_to_string` 호출의 끝에 있는 `?`도 마찬가지입니다.

`?` 연산자는 많은 boilerplate 코드를 제거하여 함수의 구현을 간소화합니다. Listing 9-8과 같이 `?` 연산자 바로 뒤에 메서드 호출을 체인하여 코드를 더욱 간결하게 만들 수도 있습니다.

Filename: src/main.rs

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-08/src/main.rs:here}}
```

Listing 9-8: `?` 연산자 뒤에 메서드 호출 체인

`username`에서 새로운 `String` 객체를 생성하는 부분은 변경되지 않았습니다. `username_file` 변수를 생성하는 대신 `File::open(\"hello.txt\")?`의 결과에 `read_to_string` 호출을 직접 체인했습니다. `read_to_string` 호출의 끝에도 여전히 `?`가 있고, `File::open`과 `read_to_string`이 모두 성공하면 `username`을 포함하는 `Ok` 값을 반환합니다. Listing 9-6과 Listing 9-7과 동일한 기능을 제공합니다. 이것은 단지 다른, 더 사용자 친화적인 방법으로 작성한 것입니다.

Listing 9-9는 `fs::read_to_string`을 사용하여 이를 더욱 간결하게 만드는 방법을 보여줍니다.

Filename: src/main.rs

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-09/src/main.rs:here}}
```

Listing 9-9: `fs::read_to_string`을 사용하여 파일을 문자열로 읽기

파일을 문자열로 읽는 것은 흔한 작업이므로 표준 라이브러리는 파일을 열고 새로운 `String` 객체를 생성하고 파일 내용을 읽어 `String`에 넣어 반환하는 편리한 `fs::read_to_string` 함수를 제공합니다. 물론 `fs::read_to_string`을 사용하면 모든 오류 처리를 설명할 기회가 없기 때문에 Listing 9-7과 같이 오류 처리를 설명했습니다.

먼저 좀 더 긴 방법을 살펴보겠습니다.

#### `?` 연산자를 사용할 수 있는 곳

`?` 연산자는 반환형이 `?`가 사용되는 값과 호환되는 함수에서만 사용할 수 있습니다. 이는 `?` 연산자가 Listing 9-6에서 정의한 `match` 표현과 동일한 방식으로 함수에서 값을 조기에 반환하도록 정의되어 있기 때문입니다. Listing 9-6에서 `match`는 `Result` 값을 사용했으며, 조기에 반환되는 부분은 `Err(e)` 값을 반환했습니다. 함수의 반환형은 이러한 `return`과 호환되도록 `Result`여야 합니다.

Listing 9-10에서 `?` 연산자를 `Result`의 형태와 호환되지 않는 `main` 함수에서 사용하면 발생하는 오류를 살펴보겠습니다.

Filename: src/main.rs

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-10/src/main.rs}}
```

Listing 9-10: `()`를 반환하는 `main` 함수에서 `?`를 사용하려고 시도하면 컴파일되지 않습니다.

이 코드는 파일을 열어서 오류가 발생할 수 있습니다. `?` 연산자는 `File::open`에서 반환되는 `Result` 값을 따릅니다. 하지만 이 `main` 함수는 `()`의 반환형을 가지고 있으며, `Result`와 호환되지 않습니다. 이 코드를 컴파일하면 다음과 같은 오류 메시지가 표시됩니다.

```console
{{#include ../listings/ch09-error-handling/listing-09-10/output.txt}}
```

이 오류 메시지는 `?` 연산자를 `Result`, `Option` 또는 `FromResidual`를 구현하는 다른 유형의 함수에서만 사용할 수 있다고 지적합니다.

오류를 해결하려면 두 가지 선택지가 있습니다. 하나는 `?` 연산자를 사용하는 값과 호환되는 함수의 반환형을 변경하는 것입니다. 다른 하나는 `match` 또는 `Result<T, E>` 메서드 중 하나를 사용하여 `Result<T, E>`를 적절한 방식으로 처리하는 것입니다.

오류 메시지는 `?`를 `Option<T>` 값에도 사용할 수 있다고 언급했습니다. `Result<T, E>`에서 `?`를 사용하는 것과 마찬가지로, `Option`에서 `?`를 사용하려면 반환형이 `Option`인 함수에서만 사용할 수 있습니다. `Option<T>`에 `?` 연산자를 호출했을 때의 동작은 `Result<T, E>`에 `?`를 호출했을 때와 유사합니다. 값이 `None`이면, `None`이 함수에서 조기에 반환됩니다. 값이 `Some`이면, `Some` 안의 값이 표현식의 결과값이 되고 함수가 계속 실행됩니다. Listing 9-11은 주어진 텍스트에서 첫 번째 줄의 마지막 문자를 찾는 함수의 예입니다.

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-11/src/main.rs:here}}
```

Listing 9-11: `Option<T>` 값에 `?` 연산자를 사용하는 예

이 함수는 `Option<char>`를 반환합니다. 왜냐하면 문자가 있을 수도 있지만 없을 수도 있기 때문입니다. 이 코드는 `text` 문자열 슬라이스 인자를 받아 `lines` 메서드를 호출합니다. `lines` 메서드는 문자열에서 줄을 나타내는 이터레이터를 반환합니다. 이 함수는 첫 번째 줄을 검토하려고 하므로 이터레이터에서 첫 번째 값을 가져오기 위해 `next`를 호출합니다. `text`가 비어 있는 경우, `next` 호출은 `None`을 반환합니다. 이 경우 `last_char_of_first_line`에서 `?`를 사용하여 `None`을 조기에 반환합니다. `text`가 비어 있지 않은 경우, `next`는 `text`의 첫 번째 줄을 나타내는 문자열 슬라이스를 포함하는 `Some` 값을 반환합니다.

`?`는 문자열 슬라이스를 추출하고, 이 문자열 슬라이스에 `chars`를 호출하여 문자의 이터레이터를 얻을 수 있습니다. 이 함수는 첫 번째 줄의 마지막 문자에 관심이 있으므로 이터레이터의 마지막 항목을 반환하기 위해 `last`를 호출합니다.
첫 번째 줄이 비어 있는 경우 `Option`이기 때문입니다. 예를 들어 `text`가 빈 줄로 시작하지만 다른 줄에는 문자가 있는 경우, 예를 들어 `\"\
hi\"`와 같이 있습니다. 그러나 첫 번째 줄에 마지막 문자가 있는 경우 `Some` 변형에 반환됩니다. 중간에 있는 `?` 연산자는 이 논리를 간결하게 표현하는 방법을 제공하여 함수를 한 줄로 구현할 수 있게 합니다. `?` 연산자를 `Option`에 사용할 수 없었다면, 더 많은 메서드 호출이나 `match` 표현식을 사용하여 이 논리를 구현해야 했습니다. 

`Result`에 `?` 연산자를 사용할 수 있는 함수에서 `Result`를 반환하는 경우, `Option`에 `?` 연산자를 사용할 수 있는 함수에서 `Option`을 반환하는 경우, 하지만 혼합해서 사용할 수는 없습니다. `?` 연산자는 `Result`를 `Option`으로 또는 그 반대로 자동으로 변환하지 않습니다. 이러한 경우 `Result`의 `ok` 메서드 또는 `Option`의 `ok_or` 메서드와 같은 메서드를 사용하여 명시적으로 변환할 수 있습니다. 

지금까지 사용해 온 모든 `main` 함수는 `()`를 반환했습니다. `main` 함수는 실행 가능한 프로그램의 시작점과 종료점이기 때문에 프로그램이 예상대로 동작하려면 반환 유형에 제한이 있습니다. 

다행히 `main`은 `Result<(), E>`를 반환할 수도 있습니다. 9-12번 목록은 9-10번 목록의 코드를 가져왔지만, `main`의 반환 유형을 `Result<(), Box<dyn Error>>`로 변경하고 끝에 `Ok(())`을 반환했습니다. 이 코드는 이제 컴파일됩니다. 

Filename: src/main.rs

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-12/src/main.rs}}
```

Listing 9-12: `main`을 `Result<(), E>`로 반환하도록 변경하면 `Result` 값에 `?` 연산자를 사용할 수 있습니다.

`Box<dyn Error>` 유형은 *trait object*이며, 17장의 “다양한 유형의 값을 허용하는 Trait Object 사용” 부분에서 다룰 것입니다. 지금은 `Box<dyn Error>`를 “모든 종류의 오류”로 읽을 수 있습니다. `main` 함수에서 `Result` 값에 `?` 연산자를 사용하는 것은 `Box<dyn Error>` 오류 유형을 사용하는 경우 허용됩니다. 이것은 `main` 함수의 본문에서 `std::io::Error` 유형의 오류만 반환될 것이지만, `main` 함수의 본문에 더 많은 오류를 반환하는 코드가 추가되더라도 이 서명이 계속해서 올바른 이유입니다. 

`main` 함수가 `Result<(), E>`를 반환하면 실행 파일은 `main`이 `Ok(())`을 반환하면 0 값으로 종료되고, `main`이 `Err` 값을 반환하면 0이 아닌 값으로 종료됩니다. C로 작성된 실행 파일은 종료 시 정수를 반환합니다. 성공적으로 종료되는 프로그램은 정수 0을 반환하고, 오류가 발생하는 프로그램은 0이 아닌 정수를 반환합니다. Rust 또한 이 규칙을 준수하여 실행 파일에서 정수를 반환합니다. 

`main` 함수는 `std::process::Termination` 트레이트를 구현하는 모든 유형을 반환할 수 있습니다. `Termination` 트레이트는 `report` 함수를 포함하며, 이 함수는 `ExitCode`를 반환합니다. 자세한 내용은 표준 라이브러리 문서를 참조하십시오. 

이제 `panic!`을 호출하거나 `Result`를 반환하는 방법에 대해 논의했으므로, 어떤 경우에 어떤 것이 적절한지 결정하는 방법에 대해 다시 살펴보겠습니다. 

[handle_failure]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result
[trait-objects]: ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[termination]: ../std/process/trait.Termination.html
