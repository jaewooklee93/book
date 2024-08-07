## 데이터 유형

Rust에서 모든 값은 특정 *데이터 유형*을 가지고 있으며, 이는 Rust가 해당 데이터의 종류를 알고 따라서 그 데이터를 어떻게 처리해야 하는지 알 수 있도록 합니다. 두 가지 데이터 유형 부분을 살펴보겠습니다: 스칼라 유형과 복합 유형.

Rust는 *정적 타입* 언어이므로 컴파일 시간에 모든 변수의 유형을 알아야 한다는 점을 기억하십시오. 컴파일러는 일반적으로 값과 그 값을 사용하는 방식을 기반으로 우리가 사용하려는 유형을 추론할 수 있습니다. `String`을 `parse`를 사용하여 숫자 유형으로 변환한 경우와 같이 여러 유형이 가능한 경우 (예: 제2장의 “추측을 비밀 숫자와 비교”][comparing-the-guess-to-the-secret-number]<!-- ignore --> 섹션), 우리는 유형 어노테이션을 추가해야 합니다. 예를 들어 다음과 같습니다.

```rust
let guess: u32 = \"42\".parse().expect(\"Not a number!\");
```

앞서 보여진 `: u32` 유형 어노테이션을 추가하지 않으면 Rust는 다음과 같은 오류를 표시하며, 컴파일러가 어떤 유형을 사용해야 하는지에 대한 추가 정보가 필요하다는 것을 의미합니다.

```console
{{#include ../listings/ch03-common-programming-concepts/output-only-01-no-type-annotations/output.txt}}
```

다른 데이터 유형에 대해서는 다른 유형 어노테이션을 사용할 것입니다.

### 스칼라 유형

*스칼라* 유형은 하나의 값을 나타냅니다. Rust에는 정수, 부동 소수점 숫자, 불리언, 문자 등 네 가지 주요 스칼라 유형이 있습니다. 다른 프로그래밍 언어에서도 이러한 유형을 알고 있을 것입니다. Rust에서 작동 방식을 살펴보겠습니다.

#### 정수 유형

*정수*는 소수 부분이 없는 숫자입니다. 제2장에서 `u32` 유형이라는 정수 유형을 사용했습니다. 이 유형 선언은 해당 값이 32비트를 사용하는 무符号 정수(정수 유형은 `i`로 시작하는 대신 `u`로 시작합니다)여야 함을 나타냅니다. 표 3-1은 Rust의 내장 정수 유형을 보여줍니다. 이러한 변형 중 어느 것을 사용하여 정수 값의 유형을 선언할 수 있습니다.

표 3-1: Rust의 정수 유형

| 길이  | 부호 있는 | 부호 없는 |
|---------|---------|----------|
| 8비트   | `i8`    | `u8`     |
| 16비트  | `i16`   | `u16`    |
| 32비트  | `i32`   | `u32`    |
| 64비트  | `i64`   | `u64`    |
| 128비트 | `i128`  | `u128`   |
| 아키텍처 | `isize` | `usize`  |

각 변형은 부호 있는 또는 부호 없는 형태로, 명시적인 크기를 가질 수 있습니다. *부호 있는*과 *부호 없는*은 숫자가 음수일 수 있는지 여부를 나타냅니다. 즉, 숫자에 부호(부호 있는)가 필요한지 여부 또는 항상 양수이므로 부호 없이 표현할 수 있는지 여부입니다. 숫자를 종이에 적는 것과 같습니다. 부호가 중요한 경우 숫자는 플러스 기호 또는 마이너스 기호와 함께 표시되지만, 숫자가 양수라고 가정할 수 있는 경우에는 부호 없이 표시됩니다.
부호 있는 숫자는 [2의 보수법][twos-complement]<!-- ignore --> 표현 방식으로 저장됩니다.

