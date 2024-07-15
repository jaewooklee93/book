## 라이브러리 기능 개발: 테스트 기반 개발

이제 논리를 *src/lib.rs* 로 추출하고 *src/main.rs* 에서 인자 수집 및 오류 처리를 남겼으므로 코드의 핵심 기능을 위한 테스트를 작성하는 것이 훨씬 쉬워졌습니다. 명령줄에서 바이너리를 호출하지 않고 다양한 인자로 함수를 직접 호출하고 반환 값을 확인할 수 있습니다.

이 섹션에서는 테스트 기반 개발(TDD) 프로세스를 사용하여 `minigrep` 프로그램에 검색 논리를 추가합니다. 다음 단계를 따릅니다.

1. 실패하는 테스트를 작성하고 실행하여 예상하는 이유로 실패하는지 확인합니다.
2. 새 테스트를 통과하도록 충분히 코드를 작성하거나 수정합니다.
3. 작성하거나 변경한 코드를 리팩터링하여 테스트가 계속 통과하는지 확인합니다.
4. 1단계부터 반복합니다!

TDD는 여러 가지 방법 중 하나이지만, 코드 설계를 촉진하는 데 도움이 될 수 있습니다. 테스트를 코드 작성 전에 작성하면 프로세스 전체에서 높은 테스트 커버리지를 유지하는 데 도움이 됩니다.

실제로 검색 문자열을 파일 내용에서 검색하고 일치하는 줄 목록을 생성하는 기능을 테스트 드라이브합니다. 이 기능을 `search` 함수로 구현합니다.

### 실패하는 테스트 작성

*src/lib.rs* 와 *src/main.rs* 에서 프로그램의 동작을 확인하기 위해 사용했던 `println!` 문을 제거합니다. 그런 다음 *src/lib.rs* 에 [Chapter 11][ch11-anatomy]<!-- ignore -->에서와 같이 테스트 모듈과 테스트 함수를 추가합니다. 테스트 함수는 `search` 함수가 어떤 동작을 해야 하는지 명시합니다. 즉, 검색 문자열과 검색할 텍스트를 받아 텍스트에서 검색 문자열이 포함된 줄만 반환해야 합니다. Listing 12-15는 이 테스트를 보여주며, 아직 컴파일되지 않습니다.

<Listing number=\"12-15\" file-name=\"src/lib.rs\" caption=\"`search` 함수를 테스트하려고 하는데 아직 존재하지 않는 테스트\">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-15/src/lib.rs:here}}
```

</Listing>

이 테스트는 `\"duct\"` 문자열을 검색합니다. 검색하는 텍스트는 세 줄이며, 그 중 하나만에 `\"duct\"`가 포함되어 있습니다. (참고로, 열린 따옴표 뒤에 백슬래시가 있는 것은 Rust가 문자열 리터럴의 시작 부분에 줄 바꿈 문자를 삽입하지 않도록 하는 것입니다.) 우리는 `search` 함수에서 반환된 값이 우리가 기대하는 줄만 포함한다고 주장합니다.

우리는 아직 테스트를 실행하고 실패하는 것을 볼 수 없습니다. 테스트는 컴파일되지 않습니다. `search` 함수가 아직 존재하지 않기 때문입니다! TDD 원칙에 따라 `search` 함수가 컴파일되고 실행되도록 충분한 코드를 추가합니다. Listing 12-16에 나와 있는 것처럼 항상 빈 벡터를 반환하는 `search` 함수의 정의를 추가합니다. 그러면 테스트가 컴파일되고 `\"safe,
fast, productive.\" ` 줄을 포함하는 벡터가 아닌 빈 벡터를 반환하기 때문에 실패합니다.

<Listing number=\"12-16\" file-name=\"src/lib.rs\" caption=\"`search` 함수를 정의하여 테스트가 컴파일되도록 합니다\">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-16/src/lib.rs:here}}
```

</Listing>

`search` 함수의 시그니처에서 명시적으로 `'a`라는 라이프타임을 정의하고 `contents` 인자와 반환 값에 해당 라이프타임을 사용하는 것을 알 수 있습니다. [Chapter 10][ch10-lifetimes]<!-- ignore -->에서 기억하시면 라이프타임 매개변수는 어떤 인자 라이프타임이 반환 값의 라이프타임에 연결되어 있는지 지정합니다. 이 경우, `search` 함수에서 반환되는 벡터가 `contents` 인자에 전달된 데이터와 동일한 기간 동안 유효해야 함을 나타냅니다.

즉, `search` 함수에서 반환되는 데이터가 `search` 함수에 `contents` 인자로 전달된 데이터와 동일한 기간 동안 유효하다는 것을 Rust에 알려줍니다. 슬라이스에 의해 참조되는 데이터가 참조가 유효하도록 해야 하기 때문입니다. 만약 컴파일러가 우리가 `search` 함수에서 반환하는 슬라이스가 `contents` 인자의 데이터보다 오래 유효하다고 가정한다면, 오류가 발생할 것입니다.

