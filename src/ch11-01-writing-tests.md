## 테스트 작성 방법

테스트는 Rust 함수로, 비테스트 코드가 예상대로 작동하는지 확인합니다.
테스트 함수의 본문은 일반적으로 다음 세 가지 작업을 수행합니다.

* 필요한 데이터 또는 상태를 설정합니다.
* 테스트하려는 코드를 실행합니다.
* 결과가 기대하는 대로인지 확인합니다.

Rust가 테스트 작성에 제공하는 기능을 살펴보겠습니다. 이 기능에는 `test` 속성, 몇 가지 매크로, 그리고 `should_panic` 속성이 포함됩니다.

### 테스트 함수의 구조

Rust에서 가장 간단한 테스트는 `test` 속성으로 표시된 함수입니다.
속성은 Rust 코드에 대한 메타데이터입니다. 예를 들어, 5장에서 구조체와 함께 사용한 `derive` 속성은 그 예시입니다. 함수를 테스트 함수로 변경하려면 `fn` 줄 바로 앞에 `#[test]`를 추가합니다. `cargo test` 명령어로 테스트를 실행하면 Rust는 테스트 함수를 실행하고 각 테스트 함수가 통과하거나 실패하는지 보고합니다.

Cargo로 새 라이브러리 프로젝트를 만들 때마다 테스트 모듈이 자동으로 생성되며, 테스트 함수가 포함되어 있습니다. 이 모듈은 테스트를 작성할 때 필요한 정확한 구조와 문법을 템플릿으로 제공하여 매번 새 프로젝트를 시작할 때 찾아보지 않아도 됩니다. 테스트 함수와 테스트 모듈을 추가할 수 있습니다!

테스트가 어떻게 작동하는지 이해하기 위해 템플릿 테스트를 실험해 보겠습니다. 그런 다음 작성한 코드를 테스트하고 동작이 올바른지 확인하는 실제 테스트를 작성할 것입니다.

`adder`라는 이름의 새 라이브러리 프로젝트를 만들어 두 개의 숫자를 더하는 코드를 작성해 보겠습니다.

```console
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

`adder` 라이브러리의 *src/lib.rs* 파일의 내용은 11-1번 목록과 같습니다.

<Listing number=\"11-1\" file-name=\"src/lib.rs\" caption=\"`cargo new`가 자동으로 생성한 코드\">

<!-- manual-regeneration
cd listings/ch11-writing-automated-tests
rm -rf listing-11-01
cargo new listing-11-01 --lib --name adder
cd listing-11-01
echo \"$ cargo test\" > output.txt
RUSTFLAGS=\"-A unused_variables -A dead_code\" RUST_TEST_THREADS=1 cargo test >> output.txt 2>&1
git diff output.txt # commit any relevant changes; discard irrelevant ones
cd ../../..
-->

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

</Listing>

지금은 `it_works` 함수에만 집중해 보겠습니다. `#[test]` 속성을 확인하십시오. 이 속성은 이 함수가 테스트 함수임을 나타내므로 테스트 런너가 이 함수를 테스트로 처리하도록 알립니다. `tests` 모듈에는 일반적인 시나리오를 설정하거나 일반적인 작업을 수행하는 비테스트 함수가 있을 수도 있습니다. 따라서 테스트 함수인지 여부를 명확히 나타내는 것이 중요합니다.

예시 함수 본문은 `assert_eq!` 매크로를 사용하여 `result` (2와 2를 더한 결과)가 4와 같다는 것을 확인합니다. 이러한 주장은 일반적인 테스트 형식의 예입니다. 실행하여 이 테스트가 통과하는지 확인해 보겠습니다.

`cargo test` 명령어는 프로젝트의 모든 테스트를 실행합니다. 11-2번 목록에 나와 있는 것처럼 테스트가 컴파일되고 실행됩니다.

