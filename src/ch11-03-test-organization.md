## 테스트 구성

이 챕터의 시작 부분에서 언급했듯이, 테스트는 복잡한 분야이며,
다른 사람들은 다른 용어와 구성을 사용합니다. Rust 커뮤니티는 테스트를 두 가지 주요 범주로 생각합니다. 단위 테스트와 통합 테스트입니다. *단위 테스트*는 하나의 모듈을 단독으로 테스트하여 작고 집중되어 있으며, 개인적인 인터페이스를 테스트할 수 있습니다. *통합 테스트*는 라이브러리 외부에 있으며, 공개 인터페이스만 사용하여 코드를 다른 외부 코드와 동일하게 사용하며, 여러 모듈을 하나의 테스트에서 실행할 수 있습니다.

두 가지 유형의 테스트를 작성하는 것은 라이브러리의 조각이 예상대로 작동하는지, 별도로 그리고 함께 작동하는지 확인하는 데 중요합니다.

### 단위 테스트

단위 테스트의 목적은 코드의 각 단위를 나머지 코드에서 분리하여 테스트하여 코드가 예상대로 작동하는지 여부를 빠르게 파악하는 것입니다. 단위 테스트는 테스트하는 코드와 동일한 *src* 디렉토리에 파일을 넣습니다. 컨벤션은 테스트 함수를 포함하는 `tests`라는 이름의 모듈을 각 파일에서 만들고 해당 모듈을 `cfg(test)`로 표시하는 것입니다.

#### 테스트 모듈 및 `#[cfg(test)]`

`tests` 모듈에 대한 `#[cfg(test)]` 어노테이션은 Rust가 `cargo test`를 실행할 때만 테스트 코드를 컴파일하고 실행하도록 지시합니다. `cargo build`를 실행할 때는 컴파일하지 않습니다. 이렇게 하면 라이브러리만 빌드할 때 컴파일 시간을 절약하고, 테스트가 포함되지 않아 컴파일된 결과물의 크기를 줄일 수 있습니다. 통합 테스트가 다른 디렉토리에 있기 때문에 `#[cfg(test)]` 어노테이션이 필요하지 않다는 것을 알 수 있습니다. 그러나 단위 테스트가 코드와 동일한 파일에 있기 때문에 `#[cfg(test)]`를 사용하여 컴파일된 결과물에 포함되지 않도록 지정합니다.

처음 챕터에서 `adder` 프로젝트를 생성했을 때 Cargo가 자동으로 생성한 코드를 기억하십시오.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

자동으로 생성된 `tests` 모듈에서 `cfg` 속성은 *구성*을 의미하며, Rust가 특정 구성 옵션이 주어진 경우 다음 항목만 포함하도록 지시합니다. 이 경우 구성 옵션은 `test`이며, Rust에서 테스트를 컴파일하고 실행하기 위해 제공됩니다. `cfg` 속성을 사용하여 Cargo는 `cargo test`로 테스트를 실행할 때만 테스트 코드를 컴파일합니다. 이에는 `#[test]`로 표시된 함수뿐만 아니라 이 모듈 내의 모든 헬퍼 함수도 포함됩니다.

#### 개인 함수 테스트

테스트 커뮤니티 내에서 개인 함수를 직접 테스트해야 하는지에 대한 논쟁이 있습니다. 다른 언어는 개인 함수를 테스트하는 것을 어렵거나 불가능하게 만듭니다. 테스트 이론에 관계없이 Rust의 개인 접근 제한은 개인 함수를 테스트할 수 있도록 합니다. 11-12번 목록의 `internal_adder` 개인 함수를 참조하십시오.

<Listing number=\"11-12\" file-name=\"src/lib.rs\" caption=\"개인 함수 테스트\">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-12/src/lib.rs}}
```

</Listing>

`internal_adder` 함수는 `pub`로 표시되지 않았습니다. 테스트는 단순히 Rust 코드이며, `tests` 모듈은 또 다른 모듈입니다. [모듈 트리에서 항목을 참조하는 경로](paths)<!-- ignore --> 섹션에서 설명했듯이, 자식 모듈의 항목은 조상 모듈의 항목을 사용할 수 있습니다. 이 테스트에서 `use super::*`를 사용하여 `tests` 모듈의 부모 모듈의 모든 항목을 스코프에 가져오고 테스트는 `internal_adder`를 호출할 수 있습니다. 개인 함수를 테스트해서는 안 된다고 생각하지 않는다면 Rust에서 그렇게 하도록 강요하는 것은 없습니다.

### 통합 테스트

Rust에서 통합 테스트는 라이브러리 외부에 있습니다. 라이브러리를 다른 코드와 동일하게 사용하므로, 공개 API에 있는 함수만 호출할 수 있습니다. 그 목적은 라이브러리의 여러 부분이 함께 올바르게 작동하는지 테스트하는 것입니다. 스스로 올바르게 작동하는 코드의 단위는 통합될 때 문제가 발생할 수 있으므로 통합된 코드에 대한 테스트 범위도 중요합니다. 통합 테스트를 만들려면 먼저 *tests* 디렉토리를 만듭니다.

#### *tests* 디렉토리

우리는 프로젝트 디렉토리의 최상위에 *tests* 디렉토리를 만들고, *src* 옆에 둡니다. Cargo는 이 디렉토리에 통합 테스트 파일이 있는지 확인합니다. 우리는 원하는 만큼 많은 테스트 파일을 만들 수 있으며, Cargo는 각 파일을 개별 크레이트로 컴파일합니다.

11-12번 목록의 코드가 여전히 *src/lib.rs* 파일에 있을 때, *tests* 디렉토리를 만들고 *tests/integration_test.rs* 라는 이름의 새 파일을 만듭니다. 디렉토리 구조는 다음과 같아야 합니다.

```text
adder
\u251c\u2500\u2500 Cargo.lock
\u251c\u2500\u2500 Cargo.toml
\u251c\u2500\u2500 src
\u2502\u00a0\u00a0 \u2514\u2500\u2500 lib.rs
\u2514\u2500\u2500 tests
    \u2514\u2500\u2500 integration_test.rs
