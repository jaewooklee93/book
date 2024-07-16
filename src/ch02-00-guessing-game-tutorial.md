## 추측 게임 프로그래밍

함께 손을 잡고 Rust를 배우는 건 어떨까요? 이
장에서는 실제 프로그램을 통해 몇 가지 일반적인 Rust 개념을 소개합니다.
`let`, `match`, 메서드, 연관 함수, 외부 crate 등을 사용하는 방법을 배울 수 있습니다.
다음 장에서는 이러한 개념들을 자세히 살펴보겠습니다. 이 장에서는 단순히 기본적인 개념을 연습하는 데 집중합니다.

우리는 흔히 초보 프로그래머가 접하는 고전적인 문제인 추측 게임을 구현합니다. 게임의 방식은 다음과 같습니다: 프로그램이 1부터 100까지의 임의의 정수를 생성합니다. 그런 다음 사용자에게 숫자를 추측하도록 요청합니다. 사용자가 숫자를 입력하면 프로그램이 추측이 너무 낮거나 너무 높은지 알려줍니다. 추측이 맞으면 프로그램은 축하 메시지를 출력하고 종료됩니다.

## 새 프로젝트 설정

1장에서 만든 *projects* 디렉토리로 이동하여 Cargo를 사용하여 새 프로젝트를 생성합니다.

```console
$ cargo new guessing_game
$ cd guessing_game
```

첫 번째 명령어인 `cargo new`는 프로젝트 이름 (`guessing_game`)을 첫 번째 인수로 받습니다. 두 번째 명령어는 새 프로젝트의 디렉토리로 이동합니다.

생성된 *Cargo.toml* 파일을 살펴보세요:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial
rm -rf no-listing-01-cargo-new
cargo new no-listing-01-cargo-new --name guessing_game
cd no-listing-01-cargo-new
cargo run > output.txt 2>&1
cd ../../..
-->

<span class=\"filename\">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/Cargo.toml}}
```

1장에서 보았듯이 `cargo new`는 "Hello, world!" 프로그램을 생성합니다. *src/main.rs* 파일을 확인해 보세요:

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/src/main.rs}}
```

이제 `cargo run` 명령어를 사용하여 이 "Hello, world!" 프로그램을 컴파일하고 실행합니다.

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/output.txt}}
```

`run` 명령어는 프로젝트를 빠르게 반복적으로 개발할 때 유용합니다. 이 게임에서도 즉시 테스트할 수 있도록 도와줍니다.

*src/main.rs* 파일을 다시 열고 모든 코드를 작성할 것입니다.

## 추측 처리

추측 게임 프로그램의 첫 번째 부분은 사용자 입력을 받고, 해당 입력을 처리하고, 입력이 예상 형식인지 확인합니다. 처음에는 사용자에게 숫자를 입력하도록 하겠습니다. Listing 2-1에 있는 코드를 *src/main.rs* 에 입력하세요.

<Listing number=\"2-1\" file-name=\"src/main.rs\" caption=\"사용자로부터 추측을 받고 출력하는 코드\">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:all}}
```

</Listing> 

이 코드에는 많은 정보가 포함되어 있으므로 줄별로 살펴보겠습니다. 사용자 입력을 받고 출력하기 위해서는 `io` 입력/출력 라이브러리를 사용해야 합니다. `io` 라이브러리는 표준 라이브러리인 `std`에서 제공됩니다:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:io}}
```

기본적으로 Rust는 모든 프로그램에 포함되는 표준 라이브러리의 일부인 *prelude* 라는 항목 세트를 제공합니다. 이러한 모든 항목은 [표준 라이브러리 문서][prelude]에서 볼 수 있습니다.

사용하려는 유형이 prelude에 없으면 `use` 문을 사용하여 해당 유형을 명시적으로 스코프에 가져와야 합니다. `std::io` 라이브러리를 사용하는 것은 이러한 유형을 가져오는 방법입니다.
Rust는 여러 유용한 기능을 제공하며, 사용자 입력을 받는 능력도 포함됩니다.

1장에서 보셨듯이 `main` 함수는 프로그램의 시작점입니다.

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:main}}
```

`fn` 문법은 새로운 함수를 선언합니다. 괄호 `()`,는 매개변수가 없음을 나타내며, 중괄호 `{`,는 함수의 몸체를 시작합니다.

1장에서 배운 것처럼 `println!`은 화면에 문자열을 출력하는 매크로입니다.

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print}}
```

이 코드는 게임이 무엇인지 나타내는 프롬프트를 출력하고 사용자로부터 입력을 요청합니다.

### 변수를 사용하여 값 저장하기

다음으로 사용자 입력을 저장하기 위해 *변수*를 만들겠습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:string}}
```

이제 프로그램이 더욱 흥미로워지고 있습니다! 이 작은 줄에는 많은 일이 일어납니다. `let` 문을 사용하여 변수를 생성합니다. 다음은 또 다른 예입니다.

```rust,ignore
let apples = 5;
```

이 줄은 `apples`라는 이름의 새로운 변수를 생성하고 5라는 값에 바인딩합니다. Rust에서 변수는 기본적으로 불변입니다. 즉, 변수에 값을 할당하면 값이 변경되지 않습니다. 이 개념은 3장의 "변수와 불변성" 섹션에서 자세히 설명됩니다. 변수를 변경 가능하게 하려면 변수 이름 앞에 `mut`를 추가합니다.

```rust,ignore
let apples = 5; // 불변
let mut bananas = 5; // 변경 가능
```

> 주의: `//` 문법은 줄의 끝까지 지속되는 주석을 시작합니다. Rust는 주석을 무시합니다. 주석에 대해서는 3장의 [주석] 섹션에서 자세히 설명합니다.

추가로 돌아가서 `let mut guess`는 `guess`라는 이름의 변경 가능한 변수를 만들 것입니다. 등호 기호 (`=`)는 Rust가 변수에 값을 할당하도록 알립니다. 등호 기호의 오른쪽에는 `guess`가 바인딩될 값이 있습니다. `String::new()`를 호출하여 얻은 값입니다. `String`은 표준 라이브러리에서 제공되는 UTF-8 인코딩된 확장 가능한 문자열 유형입니다.

`::` 문법은 `String::new` 줄에서 `new`가 `String` 유형의 연관 함수임을 나타냅니다. *연관 함수*는 특정 유형에 대한 함수입니다. 이 경우 `String` 유형입니다. 이 `new` 함수는 새로운 빈 문자열을 만듭니다.

전체적으로 `let mut guess = String::new();` 줄은 현재 빈 `String` 인스턴스에 바인딩된 변경 가능한 변수를 만들었습니다.

### 사용자 입력 받기

`use std::io;`를 프로그램의 첫 번째 줄에 포함하여 표준 라이브러리의 입력/출력 기능을 사용했습니다. 이제 `stdin` 함수를 호출하여 사용자 입력을 처리할 수 있습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:read}}
```

만약 `use std::io;`를 프로그램의 시작 부분에 포함하지 않았다면, `std::io::stdin`으로 함수를 호출할 수도 있습니다. `stdin` 함수는 터미널의 표준 입력에 대한 핸들을 반환하는 `std::io::Stdin` 인스턴스를 반환합니다.

다음 줄은 `read_line` 메서드를 사용하여 표준 입력 핸들에서 사용자 입력을 가져옵니다.

```rust,ignore
.read_line(&mut guess)
```
또한 `&mut guess` 를 `read_line` 함수의 인자로 전달하여 사용자가 입력한 문자열을 어디에 저장할지 알려줍니다. `read_line` 함수의 전체 작업은 사용자가 표준 입력에 입력한 내용을 가져와 문자열에 추가하는 것입니다(내용을 덮어쓰지 않고). 따라서 해당 문자열을 인자로 전달합니다. 문자열 인자는 함수가 문자열의 내용을 변경할 수 있도록 변경 가능해야 합니다.

`&` 는 이 인자가 *참조*임을 나타냅니다. 참조를 통해 코드의 여러 부분이 한 개의 데이터에 액세스할 수 있는 방법을 제공하지만, 데이터를 메모리에 여러 번 복사할 필요가 없습니다. 참조는 복잡한 기능이며, Rust의 주요 장점 중 하나는 참조를 안전하고 쉽게 사용할 수 있는 방식입니다. 이 프로그램을 완료하기 위해서는 참조에 대한 많은 세부 사항을 알 필요는 없습니다. 지금 당장은 변수와 마찬가지로 참조도 기본적으로 불변임을 알아두세요. 따라서 `&mut guess` 와 같이 작성하여 변경 가능하게 해야 합니다. (제4장에서 참조에 대해 자세히 설명합니다.)

<!-- Old heading. Do not remove or links may break. -->
<a id=\"handling-potential-failure-with-the-result-type\"></a>

### `Result` 유형으로 발생 가능한 오류 처리

우리는 여전히 이 코드 줄을 작업 중입니다. 다음 줄의 텍스트를 다루고 있지만, 여전히 하나의 논리적인 코드 줄의 일부임을 알아두세요. 다음 부분은 이 메서드입니다:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:expect}}
```

우리는 이 코드를 다음과 같이 작성할 수도 있었습니다.

```rust,ignore
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

그러나 한 줄의 코드는 읽기 어렵기 때문에 줄바꿈과 기타  whitespaces를 추가하여 긴 코드 줄을 분리하는 것이 좋습니다. 이제 이 줄이 무엇을 하는지 살펴보겠습니다.

앞서 언급했듯이 `read_line` 은 사용자가 입력한 내용을 우리가 전달한 문자열에 넣지만, `Result` 값을 반환합니다. [`Result`][result]<!-- ignore --> 는 *열거형* [enums]<!-- ignore --> 이라고도 불리는 유형으로, 여러 가지 가능한 상태 중 하나를 나타내는 유형입니다. 각 가능한 상태를 *변이*라고 합니다.

[Chapter 6][enums]<!-- ignore --> 에서 열거형에 대해 자세히 다룹니다. 이러한 `Result` 유형의 목적은 오류 처리 정보를 암호화하는 것입니다.

`Result` 의 변이는 `Ok` 와 `Err` 입니다. `Ok` 변이는 작업이 성공했음을 나타내며, `Ok` 안에는 성공적으로 생성된 값이 있습니다. `Err` 변이는 작업이 실패했음을 의미하며, `Err` 는 작업이 실패한 이유에 대한 정보를 포함합니다.

`Result` 유형의 값은 다른 유형의 값과 마찬가지로 메서드가 정의되어 있습니다. `Result` 인스턴스에는 `expect` 메서드가 있습니다. 이 메서드를 호출할 수 있습니다. 만약 이 `Result` 인스턴스가 `Err` 값이라면 `expect` 는 프로그램을 종료시키고 `expect` 에 전달한 메시지를 표시합니다. 만약 `read_line` 메서드가 `Err` 를 반환하면, 그것은 아마도 운영 체제에서 발생한 오류의 결과일 것입니다. 만약 이 `Result` 인스턴스가 `Ok` 값이라면 `expect` 는 `Ok` 가 가지고 있는 반환 값을 가져와서 사용할 수 있도록 그 값만 반환합니다. 이 경우 그 값은 사용자의 입력에서의 바이트 수입니다.

`expect` 를 호출하지 않으면 프로그램은 컴파일되지만 경고 메시지를 받게 됩니다.

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-02-without-expect/output.txt}}
```

Rust는 `read_line` 에서 반환된 `Result` 값을 사용하지 않았다고 경고하며, 프로그램이 가능한 오류를 처리하지 않았음을 나타냅니다.

오류를 처리하는 코드를 실제로 작성하는 것이 올바른 방법이지만, 우리의 경우 문제가 발생하면 프로그램을 종료하고 싶으므로 `expect` 를 사용합니다. [Chapter 9][recover]<!-- ignore --> 에서 오류에서 벗어나는 방법을 배웁니다.

### `println!` 템플릿 변수를 사용하여 값 출력

닫는 중괄호 외에 코드에서 논의해야 할 또 다른 줄이 있습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print_guess}}
```

이 줄은 사용자 입력을 담은 문자열을 출력합니다. `{}` 쌍은 홀더: `{}`를 작은 게게 핀셋으로 생각해 보세요. 변수의 값을 출력할 때, 변수 이름을 중괄호 안에 넣을 수 있습니다. 표현식의 결과를 출력할 때는 형식 문자열에 빈 중괄호를 사용하고, 빈 중괄호 홀더에 각각 출력할 표현식을 쉼표로 구분하여 따릅니다.

변수와 표현식의 결과를 `println!` 한 번의 호출로 출력하는 방법은 다음과 같습니다.

```rust
let x = 5;
let y = 10;

println!(\"x = {x} and y + 2 = {}\", y + 2);
```

이 코드는 `x = 5 and y + 2 = 12`를 출력합니다.

### 첫 번째 부분 테스트

게임의 첫 번째 부분을 테스트해 보겠습니다. `cargo run`을 사용하여 실행합니다.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-01/
cargo clean
cargo run
input 6 -->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 6.44s
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

이제 게임의 첫 번째 부분이 완료되었습니다. 키보드에서 입력을 받고 출력합니다.

## 비밀 숫자 생성

다음으로 사용자가 추측할 비밀 숫자를 생성해야 합니다. 비밀 숫자는 매번 다르게 생성되어 게임을 여러 번 재생하는 것이 재미있어야 합니다. 게임이 너무 어렵지 않도록 1에서 100 사이의 임의의 숫자를 사용합니다. Rust은 아직 표준 라이브러리에 임의 숫자 기능을 포함하지 않았습니다. 그러나 Rust 팀은 `rand` crate라는 임의 숫자 기능을 제공하는 crate를 제공합니다.

### 추가 기능을 위한 crate 사용

crate는 Rust 소스 코드 파일의 모음입니다. 우리가 지금까지 구축해 온 프로젝트는 실행 가능한 파일인 *binary crate*입니다. `rand` crate는 *library crate*이며, 다른 프로그램에서 사용할 수 있는 코드를 포함하고 스스로 실행할 수 없습니다.

Cargo는 외부 crate를 관리하는 데 탁월합니다. `rand`를 사용하기 전에 *Cargo.toml* 파일을 수정하여 `rand` crate를 의존성으로 추가해야 합니다. 지금 `Cargo.toml` 파일을 열고 `[dependencies]` 섹션 아래에 다음 줄을 추가합니다. Cargo가 생성한 `[dependencies]` 섹션 헤더 아래에 있는지 확인하세요.

<!-- `rand` 버전을 업데이트할 때, 이 튜토리얼의 코드 예제가 작동하도록 이 파일에서 사용하는 `rand` 버전도 업데이트해야 합니다:
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
* ch14-03-cargo-workspaces.md
-->

<span class=\"filename\">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:8:}}
```

`Cargo.toml` 파일에서 헤더 다음에 오는 모든 것은 해당 섹션에 속하며, 다른 섹션이 시작될 때까지 계속됩니다. `[dependencies]`에서 Cargo는 프로젝트가 의존하는 외부 crate와 필요한 버전을 알립니다. 이 경우 `rand` crate를 버전 `0.8.5`로 지정합니다. Cargo는 [Semantic Versioning][semver]<!-- ignore --> (때로는 *SemVer*라고도 함)을 이해합니다. SemVer는 버전 번호를 작성하는 데 사용되는 표준입니다. `0.8.5`는 실제로 `^0.8.5`의 약자로, 0.8.5 이상이지만 0.9.0 미만인 모든 버전을 의미합니다.

Cargo는 이러한 버전이 0.8.5 버전과 공개 API가 호환되는 것으로 간주하며, 이러한 명세는 이 장의 코드와 함께 컴파일될 수 있는 최신 패치 리리스를 얻게 됩니다. 0.9.0 이상의 모든 버전은 다음 예제에서 사용하는 API와 동일하지 않을 수 있습니다.

이제 코드를 변경하지 않고 프로젝트를 빌드하는 방법을 살펴보겠습니다. 2-2번 목록에 나와 있습니다.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
rm Cargo.lock
cargo clean
cargo build -->

<Listing number="2-2" caption="`cargo build`를 실행한 후 `rand` crate를 의존성으로 추가했을 때의 출력">

```console
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
  Downloaded libc v0.2.127
  Downloaded getrandom v0.2.7
  Downloaded cfg-if v1.0.0
  Downloaded ppv-lite86 v0.2.16
  Downloaded rand_chacha v0.3.1
  Downloaded rand_core v0.6.3
   Compiling libc v0.2.127
   Compiling getrandom v0.2.7
   Compiling cfg-if v1.0.0
   Compiling ppv-lite86 v0.2.16
   Compiling rand_core v0.6.3
   Compiling rand_chacha v0.3.1
   Compiling rand v0.8.5
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53s
```

</Listing>

다른 버전 번호를 볼 수도 있지만 (모두 코드와 호환되므로 SemVer 덕분입니다!), 다른 줄이 있을 수도 있으며 (운영 체제에 따라 달라질 수 있음) 줄의 순서가 다를 수 있습니다.

외부 의존성을 포함할 때 Cargo는 해당 의존성이 필요로 하는 모든 최신 버전을 *레지스트리*에서 가져옵니다. 레지스트리는 [Crates.io][cratesio]의 데이터 복사본입니다. Crates.io는 Rust 생태계의 사람들이 다른 사람들이 사용할 수 있도록 오픈 소스 Rust 프로젝트를 게시하는 곳입니다.

Cargo는 레지스트리를 업데이트한 후 `[dependencies]` 섹션을 확인하고 아직 다운로드되지 않은 목록에 있는 crate를 다운로드합니다. 이 경우 `rand`만 의존성으로 나열했지만 Cargo는 `rand`가 작동하려면 필요한 다른 crate도 가져왔습니다. crate를 다운로드한 후 Rust는 이를 컴파일하고 의존성이 있는 프로젝트를 컴파일합니다.

`cargo build`를 다시 즉시 실행하면 코드 변경 사항이 없으므로 `Finished` 줄 외에 출력이 없습니다. Cargo는 이미 의존성을 다운로드하고 컴파일했으며, *Cargo.toml* 파일에서 의존성에 대한 변경 사항이 없다는 것을 알고 있습니다. Cargo는 코드에도 변경 사항이 없다는 것을 알고 있으므로 다시 컴파일하지 않습니다. 할 일이 없으므로 단순히 종료됩니다.

*src/main.rs* 파일을 열고 사소한 변경 사항을 한 후 저장하고 다시 빌드하면 다음과 같은 두 줄의 출력만 표시됩니다.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
touch src/main.rs
cargo build -->

```console
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53 secs
```

이 줄은 Cargo가 `src/main.rs` 파일의 작은 변경 사항만으로 빌드를 업데이트한다는 것을 보여줍니다. 의존성이 변경되지 않았으므로 Cargo는 이미 다운로드하고 컴파일한 내용을 다시 사용할 수 있습니다.

#### *Cargo.lock* 파일을 사용하여 재현 가능한 빌드를 보장하는 방법

Cargo는 코드를 다시 빌드할 때마다 동일한 결과물을 얻을 수 있도록 하는 메커니즘을 제공합니다. Cargo는 지정한 의존성의 버전만 사용하여 빌드합니다. 예를 들어, 다음 주에 `rand` crate의 0.8.6 버전이 출시되고, 이 버전에는 중요한 버그 수정 사항이 포함되어 있지만 코드를 끊는 문제가 발생하는 경우, Rust는 처음 `cargo build`를 실행할 때 *Cargo.lock* 파일을 생성합니다. 이제 *guessing_game* 디렉토리에 이 파일이 있습니다.


프로젝트를 처음 빌드할 때 Cargo는 조건에 맞는 모든 의존성의 버전을 파악하고
*Cargo.lock* 파일로 기록합니다. 앞으로 프로젝트를 빌드할 때 Cargo는
*Cargo.lock* 파일이 존재한다는 것을 알아차리고 버전을 다시 파악하는 작업 대신
이 파일에서 지정된 버전을 사용합니다. 이렇게 하면 재현 가능한 빌드를 자동으로
가능하게 합니다. 즉, *Cargo.lock* 파일 덕분에 프로젝트는 명시적으로 업그레이드할
때까지 0.8.5를 유지합니다.

*Cargo.lock* 파일은 재현 가능한 빌드에 중요하기 때문에 프로젝트의 나머지 코드와
src/main.rs와 함께 소스 제어 시스템에 체크인됩니다.

#### Crate 업데이트를 통한 새 버전 가져오기

Crate를 업데이트하고 싶을 때 Cargo는 `update` 명령을 제공하며, 이 명령은
*Cargo.lock* 파일을 무시하고 *Cargo.toml* 에 있는 모든 최신 버전을 파악합니다.
Cargo는 이러한 버전을 *Cargo.lock* 파일로 기록합니다. 이 경우 Cargo는
0.8.5보다 크고 0.9.0보다 작은 버전만 찾습니다. 만약 `rand` Crate가 0.8.6과
0.9.0 두 가지 새로운 버전을 출시했다면, `cargo update`를 실행하면 다음과 같은
출력이 표시됩니다.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
cargo update
assuming there is a new 0.8.x version of rand; otherwise use another update
as a guide to creating the hypothetical output shown here -->

```console
$ cargo update
    Updating crates.io index
    Updating rand v0.8.5 -> v0.8.6
```

Cargo는 0.9.0 출시를 무시합니다. 이제 *Cargo.lock* 파일에서 `rand` Crate의 버전이
0.8.6으로 변경되었음을 알 수 있습니다. `rand` 버전 0.9.0 또는 0.9.*x* 시리즈를 사용하려면
*Cargo.toml* 파일을 다음과 같이 업데이트해야 합니다.

```toml
[dependencies]
rand = \"0.9.0\"
```

다음에 `cargo build`를 실행하면 Cargo는 사용 가능한 Crate 레지스트리를 업데이트하고
새로 지정된 버전에 따라 `rand` 요구 사항을 다시 평가합니다.

Cargo와 [그 생태계][doccratesio]에 대해서는 14장에서 자세히 설명하지만, 지금은 이만
충분합니다. Cargo는 라이브러리를 재사용하는 것을 매우 쉽게 만들기 때문에
Rustaceans는 여러 패키지로 구성된 작은 프로젝트를 작성할 수 있습니다.

### 임의의 숫자 생성하기

`rand`를 사용하여 추측할 숫자를 생성하는 방법을 살펴보겠습니다. 다음 단계는
Listing 2-3과 같이 *src/main.rs* 파일을 업데이트하는 것입니다.

<Listing number=\"2-3\" file-name=\"src/main.rs\" caption=\"임의의 숫자를 생성하는 코드 추가\">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:all}}
```

</Listing>

먼저 `use rand::Rng;` 라인을 추가합니다. `Rng` 트레이트는 임의 숫자 생성기가 구현하는 메서드를 정의하며, 이 트레이트가 범위 내에 있어야만 해당 메서드를 사용할 수 있습니다. 10장에서 트레이트에 대해 자세히 설명합니다.

다음으로 중간에 두 줄을 추가합니다. 첫 번째 줄에서는 현재 실행 스레드에 고유한 임의 숫자 생성기를 제공하는 `rand::thread_rng` 함수를 호출합니다. 이 생성기는 운영 체제에서 씨드를 사용하여 초기화됩니다. 그런 다음 `gen_range` 메서드를 임의 숫자 생성기에 호출합니다. 이 메서드는 `Rng` 트레이트에서 정의되며, 범위 표현을 인수로 받고 범위 내의 임의 숫자를 생성합니다. 여기서 사용하는 범위 표현은 `start..=end` 형태이며, 하한과 상한을 포함하므로 1부터 100까지의 숫자를 요청하려면 `1..=100`를 사용해야 합니다.

> 주의: 어떤 트레이트를 사용해야 하고 어떤 메서드와 함수를 사용해야 하는지 즉시 알 수 없습니다.


crate를 호출하는 방법을 설명하는 문서가 있습니다. Cargo의 또 다른 멋진 기능은 `cargo doc --open` 명령을 실행하면 모든 의존성이 제공하는 문서가 로컬로 빌드되고 브라우저에서 열리는 것입니다. 예를 들어 `rand` crate의 다른 기능에 관심이 있다면 `cargo doc --open`을 실행하고 왼쪽 사이드바에서 `rand`를 클릭하십시오.

두 번째 줄은 비밀 번호를 출력합니다. 개발 중이므로 테스트할 수 있도록 유용하지만 최종 버전에서는 삭제해야 합니다. 프로그램이 시작하는 즉시 답을 출력하는 것은 게임이라고 할 수 없습니다.

프로그램을 몇 번 실행해 보세요.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-03/
cargo run
4
cargo run
5
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4

$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

다른 랜덤 숫자가 나오고, 모두 1부터 100까지의 숫자여야 합니다. 잘했어요!

## 추측과 비밀 숫자의 비교

이제 사용자 입력과 랜덤 숫자를 가지고 있으므로 비교할 수 있습니다. 해당 단계는 Listing 2-4에 나와 있습니다. 이 코드는 아직 컴파일되지 않을 것입니다.

<Listing number="2-4" file-name="src/main.rs" caption="두 숫자를 비교할 때 가능한 반환 값을 처리하는 방법">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-04/src/main.rs:here}}
```

</Listing>

먼저 `std::cmp::Ordering`이라는 유형을 가져오는 또 다른 `use` 문을 추가합니다. `Ordering` 유형은 또 다른 `enum`이며 `Less`, `Greater`, `Equal`이라는 변형을 가지고 있습니다. 이러한 변형은 두 값을 비교할 때 가능한 세 가지 결과입니다.

그런 다음 `Ordering` 유형을 사용하는 다섯 줄을 추가합니다. `cmp` 메서드는 두 값을 비교할 수 있으며 어떤 값이든 비교할 수 있습니다. `guess`와 `secret_number`를 비교하는 경우 `cmp` 메서드를 사용하여 두 값을 비교합니다. `Ordering` 유형의 변형을 반환합니다. `match` 표현식을 사용하여 `cmp` 메서드에서 반환된 `Ordering` 유형의 변형에 따라 다음을 수행합니다.

`match` 표현식은 *팔*로 구성됩니다. 팔은 `match`에 주어진 값이 일치하는지 확인하는 *패턴*과 일치하는 경우 실행할 코드를 포함합니다. Rust는 `match`에 주어진 값을 보고 각 팔의 패턴을 순서대로 확인합니다. 패턴과 `match` 구조는 강력한 Rust 기능입니다. 다양한 상황을 표현하고 처리할 수 있도록 합니다. 이러한 기능은 6장과 18장에서 자세히 설명됩니다.

`Ordering::Less` 패턴을 확인하고 `Ordering::Greater` 값이 `Ordering::Less`와 일치하지 않으므로 해당 패턴의 코드를 무시합니다. 다음 팔의 패턴을 확인합니다. `Ordering::Greater` 값이 `Ordering::Greater` 패턴과 일치하므로 해당 패턴의 코드를 실행합니다.

이제 `match` 표현식이 어떻게 작동하는지 이해하셨을 것입니다.


팔을 움직여 다음 팔로 이동합니다. 다음 팔의 패턴은
`Ordering::Greater`이며, 이는 `Ordering::Greater`와 일치합니다! 해당 팔의
관련 코드가 실행되고 화면에 "너무 크다!"가 출력됩니다. `match` 표현식은
첫 번째 성공적인 일치 후에 종료되므로 이 시나리오에서는 마지막 팔의
코드를 살펴보지 않습니다.

그러나 Listing 2-4의 코드는 아직 컴파일되지 않습니다. 해보도록 하겠습니다.

<!--
이 출력의 오류 번호는 **코드를 제외한** 앵커 또는 코드 스니펫 주석이 없는 코드의 번호여야 합니다
-->

```console
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-04/output.txt}}
```

오류의 핵심은 *타입 불일치*임을 나타냅니다. Rust는 강력하고 정적 타입 시스템을 가지고 있습니다. 그러나 동시에 타입 추론 기능도 있습니다. `let mut guess = String::new()`를 작성했을 때, Rust는 `guess`가 `String`이어야 한다는 것을 추론하여 `guess`의 타입을 명시하지 않도록 했습니다. 반면 `secret_number`는 숫자 타입입니다. Rust의 몇 가지 숫자 타입은 1과 100 사이의 값을 가질 수 있습니다: `i32` (32비트 숫자), `u32` (무符号 32비트 숫자), `i64` (64비트 숫자) 등. 그렇지 않으면 명시적으로 지정하지 않는 한 Rust는 기본적으로 `i32`를 사용하며, 이는 `secret_number`의 타입입니다. 오류의 이유는 Rust가 문자열과 숫자 타입을 비교할 수 없기 때문입니다.

결국, 프로그램이 입력으로 읽는 문자열을 숫자 타입으로 변환하여 비교해야 합니다. 이를 위해 `main` 함수 본문에 다음 줄을 추가합니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/src/main.rs:here}}
```

이 줄은 다음과 같습니다.

```rust,ignore
let guess: u32 = guess.trim().parse().expect(\"Please type a number!\");
```

`guess`라는 이름의 변수를 생성합니다. 하지만 프로그램에 이미 `guess`라는 변수가 있지 않나요? 그렇습니다. 하지만 Rust는 유용하게 이전 `guess` 변수의 값을 새 `guess` 변수로 가리키도록 허용합니다. *변수 오버라이딩*은 `guess`라는 변수 이름을 재사용할 수 있도록 해주는 기능입니다. 예를 들어 `guess_str`과 `guess`와 같이 두 개의 고유한 변수를 만들지 않고도 사용할 수 있습니다. 이 기능은 종종 값을 한 타입에서 다른 타입으로 변환할 때 사용됩니다.

`guess` 변수는 `guess.trim().parse()`라는 표현으로 바인딩됩니다. `guess`는 문자열로 입력된 값을 담고 있는 원래 `guess` 변수를 가리킵니다. `String` 인스턴스의 `trim` 메서드는 시작과 끝에 있는 모든 공백을 제거합니다. 이는 `u32`와 비교하기 위해 필수적입니다. 사용자는 <kbd>enter</kbd> 키를 누르면 `read_line`을 만족시키고 자신의 추측을 입력해야 하며, 이는 문자열에 줄 바꿈 문자를 추가합니다. 예를 들어 사용자가 <kbd>5</kbd>를 입력하고 <kbd>enter</kbd> 키를 누르면 `guess`는 다음과 같습니다: `5\
`. `\
`은 줄 바꿈을 나타냅니다. (Windows에서 <kbd>enter</kbd> 키를 누르면 캐리지 리턴과 줄 바꿈이 발생하여 `\\r\
`이 됩니다.) `trim` 메서드는 `\
` 또는 `\\r\
`을 제거하여 `5`만 남깁니다.

문자열의 [`parse` 메서드][parse]<!-- ignore -->는 문자열을 다른 타입으로 변환합니다. 여기서는 문자열을 숫자로 변환하기 위해 사용합니다. `let guess: u32`를 사용하여 Rust에 원하는 정확한 숫자 타입을 알려야 합니다. 콜론(`:`)은 `guess` 뒤에 붙어 Rust가 변수의 타입을 명시하도록 합니다. Rust에는 몇 가지 내장된 숫자 타입이 있으며, 여기서 보는 `u32`는 무符号 32비트 정수입니다. 작은 긍정적인 숫자에 대한 좋은 기본 선택입니다. 타입에 대해서는 [Chapter 3][shadowing]<!-- ignore -->에서 자세히 알아볼 수 있습니다. 하지만 지금은 이 기능이 값을 한 타입에서 다른 타입으로 변환할 때 자주 사용되는 것을 알아두세요.

다른 숫자 유형은 [제3장][정수]에서 자세히 설명되어 있습니다.<!-- 무시 -->

또한 이 예제 프로그램에서 `u32` 어노테이션과 `secret_number`와의 비교는 Rust가 `secret_number`도 `u32`가 되어야 한다고 추론하도록 합니다. 이제 비교는 같은 유형의 두 값 사이에서 이루어집니다! 

`parse` 메서드는 논리적으로 숫자로 변환할 수 있는 문자열만 작동하며, 쉽게 오류를 일으킬 수 있습니다. 예를 들어 문자열에 `A👍%`가 포함되어 있다면 숫자로 변환할 방법이 없습니다. 변환에 실패할 수 있기 때문에 `parse` 메서드는 `Result` 유형을 반환합니다. 이전에 언급된 것처럼 `read_line` 메서드도 `Result` 유형을 반환합니다. ([“`Result`를 사용하여 발생할 수 있는 오류 처리”](#handling-potential-failure-with-result)<!-- 무시 -->) 우리는 이 `Result`를 `expect` 메서드를 사용하여 동일하게 처리합니다. 만약 `parse`가 문자열에서 숫자를 만들 수 없어 `Err` `Result` 변형을 반환하면 `expect` 호출은 게임을 종료시키고 우리가 제공한 메시지를 출력합니다. `parse`가 문자열을 숫자로 성공적으로 변환하면 `Ok` 변형의 `Result`를 반환하고 `expect`는 `Ok` 값에서 원하는 숫자를 반환합니다.

이제 프로그램을 실행해 보겠습니다.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/
cargo run
  76
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

멋지네요! 공백이 입력값 앞에 추가되었더라도 프로그램은 사용자가 76을 추측했다는 것을 알아냈습니다. 프로그램을 몇 번 실행하여 올바르게 숫자를 추측하는 경우, 너무 높은 숫자를 추측하는 경우, 너무 낮은 숫자를 추측하는 경우의 다양한 동작을 확인하십시오.

지금 우리는 게임의 대부분을 작동시켰지만, 사용자는 한 번만 추측할 수 있습니다. 루프를 추가하여 사용자에게 더 많은 추측 기회를 주도록 변경해 보겠습니다.

## 루프를 사용하여 여러 번 추측 가능하게 하기

`loop` 키워드는 무한 루프를 만듭니다. 사용자에게 더 많은 추측 기회를 제공하기 위해 루프를 추가합니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-04-looping/src/main.rs:here}}
```

루프 안의 각 줄을 4개의 공백으로 들여쓰고 프로그램을 다시 실행하십시오. 이제 프로그램은 영원히 또 다른 추측을 요청할 것입니다. 이는 새로운 문제를 야기합니다. 사용자는 프로그램에서 빠져나올 수 없습니다!

사용자는 <kbd>ctrl</kbd>-<kbd>c</kbd> 키 조합을 사용하여 프로그램을 중단할 수는 있지만, 사용자가 프로그램에서 빠져나올 수 있는 또 다른 방법이 있습니다. 이전에 언급된 것처럼, 사용자가 숫자가 아닌 값을 입력하면 프로그램이 종료됩니다. 이를 사용하여 사용자가 프로그램에서 빠져나갈 수 있도록 할 수 있습니다.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-04-looping/
cargo run
(너무 작은 추측)
(너무 큰 추측)
(올바른 추측)
quit
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 1.50s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
59를 맞추셨습니다.
축하합니다!
예상치를 입력하세요.
quit
main 스레드에서 panic: '숫자를 입력하세요!: ParseIntError { kind: InvalidDigit }', src/main.rs:28:47
note: `RUST_BACKTRACE=1` 환경 변수로 실행하여 백트레이스를 표시하세요
```

quit를 입력하면 게임이 종료되지만, 다른 숫자가 아닌 입력을 하면도 게임이 종료됩니다. 이는 최소한의 문제점입니다. 게임이 정답을 맞췄을 때 종료하도록 만들고 싶습니다.

### 정답 맞춤 후 종료

정답을 맞췄을 때 사용자가 게임에서 이기도록 `break` 문을 추가하여 프로그램을 만들겠습니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-05-quitting/src/main.rs:here}}
```

`You win!` 다음에 `break` 문을 추가하면 사용자가 비밀 숫자를 맞출 때 프로그램이 루프를 종료합니다. 루프를 종료하는 것은 `main`의 마지막 부분이기 때문에 프로그램을 종료하는 것과 동일합니다.

### 잘못된 입력 처리

게임의 동작을 더욱 개선하기 위해 사용자가 숫자가 아닌 입력을 하면 프로그램이 충돌하는 대신, 게임이 숫자가 아닌 입력을 무시하도록 만들겠습니다. 이를 위해 `guess`가 `String`에서 `u32`로 변환되는 줄을 수정하면 됩니다. Listing 2-5에서 보여주는 것처럼

<Listing number=\"2-5\" file-name=\"src/main.rs\" caption=\"숫자가 아닌 입력을 무시하고, 오류 발생 시 다시 입력을 요청하는 프로그램\">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:here}}
```

</Listing>

`expect` 함수를 `match` 표현식으로 바꾸어 오류를 처리하는 방식으로 변경했습니다. `parse`는 `Result` 유형을 반환하며 `Result`는 `Ok`와 `Err` 두 가지 변형을 가진 열거형입니다. `match` 표현식을 사용하는 이유는 `cmp` 메서드의 `Ordering` 결과와 같습니다.

`parse`가 문자열을 숫자로 성공적으로 변환할 수 있다면 `Ok` 값을 반환하며, 이 `Ok` 값은 첫 번째 팔의 패턴에 맞습니다. `match` 표현식은 `Ok` 값에 포함된 숫자를 `num` 변수에 저장하고 그 값을 반환합니다. 이 숫자는 우리가 만드는 새로운 `guess` 변수에 들어갑니다.

`parse`가 문자열을 숫자로 변환할 수 없다면 `Err` 값을 반환하며, 이 `Err` 값은 첫 번째 `match` 팔의 `Ok(num)` 패턴에 맞지 않습니다. 그러나 `Err` 값은 `Err(_)` 패턴의 두 번째 팔에 맞습니다. 밑줄 `_`은 모든 `Err` 값을 나타내는 잡다한 값입니다. 이 예제에서는 모든 `Err` 값에 맞는 것을 의미합니다. 따라서 프로그램은 두 번째 팔의 코드인 `continue`를 실행하고, 루프의 다음 반복으로 이동하여 다시 입력을 요청합니다. 즉, 프로그램은 `parse`가 발생시킬 수 있는 모든 오류를 무시합니다!

이제 프로그램의 모든 부분이 예상대로 작동해야 합니다. 실행해 보겠습니다.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-05/
cargo run
(너무 작은 추측)
(너무 큰 추측)
foo
(정답 추측)
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 4.45s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 61
예상치를 입력하세요.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

멋지네요! 아주 작은 마지막 수정만 더하면 추측 게임을 완성할 수 있습니다. 이전에 프로그램이 비밀 숫자를 출력하는 것을 기억하시나요? 테스트에는 잘 작동했지만, 게임을 망쳐버립니다. 비밀 숫자를 출력하는 `println!`을 삭제해 보겠습니다. 2-6번 목록은 최종 코드를 보여줍니다.

<Listing number=\"2-6\" file-name=\"src/main.rs\" caption=\"완성된 추측 게임 코드\">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-06/src/main.rs}}
```

</Listing>

이제 추측 게임을 성공적으로 구축했습니다. 축하합니다!

## 요약

이 프로젝트는 여러 새로운 Rust 개념을 직접 경험하는 좋은 방법이었습니다: `let`, `match`, 함수, 외부 crate 사용 등. 다음 몇 개의 챕터에서는 이러한 개념을 자세히 알아보겠습니다. 3장에서는 변수, 데이터 유형, 함수와 같은 대부분의 프로그래밍 언어에서 사용되는 개념을 다룹니다. 4장에서는 소유권 개념을 설명합니다. 소유권은 Rust를 다른 언어와 구분하는 특징입니다. 5장에서는 구조체와 메서드 문법을 다룹니다. 6장에서는 enum이 어떻게 작동하는지 설명합니다.

[prelude]: ../std/prelude/index.html
[variables-and-mutability]: ch03-01-variables-and-mutability.html#variables-and-mutability
[comments]: ch03-04-comments.html
[string]: ../std/string/struct.String.html
[iostdin]: ../std/io/struct.Stdin.html
[read_line]: ../std/io/struct.Stdin.html#method.read_line
[result]: ../std/result/enum.Result.html
[enums]: ch06-00-enums.html
[expect]: ../std/result/enum.Result.html#method.expect
[recover]: ch09-02-recoverable-errors-with-result.html
[randcrate]: https://crates.io/crates/rand
[semver]: http://semver.org
[cratesio]: https://crates.io/
[doccargo]: https://doc.rust-lang.org/cargo/
[doccratesio]: https://doc.rust-lang.org/cargo/reference/publishing.html
[match]: ch06-02-match.html
[shadowing]: ch03-01-variables-and-mutability.html#shadowing
[parse]: ../std/primitive.str.html#method.parse
[integers]: ch03-02-data-types.html#integer-types