<Listing number=\"11-2\" caption=\"자동으로 생성된 테스트를 실행한 결과\">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-01/output.txt}}
```

</Listing>

Cargo는 테스트를 컴파일하고 실행했습니다. `running 1 test` 라인을 볼 수 있습니다. 다음 줄은 `tests::it_works`라는 생성된 테스트 함수의 이름을 보여주며, 이 함수를 실행한 결과가 `ok`임을 나타냅니다. `test result: ok.` 라는 전체 요약은 모든 테스트가 통과했다는 것을 의미하며, `1 passed; 0 failed` 라는 부분은 통과하거나 실패한 테스트 수를 총합합니다.

특정 인스턴스에서 실행되지 않도록 테스트를 무시할 수 있습니다. 이를 `#[ignore]` 속성을 사용하여 설정할 수 있습니다. 이를 자세히 설명하는 내용은 이 장의 [“특정 요청이 없는 한 테스트를 무시하는 방법”][ignoring]<!-- ignore --> 섹션에서 다룹니다. 테스트를 무시하지 않는 한, 테스트를 실행하고 결과를 확인하는 것은 Rust 프로그램을 개발하는 데 필수적인 부분입니다.

아직 해보지 않았습니다. 요약은 `0 무시됨`을 보여줍니다.

`0 측정됨` 통계는 성능을 측정하는 벤치마크 테스트에 대한 것입니다.
벤치마크 테스트는 현재까지 밤용 Rust에서만 사용 가능합니다. 자세한 내용은
[벤치마크 테스트에 대한 설명][bench]을 참조하십시오.

`cargo test` 명령어에 인수를 전달하여 이름이 문자열과 일치하는 테스트만 실행할 수 있습니다. 이를 *필터링*이라고 하며, 이는
[“테스트 이름별 실행”][subset]<!-- ignore --> 섹션에서 다룰 것입니다. 여기서는 테스트를 필터링하지 않았으므로 요약의 끝 부분에 `0
필터링됨`이 표시됩니다.

`Doc-tests adder`로 시작하는 테스트 출력의 다음 부분은 모든 문서 테스트의 결과입니다. 아직 문서 테스트가 없지만, Rust는 API 문서에 나타나는 모든 코드 예제를 컴파일할 수 있습니다.
이 기능은 문서와 코드를 동기화하는 데 도움이 됩니다! [“문서 코멘트를 테스트로 사용하기”][doc-comments]<!-- ignore --> 챕터 14의 섹션에서 문서 테스트를 작성하는 방법에 대해 설명하겠습니다. 지금은 `Doc-tests` 출력을 무시하겠습니다.

자, 테스트를 우리의 필요에 맞게 사용하도록 변경해 보겠습니다. 먼저 `it_works` 함수의 이름을 `exploration`과 같은 다른 이름으로 변경하십시오.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/src/lib.rs}}
```

다음으로 `cargo test`를 다시 실행하십시오. 출력은 이제 `exploration` 대신 `it_works`를 보여줍니다:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/output.txt}}
```

이제 `another`라는 이름의 새로운 테스트를 추가하겠습니다. 하지만 이번에는 실패하는 테스트를 만들겠습니다! 테스트는 테스트 함수에서 panic이 발생하면 실패합니다. 각 테스트는 새로운 스레드에서 실행되며, 메인 스레드가 테스트 스레드가 죽었다는 것을 알게 되면 테스트가 실패로 표시됩니다. 9장에서는 `panic!` 매크로를 호출하는 것이 panic하는 가장 간단한 방법이라고 설명했습니다. `another`라는 이름의 새로운 테스트를 추가하여 *src/lib.rs* 파일이 11-3번째 목록과 같도록 하십시오.

<Listing number=\"11-3\" file-name=\"src/lib.rs\" caption=\"`panic!` 매크로를 호출하여 실패하는 두 번째 테스트를 추가\">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-03/src/lib.rs}}
```

</Listing>

다시 `cargo test`를 사용하여 테스트를 실행하십시오. 출력은 11-4번째 목록과 같아야 하며, `exploration` 테스트가 통과하고 `another` 테스트가 실패했습니다.

<Listing number=\"11-4\" caption=\"하나의 테스트가 통과하고 하나의 테스트가 실패할 때 테스트 결과\">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-03/output.txt}}
```