각 부호 있는 변형은 -(2<sup>n - 1</sup>)에서 2<sup>n - 1</sup> - 1까지의 숫자를 저장할 수 있으며, *n*은 해당 변형이 사용하는 비트 수입니다. 따라서 `i8`은 -(2<sup>7</sup>)에서 2<sup>7</sup> - 1까지의 숫자를 저장할 수 있으며, 이는 -128에서 127입니다. 부호 없는 변형은 0에서 2<sup>n</sup> - 1까지의 숫자를 저장할 수 있으므로 `u8`은 0에서 2<sup>8</sup> - 1까지의 숫자를 저장할 수 있으며, 이는 0에서 255입니다.

또한 `isize`와 `usize` 유형은 프로그램이 실행되는 컴퓨터의 아키텍처에 따라 달라집니다. 표에서 “아키텍처”로 표시된 것처럼 64비트 아키텍처의 경우 64비트이고 32비트 아키텍처의 경우 32비트입니다.

정수 리터럴은 표 3-2에 나열된 형식 중 어느 하나로 작성할 수 있습니다.


숫자 리터럴은 여러 유형의 숫자를 나타낼 수 있으며, 유형 접미사(예: `57u8`)를 사용하여 유형을 지정할 수 있습니다. 숫자 리터럴은 또한 `_`를 시각적 구분자로 사용하여 숫자를 더 쉽게 읽을 수 있습니다. 예를 들어 `1_000`은 `1000`과 같은 값을 가집니다.

표 3-2: Rust의 정수 리터럴

| 숫자 리터럴 | 예시 |
|---|---|
| 십진수 | `98_222` |
| 16진수 | `0xff` |
| 8진수 | `0o77` |
| 2진수 | `0b1111_0000` |
| 바이트 (`u8`만) | `b'A'` |

어떤 유형의 정수를 사용해야 할지 몰라요? 불확실하다면, Rust의 기본값은 일반적으로 좋은 시작점입니다. 정수 유형은 기본적으로 `i32`로 설정됩니다. `isize` 또는 `usize`을 사용해야 하는 주요 상황은 어떤 컬렉션을 인덱싱할 때입니다.

> ##### 정수 오버플로우
>
> `u8` 유형의 변수가 0부터 255까지의 값을 저장할 수 있다고 가정해 보겠습니다. 256과 같은 범위를 벗어나는 값으로 변수를 변경하려고 하면 *정수 오버플로우*가 발생합니다. 이는 프로그램이 실행 중에 *멈추게* 되는 결과를 초래할 수 있습니다. Rust는 프로그램이 오류로 종료될 때 *멈춤*이라고 합니다. 멈춤에 대해서는 9장의 "복구 불가능한 오류와 `panic!`"[unrecoverable-errors-with-panic]<!-- ignore --> 섹션에서 자세히 설명합니다.
>
> `--release` 플래그로 릴리즈 모드로 컴파일할 때 Rust는 오버플로우 검사를 수행하지 않습니다. 오버플로우가 발생하면 Rust는 *2의 보수 포장*을 수행합니다. 간단히 말해서, 유형이 저장할 수 있는 최대 값보다 큰 값은 유형이 저장할 수 있는 최소 값으로 *돌아갑니다*. `u8`의 경우, 256은 0이 되고, 257은 1이 되고, 이런 식으로 계속됩니다. 프로그램은 멈추지 않지만, 변수의 값이 예상치와 다를 수 있습니다. 정수 오버플로우의 포장 행동에 의존하는 것은 오류로 간주됩니다.
>
> 오버플로우 가능성을 명시적으로 처리하려면 기본 숫자 유형에 제공되는 표준 라이브러리 함수를 사용할 수 있습니다.
>
> * 모든 모드에서 포장하는 `wrapping_*` 함수, 예를 들어 `wrapping_add`를 사용합니다.
> * 오버플로우가 있는 경우 `None` 값을 반환하는 `checked_*` 함수를 사용합니다.
> * 오버플로우 여부를 나타내는 불리언과 함께 값을 반환하는 `overflowing_*` 함수를 사용합니다.
> * 최소 또는 최대 값에 포화하는 `saturating_*` 함수를 사용합니다.