```

11-13번 목록의 코드를 *tests/integration_test.rs* 파일에 입력합니다.

<Listing number=\"11-13\" file-name=\"tests/integration_test.rs\" caption=\"`adder` 크레이트의 함수에 대한 통합 테스트\">

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-13/tests/integration_test.rs}}
```

</Listing>

*tests* 디렉토리의 각 파일은 별도의 크레이트이므로, 우리의 라이브러리를 각 테스트 크레이트의 범위 내에 가져와야 합니다. 그 이유로 `use adder::add_two;`를 코드의 맨 위에 추가합니다. 이는 단위 테스트에서 필요하지 않았던 부분입니다.

*tests/integration_test.rs* 파일의 코드에 `#[cfg(test)]` 애노테이션을 추가할 필요는 없습니다. Cargo는 *tests* 디렉토리를 특별하게 처리하여 `cargo test`를 실행할 때만 이 디렉토리의 파일을 컴파일합니다. 지금 `cargo test`를 실행합니다.

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-13/output.txt}}
```

출력의 세 가지 섹션에는 단위 테스트, 통합 테스트 및 문서 테스트가 포함됩니다. 특정 섹션의 테스트가 실패하면 다음 섹션은 실행되지 않습니다. 예를 들어 단위 테스트가 실패하면 단위 테스트가 성공해야만 실행되는 통합 및 문서 테스트에 대한 출력이 없습니다.

단위 테스트의 첫 번째 섹션은 우리가 지금까지 보았던 것과 같습니다. 각 단위 테스트(11-12번 목록에서 추가한 `internal`이라는 이름의 하나)에 대한 한 줄씩, 그리고 단위 테스트의 요약 줄이 있습니다.

통합 테스트 섹션은 `Running tests/integration_test.rs` 라는 줄로 시작합니다. 다음으로 해당 통합 테스트에 있는 각 테스트 함수에 대한 줄이 나오고, `Doc-tests adder` 섹션이 시작하기 바로 전에 통합 테스트의 결과에 대한 요약 줄이 있습니다.

각 통합 테스트 파일은 자신의 섹션을 가지므로, *tests* 디렉토리에 더 많은 파일을 추가하면 통합 테스트 섹션이 더 많아집니다.

특정 통합 테스트 함수를 실행하려면 `cargo test` 명령에 테스트 함수의 이름을 인수로 지정할 수 있습니다. 특정 통합 테스트 파일의 모든 테스트를 실행하려면 `cargo test` 명령의 `--test` 인수에 파일 이름을 사용합니다.

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-05-single-integration/output.txt}}
```

이 명령은 *tests/integration_test.rs* 파일의 테스트만 실행합니다.

#### 통합 테스트에서 서브모듈

더 많은 통합 테스트를 추가하면서 여러 통합 테스트 파일에서 사용할 수 있는 도움이 되는 함수를 만들고 싶을 수 있습니다. 예를 들어, 테스트 함수를 테스트하는 기능에 따라 그룹화할 수 있습니다. 앞서 언급했듯이, *tests* 디렉토리의 각 파일은 별도의 크레이트로 컴파일되므로, 7장에서 [“모듈을 다른 파일에 분리하기”][separating-modules-into-files]<!-- ignore --> 섹션에서 설명한 것처럼, 이러한 도움이 되는 함수를 공통 모듈로 추출하려고 할 때, *src* 파일과 다른 동작을 보입니다.

*tests* 디렉토리 파일의 다른 동작은 특히 여러 통합 테스트 파일에서 사용할 수 있는 도움이 되는 함수를 만들고, 이를 *tests/common.rs* 와 같은 공통 모듈로 추출하려고 할 때 가장 눈에 띄게 나타납니다. 이 경우, *tests/common.rs* 파일에서 정의된 함수를 *tests/integration_test.rs* 와 같은 다른 통합 테스트 파일에서 사용하려면 `use` 문을 사용해야 합니다. 이는 단위 테스트에서 사용하는 것과 동일합니다.