</Listing>

<!-- manual-regeneration
rg panicked listings/ch11-writing-automated-tests/listing-11-03/output.txt
check the line number of the panic matches the line number in the following paragraph
 -->

`ok` 대신 `test tests::another` 줄에는 `FAILED`가 표시됩니다. 각 테스트 실패의 자세한 이유를 표시하는 두 개의 새로운 섹션이 개별 결과와 요약 사이에 나타납니다. 이 경우 `another`가 `panic at 'Make this test fail'`에서 실패했다는 세부 정보를 얻습니다. 17번째 줄에서 *src/lib.rs* 파일에서. 다음 섹션은 모든 실패 테스트의 이름만 나열하며, 많은 테스트와 많은 자세한 실패 테스트 출력이 있는 경우 유용합니다. 실패 테스트의 이름을 사용하여 해당 테스트만 실행하여 더 쉽게 디버깅할 수 있습니다. 테스트 실행 방법에 대해서는 [“테스트 실행 방식 제어”][controlling-how-tests-are-run]<!-- ignore --> 섹션에서 자세히 설명합니다.

요약 줄은 다음과 같이 표시됩니다. 전체적으로 테스트 결과는 `FAILED`입니다. 하나의 테스트가 통과하고 하나의 테스트가 실패했습니다.

이제 테스트 결과가 다양한 시나리오에서 어떻게 표시되는지 보았으므로,테스트에 유용한 `panic!` 이외의 몇 가지 매크로를 살펴보겠습니다.

### `assert!` 매크로를 사용하여 결과 확인

표준 라이브러리에서 제공되는 `assert!` 매크로는 테스트에서 특정 조건이 `true`로 평가되는지 확인할 때 유용합니다. `assert!` 매크로에 불린 값을 평가하는 논리 표현을 전달합니다. 값이 `true`이면 아무런 일이 일어나지 않고 테스트가 통과합니다. 값이 `false`이면 `assert!` 매크로가 `panic!`를 호출하여 테스트를 실패하게 합니다. `assert!` 매크로를 사용하면 코드가 의도한 방식으로 작동하는지 확인하는 데 도움이 됩니다.

5장에서 Listing 5-15에서 사용한 `Rectangle` 구조체와 `can_hold` 메서드는 Listing 11-5에 다시 나와 있습니다. 이 코드를 *src/lib.rs* 파일에 넣고 `assert!` 매크로를 사용하여 테스트를 작성해 보겠습니다.

<Listing number=\"11-5\" file-name=\"src/lib.rs\" caption=\"5장에서 가져온 `Rectangle` 구조체와 `can_hold` 메서드\">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-05/src/lib.rs}}
```

</Listing>

`can_hold` 메서드는 불린 값을 반환하므로 `assert!` 매크로를 사용하는 데 완벽한 경우입니다. Listing 11-6에서 `can_hold` 메서드를 테스트하는 테스트를 작성합니다. 8의 너비와 7의 높이를 가진 `Rectangle` 인스턴스를 생성하고 5의 너비와 1의 높이를 가진 다른 `Rectangle` 인스턴스를 포함할 수 있는지 확인합니다.

<Listing number=\"11-6\" file-name=\"src/lib.rs\" caption=\"`can_hold`을 테스트하는 테스트: 더 큰 사각형이 더 작은 사각형을 포함할 수 있는지 확인\">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-06/src/lib.rs:here}}
```

</Listing>

