## 제어 흐름

어떤 조건이 `true`인지에 따라 일부 코드를 실행하고, 조건이 `true`인 동안 코드를 반복 실행하는 능력은 대부분의 프로그래밍 언어에서 기본적인 구성 요소입니다. Rust 코드의 실행 흐름을 제어하는 데 사용되는 가장 일반적인 구조는 `if` 표현식과 루프입니다.

### `if` 표현식

`if` 표현식은 조건에 따라 코드를 분기하도록 허용합니다. 조건을 제공하고 “만약 이 조건이 충족되면 이 코드 블록을 실행합니다. 조건이 충족되지 않으면 이 코드 블록을 실행하지 않습니다.”라고 명시합니다.

`projects` 디렉토리에 *branches* 라는 이름의 새 프로젝트를 만들어 `if` 표현식을 탐색해 보세요. *src/main.rs* 파일에서 다음을 입력하세요.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/src/main.rs}}
```

모든 `if` 표현식은 `if` 키워드로 시작하여 조건이 옵니다. 이 경우 조건은 변수 `number`가 5보다 작은지 여부를 확인합니다. 조건이 `true`일 경우 실행할 코드 블록을 괄호 안에 넣습니다.
조건에 연결된 코드 블록은 `if` 표현식에서 *arm*이라고도 불립니다. 이는 2장의 “추측을 비밀 숫자와 비교”[comparing-the-guess-to-the-secret-number]<!--
ignore --> 섹션에서 논의한 `match` 표현식의 arm과 마찬가지입니다.

선택적으로, `else` 표현식을 포함할 수도 있습니다. 이 경우 `if` 조건이 `false`이면 프로그램에 실행할 대안 코드 블록을 제공합니다. `else` 표현식을 제공하지 않고 조건이 `false`이면 프로그램은 `if` 블록을 건너뛰고 다음 코드로 이동합니다.

이 코드를 실행해 보세요. 다음과 같은 출력을 볼 수 있습니다.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/output.txt}}
```

조건이 `false`가 되도록 `number` 값을 변경해 보세요.

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/src/main.rs:here}}
```

프로그램을 다시 실행하고 출력을 확인하세요.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/output.txt}}
```

이 코드의 조건은 *반드시* `bool`이어야 합니다. 조건이 `bool`이 아니면 오류가 발생합니다. 예를 들어 다음 코드를 실행해 보세요.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/src/main.rs}}
```

`if` 조건이 이번에는 `3` 값으로 평가되며 Rust는 오류를 발생시킵니다.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/output.txt}}
```

오류는 Rust가 `bool`을 기대했지만 정수를 받았음을 나타냅니다. Ruby나 JavaScript와 같은 언어와 달리 Rust는 비 논리형 타입을 자동으로 논리형으로 변환하려고 하지 않습니다. `if`에 항상 논리형을 명시적으로 제공해야 합니다. 예를 들어 `number`가 `0`이 아닐 때 `if` 코드 블록이 실행되도록 하려면 `if` 표현식을 다음과 같이 변경할 수 있습니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-29-if-not-equal-0/src/main.rs}}
```

이 코드를 실행하면 `number는 0이 아닌 값이었습니다`가 출력됩니다.

#### 여러 조건 처리하기 위한 `else if`

`if`와 `else`를 `else if` 표현식으로 결합하여 여러 조건을 사용할 수 있습니다. 예를 들어:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/src/main.rs}}
```

