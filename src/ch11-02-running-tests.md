## 테스트 실행 방식 제어

`cargo run`이 코드를 컴파일하고 결과물인 바이너리를 실행하는 것처럼,
`cargo test`는 코드를 테스트 모드로 컴파일하고 결과물인 테스트 바이너리를 실행합니다.
`cargo test`가 생성하는 바이너리의 기본 동작은 모든 테스트를 병렬로 실행하고 테스트 실행 중 생성된 출력을 캡처하여 출력을 표시하지 않고 테스트 결과와 관련된 출력을 읽기 쉽게 만듭니다.
그러나 명령줄 옵션을 지정하여 이 기본 동작을 변경할 수 있습니다.

일부 명령줄 옵션은 `cargo test`로, 다른 옵션은 결과 테스트 바이너리로 전달됩니다. 두 유형의 인수를 구분하기 위해 `cargo test`에 전달되는 인수를 먼저 나열한 다음 구분 기호 `--`를 사용하고 테스트 바이너리에 전달되는 인수를 나열합니다. `cargo test --help`를 실행하면 `cargo test`와 함께 사용할 수 있는 옵션이 표시되고 `cargo test -- --help`를 실행하면 구분 기호 이후에 사용할 수 있는 옵션이 표시됩니다.

### 병렬 또는 순차적으로 테스트 실행

여러 개의 테스트를 실행할 때 기본적으로 스레드를 사용하여 병렬로 실행되므로 더 빨리 완료되고 피드백을 더 빨리 받을 수 있습니다. 테스트가 동시에 실행되기 때문에 테스트 간에 의존성이 없거나 현재 작업 디렉토리 또는 환경 변수와 같은 공유 상태에 의존하지 않도록 해야 합니다.

예를 들어 각 테스트가 디스크에 *test-output.txt*라는 파일을 생성하고 그 파일로 데이터를 쓰는 코드를 실행한다고 가정해 보겠습니다. 그런 다음 각 테스트가 해당 파일에서 데이터를 읽고 파일이 특정 값을 포함하는지 확인하는 테스트를 실행합니다. 테스트가 동시에 실행되면 다른 테스트가 파일을 쓰는 동안 다른 테스트가 파일을 읽기 전에 다른 테스트가 파일을 덮어쓸 수 있습니다. 두 번째 테스트는 오류가 발생하지 않고 코드가 잘못되었기 때문에 실패합니다. 테스트가 병렬로 실행되는 동안 서로 방해하지 않도록 하려면 각 테스트가 다른 파일로 쓰도록 하거나 테스트를 한 번에 실행하는 것이 좋습니다.

병렬로 테스트를 실행하지 않거나 사용하는 스레드 수에 대한 더 세밀한 제어를 원하는 경우 `--test-threads` 플래그를 사용하여 테스트 바이너리에 스레드 수를 전달할 수 있습니다. 다음 예를 참조하십시오.

```console
$ cargo test -- --test-threads=1
```

`1`로 테스트 스레드 수를 설정하여 프로그램이 병렬 처리를 사용하지 않도록 합니다. 한 스레드를 사용하여 테스트를 실행하면 병렬 처리를 사용하여 실행하는 것보다 오래 걸리지만, 테스트가 공유 상태를 사용하는 경우 서로 방해하지 않습니다.

### 함수 출력 표시

기본적으로 테스트가 통과하면 Rust 테스트 라이브러리는 표준 출력에 출력된 모든 내용을 캡처합니다. 예를 들어, 테스트에서 `println!`을 호출하고 테스트가 통과하면 `println!` 출력을 터미널에서 볼 수 없습니다. 테스트가 실패하면 표준 출력에 출력된 내용과 함께 오류 메시지가 표시됩니다.

예를 들어, 11-10번 목록은 매개변수 값을 출력하고 10을 반환하는 어리석은 함수와 통과하는 테스트와 실패하는 테스트가 있습니다.

<Listing number=\"11-10\" file-name=\"src/lib.rs\" caption=\"`println!`을 호출하는 함수의 테스트\">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-10/src/lib.rs}}
```

</Listing>

`cargo test`를 실행하여 이 테스트를 실행하면 다음과 같은 출력이 표시됩니다.

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-10/output.txt}}
```