`tests` 모듈 내부의 `use super::*;` 줄을 주의하십시오. `tests` 모듈은 7장에서 [“모듈 트리에서 항목을 참조하는 경로”][paths-for-referring-to-an-item-in-the-module-tree]<!-- ignore -->
섹션에서 다룬 일반 모듈입니다. `tests` 모듈이 내부 모듈이기 때문에 외부 모듈에서 테스트 코드를 가져와 내부 모듈의 범위 내에 가져와야 합니다. 여기서는 모든 것을 가져오기 위해 글로브를 사용합니다.

테스트를 `larger_can_hold_smaller`로 명명했으며, 테스트에 필요한 두 개의 `Rectangle` 인스턴스를 생성했습니다. 그런 다음 `assert!` 매크로를 호출하고 `larger.can_hold(&smaller)` 호출의 결과를 전달했습니다. 이 표현은 `true`를 반환해야 하므로 테스트는 통과해야 합니다. 확인해 보겠습니다!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-06/output.txt}}
```

테스트가 통과했습니다! `can_hold` 함수에서 더 작은 사각형이 더 큰 사각형을 포함할 수 없다는 것을 확인하는 또 다른 테스트를 추가해 보겠습니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/src/lib.rs:here}}
```

이 경우 `can_hold` 함수의 올바른 결과는 `false`이므로 `assert!` 매크로에 전달하기 전에 그 결과를 반전해야 합니다. 따라서 `can_hold`이 `false`를 반환하면 테스트가 통과합니다.

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/output.txt}}
```

두 개의 테스트가 통과했습니다! 이제 코드에 버그를 도입하여 테스트 결과가 어떻게 변하는지 살펴보겠습니다. 너비를 비교할 때 `>` 기호를 `<` 기호로 바꿔 `can_hold` 메서드의 구현을 변경합니다.

```rust,not_desired_behavior,noplayground
## 테스트 작성하기

이번 장에서는 Rust에서 자동화된 테스트를 작성하는 방법을 살펴봅니다. 테스트는 코드의 정확성을 검증하고 유지보수를 돕는 데 필수적입니다. Rust는 테스트 작성을 위한 강력한 도구를 제공합니다.

### 테스트 작성하기

Rust의 테스트 프레임워크는 `cargo test` 명령어를 사용하여 테스트를 실행합니다. 테스트 파일은 일반적으로 `tests.rs` 파일로 구성되며, 테스트 함수는 `test` 함수 이름으로 시작해야 합니다.

예를 들어, 다음은 `add_two` 함수를 테스트하는 `tests.rs` 파일의 예입니다.

```rust
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/tests.rs}}
```

이 테스트 파일을 실행하면 `cargo test` 명령어를 사용하여 테스트가 실행되고 결과가 표시됩니다.

### 테스트 실패 시 출력