이 프로그램은 실행될 수 있는 네 가지 경로가 있습니다. 실행한 후에는 다음과 같은 출력을 볼 수 있습니다.

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/output.txt}}
```

이 프로그램이 실행될 때, 차례대로 각 `if` 표현식을 검사하고 조건이 `true`로 평가되는 첫 번째 몸체를 실행합니다. 6이 2로 나누어 떨어진다는 사실에도 불구하고 `number is divisible by 2` 출력을 보지 못하고 `else` 블록에서 `number is not divisible by 4, 3, or 2` 텍스트도 보지 못합니다. 그 이유는 Rust이 첫 번째 `true` 조건에 대해서만 블록을 실행하고, 한 번 찾으면 나머지를 확인하지 않기 때문입니다.

너무 많은 `else if` 표현식을 사용하면 코드가 복잡해질 수 있으므로, 여러 개가 있다면 코드를 재작성하는 것이 좋습니다. 제6장에서는 `match`라는 강력한 Rust 분기 구조를 소개합니다. 이 구조는 이러한 경우에 유용합니다.

#### `if`를 `let` 문에서 사용하기

`if`가 표현식이기 때문에, `let` 문의 오른쪽에 사용하여 결과를 변수에 할당할 수 있습니다. 예를 들어 Listing 3-2를 참조하세요.

<Listing number=\"3-2\" file-name=\"src/main.rs\" caption=\"`if` 표현식의 결과를 변수에 할당하기\">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-02/src/main.rs}}
```

</Listing>

`number` 변수는 `if` 표현식의 결과에 따라 값이 할당됩니다. 이 코드를 실행하여 발생하는 결과를 확인하세요.

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-02/output.txt}}
```

코드 블록은 그 안의 마지막 표현식을 평가하고, 숫자 자체도 표현식입니다. 이 경우, 전체 `if` 표현식의 값은 어느 코드 블록이 실행되는지에 따라 달라집니다. 즉, `if`의 각 팔에서 결과가 될 수 있는 값의 유형은 동일해야 합니다. Listing 3-2의 경우, `if` 팔과 `else` 팔의 결과 모두 `i32` 정수형입니다. 유형이 일치하지 않으면 다음과 같은 예에서 오류가 발생합니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/src/main.rs}}
```

이 코드를 컴파일하려고 할 때 오류가 발생합니다. `if` 및 `else` 팔의 값 유형이 일치하지 않기 때문입니다. Rust는 컴파일 시간에 `number` 변수의 유형을 정확히 알아야 하며, 이를 통해 `number`가 사용되는 모든 곳에서 유형이 유효하다는 것을 확인할 수 있습니다. `number`의 유형이 런타임에만 결정된다면, 컴파일러가 코드에 대한 복잡성을 증가시키고, 여러 가설적인 유형을 추적해야 하기 때문에 코드에 대한 보장을 줄여야 합니다.

### 반복문

코드 블록을 여러 번 실행하는 것은 자주 필요합니다. 이 작업을 위해 Rust는 여러 종류의 *반복문*을 제공합니다. 반복문은 루프 내부의 코드를 반복적으로 실행하고, 루프의 끝에 도달하면 다시 처음부터 시작합니다. 반복문을 시험하기 위해 *loops*라는 새로운 프로젝트를 만들어 보겠습니다.

Rust에는 `loop`, `while`, `for` 세 가지 종류의 반복문이 있습니다. 각 반복문을 시도해 보겠습니다.

#### `loop`를 사용하여 코드 반복하기

 `loop` 키워드는 Rust가 코드 블록을 반복적으로 실행하도록 지시하는 것입니다.

예를 들어, `loops` 디렉토리의 `src/main.rs` 파일을 다음과 같이 변경해 보세요.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-loop/src/main.rs}}
```

이 프로그램을 실행하면 `again!` 가 계속해서 출력되며, 프로그램을 수동으로 중지할 때까지 계속됩니다.
대부분의 터미널은 프로그램이 계속되는 루프에서 중지하는 데 사용되는 키 조합 <kbd>ctrl</kbd>-<kdb>c</kbd>를 지원합니다. 시도해 보세요:

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-32-loop
cargo run
CTRL-C
-->

```console
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished dev [unoptimized + debuginfo] target(s) in 0.29s
     Running `target/debug/loops`
