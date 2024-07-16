## 함수

함수는 Rust 코드에서 흔히 사용됩니다. 이미 언어에서 가장 중요한 함수 중 하나인 `main` 함수를 보셨습니다. `main` 함수는 많은 프로그램의 시작점입니다. 또한, 새로운 함수를 선언하는 데 사용되는 `fn` 키워드도 보셨습니다.

Rust 코드는 함수와 변수 이름에 *뱀 케이스*를 사용하는 것이 일반적인 스타일입니다. 모든 문자는 소문자이고, 단어를 구분하는 데 밑줄을 사용합니다. 다음은 함수 정의 예시를 포함하는 프로그램입니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-16-functions/src/main.rs}}
```

Rust에서 함수를 정의하려면 `fn`을 입력한 다음 함수 이름과 괄호를 입력합니다. 중괄호는 컴파일러가 함수 본문의 시작과 끝을 알 수 있도록 합니다.

정의된 모든 함수를 호출할 수 있습니다. `another_function`이 프로그램에 정의되어 있기 때문에 `main` 함수 내에서 호출할 수 있습니다. `another_function`을 소스 코드에서 `main` 함수 *앞*에 정의했을 수도 있습니다. Rust은 함수를 정의하는 위치에 신경 쓰지 않습니다. 호출자에서 볼 수 있는 범위 내에서 정의되어 있으면 됩니다.

함수를 더 자세히 탐구하기 위해 *functions*라는 이름의 새로운 이진 프로젝트를 시작해 보겠습니다. `another_function` 예시를 *src/main.rs*에 넣고 실행합니다. 다음과 같은 출력을 볼 수 있습니다.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-16-functions/output.txt}}
```

줄은 `main` 함수에 있는 순서대로 실행됩니다. 먼저 "Hello, world!" 메시지가 출력되고, 그 다음 `another_function`이 호출되어 메시지가 출력됩니다.

### 매개변수

함수에 *매개변수*를 정의할 수 있습니다. 매개변수는 함수의 서명의 일부인 특별한 변수입니다. 함수에 매개변수가 있으면 해당 매개변수에 대한 구체적인 값을 제공할 수 있습니다. 기술적으로, 구체적인 값은 *인수*라고 불리지만, 캐주얼하게 대화할 때는 함수 정의의 변수나 함수를 호출할 때 전달되는 구체적인 값을 나타내는 *매개변수*와 *인수*라는 단어를 서로 교환하여 사용합니다.

`another_function`의 이 버전에서는 매개변수를 추가합니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/src/main.rs}}
```

이 프로그램을 실행해 보세요. 다음과 같은 출력을 얻을 수 있습니다.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/output.txt}}
```

`another_function`의 선언에는 `x`라는 하나의 매개변수가 있습니다. `x`의 유형은 `i32`로 지정됩니다. `5`를 `another_function`에 전달할 때 `println!` 매크로는 형식 문자열에서 `x`를 포함하는 중괄호 쌍에 `5`를 넣습니다.

함수 서명에서는 각 매개변수의 유형을 *필수로* 선언해야 합니다. 이는 Rust 설계의 의도적인 결정입니다. 함수 정의에서 유형 선언을 요구하는 것은 컴파일러가 코드의 다른 곳에서 어떤 유형을 의미하는지 파악할 때 거의 필요하지 않도록 합니다. 컴파일러는 또한 함수가 기대하는 유형을 알면 더 유용한 오류 메시지를 제공할 수 있습니다.

여러 매개변수를 정의할 때는 쉼표로 매개변수 선언을 구분합니다. 예를 들어 다음과 같습니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/src/main.rs}}
```

이 예제는 `print_labeled_measurement`라는 이름의 함수를 만듭니다. 이 함수는 두 개의 매개변수를 가지고 있습니다. 첫 번째 매개변수는 `value`이고 `i32` 유형입니다. 두 번째 매개변수는 `unit_label`이고 `char` 유형입니다. 함수는 `value`와 `unit_label`를 모두 포함하는 텍스트를 출력합니다.

이 코드를 실행해 보겠습니다. 현재 *functions* 프로젝트의 *src/main.rs* 파일을 위의 예시로 바꾸고 `cargo run`을 사용하여 실행합니다.

```console
```

값이 `5`로 설정되었고 `'h'`로 설정되었기 때문에 프로그램 출력에는 이러한 값이 포함됩니다.

### 문과 표현식

함수 본문은 종종 표현식으로 끝나는 여러 문으로 구성됩니다. 지금까지 다룬 함수는 마지막 표현식을 포함하지 않았지만, 문에서 표현식을 사용한 적이 있습니다. Rust는 표현식 기반 언어이기 때문에 이러한 차이점을 이해하는 것이 중요합니다. 다른 언어는 동일한 차이점을 가지고 있지 않으므로, 문과 표현식이 무엇이며 함수 본문에 미치는 영향을 살펴보겠습니다.

* **문**은 특정 작업을 수행하는 지시이며 값을 반환하지 않습니다.
* **표현식**은 결과 값을 평가합니다. 몇 가지 예를 살펴보겠습니다.

실제로 이미 문과 표현식을 사용했습니다. `let` 키워드로 변수를 생성하고 값을 할당하는 것은 문입니다. 3-1번 목록에서 `let y = 6;`는 문입니다.

<Listing number=\"3-1\" file-name=\"src/main.rs\" caption=\"마지막 문을 포함하는 `main` 함수 선언\">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-01/src/main.rs}}
```