테스트가 실패하면 실패 원인과 함께 자세한 정보가 출력됩니다. 예를 들어, `add_two` 함수가 2를 더하는 대신 3을 더하는 경우 테스트가 실패하고 다음과 같은 출력이 표시됩니다.

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-01-bug-in-add-two/output.txt}}
```

### `assert!` 매크로

Rust에서 테스트를 작성하는 데 사용되는 가장 기본적인 매크로는 `assert!` 매크로입니다. 이 매크로는 주어진 조건이 참이라는 것을 확인합니다. 만약 조건이 거짓이면 테스트가 실패하고 `assertion failed` 메시지가 출력됩니다.

예를 들어, `add_two` 함수가 2를 더하는지 확인하려면 다음과 같은 코드를 사용할 수 있습니다.

```rust
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/tests.rs:here}}
```

### `assert_eq!` 및 `assert_ne!` 매크로

`assert_eq!` 매크로는 두 값이 같으면 테스트를 통과시키고 다르면 테스트를 실패시킵니다. `assert_ne!` 매크로는 두 값이 다르면 테스트를 통과시키고 같으면 테스트를 실패시킵니다.

예를 들어, `add_two` 함수가 2를 더하는지 확인하려면 다음과 같은 코드를 사용할 수 있습니다.

```rust
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-02/src/tests.rs}}
```

### 테스트의 중요성

테스트는 코드의 정확성을 검증하고 유지보수를 돕는 데 필수적입니다. 테스트를 작성함으로써 코드의 오류를 조기에 발견하고 수정할 수 있으며, 코드를 변경할 때 코드의 정확성을 유지할 수 있습니다.

## 다음 단계

이 장에서는 Rust에서 자동화된 테스트를 작성하는 기본적인 개념을 살펴보았습니다. 다음 장에서는 더욱 복잡한 테스트를 작성하는 방법을 살펴보겠습니다.값이 *될 것* 이지만, 우리는 값이 *확실히* *되어서는 안 된다는 것을 알고 있습니다.
예를 들어, 입력을 어떤 방식으로 변경하는지가 주 실행 날짜에 따라 달라지는 함수를 테스트하는 경우, 함수의 출력이 입력과 같지 않다는 것을 확인하는 것이 가장 좋습니다.

표면적으로 `assert_eq!` 및 `assert_ne!` 매크로는 `==` 및 `!=` 연산자를 사용합니다. 검사가 실패하면 이러한 매크로는 디버그 형식을 사용하여 인수를 출력하므로 비교되는 값은 `PartialEq` 및 `Debug` 트레이트를 구현해야 합니다. 기본형 타입과 대부분의 표준 라이브러리 타입은 이러한 트레이트를 구현합니다. 자신이 정의한 구조체 및 열거형의 경우 `PartialEq`를 구현하여 해당 유형의 등식을 확인해야 합니다. `Debug`를 구현하여 검사가 실패할 때 값을 출력해야 합니다. 두 트레이트 모두 도출 가능한 트레이트이므로(제5장의 5-12번 목록에서 언급됨) 일반적으로 구조체 또는 열거형 정의에 `#[derive(PartialEq, Debug)]` 애너테이션을 추가하는 것만으로 충분합니다. 자세한 내용은 부록 C, [“도출 가능한 트레이트,”][derivable-traits]<!-- ignore -->를 참조하십시오.

### 사용자 정의 오류 메시지 추가

`assert!`, `assert_eq!` 및 `assert_ne!` 매크로에 선택적으로 사용할 수 있는 사용자 정의 메시지를 추가할 수 있습니다. `format!` 매크로(제8장의 [“`+` 연산자 또는 `format!` 매크로를 사용한 연결”][concatenation-with-the--operator-or-the-format-macro]<!-- ignore --> 섹션에서 설명됨)에 전달되는 모든 인수는 `format!` 매크로에 전달되므로 `{}` 플레이스홀더와 플레이스홀더에 들어갈 값을 포함하는 형식 문자열을 전달할 수 있습니다. 사용자 정의 메시지는 어떤 검사를 의미하는지 설명하는 데 유용합니다. 테스트가 실패하면 문제가 있는 코드에 대한 더 나은 이해를 얻을 수 있습니다.

예를 들어, 이름으로 사람들을 인사하는 함수가 있고 함수에 전달하는 이름이 출력에 나타나는지 테스트하려고 합니다.

<span class=\"filename\">Filename: src/lib.rs</span>
```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-05-greeter/src/lib.rs}}
```

이 프로그램의 요구 사항은 아직 합의되지 않았으며, 인사의 시작 부분인 `Hello` 텍스트가 변경될 것으로 예상됩니다. 요구 사항이 변경될 때 테스트를 업데이트해야 하는 상황을 피하기 위해 `greeting` 함수에서 반환되는 값과 정확히 일치하는지 확인하는 대신, 출력에 입력 파라미터의 텍스트가 포함되어 있는지 확인합니다.

이제 `greeting` 함수를 수정하여 `name`을 제외하고 버그를 도입하여 기본 테스트 실패 결과를 살펴보겠습니다.

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/src/lib.rs:here}}
```

이 테스트를 실행하면 다음과 같은 결과가 나타납니다.

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/output.txt}}
```

