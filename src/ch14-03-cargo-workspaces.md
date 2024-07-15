## Cargo 작업 공간

제12장에서 바이너리 크레이트와 라이브러리 크레이트를 포함하는 패키지를 구축했습니다.
프로젝트가 발전함에 따라 라이브러리 크레이트가 계속 커지고 여러 라이브러리 크레이트로 패키지를 분할하고 싶을 수 있습니다.
Cargo는 동시에 개발되는 여러 관련 패키지를 관리하는 데 도움이 되는 *작업 공간*이라는 기능을 제공합니다.

### 작업 공간 만들기

*작업 공간*은 동일한 *Cargo.lock* 및 출력 디렉토리를 공유하는 패키지 집합입니다. 작업 공간을 사용하여 프로젝트를 구축해 보겠습니다.\u2014단순한 코드를 사용하여 작업 공간의 구조에 집중합니다. 작업 공간을 구조하는 방법은 여러 가지가 있으므로, 하나의 일반적인 방법만 보여드리겠습니다. 바이너리와 두 개의 라이브러리를 포함하는 작업 공간을 만들겠습니다. 바이너리는 두 라이브러리에 의존하여 주요 기능을 제공합니다. 하나의 라이브러리는 `add_one` 함수를 제공하고, 두 번째 라이브러리는 `add_two` 함수를 제공합니다. 이 세 개의 크레이트는 같은 작업 공간에 포함됩니다. 먼저 작업 공간에 대한 새로운 디렉토리를 만들어 보겠습니다.

```console
$ mkdir add
$ cd add
```

다음으로, *add* 디렉토리에 전체 작업 공간을 구성하는 *Cargo.toml* 파일을 만듭니다. 이 파일에는 `[package]` 섹션이 없으며, 대신 작업 공간에 구성원을 추가하여 바이너리 크레이트의 경로를 지정할 수 있는 `[workspace]` 섹션이 있습니다. 이 경우 경로는 *adder*입니다.

<span class=\"filename\">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-01-workspace-with-adder-crate/add/Cargo.toml}}
```

다음으로, `cargo new`를 사용하여 *add* 디렉토리 내에 `adder` 바이너리 크레이트를 만듭니다.

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-01-adder-crate/add
rm -rf adder
cargo new adder
copy output below
-->

```console
$ cargo new adder
     Created binary (application) `adder` package
```

이제 `cargo build`를 실행하여 작업 공간을 빌드할 수 있습니다. *add* 디렉토리의 파일은 다음과 같습니다.

```text
\u251c\u2500\u2500 Cargo.lock
\u251c\u2500\u2500 Cargo.toml
\u251c\u2500\u2500 adder
\u2502   \u251c\u2500\u2500 Cargo.toml
\u2502   \u2514\u2500\u2500 src
\u2502       \u2514\u2500\u2500 main.rs
\u2514\u2500\u2500 target
```

작업 공간은 최상위 레벨에 하나의 *target* 디렉토리를 가지며, 컴파일된 아티팩트가 저장됩니다. `adder` 패키지는 자신의 *target* 디렉토리를 가지고 있지 않습니다. `adder` 디렉토리 내에서 `cargo build`를 실행하더라도 컴파일된 아티팩트는 *add/target*에 저장되며 *add/adder/target*에 저장되지 않습니다. Cargo는 작업 공간의 크레이트가 서로 의존하도록 설계되었기 때문에 작업 공간의 *target* 디렉토리를 이렇게 구조화합니다. 각 크레이트가 자신의 *target* 디렉토리를 가지고 있다면, 각 크레이트는 다른 크레이트를 다시 컴파일하여 아티팩트를 자신의 *target* 디렉토리에 저장해야 합니다. 하나의 *target* 디렉토리를 공유하면 크레이트는 불필요한 재컴파일을 피할 수 있습니다.

### 작업 공간에 두 번째 패키지 추가

다음으로, 작업 공간에 또 다른 구성원 패키지를 만들어 `add_one`이라고 합니다. 최상위 *Cargo.toml* 파일을 수정하여 `members` 목록에 *add_one* 경로를 지정합니다.

<span class=\"filename\">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/Cargo.toml}}
```