</Listing>

함수 정의 또한 문입니다. 앞서 보았던 전체 예시는 스스로 문입니다.

문은 값을 반환하지 않습니다. 따라서 다음 코드와 같이 `let` 문을 다른 변수에 할당할 수 없습니다. 오류가 발생합니다.

Filename: src/main.rs

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/src/main.rs}}
```

이 프로그램을 실행하면 다음과 같은 오류 메시지가 표시됩니다.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/output.txt}}
```

`let y = 6` 문은 값을 반환하지 않으므로 `x`에 바인딩할 수 있는 값이 없습니다. 이는 C나 Ruby와 같은 다른 언어와 다릅니다. C와 Ruby에서는 `x = y = 6`과 같이 작성할 수 있으며 `x`와 `y` 모두 `6`의 값을 가집니다. Rust에서는 이렇게 할 수 없습니다.

표현식은 값을 평가하고 Rust에서 작성하는 대부분의 코드를 구성합니다. `5 + 6`과 같은 산술 연산은 `11`의 값을 평가하는 표현식입니다. 표현식은 문의 일부일 수 있습니다. 3-1번 목록에서 `let y = 6;` 문에서 `6`은 `6`의 값을 평가하는 표현식입니다. 함수를 호출하는 것은 표현식입니다. 매크로를 호출하는 것도 표현식입니다. 괄호로 묶인 새로운 범위 블록 또한 표현식입니다. 예를 들어 다음과 같습니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-20-blocks-are-expressions/src/main.rs}}
```

이 표현식:

```rust,ignore
{
    let x = 3;
    x + 1
}
```

은 이 경우 `4`를 평가합니다. 이 값이 `y`에 `let` 문의 일부로 바인딩됩니다. `x + 1` 줄에는 마지막에 세미콜론이 없다는 점에 유의하세요. 이는 지금까지 본 대부분의 줄과 다릅니다.
표현식에는 마지막 세미콜론이 포함되지 않습니다. 표현식의 끝에 세미콜론을 추가하면 문으로 변환되고 값을 반환하지 않습니다. 함수 반환 값과 표현식을 탐색할 때 이 점을 기억하세요.

### 반환 값이 있는 함수

함수는 호출하는 코드에 값을 반환할 수 있습니다. 반환 값은 이름을 지정하지 않지만, 화살표(`->`) 뒤에 유형을 선언해야 합니다. Rust에서 함수의 반환 값은 마지막 표현식과 동일합니다.
함수의 몸체 블록에서 표현식을 반환할 수 있습니다. `return` 키워드를 사용하여 값을 지정하여 함수에서 일찍 반환할 수 있지만, 대부분의 함수는 마지막 표현식을 암시적으로 반환합니다. 다음은 값을 반환하는 함수의 예입니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/src/main.rs}}
```

`five` 함수에는 함수 호출, 매크로, 심지어 `let` 문이 없습니다. 단지 `5`라는 숫자만 있습니다. 이것은 Rust에서 완벽하게 유효한 함수입니다. 함수의 반환 유형이 `-> i32`로 지정되어 있는 것을 알 수 있습니다. 이 코드를 실행해 보세요. 출력은 다음과 같습니다.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/output.txt}}
```

`five`에서의 `5`는 함수의 반환 값이며, 바로 그 이유로 반환 유형이 `i32`이기 때문입니다. 자세히 살펴보겠습니다. 두 가지 중요한 부분이 있습니다. 첫째, `let x = five();` 줄은 함수의 반환 값을 사용하여 변수를 초기화하고 있음을 보여줍니다. `five` 함수가 `5`를 반환하기 때문에 이 줄은 다음과 같습니다.

```rust
let x = 5;
```

둘째, `five` 함수는 매개변수가 없으며 반환 값의 유형을 정의하지만, 함수의 몸체는 `;` 없이 홀로 있는 `5`입니다. 이는 반환하려는 값을 가진 표현식이기 때문입니다.

다른 예를 살펴보겠습니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-22-function-parameter-and-return/src/main.rs}}
```

이 코드를 실행하면 `The value of x is: 6`이 출력됩니다. 하지만 `x + 1` 줄에 세미콜론을 추가하여 표현식에서 문장으로 변경하면 오류가 발생합니다.

Filename: src/main.rs

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/src/main.rs}}
```

이 코드를 컴파일하면 다음과 같은 오류가 발생합니다.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/output.txt}}
```

주요 오류 메시지인 `mismatched types`는 이 코드의 핵심 문제를 드러냅니다. `plus_one` 함수의 정의에 따르면 `i32`를 반환해야 하지만, 문장은 값을 평가하지 않으므로 `()` 즉, 단위 유형으로 표현됩니다. 따라서 아무것도 반환되지 않으며, 이는 함수 정의와 모순되어 오류를 발생시킵니다. 이 출력에서 Rust은 이 문제를 해결하는 데 도움이 될 수 있는 메시지를 제공합니다. 즉, 오류를 해결하기 위해 세미콜론을 제거해야 함을 제안합니다.
