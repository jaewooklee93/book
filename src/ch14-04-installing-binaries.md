<!-- Old link, do not remove -->
<a id=\"installing-binaries-from-cratesio-with-cargo-install\"></a>

## `cargo install`을 사용하여 바이너리 설치

`cargo install` 명령어를 사용하면 로컬로 바이너리 크레이트를 설치하고 사용할 수 있습니다. 이는 시스템 패키지를 대체하기 위한 것이 아니라, 다른 개발자가 [crates.io](https://crates.io/)에 공유한 도구를 편리하게 설치하는 방법을 제공합니다. 참고로, `cargo install`을 사용하여 설치할 수 있는 패키지는 바이너리 타겟을 가진 패키지여야 합니다. 바이너리 타겟은 크레이트에 *src/main.rs* 파일이나 다른 파일이 바이너리로 지정되어 있을 때 생성되는 실행 가능한 프로그램입니다. 반면 라이브러리 타겟은 스스로 실행할 수 없지만 다른 프로그램 내에 포함될 수 있는 라이브러리입니다. 일반적으로 크레이트의 *README* 파일에는 크레이트가 라이브러리, 바이너리 타겟, 또는 두 가지 모두인지에 대한 정보가 있습니다.

`cargo install`로 설치된 모든 바이너리는 설치 루트의 *bin* 폴더에 저장됩니다. Rust를 *rustup.rs*를 사용하여 설치했고, 사용자 정의 구성이 없는 경우 이 디렉토리는 *$HOME/.cargo/bin*이 될 것입니다. `cargo install`로 설치한 프로그램을 실행하려면 이 디렉토리가 `$PATH`에 포함되어 있어야 합니다.

예를 들어, 제12장에서 `grep` 도구의 Rust 구현인 `ripgrep`를 언급했습니다. `ripgrep`를 설치하려면 다음을 실행할 수 있습니다.

<!-- manual-regeneration
cargo install something you don't have, copy relevant output below
-->

```console
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v13.0.0
  Downloaded 1 crate (243.3 KB) in 0.88s
  Installing ripgrep v13.0.0
--snip--
   Compiling ripgrep v13.0.0
    Finished release [optimized + debuginfo] target(s) in 3m 10s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v13.0.0` (executable `rg`)
```

출력의 두 번째 줄에서 마지막 줄은 설치된 바이너리의 위치와 이름을 보여줍니다. `ripgrep`의 경우 `rg`입니다. 설치 디렉토리가 `$PATH`에 포함되어 있다면, 이전에 언급했듯이 `rg --help`를 실행하여 파일 검색에 사용할 수 있는 빠르고 Rust 기반 도구를 시작할 수 있습니다!
