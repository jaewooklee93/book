## 구조체를 사용하는 예제 프로그램

구조체를 언제 사용해야 하는지 이해하기 위해 직사각형의 면적을 계산하는 프로그램을 작성해 보겠습니다. 먼저 단일 변수를 사용하여 시작하고, 구조체를 사용하여 프로그램을 재작성할 것입니다.

Cargo로 *rectangles*라는 새로운 이진 프로젝트를 만들고, 픽셀로 지정된 직사각형의 너비와 높이를 입력받아 직사각형의 면적을 계산하도록 하겠습니다. 5-8번 표에 우리 프로젝트의 *src/main.rs*에 있는 방법을 보여줍니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:all}}
```

<span class=\"caption\">Listing 5-8: 별도의 너비와 높이 변수로 지정된 직사각형의 면적 계산</span>

이제 `cargo run`을 사용하여 이 프로그램을 실행합니다.

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/output.txt}}
```

이 코드는 `area` 함수에 각 치수를 전달하여 직사각형의 면적을 계산하는 데 성공하지만, 코드를 더 명확하고 읽기 쉽게 만들 수 있습니다.

이 코드의 문제점은 `area` 함수의 서명에서 명확합니다.

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:here}}
```

`area` 함수는 하나의 직사각형의 면적을 계산해야 하지만, 우리가 작성한 함수에는 두 개의 매개변수가 있고, 어디에도 매개변수가 관련되어 있음이 명확하지 않습니다. 너비와 높이를 함께 그룹화하여 코드를 더 읽기 쉽고 관리하기 쉽게 만들 수 있습니다. 이미 3장의 "튜플 유형"<!-- ignore --> 섹션에서 그 방법을 논의했습니다.

### 튜플을 사용한 재작성

5-9번 표는 튜플을 사용하는 프로그램의 다른 버전을 보여줍니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-09/src/main.rs}}
```

<span class=\"caption\">Listing 5-9: 튜플로 직사각형의 너비와 높이를 지정</span>

어떤 면에서 이 프로그램은 더 나은 것입니다. 튜플은 약간의 구조를 제공하며, 하나의 인수만 전달하고 있습니다. 그러나 다른 면에서 이 버전은 덜 명확합니다. 튜플은 요소에 이름을 붙이지 않으므로 튜플의 부분에 인덱싱해야 하며, 계산이 덜 명확해집니다.

너비와 높이를 섞으면 면적 계산에는 문제가 되지 않지만, 직사각형을 화면에 그리려면 문제가 발생합니다! 너비가 튜플 인덱스 `0`이고 높이가 튜플 인덱스 `1`임을 기억해야 합니다. 다른 사람이 코드를 사용하고 이해하려고 할 때 이를 기억하는 것이 더욱 어려워지고, 오류를 쉽게 만들 수 있습니다. 우리가 코드에 데이터의 의미를 전달하지 않았기 때문에, 오류를 쉽게 만들 수 있습니다.

### 구조체를 사용한 재작성: 더 많은 의미 추가

