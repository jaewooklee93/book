## D 부록 - 유용한 개발 도구

이 부록에서는 Rust 프로젝트에서 제공하는 유용한 개발 도구에 대해 논합니다. 자동 형식화, 경고 수정에 대한 빠른 방법, 코드 분석 도구, IDE와의 통합을 살펴보겠습니다.

### `rustfmt`를 사용한 자동 형식화

`rustfmt` 도구는 커뮤니티 코드 스타일을 따르도록 코드를 다시 형식화합니다. 많은 협업 프로젝트에서 `rustfmt`를 사용하여 Rust를 작성할 때 어떤 스타일을 사용할지에 대한 논쟁을 방지합니다. 모든 사람이 도구를 사용하여 코드를 형식화합니다.

`rustfmt`를 설치하려면 다음을 입력하십시오.

```console
$ rustup component add rustfmt
```

이 명령은 `rustfmt`와 `cargo-fmt`를 제공합니다. Rust가 `rustc`와 `cargo`를 제공하는 방식과 유사합니다. 현재 crate의 모든 Rust 코드를 형식화하려면 다음을 입력하십시오.

```console
$ cargo fmt
```

이 명령을 실행하면 현재 crate의 모든 Rust 코드가 다시 형식화됩니다. 이는 코드 스타일만 변경하고 코드의 의미를 변경하지 않습니다. `rustfmt`에 대한 자세한 내용은 [문서][rustfmt]를 참조하십시오.

[rustfmt]: https://github.com/rust-lang/rustfmt

### `rustfix`를 사용하여 코드 수정

`rustfix` 도구는 Rust 설치에 포함되어 있으며, 명확한 수정 방법이 있는 컴파일러 경고를 자동으로 수정할 수 있습니다. 이는 원하는 것일 가능성이 높습니다. 이전에 컴파일러 경고를 본 적이 있을 것입니다. 예를 들어 다음 코드를 살펴보십시오.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
fn do_something() {}

fn main() {
    for i in 0..100 {
        do_something();
    }
}
```

여기서 `do_something` 함수를 100번 호출하지만 `for` 루프의 몸체에서 변수 `i`를 사용하지 않습니다. Rust는 이에 대해 경고합니다.

```console
$ cargo build
   Compiling myprogram v0.1.0 (file:///projects/myprogram)
warning: unused variable: `i`
 --> src/main.rs:4:9
  |
4 |     for i in 0..100 {
  |         ^ help: consider using `_i` instead
  |
  = note: #[warn(unused_variables)] on by default

    Finished dev [unoptimized + debuginfo] target(s) in 0.50s
```

경고에서는 `_i`를 사용하라고 제안합니다. 밑줄은 이 변수를 사용하지 않도록 의도하고 있음을 나타냅니다. `cargo fix` 명령을 사용하여 이 제안을 자동으로 적용할 수 있습니다.

```console
$ cargo fix
    Checking myprogram v0.1.0 (file:///projects/myprogram)
      Fixing src/main.rs (1 fix)
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s
```

*src/main.rs*를 다시 살펴보면 `cargo fix`가 코드를 변경했음을 알 수 있습니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
fn do_something() {}

fn main() {
    for _i in 0..100 {
        do_something();
    }
}
```

`for` 루프 변수는 `_i`로 변경되었으며 경고가 더 이상 나타나지 않습니다.

`cargo fix` 명령을 사용하여 코드를 다른 Rust 버전 간에 전환할 수도 있습니다. 버전은 [D 부록][editions]에서 다룹니다.

### Clippy를 사용한 더 많은 코드 분석

Clippy 도구는 코드를 분석하여 일반적인 오류를 찾고 Rust 코드를 개선할 수 있는 코드 분석 도구 모음입니다.

Clippy를 설치하려면 다음을 입력하십시오.

```console
$ rustup component add clippy
```

어떤 Cargo 프로젝트에 Clippy의 코드 분석을 실행하려면 다음을 입력하십시오.

```console
$ cargo clippy
```

예를 들어, `f{32, 64}::consts::PI`와 같은 수학적 상수의 근사치를 사용하는 프로그램을 작성했다고 가정해 보겠습니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
fn main() {
    let x = 3.1415;
    let r = 8.0;
    println!(\"the area of the circle is {}\", x * r * r);
}
```

이 프로젝트에 `cargo clippy`를 실행하면 다음과 같은 오류가 발생합니다.

```text
error: approximate value of `f{32, 64}::consts::PI` found
 --> src/main.rs:2:13
  |
2 |     let x = 3.1415;
  |             ^^^^^^
  |
  = note: `#[deny(clippy::approx_constant)]` on by default
  = help: consider using the constant directly
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#approx_constant
```

이 오류는 Rust가 이미 더 정확한 `PI` 상수를 정의하고 있음을 알려줍니다.
프로그램이 상수를 사용하는 것이 더 정확하다는 것입니다.
코드를 `PI` 상수를 사용하도록 변경해야 합니다. 다음 코드는 Clippy에서 오류나 경고를 발생시키지 않습니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
fn main() {
    let x = std::f64::consts::PI;
    let r = 8.0;
    println!(\"the area of the circle is {}\", x * r * r);
}
```

Clippy에 대한 자세한 내용은 [문서][clippy]를 참조하십시오.

[clippy]: https://github.com/rust-lang/rust-clippy

### IDE 통합을 위한 `rust-analyzer` 사용

Rust 커뮤니티는 IDE 통합을 위해 [`rust-analyzer`][rust-analyzer]를 권장합니다. 이 도구는 [Language Server Protocol][lsp]<!-- ignore -->를 사용하는 컴파일러 중심 유틸리티 세트입니다. 이는 IDE와 프로그래밍 언어가 서로 소통하는 데 사용되는 명세입니다. 다양한 클라이언트가 `rust-analyzer`를 사용할 수 있으며, [Visual Studio Code용 Rust 분석기 플러그인][vscode]과 같습니다.

[lsp]: http://langserver.org/
[vscode]: https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer

`rust-analyzer` 프로젝트의 [홈페이지][rust-analyzer]<!-- ignore -->를 방문하여 설치 지침을 확인한 후 특정 IDE에서 언어 서버 지원을 설치하십시오. IDE는 자동 완성, 정의로 이동 및 줄 단위 오류와 같은 기능을 갖게 됩니다.

[rust-analyzer]: https://rust-analyzer.github.io
[editions]: appendix-05-editions.md