#### 부동 소수점 유형

Rust에는 소수점이 있는 숫자를 나타내는 두 가지 기본 유형이 있습니다. Rust의 부동 소수점 유형은 32비트인 `f32`와 64비트인 `f64`입니다. 기본 유형은 `f64`입니다. 왜냐하면 현대 CPU에서 `f32`와 동일한 속도로 실행되지만 더 정확한 값을 표현할 수 있기 때문입니다. 모든 부동 소수점 유형은 부호가 있습니다.

다음 코드는 부동 소수점 숫자를 사용하는 방법을 보여줍니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-06-floating-point/src/main.rs}}
```

부동 소수점 숫자는 IEEE-754 표준에 따라 표현됩니다. `f32` 유형은 단일 정밀도 부동 소수점이고, `f64`는 이중 정밀도입니다.

#### 숫자 연산

Rust는 모든 숫자 유형에 대해 기대하는 기본적인 수학 연산을 지원합니다. 덧셈, 뺄셈, 곱셈, 나눗셈, 나머지 연산입니다. 정수 나눗셈은 0으로 가까운 값으로 잘립니다. 다음 코드는 각 숫자 연산을 `let` 문에서 사용하는 방법을 보여줍니다.

파일 이름: src/main.rs

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-07-numeric-operations/src/main.rs}}
```

이러한 문장의 각 표현식은 수학 연산자를 사용하며, 하나의 값으로 평가되어 변수에 바인딩됩니다. [부록
B][appendix_b]<!-- ignore -->는 Rust가 제공하는 모든 연산자의 목록이 포함되어 있습니다.

#### 불린형

대부분의 다른 프로그래밍 언어와 마찬가지로 Rust의 불린형에는 `true`와 `false`의 두 가지 가능한 값이 있습니다. 불린형은 1 바이트 크기입니다. Rust의 불린형은 `bool`로 지정됩니다. 예를 들어:

파일 이름: src/main.rs

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-08-boolean/src/main.rs}}
```

불린 값을 사용하는 주요 방법은 `if` 표현식과 같은 조건문을 통해서입니다. `if` 표현식이 Rust에서 어떻게 작동하는지에 대한 자세한 내용은 [“제어 흐름”][control-flow]<!-- ignore --> 섹션에서 확인하십시오.

#### 문자형

Rust의 `char`형은 언어의 가장 기본적인 알파벳 형입니다. `char` 값을 선언하는 몇 가지 예는 다음과 같습니다.

파일 이름: src/main.rs

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-09-char/src/main.rs}}
```

참고로, Rust의 `char` 값은 문자열 리터럴과 달리 싱글 쿼우트로 지정합니다. Rust의 `char`형은 4 바이트 크기이며 Unicode Scalar Value를 나타냅니다. 즉, ASCII 외에도 많은 것을 나타낼 수 있습니다. 억양이 있는 글자, 중국어, 일본어, 한국어 문자, emoji, 빈 공간 등이 Rust의 `char` 값으로 유효합니다. Unicode Scalar Value는 `U+0000`부터 `U+D7FF` 및 `U+E000`부터 `U+10FFFF`까지를 포함합니다. 그러나 “문자”라는 개념은 Unicode에서 정확히 정의되지 않았기 때문에, “문자”라는 Rust의 `char` 값과의 인식이 일치하지 않을 수 있습니다. 이 주제에 대해서는 8장의 [“UTF-8 인코딩된 문자열을 저장하는 방법”][strings]<!-- ignore -->에서 자세히 설명합니다.

### 복합형

*복합형*은 여러 값을 하나의 형으로 그룹화할 수 있습니다. Rust에는 튜플과 배열이라는 두 가지 기본 복합형이 있습니다.

#### 튜플형

A *튜플*은 다양한 형의 여러 값을 하나의 복합형으로 그룹화하는 일반적인 방법입니다. 튜플은 고정 길이를 가지므로 선언되면 크기가 변경될 수 없습니다.

