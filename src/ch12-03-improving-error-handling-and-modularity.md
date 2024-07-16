## 오류 처리 및 모듈성 향상을 위한 리팩토링

프로그램을 개선하기 위해 `main` 함수의 구조와 오류 처리 방식과 관련된 네 가지 문제를 해결합니다. 첫째, `main` 함수는 현재 인자를 분석하고 파일을 읽는 두 가지 작업을 수행합니다. 프로그램이 커질수록 `main` 함수가 처리하는 작업 수가 증가합니다. 함수가 책임을 맡을수록 논리적 사고가 어려워지고 테스트하기 어려워지며, 부분을 손상시키지 않고 변경하기 어려워집니다. 각 함수가 하나의 작업에 책임을 맡도록 기능을 분리하는 것이 좋습니다.

이 문제는 또한 `query`와 `file_path`와 같은 변수가 프로그램의 구성 변수이지만, `contents`와 같은 변수는 프로그램의 논리를 수행하는 데 사용된다는 점과도 관련이 있습니다. `main` 함수가 길어질수록 스코프 내에 가져올 변수 수가 증가합니다. 스코프 내 변수가 많을수록 각 변수의 목적을 추적하기 어려워집니다. 구성 변수를 하나의 구조체에 그룹화하여 목적을 명확하게 하는 것이 좋습니다.

셋째, 파일 읽기 실패 시 오류 메시지를 출력하기 위해 `expect`를 사용했지만, 오류 메시지는 `파일을 읽을 수 있어야 했습니다`라고만 출력합니다. 파일을 읽는 것은 여러 가지 방법으로 실패할 수 있습니다. 예를 들어, 파일이 누락되었거나 열 권한이 없을 수 있습니다. 현재 상황에 관계없이 모든 상황에 대해 동일한 오류 메시지를 출력하므로 사용자에게는 정보가 전달되지 않습니다.

넷째, `expect`를 사용하여 오류를 처리하고, 사용자가 충분한 인수를 지정하지 않고 프로그램을 실행하면 Rust에서 `인덱스가 범위를 벗어났습니다` 오류가 발생합니다. 이 오류는 문제를 명확하게 설명하지 않습니다. 모든 오류 처리 코드가 하나의 위치에 있으면 미래의 유지보수자가 오류 처리 논리 변경이 필요한 경우 참조할 수 있는 유일한 위치가 됩니다. 모든 오류 처리 코드를 하나의 위치에 두면 사용자에게 의미 있는 메시지를 출력하는 것도 보장됩니다.

이 네 가지 문제를 해결하기 위해 프로젝트를 리팩토링하겠습니다.

### 이진 프로젝트를 위한 책임 분담

`main` 함수에 여러 작업의 책임을 할당하는 조직적 문제는 많은 이진 프로젝트에서 흔히 발생합니다. 따라서 Rust 커뮤니티는 `main` 함수가 커지기 시작하면 이진 프로그램의 별개의 우려 사항을 분할하는 지침을 개발했습니다. 이 프로세스는 다음 단계를 포함합니다.

* 프로그램을 `main.rs` 와 `lib.rs` 로 분리하고 프로그램 논리를 `lib.rs` 로 이동합니다.
* 명령줄 분석 논리가 작다면 `main.rs` 에 남겨둘 수 있습니다.
* 명령줄 분석 논리가 복잡해지면 `main.rs` 에서 추출하여 `lib.rs` 로 이동합니다.

이 프로세스 후 `main` 함수에 남아있는 책임은 다음과 같습니다.

* 인자 값으로 명령줄 분석 논리를 호출합니다.
* 다른 구성을 설정합니다.
* `lib.rs` 에서 `run` 함수를 호출합니다.
* `run` 함수가 오류를 반환하면 오류를 처리합니다.