Rust는 인수와 반환 값을 연결하는 데 있어서 명확한 규칙을 가지고 있습니다. 이러한 규칙은 라이프타임을 사용하여 정의됩니다. 라이프타임은 변수가 유효한 동안의 기간을 나타냅니다.

예를 들어, `search` 함수는 `contents`라는 문자열을 인수로 받고, `contents` 안에 있는 문자열 부분을 반환합니다. 이 경우, `contents`의 라이프타임은 `search` 함수가 실행되는 동안 유효해야 합니다.

하지만 `search` 함수에서 `query`라는 문자열 슬라이스를 사용하는 경우, `contents`가 아닌 `query`의 라이프타임을 사용해야 합니다. 이렇게 하지 않으면 Rust는 어떤 인수를 사용해야 하는지 알 수 없기 때문에 오류가 발생합니다.

다음은 `search` 함수의 예시입니다.

```rust
fn search(contents: &str, query: &str) -> Vec<&str> {
    let mut results = Vec::new();
    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }
    results
}
```

이 함수는 `contents` 문자열을 줄 단위로 반복하고, 각 줄이 `query` 문자열을 포함하는지 확인합니다. 포함된다면, 해당 줄을 `results` 벡터에 추가합니다. 마지막으로, `results` 벡터를 반환합니다.

이 함수를 테스트하려면 다음과 같은 테스트 코드를 사용할 수 있습니다.

```rust
#[test]
fn test_search() {
    let contents = "This is a test string.
This is another test string.";
    let query = "test";
    let results = search(contents, query);
    assert_eq!(results.len(), 2);
}
```

이 테스트 코드는 `search` 함수를 호출하고, 반환된 결과가 `2`개인지 확인합니다.

이 예시를 통해 Rust에서 인수와 반환 값을 연결하는 방법을 이해할 수 있습니다. 라이프타임을 사용하여 변수의 유효성을 명확하게 정의함으로써, Rust는 코드의 안전성을 보장합니다.


이제 `search` 함수는 `query`를 포함하는 줄만 반환해야 하며, 테스트가 통과해야 합니다. 테스트를 실행해 보겠습니다.

```console
{{#include ../listings/ch12-an-io-project/listing-12-19/output.txt}}
```

테스트가 통과했으므로 작동한다는 것을 알 수 있습니다.

이제 `search` 함수의 구현을 재작성하면서 테스트가 통과하도록 유지하여 동일한 기능을 유지할 수 있는 기회를 고려할 수 있습니다. `search` 함수의 코드는 그렇게 나쁘지 않지만, 이터레이터의 유용한 기능을 활용하지는 않습니다. 이 예제는 [Chapter 13][ch13-iterators]<!-- ignore -->에서 자세히 살펴보고 개선하는 방법을 알아보겠습니다.

#### `run` 함수에서 `search` 함수 사용

이제 `search` 함수가 작동하고 테스트가 완료되었으므로 `run` 함수에서 `search`를 호출해야 합니다. `run` 함수는 `config.query` 값과 `run`이 파일에서 읽는 `contents`를 `search` 함수에 전달해야 합니다. 그러면 `run`은 `search`에서 반환된 각 줄을 출력합니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/src/lib.rs:here}}
```

여전히 `search`에서 각 줄을 반환하고 출력하기 위해 `for` 루프를 사용하고 있습니다.

이제 전체 프로그램이 작동해야 합니다. \u201cfrog\u201d라는 단어로 시작하여 Emily Dickinson 시에서 정확히 한 줄만 반환해야 하는 단어로 시도해 보겠습니다.

```console
{{#include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/output.txt}}
```

멋지네요! 이제 \u201cbody\u201d와 같은 여러 줄에 일치하는 단어로 시도해 보겠습니다.

```console
{{#include ../listings/ch12-an-io-project/output-only-03-multiple-matches/output.txt}}
```

마지막으로, 시에 없는 단어인 \u201cmonomorphization\u201d과 같은 단어를 검색하면 줄이 없도록 확인해 보겠습니다.

```console
{{#include ../listings/ch12-an-io-project/output-only-04-no-matches/output.txt}}
```

훌륭합니다! 우리는 고전 도구의 미니 버전을 만들고 많은 것을 배우었습니다. 파일 입력 및 출력, 라이프타임, 테스트 및 명령줄 분석에 대해서도 약간 배웠습니다.

이 프로젝트를 마무리하기 위해 환경 변수와 표준 오류로 출력하는 방법을 간단히 보여드리겠습니다. 이 두 가지는 명령줄 프로그램을 작성할 때 유용합니다.

[validating-references-with-lifetimes]:
ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[ch11-anatomy]: ch11-01-writing-tests.html#the-anatomy-of-a-test-function
[ch10-lifetimes]: ch10-03-lifetime-syntax.html
[ch3-iter]: ch03-05-control-flow.html#looping-through-a-collection-with-for
[ch13-iterators]: ch13-02-iterators.html