그런 다음 `cargo new add_one --lib` 명령을 사용하여 `add_one`이라는 이름의 새로운 라이브러리 크레이트를 생성합니다.

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-02-add-one/add
rm -rf add_one
cargo new add_one --lib
copy output below
-->

```console
$ cargo new add_one --lib
     Created library `add_one` package
```

이제 *add* 디렉토리에는 다음과 같은 디렉토리와 파일이 있습니다.

```text
\u251c\u2500\u2500 Cargo.lock
\u251c\u2500\u2500 Cargo.toml
\u251c\u2500\u2500 add_one
\u2502   \u251c\u2500\u2500 Cargo.toml
\u2502   \u2514\u2500\u2500 src
\u2502       \u2514\u2500\u2500 lib.rs
\u251c\u2500\u2500 adder
\u2502   \u251c\u2500\u2500 Cargo.toml
\u2502   \u2514\u2500\u2500 src
\u2502       \u2514\u2500\u2500 main.rs
\u2514\u2500\u2500 target
```

add_one/src/lib.rs" 파일에서 `add_one` 함수를 추가해 보겠습니다.

<span class=\"filename\">Filename: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/add_one/src/lib.rs}}
```

이제 `adder` 패키지가 우리의 바이너리에 `add_one` 패키지가 의존하도록 할 수 있습니다. 먼저 `add_one` 에 대한 경로 의존성을 *adder/Cargo.toml* 에 추가해야 합니다.

<span class=\"filename\">Filename: adder/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/adder/Cargo.toml:6:7}}
```

Cargo는 작업 공간의 crate가 서로 의존할 것이라고 가정하지 않습니다. 따라서 의존성 관계에 대해 명시적으로 알려줘야 합니다.

다음으로 `adder` crate에서 `add_one` 함수( `add_one` crate에서)를 사용하는 방법을 살펴보겠습니다. *adder/src/main.rs* 파일을 열고 `add_one` 라이브러리 crate를 스코프에 가져오기 위해 `use` 문을 맨 위에 추가합니다. 그런 다음 Listing 14-7과 같이 `main` 함수를 `add_one` 함수를 호출하도록 변경합니다.

<Listing number=\"14-7\" file-name=\"adder/src/main.rs\" caption=\"`add_one` 라이브러리 crate를 `adder` crate에서 사용\">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-07/add/adder/src/main.rs}}
```

</Listing>

`add` 폴더의 최상위에서 `cargo build`를 실행하여 작업 공간을 빌드합니다!

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.68s
```

`adder` crate에서 바이너리 crate를 실행하려면 작업 공간의 `-p` 인수와 패키지 이름을 사용하여 `cargo run` 명령을 사용하여 실행하려는 패키지를 지정할 수 있습니다.

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo run -p adder
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo run -p adder
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

이렇게 하면 `adder/src/main.rs`의 코드가 실행되며, `add_one` crate에 의존합니다.

#### 작업 공간에서 외부 패키지 의존

작업 공간에는 최상위에 하나의 *Cargo.lock* 파일만 있음을 알 수 있습니다. 각 crate 디렉토리에 *Cargo.lock* 파일이 있는 것이 아니라요. 이는 모든 crate가 모든 의존성의 동일한 버전을 사용하도록 보장하기 때문입니다. `rand` 패키지를 `adder/Cargo.toml`과 `add_one/Cargo.toml` 파일에 추가하면 Cargo는 두 패키지 모두를 하나의 `rand` 버전으로 해결하고 하나의 *Cargo.lock*에 기록합니다. 작업 공간의 모든 crate에 동일한 의존성을 사용하면 crate가 항상 서로 호환될 수 있습니다. `add_one` crate에서 `rand` crate를 사용할 수 있도록 `add_one/Cargo.toml` 파일에 `[dependencies]` 섹션에 `rand` crate를 추가해 보겠습니다.

<!-- `rand` 버전을 업데이트할 때, 이러한 파일에서 사용하는 `rand` 버전도 모두 일치하도록 업데이트해야 합니다:
* ch02-00-guessing-game-tutorial.md
ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
-->