again!
again!
again!
again!
^Cagain!
```

`^C` 기호는 <kbd>ctrl</kbd>-<kbd>c</kbd> 키를 누른 위치를 나타냅니다. 루프에서 중지 신호를 받았을 때 코드가 어디에 있었는지에 따라 `again!` 가 출력될지 여부는 다를 수 있습니다.

다행히 Rust는 코드를 사용하여 루프에서 빠져나갈 수 있는 방법도 제공합니다. `break` 키워드를 루프 안에 넣으면 프로그램이 루프를 실행하는 것을 언제 중지해야 하는지 알 수 있습니다.

이전에 챕터 2의 [“정답을 맞추면 종료”][quitting-after-a-correct-guess]<!-- ignore
--> 섹션에서 사용했던 추측 게임에서도 이를 사용했습니다. 사용자가 정답을 맞춰 게임에서 이겼을 때 프로그램을 종료하기 위해 사용했습니다.

또한 추측 게임에서 `continue`를 사용했는데, 루프에서 코드를 실행하는 것을 건너뛰고 다음 반복으로 이동하도록 프로그램에 지시합니다.

#### 루프에서 값 반환하기

`loop` 중 하나의 사용법은 작업이 실패할 수 있는 경우를 반복하여 시도하는 것입니다. 예를 들어, 스레드가 작업을 완료했는지 확인하는 것입니다. 또한 작업의 결과를 루프에서 나머지 코드로 전달해야 할 수도 있습니다. 이를 위해 `break` 표현식 뒤에 반환하려는 값을 추가할 수 있습니다. 이 값은 루프에서 반환되어 나머지 코드에서 사용할 수 있습니다. 다음은 예입니다.

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-33-return-value-from-loop/src/main.rs}}
```

루프 전에 `counter` 라는 변수를 선언하고 0으로 초기화합니다. `result` 라는 변수를 루프에서 반환되는 값을 저장하기 위해 선언합니다. 루프의 각 반복에서 `counter` 변수에 1을 더하고 `counter`가 10과 같으면 `break` 키워드를 사용하여 `counter * 2` 값을 사용합니다. 루프 후에는 `result` 변수에 값을 할당하는 문장의 끝에 세미콜론을 사용합니다. 마지막으로 `result`에 있는 값을 출력합니다. 이 경우 `20`입니다.

루프에서 `return`을 사용할 수도 있습니다. `break`는 현재 루프에서만 나가지만, `return`은 항상 현재 함수에서 나갑니다.

#### 루프 레이블을 사용하여 여러 루프를 구분하기

루프 안에 루프가 있으면 `break`와 `continue`는 해당 위치의 가장 안쪽 루프에 적용됩니다. `break` 또는 `continue` 키워드가 해당 루프에 적용되도록 지정하려면 루프에 옵셔널한 *루프 레이블*을 지정할 수 있습니다. 루프 레이블은 싱글 쿼우트로 시작해야 합니다. 다음은 두 개의 중첩된 루프의 예입니다.

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/src/main.rs}}
```

외부 루프는 `'counting_up` 레이블을 가지고 있으며 0부터 2까지 세어갑니다. 레이블이 없는 내부 루프는 10에서 9까지 세어갑니다. 첫 번째 `break`는 `'counting_up` 레이블을 가진 외부 루프를 중지합니다.

문서에 레이블이 지정되지 않으면 내부 루프만 종료됩니다. `break
'counting_up;` 문은 외부 루프를 종료합니다. 이 코드는 다음과 같이 출력됩니다.

```console
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/output.txt}}
```

#### 조건 루프와 `while`

프로그램은 종종 루프 내에서 조건을 평가해야 합니다. 조건이 `true`일 때 루프가 실행됩니다. 조건이 더 이상 `true`가 아니면 프로그램은 `break`를 호출하여 루프를 중단합니다. `loop`, `if`, `else`, `break`의 조합을 사용하여 이러한 동작을 구현할 수 있습니다. 이제 프로그램에서 시도해 볼 수 있습니다. 그러나 이 패턴이 너무 일반적이기 때문에 Rust에는 `while` 루프라고 불리는 내장 언어 구문이 있습니다. 3-3번 목록에서 `while`을 사용하여 프로그램을 세 번 실행하고 각 번에 카운트를 내리고, 그 후 메시지를 출력하고 종료합니다.