이 패턴은 책임 분담에 관한 것입니다. `main.rs` 는 프로그램 실행을 처리하고, `lib.rs` 는 작업의 모든 논리를 처리합니다. `main` 함수를 직접 테스트할 수 없기 때문에 이러한 구조는 모든 프로그램 논리를 `lib.rs` 에 있는 함수로 이동하여 테스트할 수 있게 합니다. `main.rs` 에 남아있는 코드는 읽으면서도 올바르게 작동하는지 확인할 수 있을 만큼 작습니다. 이러한 프로세스를 따르면 프로그램을 다시 작성할 수 있습니다.

#### 인자 분석기 추출

인자를 분석하는 기능을 `parse_config` 함수로 추출하여 `main` 함수가 호출하는 방식으로 변경합니다. `parse_config` 함수는 `src/main.rs` 에서 일단 정의됩니다. 12-5번 목록은 인자를 분석하는 `parse_config` 함수를 호출하는 `main` 의 새로운 시작 부분을 보여줍니다.

<Listing number=\"12-5\" file-name=\"src/main.rs\" caption=\"`parse_config` 함수를 `main` 에서 추출\">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-05/src/main.rs:here}}
```

</Listing>

인자를 분석하는 기능을 함수로 추출하면 `main` 함수가 깔끔해지고 테스트하기 쉬워집니다. 이제 `parse_config` 함수는 명령줄 인자를 처리하고 프로그램에 필요한 구성을 반환합니다.

main 함수에서 `file_path` 변수에 대한 인덱스 2의 인수 값을 가리키는 오류입니다.

이 오류는 `parse_config` 함수에 전달되는 인수가 `main` 함수에서 정의된 변수와 일치하지 않기 때문입니다. `parse_config` 함수는 명령줄 인수를 처리하고 `query`와 `file_path` 변수에 할당하기 위한 논리를 담고 있습니다. `main` 함수는 이 논리를 담당하지 않고, `parse_config` 함수가 처리하는 것으로 변경되었습니다.

이러한 변경은 작은 프로그램에 과도한 작업으로 보일 수 있지만, 작고 단계적인 리팩토링을 통해 진행하고 있습니다. 이 변경을 적용한 후 프로그램을 다시 실행하여 명령줄 인수 처리가 여전히 제대로 작동하는지 확인하십시오. 문제가 발생했을 때 원인을 파악하는 데 도움이 되기 때문에 진행 상황을 자주 확인하는 것이 좋습니다.

#### 구성 값 그룹화

`parse_config` 함수를 더욱 개선하기 위해 한 가지 더 작은 단계를 취할 수 있습니다. 현재는 튜플을 반환하지만, 즉시 해당 튜플을 개별 부분으로 분해합니다. 이는 아직 적절한 추상화를 가지고 있지 않음을 나타냅니다.

또 다른 개선 지표는 `parse_config`의 `config` 부분으로, 두 개의 반환 값이 관련이 있고 하나의 구성 값의 일부임을 의미합니다. 현재는 두 값을 튜플로 그룹화하여 이 의미를 전달하는 것 외에는 다른 방법이 없습니다. 대신 두 값을 하나의 구조체에 넣고 각 구조체 필드에 의미 있는 이름을 붙이면 됩니다. 이렇게 하면 미래에 이 코드를 유지 관리하는 사람들이 다양한 값 간의 관계와 각 값의 목적을 더 쉽게 이해할 수 있습니다.

표 12-6은 `parse_config` 함수의 개선 사항을 보여줍니다.

<Listing number=\"12-6\" file-name=\"src/main.rs\" caption=\"`parse_config` 함수를 `Config` 구조체의 인스턴스를 반환하도록 리팩토링\">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-06/src/main.rs:here}}
```

</Listing>

`Config`라는 이름의 구조체를 추가했습니다. `query`와 `file_path`라는 필드를 가지도록 정의했습니다. `parse_config`의 반환형은 이제 `Config` 값을 나타냅니다. `parse_config` 함수의 본문에서 이전에 `args`에서 참조하는 `String` 값을 반환하는 문자열 슬라이스를 사용했던 부분은 이제 소유된 `String` 값을 포함하는 `Config`를 정의합니다. `main` 함수의 `args` 변수는 인수 값의 소유자이며, `parse_config` 함수에만 대출할 수 있습니다. 따라서 `Config`가 `args`에서 값을 소유하려고 하면 Rust의 대출 규칙을 위반하게 됩니다.