<span class=\"filename\">Filename: add_one/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add/add_one/Cargo.toml:6:7}}
```

이제 *add_one/src/lib.rs* 파일에 `use rand;`를 추가하고, *add* 디렉토리에서 `cargo build`를 실행하여 작업 공간 전체를 빌드하면 `rand` crate가 가져와서 컴파일됩니다. `rand`를 사용하지 않고 있기 때문에 다음과 같은 경고 메시지가 표시됩니다.

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
   --snip--
   Compiling rand v0.8.5
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: `rand`
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

warning: `add_one` (lib) generated 1 warning
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 10.18s
```

최상위 *Cargo.lock* 파일에는 이제 `add_one` 의 `rand` 의존성 정보가 포함됩니다. 그러나 `rand` 가 작업 공간 어딘가에서 사용되더라도 `adder` 패키지의 *Cargo.toml* 파일에 `rand` 를 추가하지 않으면 다른 작업 공간의 crate 에서는 사용할 수 없습니다. 예를 들어, `adder/src/main.rs` 파일에 `use rand;` 를 추가하면 다음과 같은 오류가 발생합니다.

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-03-use-rand/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import `rand`
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate `rand`
```

이 오류를 해결하려면 `adder` 패키지의 *Cargo.toml* 파일을 편집하고 `rand` 를 `adder` 의 의존성으로 지정합니다. `adder` 패키지를 빌드하면 `rand` 가 `adder` 의 의존성 목록에 `Cargo.lock` 에 추가되지만, `rand` 의 추가 복사본은 다운로드되지 않습니다. Cargo는 작업 공간의 모든 crate 에서 `rand` 패키지를 사용하는 데 동일한 버전을 사용하도록 보장하며, 공간을 절약하고 작업 공간의 crate 간 호환성을 보장합니다.

만약 작업 공간의 crate 가 동일한 의존성에 대해 호환되지 않는 버전을 지정하면 Cargo는 각각을 해결하지만, 가능한 한 적은 버전을 해결하려고 합니다.

#### 작업 공간에 테스트 추가하기

다른 개선 사항으로서, `add_one::add_one` 함수를 `add_one` crate 내에서 테스트합니다.

<span class=\"filename\">Filename: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add/add_one/src/lib.rs}}
```

이제 최상위 *add* 디렉토리에서 `cargo test` 를 실행합니다. 이러한 구조의 작업 공간에서 `cargo test` 를 실행하면 작업 공간의 모든 crate 의 테스트가 실행됩니다.

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test
아래 출력을 복사하세요. 출력 업데이트 스크립트가 경로 내부의 하위 디렉토리를 제대로 처리하지 않습니다.
-->

```console
$ cargo test
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.27s
     Running unittests src/lib.rs (target/debug/deps/add_one-f0253159197f7841)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-49979ff40686fa8e)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

첫 번째 출력 섹션은 `add_one` 크레인의 `it_works` 테스트가 통과했다는 것을 보여줍니다.
다음 섹션은 `adder` 크레인에서 테스트를 찾지 못했다고 보여주며, 마지막 섹션은 `add_one` 크레인에서 문서 테스트를 찾지 못했다고 보여줍니다.

우리는 또한 작업 공간의 특정 크레인의 테스트를 탑 레벨 디렉토리에서 `-p` 플래그를 사용하여 크레인 이름을 지정하여 실행할 수 있습니다.

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test -p add_one
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo test -p add_one
    Finished test [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-b3235fea9a156f74)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

이 출력은 `cargo test`가 `add_one` 크레인의 테스트만 실행했고 `adder` 크레인 테스트는 실행하지 않았음을 보여줍니다.

만약 작업 공간의 크레인을 [crates.io](https://crates.io/)에 게시한다면, 작업 공간의 각 크레인은 별도로 게시해야 합니다. `cargo test`와 마찬가지로, 특정 크레인을 게시하려면 `-p` 플래그를 사용하여 게시하려는 크레인의 이름을 지정할 수 있습니다.

추가 연습으로, `add_one` 크레인과 유사하게 이 작업 공간에 `add_two` 크레인을 추가해 보세요!

프로젝트가 커질수록 작업 공간을 사용하는 것이 좋습니다. 작고 개별적인 구성 요소를 이해하는 것이 큰 코드 블록을 이해하는 것보다 쉽기 때문입니다. 또한 크레인을 작업 공간에 유지하면 동시에 자주 변경되는 경우 크레인 간의 협업을 쉽게 할 수 있습니다.