<Listing number=\"3-3\" file-name=\"src/main.rs\" caption=\"`while` 루프를 사용하여 조건이 참일 때 코드를 실행\">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-03/src/main.rs}}
```

</Listing>

이 구문은 `loop`, `if`, `else`, `break`를 사용했을 때 필요했던 많은 중첩을 제거하고 더 명확합니다. 조건이 `true`일 때 코드가 실행되며, 그렇지 않으면 루프를 종료합니다.

#### `for` 루프를 사용하여 수집을 반복

`while` 구문을 사용하여 배열과 같은 수집의 요소를 반복할 수도 있습니다. 예를 들어, 3-4번 목록의 루프는 배열 `a`의 각 요소를 출력합니다.

<Listing number=\"3-4\" file-name=\"src/main.rs\" caption=\"`while` 루프를 사용하여 수집의 각 요소를 반복\">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-04/src/main.rs}}
```

</Listing>

여기서 코드는 배열의 요소를 순회합니다. 인덱스 `0`부터 시작하고, 배열의 마지막 인덱스(즉, `index < 5`가 더 이상 `true`가 아닐 때)까지 루프를 반복합니다. 이 코드를 실행하면 배열의 모든 요소가 출력됩니다.

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-04/output.txt}}
```

모든 다섯 배열 값이 기대대로 콘솔에 나타납니다. `index`가 어느 시점에 값 `5`를 달성하더라도, 루프는 배열에서 여섯 번째 값을 가져오려고 시도하기 전에 실행을 중단합니다.

그러나 이 접근 방식은 오류 취약합니다. 인덱스 값이나 테스트 조건이 잘못된 경우 프로그램이 오류를 일으킬 수 있습니다. 예를 들어, `a` 배열의 정의를 네 개의 요소로 변경했지만 `while index < 4` 조건을 업데이트하지 않았다면 코드가 오류를 일으킬 것입니다. 또한, 배열의 인덱스 범위 내에서인지 여부를 매 반복 시점에 실행 시간 코드를 추가하기 때문에 느립니다.

보다 간결한 대안으로 `for` 루프를 사용하여 수집의 각 항목에 대해 코드를 실행할 수 있습니다. `for` 루프는 3-5번 목록의 코드와 같습니다.

<Listing number=\"3-5\" file-name=\"src/main.rs\" caption=\"`for` 루프를 사용하여 수집의 각 요소를 반복\">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-05/src/main.rs}}
```

</Listing>

이 코드를 실행하면 3-4번 목록과 동일한 출력이 나타납니다. 더 중요한 것은 `for` 루프를 사용하면 배열의 값 수를 변경하더라도 다른 코드를 변경할 필요가 없다는 것입니다.

`for` 루프의 안전성과 간결성은 `for` 루프를 가장 일반적으로 사용되는 루프로 만듭니다.

Rust에서 코드를 작성하는 방법을 배우셨습니다. `while` 루프를 사용하여 카운트다운 예제와 같이 특정 횟수 동안 코드를 실행하고 싶은 경우에도, 대부분의 Rust 개발자들은 `for` 루프를 사용합니다. 이를 위해서는 표준 라이브러리에서 제공되는 `Range`를 사용하는 것이며, `Range`는 특정 숫자부터 시작하여 다른 숫자보다 작은 숫자까지 순차적으로 모든 숫자를 생성합니다.

다음은 `for` 루프와 아직 배우지 않은 또 다른 방법인 `rev`를 사용하여 범위를 뒤집어 카운트다운을 표현하는 방법입니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-34-for-range/src/main.rs}}
```

이 코드는 좀 더 보기 좋지 않나요?

## 요약

이제 마무리했습니다! 이 장은 상당히 깁니다. 변수, 스칼라 및 복합 데이터 유형, 함수, 주석, `if` 표현식 및 루프에 대해 배웠습니다. 이 장에서 다룬 개념을 연습하려면 다음과 같은 프로그램을 작성해 보세요.

* 화씨에서 섭씨로, 섭씨에서 화씨로 온도를 변환합니다.
* *n*번째 피보나치 수를 생성합니다.
* 크리스마스 캐롤 “The Twelve Days of Christmas”의 가사를 출력하고 노래의 반복을 활용합니다.

준비가 되면 Rust에서 다른 프로그래밍 언어에서 일반적으로 존재하지 않는 개념인 소유권에 대해 알아보겠습니다.

[comparing-the-guess-to-the-secret-number]:
ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[quitting-after-a-correct-guess]:
ch02-00-guessing-game-tutorial.html#quitting-after-a-correct-guess