구조체를 사용하여 데이터에 이름을 붙여 의미를 추가합니다. 우리가 사용하는 튜플을 `Rectangle`이라는 이름의 구조체로 변환하고, 전체에 대한 이름과 부분에 대한 이름을 정의할 수 있습니다. 5-10번 표를 참조하세요.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-10/src/main.rs}}
```

<span class=\"caption\">Listing 5-10: `Rectangle` 구조체 정의</span>

여기서는 `Rectangle`이라는 구조체를 정의했습니다. 괄호 안에는 `width`와 `height`라는 필드를 정의했으며, 두 필드 모두 `u32` 유형입니다. 그런 다음 `main`에서 `Rectangle`의 특정 인스턴스를 생성하여 너비가 `30`이고 높이가 `50`인 직사각형을 만듭니다.

이제 `area` 함수는 하나의 매개변수로 정의되며, 이를 `Rectangle`이라고 이름 지었습니다.
 `rectangle` 이라는 변수를 선언합니다. 이 변수의 타입은 `Rectangle` 구조체의 불변 참조입니다. 4장에서 언급했듯이, 구조체를 빌려서 소유권을 넘기지 않고 싶습니다. 이렇게 하면 `main` 함수는 여전히 소유권을 가지고 `rect1`을 계속 사용할 수 있습니다. 따라서 함수 선언과 함수 호출에서 `&`를 사용합니다.

`area` 함수는 `Rectangle` 인스턴스의 `width`와 `height` 필드에 접근합니다. (빌려진 구조체 인스턴스의 필드에 접근하는 것은 필드 값을 이동시키지 않기 때문에 자주 볼 수 있습니다). `area` 함수의 함수 선언은 이제 정확히 무엇을 의미하는지 명시합니다: `Rectangle`의 면적을 계산하고, `width`와 `height` 필드를 사용합니다. 이는 `width`와 `height`가 서로 관련되어 있음을 나타내며, 값에 설명적인 이름을 사용하여 튜플 인덱스 값 `0`과 `1`을 사용하는 것보다 명확합니다.

### 유용한 기능 추가: 파생된 트레이트

`Rectangle` 인스턴스를 출력하여 프로그램을 디버깅하는 동안 값을 확인하는 것이 유용할 것입니다. 5-11번 목록은 이전 장에서 사용했던 `println!` 매크로를 사용하려고 시도합니다. 그러나 이것은 작동하지 않습니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/src/main.rs}}
```

<span class=\"caption\">Listing 5-11: `Rectangle` 인스턴스를 출력하려는 시도</span>

이 코드를 컴파일하면 다음과 같은 오류 메시지가 나타납니다.

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:3}}
```

`println!` 매크로는 다양한 형식을 지원하며, 기본적으로 중괄호는 `Display` 형식이라고 하는 형식을 사용하도록 지시합니다. `Display` 형식은 직접 사용자에게 보여줄 출력을 의미합니다. 우리가 지금까지 본 기본형은 `1`이나 다른 기본형을 사용자가 볼 때 원하는 유일한 방식으로 표시되기 때문에 기본적으로 `Display`를 구현합니다. 하지만 구조체의 경우 `println!`이 출력을 어떻게 형식화해야 하는지 불분명합니다. 쉼표를 사용할지, 중괄호를 사용할지, 모든 필드를 표시할지 등 여러 가지 가능성이 있습니다. 이러한 모호성 때문에 Rust는 무엇을 원하는지 추측하지 않으며, `println!`과 `{}` 플레이스홀더를 사용하여 구조체에 기본 `Display` 구현이 없습니다.

오류 메시지를 계속 읽으면 다음과 같은 유용한 메시지가 있습니다.

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:9:10}}
```

이를 시도해 보겠습니다! `println!` 매크로 호출은 이제 `println!(\"rect1 is
{rect1:?}\");`와 같이 보입니다. `:?`을 중괄호 안에 넣으면 `println!`에 `Debug` 형식이라고 하는 형식을 사용하도록 지시합니다. `Debug` 트레이트는 개발자가 코드를 디버깅하는 동안 구조체의 값을 볼 수 있도록 출력하는 데 사용됩니다.

이 변경 사항으로 코드를 컴파일해 보세요. 아직 오류가 발생합니다.

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:3}}
```

하지만 컴파일러는 다시 도움이 되는 메시지를 제공합니다.

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:9:10}}
```

Rust는 디버깅 정보를 출력하는 기능을 포함하고 있지만, 이 기능을 구조체에 명시적으로 적용해야 합니다. 구조체 정의 바로 앞에 `#[derive(Debug)]`라는 외부 속성을 추가하면 이를 수행할 수 있습니다. 5-12번 목록에서 보여주는 것처럼.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
```

<span class=\"caption\">Listing 5-12: `Debug` 트레이트를 추가하여 `Rectangle` 인스턴스를 디버그 형식으로 출력</span>

이제 프로그램을 실행하면 오류가 발생하지 않고 다음과 같은 출력을 볼 수 있습니다.

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/output.txt}}
```