이 결과는 검사가 실패했고 검사가 있는 줄이 무엇인지만 나타냅니다. 더 유용한 오류 메시지는 `greeting` 함수에서 가져온 값을 출력합니다. `greeting` 함수에서 가져온 값으로 채워진 형식 문자열을 사용하여 사용자 정의 오류 메시지를 추가해 보겠습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/src/lib.rs:here}}
```

이제 테스트를 실행하면 더 자세한 오류 메시지가 표시됩니다.

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/output.txt}}
```

테스트 출력에서 실제로 얻은 값을 볼 수 있습니다. 이는 예상되는 값이 아닌 실제로 발생한 문제를 파악하는 데 도움이 됩니다.

### `should_panic`을 사용하여 Panic 검사

반환값을 확인하는 것 외에도 코드가 예상대로 오류 조건을 처리하는지 확인하는 것이 중요합니다. 예를 들어, 9장에서 소개한 `Guess` 유형을 생각해 보세요. 다른 코드가 `Guess`를 사용하는 경우 `Guess` 인스턴스에는 1부터 100까지의 값만 포함되어 있다는 보장에 의존합니다. `Guess` 인스턴스를 생성하려고 할 때 범위 밖의 값을 사용하면 `panic`하는지 확인하는 테스트를 작성할 수 있습니다.

`should_panic` 속성을 테스트 함수에 추가하여 이를 수행합니다. 테스트 함수 내부의 코드가 `panic`하면 테스트가 통과하고, 테스트 함수 내부의 코드가 `panic`하지 않으면 테스트가 실패합니다.

11-8번 목록은 `Guess::new`의 오류 조건이 예상대로 발생하는지 확인하는 테스트를 보여줍니다.

<Listing number=\"11-8\" file-name=\"src/lib.rs\" caption=\"특정 조건이 `panic!`를 유발하는지 테스트\">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-08/src/lib.rs}}
```

</Listing>

`#[should_panic]` 속성은 `#[test]` 속성 뒤에 테스트 함수에 적용되는 위치에 배치합니다. 테스트가 통과할 때 나타나는 결과를 살펴보겠습니다.

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-08/output.txt}}
```

잘 되었습니다! 이제 테스트가 실패하는 경우를 살펴보기 위해 코드에 버그를 추가해 보겠습니다. `new` 함수가 값이 100보다 클 경우 `panic`하지 않도록 조건을 제거합니다.

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/src/lib.rs:here}}
```

11-8번 목록의 테스트를 실행하면 실패합니다.

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/output.txt}}
```

이 경우에는 매우 도움이 되는 메시지가 나타나지 않습니다. 하지만 테스트 함수를 살펴보면 `#[should_panic]` 속성이 적용되어 있는 것을 알 수 있습니다. 테스트가 실패한 것은 테스트 함수 내부의 코드가 `panic`하지 않았기 때문입니다.

`should_panic`을 사용하는 테스트는 정확도가 떨어질 수 있습니다. `should_panic` 테스트는 테스트가 예상했던 이유와 다른 이유로 `panic`하더라도 통과할 수 있습니다. `should_panic` 테스트의 정확도를 높이기 위해 `expected` 매개변수를 추가할 수 있습니다. 테스트 래퍼는 실패 메시지에 제공된 텍스트가 포함되어 있는지 확인합니다. 예를 들어, 11-9번 목록에서 `Guess::new` 함수가 값이 너무 작거나 너무 클 때 다른 메시지로 `panic`하는 경우를 생각해 보세요.