이 출력에서 어디에도 통과하는 테스트가 실행될 때 출력되는 `I got the value 4`가 없습니다. 이 출력은 캡처되었습니다. 실패하는 테스트에서 출력되는 `I got the value 8`은 테스트 요약 출력의 일부로 표시되며 테스트 실패의 원인도 함께 표시됩니다.

통과하는 테스트의 출력도 표시하려면 `--show-output`를 사용하여 Rust에 알려주세요.

```console
$ cargo test -- --show-output
```

11-10번 목록의 테스트를 다시 `--show-output` 플래그를 사용하여 실행하면 다음과 같은 출력이 표시됩니다.

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-10/output-with-show-output.txt}}
```

## 테스트 실행: 부분 실행 및 무시된 테스트


### 테스트 이름으로 부분 테스트 실행

 전체 테스트를 실행하는 것은 시간이 오래 걸릴 수 있습니다. 특정 코드를 작업하는 경우 해당 코드에 대한 테스트만 실행하는 것이 좋습니다. `cargo test` 명령어에 원하는 테스트 이름 또는 이름들을 인수로 전달하여 어떤 테스트를 실행할지 선택할 수 있습니다.

 세 가지 테스트를 `add_two` 함수에 대해 만들어 테스트 이름을 사용하여 어떤 테스트를 실행할지 보여드리겠습니다. Listing 11-11을 참조하세요.

<Listing number=\"11-11\" file-name=\"src/lib.rs\" caption=\"세 가지 테스트 이름\">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-11/src/lib.rs}}
```

</Listing>

 이전에 언급했듯이 인수를 전달하지 않고 테스트를 실행하면 모든 테스트가 병렬로 실행됩니다.

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-11/output.txt}}
```

#### 단일 테스트 실행

 `cargo test` 에 테스트 함수 이름을 전달하여 해당 테스트만 실행할 수 있습니다.

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-02-single-test/output.txt}}
```

 `one_hundred`이라는 이름의 테스트만 실행되었습니다. 다른 두 테스트는 해당 이름과 일치하지 않았습니다. 테스트 출력은 `2 filtered out`이라는 메시지로 더 실행되지 않은 테스트가 있음을 알려줍니다.

 단일 테스트 이름만 지정할 수 있습니다. 여러 테스트를 실행하려면 다른 방법을 사용해야 합니다.

#### 여러 테스트 실행

 테스트 이름의 일부를 지정할 수 있으며, 해당 값과 일치하는 모든 테스트가 실행됩니다. 예를 들어, 두 개의 테스트 이름에 `add`가 포함되어 있으므로 `cargo test add`를 실행하여 두 테스트를 실행할 수 있습니다.

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-03-multiple-tests/output.txt}}
```

 이 명령어는 `add`가 포함된 모든 테스트를 실행하고 `one_hundred`이라는 이름의 테스트는 제외합니다. 또한 테스트가 포함된 모듈이 테스트 이름의 일부가 되므로 모듈 이름을 기반으로 모든 테스트를 실행할 수 있습니다.

### 특정 요청이 없는 한 무시된 테스트

 일부 테스트는 실행하는 데 시간이 오래 걸릴 수 있으므로 대부분의 `cargo test` 실행 중에는 제외하는 것이 좋습니다. `cargo test` 결과가 빠르게 반환되도록 원하는 테스트만 실행하는 것이 좋습니다. `#[ignore]` 속성을 사용하여 시간이 오래 걸리는 테스트를 지정하여 제외할 수 있습니다.

Filename: src/lib.rs

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/src/lib.rs:here}}
```

 `#[test]` 다음에 `#[ignore]` 줄을 추가하여 무시할 테스트를 지정합니다. 이제 `cargo test`를 실행하면 `it_works`가 실행되지만 `expensive_test`는 실행되지 않습니다.

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/output.txt}}
```

 `expensive_test` 함수는 `ignored`로 표시됩니다. `cargo test -- --ignored`를 실행하면 무시된 테스트만 실행됩니다.

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-04-running-ignored/output.txt}}
```

 테스트를 제어하여 `cargo test` 결과가 빠르게 반환되도록 할 수 있습니다. `ignored` 테스트의 결과를 확인하고 시간이 있을 때 `cargo test -- --ignored`를 실행하여 모든 테스트를 실행할 수 있습니다. 모든 테스트를 실행하려면 `cargo test -- --include-ignored`를 실행합니다.
