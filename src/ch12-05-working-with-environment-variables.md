## 환경 변수 사용하기

`minigrep`를 개선하여 대소문자를 구분하지 않는 검색 옵션을 추가합니다. 사용자가 이 옵션을 환경 변수를 통해 켜고 끌 수 있도록 합니다. 이 기능을 명령줄 옵션으로 만들고 사용자가 매번 입력해야 하는 대신 환경 변수를 한 번 설정하면 해당 터미널 세션에서 모든 검색이 대소문자를 구분하지 않도록 할 수 있습니다.

### 대소문자 구분하지 않는 `search` 함수를 위한 실패 테스트 작성

먼저 환경 변수가 값을 가질 때 호출될 새로운 `search_case_insensitive` 함수를 추가합니다. TDD 프로세스를 계속 따라가므로 첫 번째 단계는 다시 실패하는 테스트를 작성하는 것입니다. `search_case_insensitive` 함수에 대한 새로운 테스트를 추가하고 `one_result`라는 기존 테스트를 `case_sensitive`로 이름을 바꿔 두 테스트의 차이를 명확히 합니다. Listing 12-20과 같이 보여줍니다.

<Listing number=\"12-20\" file-name=\"src/lib.rs\" caption=\"추가하려는 대소문자를 구분하지 않는 함수에 대한 새로운 실패 테스트 추가\">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-20/src/lib.rs:here}}
```

</Listing>

기존 테스트의 `contents`도 수정했습니다. `\"Duct tape.\"`라는 새로운 줄을 대문자 D를 사용하여 추가했습니다. 이는 대소문자를 구분하는 검색 방식으로는 `\"duct\"` 쿼리에 일치하지 않아야 합니다. 기존 테스트를 이렇게 변경하면 대소문자를 구분하는 검색 기능이 이미 구현되었기 때문에 실수로 손상되지 않도록 합니다. 이 테스트는 현재 통과해야 하며 대소문자를 구분하지 않는 검색을 구현하는 동안에도 계속 통과해야 합니다.

대소문자를 구분하지 않는 검색을 위한 새로운 테스트는 `\"rUsT\"`를 쿼리로 사용합니다. `search_case_insensitive` 함수에서 `\"rUsT\"` 쿼리는 대문자 R를 가진 `\"Rust:\"` 라는 줄과 대문자 T를 가진 `\"Trust me.\"` 라는 줄에 일치해야 합니다. 이것이 우리의 실패 테스트이며, `search_case_insensitive` 함수를 아직 정의하지 않았기 때문에 컴파일되지 않습니다. Listing 12-16에서 `search` 함수를 구현한 방식과 유사하게 항상 빈 벡터를 반환하는 스켈레톤 구현을 추가하면 테스트가 컴파일되고 실패하는 것을 볼 수 있습니다.

### `search_case_insensitive` 함수 구현

Listing 12-21에 나와 있는 `search_case_insensitive` 함수는 `search` 함수와 거의 동일합니다. 유일한 차이점은 쿼리와 각 줄을 소문자로 변환하기 때문에 입력 인수의 대소문자와 상관없이 줄이 쿼리에 포함되어 있는지 확인할 때 동일한 대소문자를 사용한다는 것입니다.

<Listing number=\"12-21\" file-name=\"src/lib.rs\" caption=\"대소문자를 구분하지 않는 `search_case_insensitive` 함수를 정의하여 쿼리와 줄을 소문자로 변환하기 전에 비교합니다\">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-21/src/lib.rs:here}}
```

</Listing>

먼저 `query` 문자열을 소문자로 변환하고 동일한 이름의 새 변수에 저장합니다. `to_lowercase`를 쿼리에 호출하는 것은 사용자의 쿼리가 `\"rust\"`, `\"RUST\"`, `\"Rust\"` 또는 `\"rUsT\"`인 경우에 상관없이 쿼리를 `\"rust\"`로 처리하기 위해 필요합니다. `to_lowercase`는 기본적인 Unicode를 처리하지만 100% 정확하지는 않습니다. 실제 응용 프로그램을 작성하는 경우 여기에서 조금 더 많은 작업을 해야 할 것입니다. 그러나 이 섹션은 환경 변수에 대한 것이므로 Unicode가 아니라 이 부분에 집중할 것입니다.

참고로 `query`는 문자열 슬라이스가 아닌 `String`입니다. `to_lowercase`를 호출하면 기존 데이터를 참조하는 것이 아니라 새로운 데이터를 생성하기 때문입니다. 예를 들어 쿼리가 `\"rUsT\"` 라고 가정하면 문자열 슬라이스에는 소문자 `u` 또는 `t`가 없기 때문에 소문자 `u` 또는 `t`를 포함하는 새로운 `String`을 할당해야 합니다. `\"rust\"`. `query`를 `contains` 메서드에 인자로 전달할 때, 이제 `&`를 추가해야 합니다. `contains`의 서명은 문자열 슬라이스를 가져오도록 정의되어 있기 때문입니다.

다음으로, 각 `line`에 `to_lowercase`를 호출하여 모든 문자를 소문자로 변환합니다. 이제 `line`과 `query`를 소문자로 변환했으므로, 쿼리의 경우에 관계없이 일치하는 항목을 찾을 수 있습니다.

`search_case_insensitive` 함수가 테스트를 통과하는지 확인해 보겠습니다.

```console
{{#include ../listings/ch12-an-io-project/listing-12-21/output.txt}}
```