좋습니다! 가장 아름다운 출력은 아니지만, 이 인스턴스의 모든 필드 값이 표시되어 디버깅에 도움이 될 것입니다. 더 큰 구조체를 사용하는 경우 출력이 읽기 쉽도록 하는 것이 유용합니다. 그러한 경우 `println!` 문자열에서 `{:#?}` 대신 `{:?}`을 사용할 수 있습니다. 이 예제에서 `{:#?}` 스타일을 사용하면 다음과 같은 출력이 생성됩니다.

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-02-pretty-debug/output.txt}}
```

`Debug` 형식을 사용하여 값을 출력하는 또 다른 방법은 [`dbg!` 매크로][dbg]<!-- ignore -->를 사용하는 것입니다. `dbg!` 매크로는 표현식의 소유권을 가져와서 ( `println!`과 달리 참조를 가져옵니다), 코드에서 해당 `dbg!` 매크로 호출이 발생하는 파일과 줄 번호와 함께 그 표현식의 결과 값을 출력하고, 표현식의 소유권을 반환합니다.

> 주의: `dbg!` 매크로는 표준 오류 콘솔 스트림 (`stderr`)에 출력합니다. `println!`은 표준 출력 콘솔 스트림 (`stdout`)에 출력합니다. `stderr`와 `stdout`에 대해서는 제12장의 \u201cWriting Error Messages to Standard Error Instead of Standard Output\u201d 섹션에서 자세히 설명합니다.[err]<!-- ignore -->.

`width` 필드에 할당되는 값과 `rect1` 전체 구조체의 값에 관심이 있는 경우 다음과 같은 예제를 살펴보겠습니다.

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/src/main.rs}}
```

`dbg!`를 `30 * scale` 표현식 주변에 둘 수 있으며, `dbg!`가 표현식의 소유권을 가져오기 때문에 `width` 필드는 `dbg!` 호출이 없었다면과 같은 값을 가져옵니다. `dbg!`가 `rect1`의 소유권을 가져가지 않으려면 다음 호출에서 `rect1`의 참조를 사용해야 합니다. 이 예제의 출력은 다음과 같습니다.

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/output.txt}}
```

첫 번째 출력은 *src/main.rs*의 10번째 줄에서 `dbg!`를 사용하여 `30 * scale` 표현식을 디버깅했기 때문에 나타났습니다. 그 결과 값은 `60`입니다. (정수에 대한 `Debug` 형식은 값만 출력하도록 구현되었습니다). `rect1`의 값을 출력하는 `dbg!` 호출은 *src/main.rs*의 14번째 줄에서 발생합니다. 이 출력은 `Rectangle` 유형의 예쁘게 포맷된 `Debug` 형식을 사용합니다. `dbg!` 매크로는 코드가 무엇을 하는지 파악하려고 할 때 매우 유용할 수 있습니다!

`Debug` 트레이트 외에도 Rust는 `derive` 속성을 사용하여 사용자 정의 유형에 유용한 동작을 추가할 수 있는 여러 트레이트를 제공합니다. 이러한 트레이트와 그 동작은 [부록 C][app-c]<!-- ignore -->에 나와 있습니다. 사용자 정의 동작으로 이러한 트레이트를 구현하는 방법과 자신의 트레이트를 만드는 방법은 제10장에서 다룹니다. `derive` 외에도 많은 속성이 있습니다. 자세한 내용은 Rust 참조의 \u201cAttributes\u201d 섹션을 참조하십시오.[attributes]

우리의 `area` 함수는 매우 특수합니다. 즉, 직사각형의 면적만 계산합니다. 이 동작을 `Rectangle` 구조체에 더욱 밀접하게 연결하는 것이 도움이 될 것입니다. 다른 유형과 함께 작동하지 않기 때문입니다. `Rectangle` 구조체와 더 밀접하게 연결하여 이 동작을 구현하는 방법을 살펴보겠습니다.

구현된 `area` 함수를 `Rectangle` 유형에 정의된 `area` *메서드*로 변경하여 코드를 재정의합니다.

[tuple-type]: ch03-02-data-types.html#the-tuple-type
[app-c]: appendix-03-derivable-traits.md
[println]: ../std/macro.println.html
[dbg]: ../std/macro.dbg.html
[err]: ch12-06-writing-to-stderr-instead-of-stdout.html
[attributes]: ../reference/attributes.html