그리고 `setup`이라는 함수를 넣으면, 여러 테스트 함수에서 공유할 수 있는 코드를 `setup`에 추가할 수 있습니다.

<span class=\"filename\">Filename: tests/common.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/tests/common.rs}}
```

테스트를 다시 실행하면, `common.rs` 파일의 새로운 섹션이 테스트 출력에 나타납니다. 하지만 이 파일에는 테스트 함수도 없고 `setup` 함수를 어디에서도 호출하지 않았습니다.

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/output.txt}}
```

`common`이 테스트 결과에 `running 0 tests`로 표시되는 것은 원치 않는 결과입니다. 우리는 다른 통합 테스트 파일과 공유하려는 코드가 있을 뿐입니다. `common`이 테스트 출력에 나타나지 않도록 하려면, `tests/common.rs` 대신 `tests/common/mod.rs`를 만들어야 합니다. 프로젝트 디렉토리가 이제 다음과 같습니다.

```text
\u251c\u2500\u2500 Cargo.lock
\u251c\u2500\u2500 Cargo.toml
\u251c\u2500\u2500 src
\u2502\u00a0\u00a0 \u2514\u2500\u2500 lib.rs
\u2514\u2500\u2500 tests
    \u251c\u2500\u2500 common
    \u2502\u00a0\u00a0 \u2514\u2500\u2500 mod.rs
    \u2514\u2500\u2500 integration_test.rs
```

이것은 Rust에서도 이해하는 더 오래된 파일 이름 규칙입니다. 7장의 [\u201cAlternate File Paths\u201d][alt-paths]<!-- ignore --> 섹션에서 언급했듯이, 이러한 파일 이름을 사용하면 `common` 모듈을 통합 테스트 파일로 취급하지 않습니다. `setup` 함수 코드를 `tests/common/mod.rs`로 이동하고 `tests/common.rs` 파일을 삭제하면 테스트 출력에서 해당 섹션이 사라집니다.

`tests` 디렉토리의 하위 디렉토리에 있는 파일은 별도의 crate로 컴파일되지 않으며 테스트 출력에서도 섹션이 나타나지 않습니다.

`tests/common/mod.rs`를 만들면, 이를 모든 통합 테스트 파일에서 모듈로 사용할 수 있습니다. 예를 들어, `tests/integration_test.rs`의 `it_adds_two` 테스트에서 `setup` 함수를 호출하는 방법은 다음과 같습니다.

<span class=\"filename\">Filename: tests/integration_test.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-13-fix-shared-test-code-problem/tests/integration_test.rs}}
```

참고로 `mod common;` 선언은 7-21번째 목록에서 보여준 모듈 선언과 동일합니다. 테스트 함수에서 `common::setup()` 함수를 호출할 수 있습니다.

#### 바이너리 크레이트의 통합 테스트

프로젝트가 *src/main.rs* 파일만 포함하는 바이너리 크레이트이고 *src/lib.rs* 파일이 없는 경우, *tests* 디렉토리에 통합 테스트를 만들고 `use` 문으로 `src/main.rs` 파일에서 정의된 함수를 범위 내에 가져올 수 없습니다. 바이너리 크레이트는 다른 크레이트가 사용할 수 있는 함수를 노출하지 않기 때문입니다. 바이너리 크레이트는 스스로 실행되도록 설계되었습니다.

이것은 바이너리 프로젝트에 대한 Rust의 일반적인 구조입니다. 바이너리 프로젝트는 *src/main.rs* 파일에서 로직을 호출하는 간단한 구조를 가지고 있습니다. 이 로직은 *src/lib.rs* 파일에서 정의됩니다. 이러한 구조를 사용하면 통합 테스트가 라이브러리 크레이트를 테스트할 수 있습니다. 라이브러리 크레이트의 중요한 기능이 작동한다면, *src/main.rs* 파일의 작은 코드도 작동할 것입니다. 이 작은 코드는 테스트할 필요가 없습니다.

## 요약

Rust의 테스트 기능은 코드가 예상대로 작동하도록 지속적으로 확인하는 방법을 제공합니다. 단위 테스트는 라이브러리의 각 부분을 별도로 실행하고 개인적인 구현 세부 사항을 테스트할 수 있습니다. 통합 테스트는 라이브러리의 여러 부분이 함께 올바르게 작동하는지 확인하며, 라이브러리의 공개 API를 사용하여 코드가 외부 코드에서 사용되는 방식으로 테스트합니다. Rust의 타입 시스템과 소유권 규칙은 일부 오류를 방지하는 데 도움이 되지만, 코드의 예상된 동작 방식에 대한 논리 오류를 줄이기 위해 테스트는 여전히 중요합니다.

이 장에서 배운 지식과 이전 장의 지식을 결합하여 프로젝트를 진행해 보세요!

**참고:**

* [paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
* [separating-modules-into-files]:
ch07-05-separating-modules-into-different-files.html
* [alt-paths]: ch07-05-separating-modules-into-different-files.html#alternate-file-paths