<Listing number=\"11-9\" file-name=\"src/lib.rs\" caption=\"특정 문자열을 포함하는 `panic!`를 테스트\">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-09/src/lib.rs:here}}
```

</Listing>

이 테스트는 `should_panic` 속성의 `expected` 매개변수에 입력한 값이 `Guess::new` 함수가 `panic`할 때 나타나는 메시지의 부분 문자열이기 때문에 통과합니다. 우리는 `panic` 메시지의 전체 내용을 지정할 수도 있습니다. 이 경우 `Guess value must be less than or equal to 100, got 200`가 될 것입니다. 어떤 부분을 지정할지는 `panic` 메시지가 얼마나 고유하거나 동적이며 테스트가 얼마나 정확해야 하는지에 따라 다릅니다. 이 경우 `panic` 메시지의 부분 문자열만으로도 테스트 함수 내부의 코드가 `else if value > 100` 블록을 실행하는지 확인하는 데 충분합니다.

`expected` 메시지가 포함된 `should_panic` 테스트가 실패하는 경우를 보려면 코드에 다시 버그를 추가하여 `if value < 1`과 `else if value > 100` 블록의 내용을 바꿉니다.

```rust,ignore,not_desired_behavior
"


```## 테스트 작성하기

이전에 작성한 `guess` 함수를 테스트하는 데 사용할 수 있는 `should_panic` 어트리뷰트를 사용해 보겠습니다. 이 어트리뷰트는 테스트가 멈추고 에러 메시지를 출력하는 것을 기대하는 경우에 유용합니다.

```rust
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/src/lib.rs:here}}
```

이 코드를 실행하면 다음과 같은 출력을 얻을 수 있습니다.

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/output.txt}}
```

이 출력은 테스트가 예상대로 `panic`을 일으켰음을 보여줍니다. 하지만 `panic` 메시지에 `100 이하`라는 문자열이 포함되어 있지 않습니다. 대신 `Guess value must
be greater than or equal to 1, got 200.`라는 메시지를 얻었습니다. 이제 버그의 원인을 파악하기 시작할 수 있습니다!

### 테스트에서 `Result<T, E>` 사용하기

지금까지 작성한 테스트는 모두 실패 시 `panic`을 일으켰습니다. `Result<T, E>`를 사용하여 테스트를 작성할 수도 있습니다. 다음은 Listing 11-1에서 가져온 테스트를 `Result<T, E>`를 사용하여 다시 작성한 것입니다.

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-10-result-in-tests/src/lib.rs:here}}
```

`it_works` 함수는 이제 `Result<(), String>` 반환형을 가지고 있습니다. 함수 내부에서 `assert_eq!` 매크로를 호출하는 대신, 테스트가 통과하면 `Ok(())`을 반환하고, 테스트가 실패하면 `Err`와 `String`을 포함하여 반환합니다.

`Result<T, E>`를 사용하여 테스트를 작성하면 테스트 내부에서 질문표시 연산자를 사용할 수 있습니다. 질문표시 연산자는 `Err` 변형을 반환하는 작업이 있는 경우 테스트를 실패시키는 데 유용한 방법입니다.

`Result<T, E>`를 사용하는 테스트에는 `#[should_panic]` 어트리뷰트를 사용할 수 없습니다. `Err` 변형을 반환하는지 확인하려면 `Result<T, E>` 값에 질문표시 연산자를 사용하지 마세요. 대신 `assert!(value.is_err())`을 사용하세요.

이제 여러 가지 테스트 작성 방법을 알고 있으므로, 테스트 실행 시 발생하는 일을 살펴보고 `cargo test`를 사용하여 테스트를 실행할 때 사용할 수 있는 다양한 옵션을 탐색해 보겠습니다.

[concatenation-with-the--operator-or-the-format-macro]:
ch08-02-strings.html#concatenation-with-the--operator-or-the-format-macro
[bench]: ../unstable-book/library-features/test.html
[ignoring]: ch11-02-running-tests.html#ignoring-some-tests-unless-specifically-requested
[subset]: ch11-02-running-tests.html#running-a-subset-of-tests-by-name
[controlling-how-tests-are-run]:
ch11-02-running-tests.html#controlling-how-tests-are-run
[derivable-traits]: appendix-03-derivable-traits.html
[doc-comments]: ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests
[paths-for-referring-to-an-item-in-the-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