훌륭합니다! 테스트가 통과했습니다. 이제 `run` 함수에서 새 `search_case_insensitive` 함수를 호출해 보겠습니다. 먼저 `Config` 구조체에 사례에 민감하거나 사례에 민감하지 않은 검색을 전환하는 구성 옵션을 추가합니다. 이 필드를 추가하면 `ignore_case` 필드를 초기화하지 않았기 때문에 컴파일 오류가 발생합니다.

Filename: src/lib.rs

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/lib.rs:here}}
```

`ignore_case` 필드를 추가했습니다. 이 필드는 불리언 값을 저장합니다. 다음으로 `run` 함수가 `ignore_case` 필드의 값을 확인하고 `search` 함수 또는 `search_case_insensitive` 함수를 호출하는지 여부를 결정해야 합니다. Listing 12-22와 같이 구현했습니다. 이것도 아직 컴파일되지 않습니다.

<Listing number=\"12-22\" file-name=\"src/lib.rs\" caption=\"`config.ignore_case` 값에 따라 `search` 또는 `search_case_insensitive`를 호출합니다\">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/lib.rs:there}}
```

</Listing>

마지막으로 환경 변수를 확인해야 합니다. 환경 변수를 처리하는 함수는 표준 라이브러리의 `env` 모듈에 있으므로 `src/lib.rs`의 맨 위에 해당 모듈을 스코프에 가져옵니다. 다음으로 `IGNORE_CASE`라는 이름의 환경 변수에 대해 값이 설정되었는지 확인하는 `env` 모듈의 `var` 함수를 사용합니다. Listing 12-23과 같이 구현했습니다.

<Listing number=\"12-23\" file-name=\"src/lib.rs\" caption=\"`IGNORE_CASE`라는 이름의 환경 변수에 대해 값이 있는지 확인합니다\">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-23/src/lib.rs:here}}
```

</Listing>

여기서 `ignore_case`라는 새로운 변수를 생성합니다. 이 변수의 값을 설정하려면 `env::var` 함수를 호출하고 `IGNORE_CASE` 환경 변수의 이름을 전달합니다. `env::var` 함수는 환경 변수가 설정되어 있으면 성공적인 `Ok` 변형을 반환하고, 그 값을 포함합니다. 환경 변수가 설정되지 않았으면 `Err` 변형을 반환합니다.

`Result`에서 환경 변수가 설정되었는지 확인하려면 `is_ok` 메서드를 사용합니다. 이는 프로그램이 사례에 민감하지 않은 검색을 수행해야 함을 의미합니다. `IGNORE_CASE` 환경 변수가 아무것도 설정되지 않았다면 `is_ok`은 `false`를 반환하고 프로그램은 사례에 민감한 검색을 수행합니다. 환경 변수의 *값*에 관심이 없으므로 `unwrap`, `expect` 또는 `Result`에서 본 다른 메서드 대신 `is_ok`를 사용합니다.

`ignore_case` 변수의 값을 `Config` 인스턴스에 전달하여 `run` 함수가 해당 값을 읽고 `search_case_insensitive` 또는 `search`를 호출할 수 있도록 합니다.

이제 시도해 보겠습니다! 먼저 환경 변수를 설정하지 않고 프로그램을 실행하고 `to`라는 쿼리를 사용하여 `to`라는 단어가 소문자로 포함된 모든 줄을 찾습니다.

```console
"


``````

여전히 작동하는 것처럼 보입니다! 이제 `IGNORE_CASE`를 `1`로 설정하지만 동일한 쿼리 `to`를 사용하여 프로그램을 실행해 보겠습니다.

```console
$ IGNORE_CASE=1 cargo run -- to poem.txt
```

PowerShell을 사용하는 경우, 환경 변수를 설정하고 프로그램을 별도의 명령으로 실행해야 합니다.

```console
PS> $Env:IGNORE_CASE=1; cargo run -- to poem.txt
```

이렇게 하면 `IGNORE_CASE`가 셸 세션의 나머지 동안 유지됩니다. `Remove-Item` cmdlet를 사용하여 해제할 수 있습니다.

```console
PS> Remove-Item Env:IGNORE_CASE
```

대소문자를 구분하지 않고 `to`를 포함하는 줄을 가져와야 합니다.

<!-- manual-regeneration
cd listings/ch12-an-io-project/listing-12-23
IGNORE_CASE=1 cargo run -- to poem.txt
can't extract because of the environment variable
-->

```console
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

훌륭합니다! `To`를 포함하는 줄도 가져왔습니다! `minigrep` 프로그램은 이제 환경 변수로 제어되는 대소문자 구분 없이 검색할 수 있습니다. 이제 명령줄 인수 또는 환경 변수를 사용하여 옵션을 설정하는 방법을 알고 있습니다. 일부 프로그램은 동일한 구성에 대해 *인수*와 *환경 변수*를 모두 허용합니다. 그러한 경우 프로그램은 하나 또는 다른 하나가 우선순위를 가지도록 결정합니다. 혼자서 해보는 다른 연습으로, 명령줄 인수 또는 환경 변수를 통해 대소문자 민감도를 제어해 보세요. 프로그램이 대소문자를 구분하는 것으로 설정되고 다른 하나는 대소문자를 무시하는 것으로 설정되어 실행될 때, 명령줄 인수 또는 환경 변수가 우선순위를 가지도록 결정하세요.

`std::env` 모듈에는 환경 변수를 처리하는 데 사용할 수 있는 많은 유용한 기능이 있습니다. 사용 가능한 기능을 확인하려면 설명서를 참조하세요.
