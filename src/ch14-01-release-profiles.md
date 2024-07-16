## 릴리즈 프로파일을 사용한 빌드 커스터마이징

Rust에서 *릴리즈 프로파일*은 미리 정의되고 사용자 정의 가능한 프로파일로,
다른 구성을 통해 프로그래머가 코드 컴파일 옵션에 대한 더 많은 제어를 할 수 있도록 합니다. 각 프로파일은 독립적으로 구성됩니다.

Cargo는 `cargo build`를 실행할 때 사용하는 `dev` 프로파일과 `cargo build --release`를 실행할 때 사용하는 `release` 프로파일의 두 가지 주요 프로파일을 가지고 있습니다. `dev` 프로파일은 개발에 적합한 좋은 기본값으로 정의되며, `release` 프로파일은 릴리즈 빌드에 적합한 좋은 기본값을 가지고 있습니다.

이러한 프로파일 이름은 빌드 출력에서 익숙할 수 있습니다.

<!-- manual-regeneration
어디에서든 실행하세요:
cargo build
cargo build --release
그리고 아래 출력이 정확한지 확인하세요
-->

```console
$ cargo build
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
$ cargo build --release
    Finished release [optimized] target(s) in 0.0s
```

`dev`와 `release`는 컴파일러가 사용하는 다른 프로파일입니다.

Cargo는 프로젝트의 *Cargo.toml* 파일에서 명시적으로 추가하지 않으면 각 프로파일에 적용되는 기본 설정을 가지고 있습니다. `[profile.*]` 섹션을 추가하여 원하는 모든 프로파일을 사용자 정의하면 기본 설정의 일부를 재정의할 수 있습니다. 예를 들어, `dev`와 `release` 프로파일의 `opt-level` 설정의 기본값은 다음과 같습니다.

Filename: Cargo.toml

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

`opt-level` 설정은 Rust이 코드에 적용할 최적화 수준을 제어하며, 0부터 3까지의 범위를 가지고 있습니다. 더 많은 최적화를 적용하면 컴파일 시간이 늘어나므로 개발 중이고 코드를 자주 컴파일하는 경우, 실행 속도가 느려지더라도 컴파일 속도가 빠르도록 최적화 수준을 줄이는 것이 좋습니다. 따라서 `dev`의 기본 `opt-level`은 `0`입니다. 코드를 릴리즈할 준비가 되면 컴파일 시간을 더 많이 투자하는 것이 좋습니다. 릴리즈 모드로 컴파일하는 것은 한 번만 하지만, 컴파일된 프로그램을 여러 번 실행하므로 릴리즈 모드는 컴파일 시간이 길어지지만 실행 속도가 빠른 코드를 제공합니다. 그렇기 때문에 `release` 프로파일의 기본 `opt-level`은 `3`입니다.

*Cargo.toml*에서 다른 값을 추가하여 기본 설정을 재정의할 수 있습니다. 예를 들어, 개발 프로파일에서 최적화 수준 1을 사용하려면 프로젝트의 *Cargo.toml* 파일에 다음 두 줄을 추가할 수 있습니다.

Filename: Cargo.toml

```toml
[profile.dev]
opt-level = 1
```

이 코드는 기본 설정을 재정의합니다. 이제 `cargo build`를 실행하면 Cargo는 `dev` 프로파일의 기본값과 `opt-level`에 대한 사용자 정의를 사용하여 컴파일합니다. `opt-level`을 1로 설정했기 때문에 Cargo는 기본값보다 더 많은 최적화를 적용하지만 릴리즈 빌드만큼은 적용하지 않습니다.

각 프로파일의 구성 옵션 및 기본값의 전체 목록은 [Cargo 문서](https://doc.rust-lang.org/cargo/reference/profiles.html)를 참조하십시오.