`String` 데이터를 관리하는 방법은 여러 가지가 있습니다. 가장 쉬운 방법은 `clone` 메서드를 호출하는 것입니다. 이렇게 하면 `Config` 인스턴스가 소유할 수 있도록 데이터의 완전한 복사본이 생성됩니다. 이는 `String` 데이터에 대한 참조를 저장하는 것보다 시간과 메모리를 더 많이 사용하지만, `Config`가 데이터에 대한 소유권을 가지도록 하는 것은 매우 간단합니다.

> ### `clone` 사용의 트레이드오프
>
> 많은 Rust 개발자들은 `clone`을 사용하여 소유권 문제를 해결하는 경향이 있습니다. 그러나 `clone`은 런타임에 비용이 발생하기 때문에 피하는 것이 좋습니다. [Chapter 13][ch13]<!-- ignore -->에서 더 효율적인 방법을 배울 수 있습니다. 하지만 지금은 작은 프로그램에서 몇 개의 문자열을 복사하는 것이 계속해서 진행되는 것이 좋습니다. 이러한 복사는 한 번만 수행되며 파일 경로와 쿼리 문자열이 매우 작기 때문에 성능이 떨어지는 것보다 작동하는 프로그램이 더 중요합니다. 경험이 쌓이면 가장 효율적인 해결책으로 시작하는 것이 쉬워지지만, 지금은 `clone`을 호출하는 것이 완벽하게 허용됩니다.

`main` 함수는 `parse_config` 함수에서 반환된 `Config` 인스턴스를 `config`라는 변수에 저장하고, 이전에 `query`와 `file_path` 변수를 사용했던 코드를 `Config` 구조체의 필드를 사용하도록 업데이트했습니다.

이제 `query`와 `file_path`가 관련이 있고 서로 의존적인 값임을 코드가 명확하게 전달합니다.
프로그램이 작동하는 방식을 구성하는 방법을 나타냅니다. 이러한 값을 사용하는 모든 코드는 `config` 인스턴스에서 목적에 따라 명명된 필드에서 이러한 값을 찾는 방법을 알고 있습니다.

#### `Config` 생성자 만들기

지금까지 명령줄 인수를 파싱하는 논리를 `main`에서 추출하여 `parse_config` 함수에 넣었습니다. 이렇게 하면 `query`와 `file_path` 값이 관련이 있음을 알 수 있었고, 이 관계는 코드에 반영되어야 했습니다. 그런 다음 `query`와 `file_path`의 관련 목적을 명명하고 `parse_config` 함수에서 값의 이름을 구조체 필드 이름으로 반환할 수 있도록 `Config` 구조체를 추가했습니다.

이제 `parse_config` 함수의 목적이 `Config` 인스턴스를 생성하는 것이므로, `parse_config`를 단순한 함수에서 `Config` 구조체와 연결된 `new` 함수로 변경할 수 있습니다. 이 변경은 코드를 더욱 자연스럽게 만들 것입니다. 표준 라이브러리의 유형, 예를 들어 `String`을 생성하는 방법은 `String::new`를 호출하는 것입니다. 마찬가지로 `parse_config`를 `Config::new`과 같은 `new` 함수로 변경하면 `Config` 인스턴스를 호출하여 `Config::new`를 호출하여 생성할 수 있습니다. 12-7번 목록은 변경 사항을 보여줍니다.