우리는 괄호 안에 쉼표로 구분된 값 목록을 작성하여 튜플을 만듭니다. 튜플의 각 위치는 유형을 가지며, 튜플 내의 다양한 값의 유형은 동일하지 않을 수 있습니다. 예를 들어, 다음과 같이 유형 지정을 추가했습니다.

파일 이름: src/main.rs

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-10-tuples/src/main.rs}}
```

변수 `tup`은 전체 튜플에 바인딩됩니다. 튜플은 하나의 복합 요소로 간주되기 때문입니다. 튜플에서 개별 값을 가져오려면 패턴 매칭을 사용하여 튜플 값을 해체할 수 있습니다. 예를 들어 다음과 같습니다.

파일 이름: src/main.rs

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-11-destructuring-tuples/src/main.rs}}
```

이 프로그램은 먼저 튜플을 생성하고 변수 `tup`에 바인딩합니다. 그런 다음 `let`을 사용하는 패턴으로 `tup`을 가져와 세 개의 별도의 변수 `x`, `y`, `z`로 만듭니다. 이를 *해체*라고 합니다. 즉, 하나의 튜플을 세 부분으로 분해하는 것입니다. 마지막으로 프로그램은 `y` 값을 출력합니다. `y` 값은 `6.4`입니다.

또한 점 (`.`)를 사용하여 원하는 값의 인덱스를 따라 튜플 요소에 직접 액세스할 수 있습니다. 예를 들어 다음과 같습니다.

파일 이름: src/main.rs

```rust
## 데이터 유형"
}이 코드는 성공적으로 컴파일됩니다. `cargo run`을 사용하여 이 코드를 실행하고
`0`, `1`, `2`, `3`, 또는 `4`를 입력하면 프로그램은 배열에서 해당 인덱스에 있는 값을 출력합니다.
대신 배열의 끝을 넘는 숫자, 예를 들어 `10`을 입력하면 다음과 같은 출력을 얻게 됩니다.

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access
cargo run
10
-->

```console
thread 'main' panicked at src/main.rs:19:19:
index out of bounds: the len is 5 but the index is 10
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

프로그램은 유효하지 않은 값을 인덱싱 연산에 사용한 지점에서 *런타임* 오류가 발생했습니다.
프로그램은 오류 메시지와 함께 종료되었으며 마지막 `println!` 문을 실행하지 않았습니다. 인덱싱을 사용하여 요소에 액세스할 때 Rust은 지정한 인덱스가 배열 길이보다 작은지 확인합니다.
인덱스가 길이보다 크거나 같으면 Rust는 panic합니다. 이 확인은 특히 이 경우 런타임에 있어야 하며, 컴파일러는 코드를 실행할 때 사용자가 어떤 값을 입력할지 알 수 없기 때문입니다.

이것은 Rust의 메모리 안전성 원칙이 작동하는 예입니다. 많은 저수준 언어에서는 이러한 종류의 확인이 수행되지 않으며, 잘못된 인덱스를 제공하면 무효한 메모리에 액세스할 수 있습니다.
Rust는 메모리 액세스를 허용하지 않고 즉시 종료함으로써 이러한 유형의 오류로부터 보호합니다.
제9장에서는 Rust의 오류 처리에 대해 더 자세히 설명하고, panic하지 않고 유효하지 않은 메모리 액세스를 허용하지 않는 읽기 쉬운 안전한 코드를 작성하는 방법을 다룹니다.

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[twos-complement]: https://en.wikipedia.org/wiki/Two%27s_complement
[control-flow]: ch03-05-control-flow.html#control-flow
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[stack-and-heap]: ch04-01-what-is-ownership.html#the-stack-and-the-heap
[vectors]: ch08-01-vectors.html
[unrecoverable-errors-with-panic]: ch09-01-unrecoverable-errors-with-panic.html
[appendix_b]: appendix-02-operators.md
