## Cargo 인사!

Cargo는 Rust의 빌드 시스템과 패키지 관리자입니다. 대부분의 Rust 개발자들은 Cargo를 사용하여 Rust 프로젝트를 관리합니다. Cargo는 코드를 빌드하는 것뿐만 아니라 코드가 의존하는 라이브러리를 다운로드하고 빌드하는 등 여러 작업을 처리합니다. (코드가 필요로 하는 라이브러리를 *의존성*이라고 부릅니다.)

가장 간단한 Rust 프로그램은 아직까지 작성한 것과 같이 의존성이 없습니다. 만약 Cargo를 사용하여 “Hello, world!” 프로젝트를 빌드했다면, Cargo의 코드 빌드 기능만 사용할 것입니다. 더 복잡한 Rust 프로그램을 작성하면서 의존성을 추가하게 되면, Cargo를 사용하면 의존성 추가가 훨씬 쉬워집니다.

Rust 프로젝트의 대부분이 Cargo를 사용하기 때문에, 이 책의 나머지 부분에서는 Cargo를 사용하는 것으로 가정합니다. 공식 설치 프로그램(설치 섹션 참조)을 사용하여 Rust를 설치했다면 Cargo도 함께 설치됩니다. 다른 방법으로 Rust를 설치했다면, 다음 명령어를 터미널에 입력하여 Cargo가 설치되었는지 확인하십시오.

```console
$ cargo --version
```

버전 번호가 표시되면 Cargo가 설치되어 있습니다. `command not found`와 같은 오류가 표시되면 설치 방법의 설명서를 참조하여 Cargo를 별도로 설치하는 방법을 확인하십시오.

### Cargo로 프로젝트 생성

Cargo를 사용하여 새로운 프로젝트를 만들고, 이전에 작성한 “Hello, world!” 프로젝트와의 차이점을 살펴보겠습니다. *projects* 디렉토리(또는 코드를 저장하는 곳)로 이동한 후, 다음 명령어를 실행하십시오.

```console
$ cargo new hello_cargo
$ cd hello_cargo
```

첫 번째 명령어는 *hello_cargo*라는 새로운 디렉토리와 프로젝트를 생성합니다. 우리는 프로젝트를 *hello_cargo*로 명명했으며, Cargo는 동일한 이름의 디렉토리에 파일을 생성합니다.

*hello_cargo* 디렉토리로 이동하여 파일 목록을 확인하십시오. Cargo가 두 개의 파일과 *src* 디렉토리를 생성했음을 알 수 있습니다. *src* 디렉토리에는 *main.rs* 파일이 포함되어 있습니다.

Cargo는 또한 새로운 Git 저장소를 초기화하고 *.gitignore* 파일을 생성했습니다. Git 파일은 기존 Git 저장소 내에서 `cargo new`를 실행하면 생성되지 않습니다. 다른 버전 관리 시스템 또는 버전 관리 시스템 없이 `cargo new`를 사용하려면 `--vcs` 플래그를 사용하십시오. `cargo new --help`를 실행하여 사용 가능한 옵션을 확인하십시오.

> 주의: Git는 일반적인 버전 관리 시스템입니다. `cargo new`를 다른 버전 관리 시스템 또는 버전 관리 시스템 없이 사용하려면 `--vcs` 플래그를 사용하십시오. `cargo new --help`를 실행하여 사용 가능한 옵션을 확인하십시오.

*Cargo.toml* 파일을 텍스트 편집기로 열어보십시오. Listing 1-2와 유사한 코드를 볼 수 있습니다.

<Listing number=\"1-2\" file-name=\"Cargo.toml\" caption=\"`cargo new`에 의해 생성된 *Cargo.toml* 파일 내용\">

```toml
[package]
name = \"hello_cargo\"
version = \"0.1.0\"
edition = \"2021\"

# https://doc.rust-lang.org/cargo/reference/manifest.html 에서 더 많은 키와 정의를 확인하십시오.

[dependencies]
```

</Listing>

