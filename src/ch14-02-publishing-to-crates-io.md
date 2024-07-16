## crates.io에 패키지 게시하기

우리는 [crates.io](https://crates.io/)<!-- ignore --> 에서 패키지를 가져와서 프로젝트의 의존성으로 사용했지만, 다른 사람들과 당신의 코드를 공유하기 위해 당신의 패키지를 게시할 수도 있습니다. [crates.io](https://crates.io/)<!-- ignore --> 에서 제공되는 패키지 레지스트리는 당신의 패키지의 소스 코드를 배포하기 때문에, 주로 오픈 소스 코드를 호스팅합니다.

Rust와 Cargo는 게시된 패키지를 다른 사람들이 쉽게 찾고 사용하도록 하는 기능을 제공합니다. 다음으로 이러한 기능에 대해 이야기하고 패키지를 게시하는 방법을 설명하겠습니다.

### 유용한 설명서 주석 작성하기

정확하게 패키지를 설명하면 다른 사용자가 언제 어떻게 사용해야 하는지 알 수 있으므로 설명서를 작성하는 데 시간을 투자하는 것이 좋습니다. 3장에서 두 개의 슬래시 `//`를 사용하여 Rust 코드에 주석을 붙이는 방법에 대해 설명했습니다. Rust는 프로그래머가 당신의 crate를 *사용하는 방법*을 알고 싶어하는 대신 당신의 crate가 *구현되는 방법*에 대해 알고 싶어하는 경우, 공개 API 항목에 대한 HTML 설명서를 생성하는 특별한 종류의 주석이 있습니다. 이를 *설명서 주석*이라고 부릅니다.

설명서 주석은 두 개의 슬래시 대신 세 개의 슬래시 `///`를 사용하고 Markdown 표현을 지원하여 텍스트를 서식화합니다. 설명서 주석을 설명하는 항목 바로 앞에 놓습니다. 14-1번 표는 `my_crate`라는 crate의 `add_one` 함수에 대한 설명서 주석을 보여줍니다.

<Listing number=\"14-1\" file-name=\"src/lib.rs\" caption=\"함수에 대한 설명서 주석\">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-01/src/lib.rs}}
```

</Listing>

여기서 `add_one` 함수가 무엇을 하는지 설명하고 `Examples` 제목으로 시작하는 섹션을 만들고 `add_one` 함수를 사용하는 코드를 제공합니다. `cargo doc`를 실행하면 이 설명서 주석에서 HTML 설명서를 생성할 수 있습니다. 이 명령은 Rust와 함께 제공되는 `rustdoc` 도구를 실행하고 생성된 HTML 설명서를 *target/doc* 디렉토리에 넣습니다.

편의를 위해 `cargo doc --open`을 실행하면 현재 crate의 설명서(및 모든 의존성의 설명서)를 빌드하고 결과를 웹 브라우저에서 열어줍니다. `add_one` 함수로 이동하면 설명서 주석의 텍스트가 표시되는 것을 볼 수 있습니다. 14-1번 그림과 같이:

<img alt=\"`my_crate`의 `add_one` 함수에 대한 렌더링된 HTML 설명서\" src=\"img/trpl14-01.png\" class=\"center\" />

<span class=\"caption\">그림 14-1: `add_one` 함수에 대한 HTML 설명서</span>

#### 일반적으로 사용되는 섹션

14-1번 표에서 `# Examples` Markdown 제목을 사용하여 HTML에서 제목이 “Examples”인 섹션을 만들었습니다. crate 작성자는 일반적으로 설명서에서 다음과 같은 섹션을 사용합니다.

* **Panics**: 설명서에 있는 함수가 panic할 수 있는 상황을 나열합니다. 프로그램이 panic하지 않도록 하려는 함수 호출자는 이러한 상황에서 함수를 호출하지 않도록 해야 합니다.
* **Errors**: 함수가 `Result`를 반환하는 경우, 발생할 수 있는 오류 유형과 이러한 오류가 반환되는 조건을 설명하는 것이 도움이 될 수 있습니다. 이렇게 하면 호출자는 다양한 유형의 오류를 다르게 처리하는 코드를 작성할 수 있습니다.
* **Safety**: 함수가 `unsafe`로 호출될 수 있는 경우(19장에서 unsafety에 대해 설명합니다) 함수가 unsafe인 이유를 설명하고 호출자에게 유지해야 하는 불변성에 대한 설명이 있어야 합니다.

대부분의 설명서 주석은 이러한 모든 섹션이 필요하지 않지만, 사용자가 알고 싶어하는 코드의 측면에 대한 확인 목록으로 유용합니다.

#### 설명서 주석을 테스트로 사용하기

설명서 주석에 예제 코드 블록을 추가하면 라이브러리를 사용하는 방법을 보여줄 수 있으며, 이렇게 하면 추가적인 이점이 있습니다. `cargo test`를 실행하면 설명서의 코드 예제를 테스트로 실행합니다! 설명서에 예제가 있는 것은 가장 좋지만, 설명서가 작성된 이후 코드가 변경되어 예제가 작동하지 않는 것은 가장 나쁜 일입니다. 14-1번 표의 `add_one` 함수의 설명서를 `cargo test`로 실행하면 테스트 결과에 다음과 같은 섹션이 나타납니다.

<!-- manual-regeneration -->cd listings/ch14-more-about-cargo/listing-14-01/
cargo test
doc-tests 섹션만 복사하세요
-->

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
```

만약 `add_one` 함수나 예제에서 `assert_eq!`이 예제에서 panic하도록 변경하고 `cargo test`를 다시 실행하면, doc tests가 예제와 코드가 서로 일치하지 않는 것을 잡아낼 것입니다!

#### 주석으로 묶인 항목

`//!` 스타일의 문서 주석은 주석이 포함된 항목에 설명을 추가하는 대신, 주석이 뒤따르는 항목에 설명을 추가합니다. 일반적으로 이러한 문서 주석은 crate 루트 파일(*src/lib.rs* 예시)이나 모듈 내부에 사용하여 crate 또는 모듈 전체를 설명합니다.

예를 들어 `my_crate` crate에 포함된 `add_one` 함수를 설명하는 문서 주석을 추가하려면 `//!`로 시작하는 문서 주석을 *src/lib.rs* 파일의 처음에 추가해야 합니다. 14-2번 목록을 참조하세요.

<Listing number=\"14-2\" file-name=\"src/lib.rs\" caption=\"`my_crate` crate 전체에 대한 문서화\">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-02/src/lib.rs:here}}
```

</Listing>

마지막 `//!`로 시작하는 줄 이후에는 코드가 없습니다. `//!`로 시작하여 `///`로 시작하지 않기 때문에, 이 주석은 이 주석이 포함된 항목인 *src/lib.rs* 파일을 설명하고 있습니다. 이러한 주석은 전체 crate를 설명합니다.

`cargo doc --open`을 실행하면 이러한 주석은 `my_crate`의 문서의 첫 페이지에 crate의 공개 항목 목록 위에 표시됩니다. 14-2번 그림을 참조하세요.

<img alt=\"crate 전체에 대한 주석이 포함된 렌더링된 HTML 문서\" src=\"img/trpl14-02.png\" class=\"center\" />

<span class=\"caption\">그림 14-2: `my_crate`의 렌더링된 문서, crate 전체를 설명하는 주석 포함</span>

항목 내부의 문서 주석은 특히 crate 및 모듈을 설명하는 데 유용합니다. 사용자가 컨테이너의 전체 목적을 이해하도록 돕기 위해 컨테이너에 대한 설명을 추가합니다.

### `pub use`를 사용하여 편리한 공개 API를 제공하기

crates를 게시할 때 공개 API의 구조는 중요한 고려 사항입니다. crate를 사용하는 사람들은 crate의 구조에 대해 당신만큼 익숙하지 않을 수 있으며, crate가 복잡한 모듈 계층 구조를 가지고 있다면 사용자들이 원하는 부분을 찾는 데 어려움을 겪을 수 있습니다.

7장에서 `pub` 키워드를 사용하여 항목을 공개하고 `use` 키워드를 사용하여 항목을 스코프로 가져오는 방법을 설명했습니다. 그러나 개발 중에 의미있는 구조를 가지고 있는 crate의 구조가 다른 사람들이 사용하기에 편리하지 않을 수 있습니다. 여러 레벨의 모듈 계층 구조로 구조화된 struct를 가질 수 있지만, 사용자가 계층 구조의 깊은 곳에서 정의된 유형에 대해 알아내기 어려울 수 있습니다. 또한 `use my_crate::some_module::another_module::UsefulType;`와 같이 복잡한 문법을 사용해야 할 수 있습니다.

좋은 소식은 내부 조직을 다시 정리하지 않아도 됩니다. 내부 조직이 다른 라이브러리에서 사용하기에 편리하지 않다면, `pub use`를 사용하여 다른 위치에서 공개 구조를 다시 내보낼 수 있습니다. 다시 내보내는 것은 공개 항목을 한 위치에서 다른 위치로 가져와서, 마치 다른 위치에서 정의된 것처럼 만드는 것입니다.

예를 들어 `art`라는 이름의 라이브러리를 만들었고, 이 라이브러리는 `kinds` 모듈( `PrimaryColor` 와 `SecondaryColor` 라는 두 개의 enum을 포함)과 `utils` 모듈( `mix` 라는 함수를 포함)을 가지고 있다고 가정해 보겠습니다. 14-3번 목록을 참조하세요.

## crates.io에 게시하기

다음은 `art`라는 라이브러리를 `kinds`와 `utils` 모듈로 구성된 예제입니다.

<Listing number=\"14-3\" file-name=\"src/lib.rs\" caption=\"`art` 라이브러리: 항목은 `kinds`와 `utils` 모듈로 구성됨\">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-03/src/lib.rs:here}}
```

</Listing>

이 라이브러리의 문서를 `cargo doc`로 생성하면 다음과 같은 화면이 표시됩니다.

<img alt=\"`art` crate의 문서의 맨 앞 페이지: `kinds`와 `utils` 모듈이 나열됨\" src=\"img/trpl14-03.png\" class=\"center\" />

<span class=\"caption\">그림 14-3: `art` crate의 문서의 맨 앞 페이지
`kinds`와 `utils` 모듈이 나열됨</span>

`PrimaryColor`와 `SecondaryColor` 타입 및 `mix` 함수는 맨 앞 페이지에 나열되지 않습니다. `kinds`와 `utils`를 클릭해야 볼 수 있습니다.

이 라이브러리를 의존하는 다른 crate는 `art`에서 항목을 가져오는 `use` 문을 필요로 합니다. 현재 정의된 모듈 구조를 명시해야 합니다. 14-4번 목록은 `art` crate의 `PrimaryColor`와 `mix` 항목을 사용하는 crate의 예입니다.

<Listing number=\"14-4\" file-name=\"src/main.rs\" caption=\"`art` crate를 사용하는 crate: 내부 구조가 내보내짐\">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-04/src/main.rs:here}}
```

</Listing>

14-4번 목록의 코드 작성자는 `PrimaryColor`가 `kinds` 모듈에 있고 `mix`가 `utils` 모듈에 있다는 것을 알아야 합니다. `art` crate의 내부 구조는 `art` crate를 개발하는 개발자에게 더 중요하며, 사용하는 개발자에게는 유용한 정보가 아닙니다. 내부 구조는 사용자가 항목을 찾는 데 어려움을 겪게 하고, `use` 문에서 모듈 이름을 명시해야 하는 불필요한 복잡성을 야기합니다.

내부 구조를 공개 API에서 제거하려면 14-3번 목록의 `art` crate 코드에 `pub use` 문을 추가하여 최상위에서 항목을 다시 내보내도록 수정할 수 있습니다. 14-5번 목록을 참조하세요.

<Listing number=\"14-5\" file-name=\"src/lib.rs\" caption=\"`pub use` 문을 추가하여 항목을 다시 내보냄\">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-05/src/lib.rs:here}}
```

</Listing>

이 라이브러리의 문서를 `cargo doc`로 생성하면 맨 앞 페이지에 다시 내보낸 항목이 나열되고 링크가 생성됩니다. 14-4번 그림을 참조하세요.

<img alt=\"`art` crate의 문서의 맨 앞 페이지: 다시 내보낸 항목이 나열됨\" src=\"img/trpl14-04.png\" class=\"center\" />

<span class=\"caption\">그림 14-4: `art` crate의 문서의 맨 앞 페이지
다시 내보낸 항목이 나열됨</span>

`art` crate 사용자는 여전히 14-3번 목록의 내부 구조를 볼 수 있습니다. 14-4번 목록과 같이 사용할 수도 있습니다. 또는 14-5번 목록과 같이 더 편리한 구조를 사용할 수도 있습니다. 14-6번 목록을 참조하세요.

<Listing number=\"14-6\" file-name=\"src/main.rs\" caption=\"`art` crate에서 다시 내보낸 항목을 사용하는 프로그램\">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-06/src/main.rs:here}}
```

</Listing>

여러 개의 중첩된 모듈이 있는 경우 `pub use`를 사용하여 최상위에서 타입을 다시 내보내면 사용자 경험에 큰 차이를 줄 수 있습니다crates.io에 게시하기 전에, 현재 crate의 의존성에 대한 정의를 만들어 해당 crate의 정의를 현재 crate의 공개 API 부분으로 만들어야 합니다.

 유용한 공개 API 구조를 만들기는 예술적 측면이 더 크며, 사용자에게 가장 적합한 API를 찾기 위해 반복적으로 수정할 수 있습니다. `pub use`를 선택하면 내부 구조를 어떻게 구성하든 유연성을 제공하며, 내부 구조를 사용자에게 보여주는 것과 분리됩니다. 설치한 crate의 일부 코드를 살펴보고 내부 구조가 공개 API와 다르게 구성되어 있는지 확인해 보세요.

### crates.io 계정 설정

 어떤 crate를 게시하기 전에 crates.io에 계정을 만들고 API 토큰을 가져와야 합니다. [crates.io](https://crates.io/)로 이동하여 GitHub 계정으로 로그인합니다. (현재 GitHub 계정은 필수 요건이며, 사이트가 미래에 다른 계정 생성 방법을 지원할 수도 있습니다.) 로그인한 후 [https://crates.io/me/](https://crates.io/me/)로 이동하여 계정 설정을 확인하고 API 키를 가져옵니다. 그런 다음 `cargo login` 명령을 실행하고 요청 시 API 키를 붙여넣습니다.

```console
$ cargo login
abcdefghijklmnopqrstuvwxyz012345
```

이 명령은 Cargo에 API 토큰을 알리고 `*~/.cargo/credentials*`에 로컬로 저장합니다. 이 토큰은 *비밀*이므로 다른 사람과 공유하지 마세요. 어떤 이유로든 다른 사람과 공유하면 crates.io에서 토큰을 취소하고 새 토큰을 생성해야 합니다.

### 새 crate에 메타데이터 추가

 게시하려는 crate가 있다고 가정해 보겠습니다. 게시하기 전에 crate의 *Cargo.toml* 파일의 `[package]` 섹션에 몇 가지 메타데이터를 추가해야 합니다.

 crate는 고유한 이름이 필요합니다. 로컬로 crate를 작업하는 동안 crate 이름을 원하는 대로 지정할 수 있습니다. 그러나 crates.io의 crate 이름은 선착순으로 할당됩니다. crate 이름이 사용되면 다른 사람은 해당 이름으로 crate를 게시할 수 없습니다. crate를 게시하기 전에 원하는 이름을 검색하십시오. 이름이 이미 사용된 경우 *Cargo.toml* 파일의 `[package]` 섹션에서 `name` 필드를 편집하여 게시 시 사용할 새로운 이름으로 변경해야 합니다.

<span class=\"filename\">Filename: Cargo.toml</span>

```toml
[package]
name = \"guessing_game\"
```

에도 불구하고 `cargo publish` 명령을 실행하여 crate를 게시하려고 하면 경고 메시지가 표시되고 오류가 발생합니다.

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-01/
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
warning: manifest has no description, license, license-file, documentation, homepage or repository.
See https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata for more info.
--snip--
error: failed to publish to registry at https://crates.io

Caused by:
  the remote server responded with an error: missing or empty metadata fields: description, license. Please see https://doc.rust-lang.org/cargo/reference/manifest.html for how to upload metadata
```

 이 오류는 설명과 라이선스가 누락되어 있기 때문에 발생합니다. 사람들이 crate가 무엇을 하는지, 어떤 조건으로 사용할 수 있는지 알 수 없기 때문입니다. *Cargo.toml*에서 검색 결과에 함께 표시될 간단한 문장이나 두 문장으로 구성된 설명을 추가합니다. `license` 필드에는 사람들이 crate를 사용할 수 있는 조건을 명시하는 *라이선스 식별자 값*을 제공해야 합니다. [Linux Foundation의 소프트웨어 패키지 데이터 교환(SPDX)][spdx]에서 사용할 수 있는 식별자 목록을 확인할 수 있습니다. 예를 들어, crate를 MIT 라이선스로 라이선스했다면 `MIT` 식별자를 추가합니다.

<span class=\"filename\">Filename: Cargo.toml</span>

```toml
[package]
name = \"guessing_game\"
license = \"MIT\"
```

 SPDX에 없는 라이선스를 사용하려면 라이선스 파일을 추가해야 합니다. 라이선스 파일은 `license-file` 필드에 지정됩니다. 예를 들어, crate가 GPL 라이선스를 사용한다면 다음과 같이 추가할 수 있습니다.

<span class=\"filename\">Filename: Cargo.toml</span>

```toml
[package]
name = \"guessing_game\"
license = \"GPL-3.0\"
license-file = \"LICENSE\"
```

 이제 `cargo publish` 명령을 실행하면 crate를 crates.io에 게시할 수 있습니다.

[spdx]: https://spdx.org/"]라이선스 파일의 텍스트를 파일로 포함하고, 해당 파일을 프로젝트에 포함한 후 `license-file` 을 사용하여 해당 파일의 이름을 지정합니다. `license` 키를 사용하지 않습니다.

프로젝트에 적합한 라이선스가 무엇인지에 대한 자세한 내용은 책의 범위를 벗어납니다. Rust 커뮤니티의 많은 사람들이 Rust와 동일한 방식으로 프로젝트를 라이선스하여 `MIT OR Apache-2.0` 의 이중 라이선스를 사용합니다. 이러한 관행은 프로젝트에 여러 라이선스를 지정할 수 있다는 것을 보여줍니다. `OR` 로 구분된 여러 라이선스 식별자를 사용할 수 있습니다.

유일한 이름, 버전, 설명 및 라이선스가 추가되면 게시를 준비한 프로젝트의 *Cargo.toml* 파일은 다음과 같습니다.

<span class=\"filename\">Filename: Cargo.toml</span>

```toml
[package]
name = \"guessing_game\"
version = \"0.1.0\"
edition = \"2021\"
description = \"컴퓨터가 선택한 숫자를 맞추는 재미있는 게임입니다.\"
license = \"MIT OR Apache-2.0\"

[dependencies]
```

[Cargo’s documentation](https://doc.rust-lang.org/cargo/) 에서는 다른 메타데이터를 지정하여 다른 사람들이 크레이트를 더 쉽게 찾고 사용할 수 있도록 하는 방법에 대한 설명을 제공합니다.

### Crates.io로 게시

이제 계정을 만들고 API 토큰을 저장하고 크레이트에 대한 이름을 선택하고 필요한 메타데이터를 지정했다면 게시할 준비가 되었습니다! 크레이트를 게시하는 것은 특정 버전을 [crates.io](https://crates.io/)에 업로드하는 것을 의미합니다.<!-- ignore --> 다른 사람들이 사용할 수 있도록 합니다.

주의해야 할 점은 게시는 *영구적* 이라는 것입니다. 버전은 다시 쓰기 불가능하며 코드를 삭제할 수 없습니다. [crates.io](https://crates.io/)<!-- ignore --> 의 주요 목표 중 하나는 코드의 영구적인 아카이브 역할을 하여 [crates.io](https://crates.io/)<!-- ignore --> 에서 크레이트를 가져온 모든 프로젝트의 빌드가 계속 작동하도록 하는 것입니다. 버전 삭제를 허용하면 이 목표를 달성하는 것이 불가능해집니다. 그러나 게시할 수 있는 크레이트 버전의 수에는 제한이 없습니다.

`cargo publish` 명령을 다시 실행합니다. 이제 성공해야 합니다.

<!-- manual-regeneration
어떤 유효한 크레이트로 이동하여 새 버전을 게시하십시오
cargo publish
아래에 관련된 줄만 복사하십시오
-->

```console
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
   Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
   Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished dev [unoptimized + debuginfo] target(s) in 0.19s
   Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
```

축하합니다! 이제 코드를 Rust 커뮤니티와 공유했으며 누구나 쉽게 프로젝트의 의존성으로 크레이트를 추가할 수 있습니다.

### 기존 크레이트의 새 버전 게시

크레이트에 변경 사항을 적용하고 새 버전을 출시할 준비가 되면 *Cargo.toml* 파일에서 지정된 `version` 값을 변경하고 다시 게시합니다. [Semantic Versioning 규칙][semver]을 사용하여 변경 사항의 유형에 따라 적절한 다음 버전 번호를 결정합니다. 그런 다음 `cargo publish`를 실행하여 새 버전을 업로드합니다.

<!-- Old link, do not remove -->
<a id=\"removing-versions-from-cratesio-with-cargo-yank\"></a>

### `cargo yank`을 사용하여 Crates.io에서 버전 삭제

크레이트 버전을 삭제할 수는 없지만, 새 프로젝트가 해당 버전을 새 의존성으로 추가하는 것을 방지할 수 있습니다. 이는 크레이트 버전이 어떤 이유로든 문제가 있는 경우 유용합니다. 이러한 상황에서 Cargo는 크레이트 버전을 *yank*하는 것을 지원합니다.

크레이트 버전을 yank하는 것은 새 프로젝트가 해당 버전에 의존하지 못하도록 하면서 기존에 의존하는 모든 프로젝트가 계속 작동하도록 하는 것입니다. 즉, yank은 모든 *Cargo.lock* 파일이 손상되지 않으며, 모든 미래 *Cargo.lock* 파일은 yank된 버전을 사용하지 않도록 합니다.

이전에 게시한 크레이트의 디렉토리에서 `cargo yank`을 실행하고 yank하려는 버전을 지정하여 크레이트 버전을 yank합니다. 예를 들어, `guessing_game`이라는 이름의 크레이트 버전 `0.1.0`을 yank하려면 다음과 같이 실행합니다.
```
$ cargo yank 0.1.0
```

1.0.1 버전을 삭제하고 싶다면, `guessing_game` 프로젝트 디렉토리에서 다음 명령을 실행합니다.

<!-- manual-regeneration:
cargo yank carol-test --version 2.1.0
cargo yank carol-test --version 2.1.0 --undo
-->

```console
$ cargo yank --vers 1.0.1
    crates.io 인덱스 업데이트
        guessing_game@1.0.1 삭제
```

`--undo` 옵션을 명령에 추가하면 삭제를 되돌리고 프로젝트가 다시 특정 버전에 의존하도록 할 수 있습니다.

```console
$ cargo yank --vers 1.0.1 --undo
    crates.io 인덱스 업데이트
      guessing_game@1.0.1 복원
```

삭제는 코드를 삭제하지 않습니다. 예를 들어, 실수로 업로드된 비밀 정보를 삭제할 수 없습니다. 그런 경우 비밀 정보를 즉시 재설정해야 합니다.

[spdx]: http://spdx.org/licenses/
[semver]: http://semver.org/