<Listing number=\"12-7\" file-name=\"src/main.rs\" caption=\"`parse_config`를 `Config::new`로 변경\">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-07/src/main.rs:here}}
```

</Listing>

`main`에서 `parse_config`를 호출하는 부분을 `Config::new`로 변경했습니다. `parse_config`의 이름을 `new`로 변경하고 `impl` 블록 내부로 이동하여 `new` 함수를 `Config`와 연결했습니다. 이 코드를 다시 컴파일하여 작동하는지 확인해 보세요.

### 오류 처리 개선

이제 오류 처리를 수정하는 데 집중해 보겠습니다. `args` 벡터의 인덱스 1 또는 2에 있는 값에 액세스하려고 시도하면 벡터에 3개 미만의 항목이 있는 경우 프로그램이 panic합니다. 3개 미만의 인수로 프로그램을 실행해 보세요. 다음과 같이 보입니다.

```console
{{#include ../listings/ch12-an-io-project/listing-12-07/output.txt}}
```

`index out of bounds: the len is 1 but the index is 1`은 개발자를 위한 오류 메시지입니다. 사용자가 무엇을 해야 하는지 이해하는 데 도움이 되지 않습니다. 지금 바로 수정해 보겠습니다.

#### 오류 메시지 개선

12-8번 목록에서 `new` 함수에 추가된 검사는 `slice`의 길이가 충분한지 확인합니다. `slice`가 충분하지 않으면 프로그램이 종료되고 더 나은 오류 메시지가 표시됩니다.

<Listing number=\"12-8\" file-name=\"src/main.rs\" caption=\"인수 수를 확인하는 추가\">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-08/src/main.rs:here}}
```

</Listing>

이 코드는 [9-13번 목록에서 작성한 `Guess::new` 함수와 유사합니다.]<!-- ignore -->, 그곳에서 `value` 인수가 유효한 값 범위를 벗어나면 `panic!`을 호출했습니다. 여기서는 값 범위를 확인하는 대신 `args`의 길이가 3 이상인지 확인합니다. 이 조건이 충족되지 않으면 `args`의 길이가 3 미만이므로 이 조건이 참이 되고 `panic!` 매크로를 사용하여 프로그램을 즉시 종료합니다.

`new`에 추가된 몇 줄의 코드로 인해 인수를 입력하지 않고 프로그램을 실행하면 오류 메시지가 다음과 같이 표시됩니다.

```console
{{#include ../listings/ch12-an-io-project/listing-12-08/output.txt}}
```

이 출력은 더 나은 오류 메시지입니다. 그러나 사용자에게 전달하지 않아야 할 불필요한 정보도 있습니다. 예를 들어, `Config` 구조체의 필드 이름을 사용자에게 보여주는 것은 좋지 않을 수 있습니다.

Listing 9-13에서 사용한 기법은 여기서 가장 적합하지 않습니다. `panic!` 함수는 사용 문제보다 프로그래밍 문제에 더 적합합니다. (Chapter 9에서 논의된 바와 같습니다)[ch9-error-guidelines]<!-- ignore -->. 대신, Chapter 9에서 배운 다른 기법인 `Result` 반환을 사용할 것입니다. `Result`는 성공 또는 오류를 나타냅니다.[ch9-result]<!-- ignore -->

<!-- 이전 제목. 링크가 깨지지 않도록 제거하지 마세요. -->
<a id=\"returning-a-result-from-new-instead-of-calling-panic\"></a>

#### `Result` 반환 대신 `panic!` 호출

`Config` 인스턴스를 성공 시 반환하고 오류 시 문제를 설명하는 `Result` 값을 반환할 수 있습니다. `new` 함수 이름을 `build`로 변경하는 것도 좋습니다. 많은 프로그래머들이 `new` 함수가 절대 실패하지 않는다고 기대하기 때문입니다. `Config::build`가 `main`에 메시지를 전달할 때 `Result` 유형을 사용하여 문제가 발생했음을 알릴 수 있습니다. 그런 다음 `main`을 `Err` 변형을 사용자에게 더 실용적인 오류로 변환하도록 변경할 수 있습니다. `panic!` 호출로 인해 발생하는 `thread 'main'`과 `RUST_BACKTRACE`에 대한 추가 텍스트 없이.

Listing 12-9는 `Config::build` 함수의 반환 값에 대한 변경 사항과 함수 본문에 필요한 변경 사항을 보여줍니다. 이 변경 사항은 `main`을 업데이트할 때까지 컴파일되지 않습니다. 다음 Listing에서 `main`을 업데이트하는 방법을 보여드리겠습니다.

<Listing number=\"12-9\" file-name=\"src/main.rs\" caption=\"`Config::build`에서 `Result` 반환\">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-09/src/main.rs:here}}
```

</Listing>

`build` 함수는 성공 시 `Config` 인스턴스를 포함하는 `Result`를 반환하고 오류 시 `&'static str`을 반환합니다. 오류 값은 항상 `'static` 라이프타임을 가진 문자열 리터럴입니다.

함수 본문에서 두 가지 변경 사항을 적용했습니다. 사용자가 충분한 인수를 전달하지 않을 때 `panic!` 함수를 호출하는 대신 `Err` 값을 반환하고, `Config` 반환 값을 `Ok`로 감싸었습니다. 이러한 변경 사항은 함수가 새 유형 시그니처에 맞도록 합니다.

`Config::build`에서 `Err` 값을 반환하면 `main` 함수가 `Config::build`에서 반환되는 `Result` 값을 처리하고 오류 발생 시 프로세스를 더 깔끔하게 종료할 수 있습니다.

<!-- 이전 제목. 링크가 깨지지 않도록 제거하지 마세요. -->
<a id=\"calling-confignew-and-handling-errors\"></a>

#### `Config::build` 호출 및 오류 처리

`Config::build`에서 반환되는 `Result` 값을 처리하고 사용자 친화적인 메시지를 출력하기 위해 `main`을 업데이트해야 합니다. Listing 12-10에서 보여주는 것처럼 `Config::build`에서 반환되는 `Result` 값을 처리하는 방법을 보여줍니다. 또한 `panic!`에서 벗어나 명령줄 도구가 오류 상태로 종료되는 것을 책임지도록 변경합니다. 비율이 0이 아닌 종료 상태는 프로그램이 우리 프로그램을 호출한 프로세스에 오류 상태로 프로그램이 종료되었음을 알리는 전통입니다.

<Listing number=\"12-10\" file-name=\"src/main.rs\" caption=\"`Config` 생성 실패 시 오류 코드로 종료\">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-10/src/main.rs:here}}
```

</Listing>

이 Listing에서 아직 자세히 다루지 않았지만 사용하는 메서드가 있습니다. `unwrap_or_else`는 표준 라이브러리에서 `Result<T, E>`에 정의되어 있습니다. `unwrap_or_else`를 사용하면 사용자 정의 오류 처리를 정의할 수 있습니다. `Result`가 `Ok` 값이면 이 메서드의 동작은 `unwrap`과 유사합니다. 즉, `Ok`이 감싸고 있는 내부 값을 반환합니다. 그러나 값이 `Err` 값이면 이 메서드는 *closure*에서 실행되는 코드를 호출합니다. `closure`는 `unwrap_or_else`에 전달하는 익명 함수입니다. 우리는 `unwrap_or_else`를 사용하여 오류가 발생하면 사용자에게 더 명확하고 유용한 메시지를 표시할 수 있습니다.

## 오류 처리 및 모듈성 개선

이제 `unwrap_or_else`를 사용하여 오류를 처리하는 방법을 살펴보겠습니다. `unwrap_or_else`는 `Err`의 내부 값을 즉시 반환합니다. 이 경우에는 Listing 12-9에서 추가한 정적 문자열 `\"not enough arguments\"`가 `err`라는 매개변수로 폐쇄 함수에 전달됩니다. 폐쇄 함수 내부에서 `err` 값을 사용할 수 있습니다.

Listing 12-10에서 `process`를 표준 라이브러리에서 가져오기 위해 `use` 문을 추가했습니다. 오류 발생 시 실행될 폐쇄 함수 코드는 두 줄로 구성되어 있습니다. `err` 값을 출력하고 `process::exit` 함수를 호출합니다. `process::exit` 함수는 프로그램을 즉시 종료하고 전달된 값이 종료 상태 코드로 사용됩니다. 이는 Listing 12-8에서 사용한 `panic!` 기반 처리와 유사하지만, 추가 출력이 발생하지 않습니다. 결과를 확인해 보겠습니다.

```console
{{#include ../listings/ch12-an-io-project/listing-12-10/output.txt}}
```

훌륭합니다! 이 출력은 사용자에게 더욱 친절합니다.

### `main` 함수에서 논리 추출

이제 구성 파싱을 재작성했으므로 프로그램 논리로 넘어가겠습니다. [\"이진 프로젝트의 책임 분담\"](#separation-of-concerns-for-binary-projects)에서 언급했듯이, `run`이라는 함수를 만들어 현재 `main` 함수에 있는 구성 설정이나 오류 처리와 관련 없는 모든 논리를 담을 것입니다. 이렇게 하면 `main` 함수가 간결하고 직관적으로 검증 가능해지며, 다른 논리에 대한 테스트를 작성할 수 있습니다.

Listing 12-11은 추출된 `run` 함수를 보여줍니다. 현재는 함수를 추출하는 작고 단계적인 개선만 수행하고 있습니다. `src/main.rs`에서 여전히 함수를 정의하고 있습니다.

<Listing number=\"12-11\" file-name=\"src/main.rs\" caption=\"`run` 함수 추출\">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-11/src/main.rs:here}}
```

</Listing>

`run` 함수에는 이제 `main` 함수에서 나머지 논리, 파일 읽기부터 시작하는 모든 논리가 포함되어 있습니다. `run` 함수는 `Config` 인스턴스를 매개변수로 받습니다.

#### `run` 함수에서 오류 반환

`run` 함수에 나머지 프로그램 논리를 분리한 후, `run` 함수에서 `Result<T, E>`를 반환하여 오류 처리를 개선할 수 있습니다. 이렇게 하면 `main` 함수에서 오류 처리 논리를 사용자 친화적으로 통합할 수 있습니다. Listing 12-12는 `run` 함수의 서명과 몸체에 적용해야 하는 변경 사항을 보여줍니다.

<Listing number=\"12-12\" file-name=\"src/main.rs\" caption=\"`run` 함수를 `Result` 반환형으로 변경\">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-12/src/main.rs:here}}
```

</Listing>

여기서 세 가지 중요한 변경 사항이 있습니다. 첫째, `run` 함수의 반환 유형을 `Result<(), Box<dyn Error>>`로 변경했습니다. 이 함수는 이전에 단위 유형 `()`을 반환했으며, `Ok` 케이스에서 여전히 그대로 유지됩니다.

둘째, 오류 유형으로는 *trait object* `Box<dyn Error>`를 사용했습니다. (맨 위에 `std::error::Error`를 `use` 문으로 가져왔습니다.) `Box<dyn Error>`는 함수가 `Error` trait를 구현하는 유형을 반환하지만, 특정 유형이 무엇인지 명시할 필요가 없습니다. 이는 다양한 오류 케이스에서 다양한 유형의 오류 값을 반환할 수 있도록 유연성을 제공합니다. `dyn` 키워드는 “동적”의 약자입니다.

두 번째로, `expect` 함수 호출 대신 [9장](ch9-question-mark)에서 논의했던 `?` 연산자를 사용했습니다. `expect`는 오류 발생 시 `panic!`을 호출하지만, `?`는 현재 함수에서 오류 값을 반환하여 호출자에서 처리하도록 합니다.

세 번째로, `run` 함수는 이제 성공 시 `Ok` 값을 반환합니다. `run` 함수의 성공 유형을 시그니처에서 `()`로 선언했기 때문에, 단위 유형 값을 `Ok` 값에 감싸야 합니다. 이 `Ok(())` 구문은 처음에는 약간 이상해 보일 수 있지만, 이렇게 `()`을 사용하는 것은 `run`을 그 측면 효과만을 위해 호출하고 있음을 나타내는 전통적인 방법입니다. 즉, 처리해야 할 값이 아닙니다.

이 코드를 실행하면 컴파일되지만 경고 메시지를 표시합니다.

```console
{{#include ../listings/ch12-an-io-project/listing-12-12/output.txt}}
```

Rust는 코드가 `Result` 값을 무시했고, `Result` 값이 오류를 나타낼 수 있다고 알립니다. 하지만 우리는 오류 여부를 확인하지 않고 컴파일러는 아마도 여기에 오류 처리 코드가 필요하다는 것을 알려줍니다! 이제 문제를 해결해 보겠습니다.

#### `main`에서 `run`으로부터 반환되는 오류 처리

Listing 12-10의 `Config::build`과 유사한 기법을 사용하여 `run`에서 반환되는 오류를 확인하고 처리합니다. 하지만 작은 차이점이 있습니다.

Filename: src/main.rs

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-01-handling-errors-in-main/src/main.rs:here}}
```

`if let`을 사용하여 `run`이 `Err` 값을 반환하는지 확인합니다. 그렇다면 오류를 출력하고 `process::exit(1)`을 호출합니다. `run` 함수는 성공 시 `()`을 반환하기 때문에, `unwrap_or_else`를 사용하여 래핑된 값을 반환할 필요가 없습니다. `unwrap_or_else`는 `()`이라는 단위 유형을 반환할 뿐입니다.

`if let`과 `unwrap_or_else` 함수의 코드는 동일합니다.

### 코드를 라이브러리 크레이트로 분리

`minigrep` 프로젝트가 이제까지 잘 진행되었습니다! 이제 `src/main.rs` 파일을 분리하여 일부 코드를 `src/lib.rs` 파일로 옮겨서 테스트 코드를 작성하고 `src/main.rs` 파일의 책임을 줄일 수 있습니다.

`main` 함수가 아닌 코드를 `src/main.rs`에서 `src/lib.rs`로 옮겨보겠습니다.

* `run` 함수 정의
* 관련 `use` 문
* `Config` 정의
* `Config::build` 함수 정의

`src/lib.rs`의 내용은 Listing 12-13에 표시된 시그니처를 가져야 합니다. (코드 본문은 간결성을 위해 생략했습니다). 이것은 `src/main.rs`를 수정할 때까지 컴파일되지 않습니다. Listing 12-14를 참조하세요.

<Listing number=\"12-13\" file-name=\"src/lib.rs\" caption=\"`Config`와 `run`을 `src/lib.rs`로 옮기기\">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-13/src/lib.rs:here}}
```

</Listing>

`pub` 키워드를 적극적으로 사용했습니다. `Config`, `Config` 필드 및 `Config::build` 메서드, 그리고 `run` 함수에 적용되었습니다. 이제 테스트할 수 있는 공개 API를 가진 라이브러리 크레이트를 가지고 있습니다!

이제 `src/lib.rs`에 옮긴 코드를 `src/main.rs`에서 사용할 수 있도록 가져와야 합니다. Listing 12-14를 참조하세요.

## 12.3. 오류 처리 및 모듈성 개선

<Listing number=\"12-14\" file-name=\"src/main.rs\" caption=\"*src/main.rs*에서 `minigrep` 라이브러리 crate를 사용\">
```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-14/src/main.rs:here}}
```
</Listing>

`Config` 유형을 라이브러리 crate에서 가져와 이진 crate의 범위로 가져옵니다. `run` 함수에 crate 이름을 접두사로 붙여 모든 기능이 연결되고 작동해야 합니다. `cargo run` 명령어로 프로그램을 실행하고 모든 것이 제대로 작동하는지 확인합니다.

우와! 많은 작업이었지만, 앞으로의 성공을 위한 기반을 마련했습니다. 이제 오류 처리가 훨씬 쉬워졌고 코드가 더욱 모듈화되었습니다. 앞으로는 대부분의 작업이 *src/lib.rs*에서 이루어질 것입니다.

새로운 모듈성을 활용하여 이전 코드에서는 어려웠지만 새로운 코드에서는 쉬운 작업을 수행해 보겠습니다. 테스트를 작성해 보겠습니다!

[ch13]: ch13-00-functional-features.html
[ch9-custom-types]: ch09-03-to-panic-or-not-to-panic.html#creating-custom-types-for-validation
[ch9-error-guidelines]: ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling
[ch9-result]: ch09-02-recoverable-errors-with-result.html
[ch17]: ch17-00-oop.html
[ch9-question-mark]: ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator



