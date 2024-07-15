## 명령줄 인수 받기

`cargo new`를 사용하여 항상처럼 새로운 프로젝트를 만들겠습니다. 우리 프로젝트를 `minigrep` 라고 이름 지어서 시스템에 이미 있는 `grep` 도구와 구분하겠습니다.

```console
$ cargo new minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

첫 번째 작업은 `minigrep`가 파일 경로와 검색할 문자열을 두 개의 명령줄 인수를 받도록 하는 것입니다. 즉, `cargo run`, 두 개의 하이픈으로 명령줄 인수가 우리 프로그램을 위해서인지 `cargo`를 위해서인지 나타내고, 검색할 문자열, 그리고 검색할 파일 경로를 입력하여 프로그램을 실행할 수 있어야 합니다.

```console
$ cargo run -- searchstring example-filename.txt
```

지금 당장 `cargo new`가 생성한 프로그램은 우리가 제공하는 인수를 처리할 수 없습니다. [crates.io](https://crates.io/)에 있는 몇몇 기존 라이브러리가 명령줄 인수를 받는 프로그램을 작성하는 데 도움을 줄 수 있지만, 이 개념을 배우는 중이므로 직접 구현해 보겠습니다.

### 인수 값 읽기

`minigrep`가 우리가 전달하는 명령줄 인수의 값을 읽도록 하려면 Rust의 표준 라이브러리에서 제공되는 `std::env::args` 함수를 사용해야 합니다. 이 함수는 `minigrep`에 전달된 명령줄 인수의 이터레이터를 반환합니다. 이터레이터에 대해서는 [Chapter 13][ch13]에서 자세히 다룰 것입니다. 지금은 이터레이터에 대해 두 가지만 알아야 합니다. 첫째, 이터레이터는 값의 시퀀스를 생성합니다. 둘째, 이터레이터의 `collect` 메서드를 사용하면 이터레이터가 생성하는 모든 요소를 포함하는 벡터와 같은 컬렉션으로 변환할 수 있습니다.

Listing 12-1의 코드는 `minigrep` 프로그램이 전달된 명령줄 인수를 읽고 벡터에 값을 저장하도록 합니다.

<Listing number=\"12-1\" file-name=\"src/main.rs\" caption=\"명령줄 인수를 벡터로 수집하고 출력\">

```rust
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-01/src/main.rs}}
```

</Listing>

먼저 `use` 문을 사용하여 `std::env` 모듈을 스코프에 가져와 `args` 함수를 사용할 수 있도록 합니다. `std::env::args` 함수가 여러 개의 모듈 안에 깊이 있게 삽입되어 있는 것을 알 수 있습니다. [Chapter 7][ch7-idiomatic-use]에서 설명했듯이, 원하는 함수가 여러 개의 모듈 안에 깊이 있게 삽입된 경우, 원하는 함수를 사용하기 위해 부모 모듈을 스코프에 가져오는 것이 일반적입니다. 이렇게 하면 `std::env` 모듈에서 다른 함수도 쉽게 사용할 수 있습니다. 또한 `args`를 현재 모듈에서 정의된 함수로 착각할 가능성이 있기 때문에, `use std::env::args`를 사용하는 것보다 명확합니다.

> ### `args` 함수와 유효하지 않은 Unicode
>
> `std::env::args`는 인수에 유효하지 않은 Unicode가 포함되어 있는 경우 panic합니다. 프로그램이 유효하지 않은 Unicode를 포함하는 인수를 받아야 하는 경우 `std::env::args_os`를 사용하세요. `args_os` 함수는 `OsString` 값을 생성하는 이터레이터를 반환합니다. 이 문단에서는 간단함을 위해 `std::env::args`를 사용했지만, `OsString` 값은 플랫폼마다 다르고 `String` 값보다 처리하기 복잡합니다.

`main` 함수의 첫 번째 줄에서 `env::args`를 호출하고 즉시 `collect`를 사용하여 이터레이터를 벡터로 변환합니다. `collect` 함수를 사용하여 다양한 종류의 컬렉션을 만들 수 있으므로, `args`의 유형을 명시적으로 지정하여 원하는 컬렉션 유형을 나타내야 합니다. Rust에서 유형을 명시적으로 지정해야 하는 경우는 매우 드물지만, `collect` 함수는 유형을 명시적으로 지정해야 하는 함수 중 하나입니다.

마지막으로 `debug` 매크로를 사용하여 벡터를 출력합니다. 아래는 인수를 전달하지 않고 두 개의 인수를 전달하여 코드를 실행하는 방법입니다.

```console
{{#include ../listings/ch12-an-io-project/listing-12-01/output.txt}}
```

```console
```

`\"target/debug/minigrep\"`라는 값이 벡터의 첫 번째 요소임을 주목하세요. 이것은 우리의 이진 파일의 이름과 일치합니다. 이것은 C에서 인수 목록의 동작과 일치하며, 프로그램이 실행될 때 사용된 이름으로 프로그램이 자신을 참조하도록 합니다. 프로그램 이름에 액세스하는 것은 종종 편리합니다. 프로그램 이름을 메시지에 출력하거나 프로그램이 호출된 명령줄 별칭에 따라 프로그램의 동작을 변경하려는 경우에 유용합니다. 그러나 이 장의 목적에 따라 우리는 이를 무시하고 필요한 두 개의 인수만 저장합니다.

### 인수 값을 변수에 저장하기

프로그램은 현재 명령줄 인수로 지정된 값에 액세스할 수 있습니다. 이제 두 개의 인수 값을 변수에 저장하여 프로그램의 나머지 부분에서 값을 사용할 수 있도록 해야 합니다. Listing 12-2에서 이 작업을 수행합니다.

<Listing number=\"12-2\" file-name=\"src/main.rs\" caption=\"쿼리 인수와 파일 경로 인수를 저장하는 변수 생성\">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-02/src/main.rs}}
```

</Listing>

우리가 인수 벡터를 출력했을 때와 같이 프로그램의 이름은 `args[0]`에서 벡터의 첫 번째 값을 차지하므로 인수는 인덱스 `1`부터 시작합니다. 프로그램이 받는 첫 번째 인수인 `minigrep`은 우리가 검색하는 문자열이므로 `query` 변수에 첫 번째 인수의 참조를 넣습니다. 두 번째 인수는 파일 경로이므로 `file_path` 변수에 두 번째 인수의 참조를 넣습니다.

이 변수의 값을 일시적으로 출력하여 코드가 예상대로 작동하는지 확인합니다. `test`와 `sample.txt` 인수로 이 프로그램을 다시 실행해 보겠습니다.

```console
{{#include ../listings/ch12-an-io-project/listing-12-02/output.txt}}
```

훌륭합니다! 필요한 인수 값이 올바른 변수에 저장되고 있습니다. 나중에 사용자가 인수를 제공하지 않은 경우와 같은 특정 오류 상황을 처리하는 오류 처리를 추가할 것입니다. 지금은 파일 읽기 기능을 추가하는 데 집중하겠습니다.

[ch13]: ch13-00-functional-features.html
[ch7-idiomatic-use]: ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#creating-idiomatic-use-paths