이 파일은 Cargo의 구성 파일인 [*TOML*][toml](*Tom's Obvious, Minimal Language*) 형식으로 작성되어 있습니다.

첫 번째 줄, `[package]`는 다음 문장이 패키지를 구성한다는 것을 나타내는 섹션 제목입니다. 이 파일에 더 많은 정보를 추가하면서 다른 섹션도 추가할 것입니다.

다음 세 줄은 Cargo가 프로그램을 컴파일하기 위해 필요한 구성 정보를 설정합니다. 이름, 버전, 사용할 Rust 에디션입니다. `edition` 키에 대해서는 [Appendix E][appendix-e]에서 설명합니다.

마지막 줄, `[dependencies]`는 프로젝트의 의존성을 나열하는 섹션의 시작입니다. Rust에서 코드 패키지는 *crates*라고 부릅니다. 이 프로젝트에는 다른 crates가 필요하지 않지만, 제2장의 첫 번째 프로젝트에서는 사용할 것이므로 이 의존성 섹션을 사용할 것입니다.

이제 *src/main.rs*를 열어보십시오.

Filename: src/main.rs

```rust
fn main() {
    println!(\"Hello, world!\");
}
```

Cargo가 “Hello, world!” 프로그램을 생성해 주었습니다. Listing 1-1에서 작성한 것과 동일합니다! 이제까지 우리 프로젝트와 이전 프로젝트의 차이점은 무엇인지 살펴보았습니다. 이제 Cargo를 사용하여 Rust 프로젝트를 관리하는 방법을 더 자세히 알아보겠습니다.

Cargo로 생성된 프로젝트는 Cargo가 코드를 *src* 디렉토리에
두고 *Cargo.toml* 구성 파일을 최상위 디렉토리에 놓는 것입니다.

Cargo는 소스 파일이 *src* 디렉토리 안에 있어야 한다고 기대합니다.
최상위 프로젝트 디렉토리는 README 파일, 라이선스 정보,
구성 파일 및 코드와 관련 없는 다른 모든 것들을 위한 것입니다.
Cargo를 사용하면 프로젝트를 구성하는 데 도움이 됩니다. 모든 곳에
장소가 있고, 모든 것이 그 자리에 있습니다.

Cargo를 사용하지 않는 프로젝트를 시작했다면, "Hello, world!" 프로젝트와
같은 경우, Cargo를 사용하는 프로젝트로 변환할 수 있습니다. 프로젝트 코드를
*src* 디렉토리로 이동하고 적절한 *Cargo.toml* 파일을 만듭니다.

### Cargo 프로젝트 빌드 및 실행

이제 Cargo로 "Hello, world!" 프로그램을 빌드하고 실행할 때
차이점을 살펴보겠습니다. *hello_cargo* 디렉토리에서 다음 명령어를 입력하여
프로젝트를 빌드합니다.

```console
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

이 명령어는 *target/debug/hello_cargo* (또는 Windows에서는
*target\\debug\\hello_cargo.exe*)와 같은 디렉토리에 실행 파일을 생성합니다.
default build가 디버그 빌드이기 때문에 Cargo는 *debug* 라는 이름의
디렉토리에 바이너리를 넣습니다. 다음 명령어로 실행 파일을 실행할 수 있습니다.

```console
$ ./target/debug/hello_cargo # or .\\target\\debug\\hello_cargo.exe on Windows
Hello, world!
```

만약 모든 것이 잘 작동하면 `Hello, world!` 가 터미널에 출력되어야 합니다.
`cargo build`를 처음 실행하면 Cargo가 최상위에 새로운 파일을
만들게 됩니다: *Cargo.lock*. 이 파일은 프로젝트의 의존성의 정확한 버전을
추적합니다. 이 프로젝트에는 의존성이 없으므로 파일이 약간 빈껍니다.
이 파일을 수동으로 변경할 필요는 없습니다. Cargo가 그 내용을 관리합니다.

`cargo build`로 프로젝트를 빌드하고 `./target/debug/hello_cargo`로 실행했지만,
`cargo run`을 사용하여 코드를 컴파일하고 결과 실행 파일을 한 번에 실행할 수도 있습니다.

```console
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

`cargo run`을 사용하면 `cargo build`를 기억해야 하고 실행 파일의 전체 경로를
입력해야 하는 번거로움을 피할 수 있으므로, 대부분의 개발자가 `cargo run`을
사용합니다.

이번에는 Cargo가 코드를 컴파일하는 출력을 보지 않았습니다. Cargo는 파일이
변하지 않았음을 알아내어 컴파일하지 않았지만 실행 파일을 실행했습니다.
만약 소스 코드를 수정했다면, Cargo는 실행하기 전에 프로젝트를 다시
빌드했을 것이며, 다음과 같은 출력을 보았을 것입니다.

```console
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Cargo는 `cargo check`라는 명령어도 제공합니다. 이 명령어는 코드를 컴파일
되도록 빠르게 확인하지만 실행 파일을 생성하지 않습니다.

```console
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

실행 파일을 생성하지 않으려면 왜 `cargo check`를 사용할까요? 종종 `cargo check`는
`cargo build`보다 훨씬 빠르기 때문입니다. 코드를 작성하는 동안 작업을
지속적으로 확인하는 경우 `cargo check`를 사용하면 프로젝트가 여전히
컴파일되는지 확인하는 데 시간을 단축할 수 있습니다. 따라서 많은 Rust 개발자들이
코드를 작성하는 동안 `cargo check`를 주기적으로 실행하여 프로젝트가
컴파일되는지 확인하고, 실행 파일을 사용할 준비가 되면 `cargo build`를 실행합니다.

Cargo에 대해서 아직 배운 내용을 요약해 보겠습니다.

* `cargo new`를 사용하여 프로젝트를 만들 수 있습니다.
* `cargo build`를 사용하여 프로젝트를 빌드할 수 있습니다.
* `cargo run`을 사용하여 프로젝트를 빌드하고 실행할 수 있습니다.
 * 오류를 확인하기 위해 바이너리를 생성하지 않고 프로젝트를 빌드할 수 있습니다.
  `cargo check`를 사용하여
 * Cargo는 코드와 같은 디렉토리에 결과를 저장하는 대신 *target/debug* 디렉토리에 저장합니다.

Cargo를 사용하는 또 다른 장점은 운영 체제에 관계없이 명령어가 동일하다는 것입니다. 따라서 이제는 Linux와 macOS와 Windows에 대한 구체적인 지침을 제공하지 않습니다.

### 배포용 빌드

프로젝트가 마침내 배포 준비가 되면 `cargo build --release`를 사용하여 최적화된 컴파일을 수행할 수 있습니다. 이 명령어는 *target/release*에 실행 파일을 생성합니다. *target/debug* 대신. 최적화를 사용하면 Rust 코드가 더 빠르게 실행되지만, 프로그램 컴파일 시간이 길어집니다. 이것이 개발용 프로필과 배포용 프로필이 있는 이유입니다. 개발용 프로필은 컴파일을 자주하고 빠르게 다시 빌드해야 할 때 사용되며, 배포용 프로필은 사용자에게 제공할 최종 프로그램을 빌드할 때 사용됩니다. 이 프로그램은 반복적으로 다시 빌드되지 않으며 가능한 한 빠르게 실행됩니다. 코드 실행 시간을 측정하는 경우 `cargo build --release`를 실행하고 *target/release*에 있는 실행 파일을 사용하여 측정해야 합니다.

### Cargo는 관습으로

단순한 프로젝트의 경우 Cargo가 `rustc`만 사용하는 것보다 큰 가치를 제공하지 않지만, 프로그램이 더 복잡해질수록 그 가치가 증가합니다. 프로그램이 여러 파일로 구성되거나 의존성이 필요해지면 Cargo가 빌드를 조정하는 것이 훨씬 쉬워집니다.

`hello_cargo` 프로젝트가 간단하더라도 이제는 Rust 커리어의 나머지 부분에서 사용할 실제 도구의 대부분을 사용하고 있습니다. 실제로 기존 프로젝트에 작업하려면 Git을 사용하여 코드를 확인하고 프로젝트 디렉토리로 이동한 후 빌드할 수 있습니다.

```console
$ git clone example.org/someproject
$ cd someproject
$ cargo build
```

Cargo에 대한 자세한 내용은 [문서](https://doc.rust-lang.org/cargo/)를 참조하십시오.

## 요약

Rust 여정을 시작하는 데 이미 훌륭한 시작을 하셨습니다! 이 장에서는 다음을 배웠습니다.

* `rustup`을 사용하여 Rust의 최신 안정 버전을 설치하는 방법
* Rust 버전을 업데이트하는 방법
* 로컬로 설치된 설명서를 열기 위한 방법
* `rustc`를 직접 사용하여 “Hello, world!” 프로그램을 작성하고 실행하는 방법
* Cargo의 규칙을 사용하여 새 프로젝트를 만들고 실행하는 방법

이제 Rust 코드를 읽고 작성하는 데 익숙해지기 위해 더 중요한 프로그램을 빌드하는 것이 좋습니다. 따라서 제2장에서는 추측 게임 프로그램을 빌드할 것입니다. Rust에서 일반적인 프로그래밍 개념이 어떻게 작동하는지 먼저 배우고 싶다면 제3장을 참조한 후 제2장으로 돌아오십시오.

[installation]: ch01-01-installation.html#installation
[toml]: https://toml.io
[appendix-e]: appendix-05-editions.html
[cargo]: https://doc.rust-lang.org/cargo/
