## 슬라이스 자료형

*슬라이스*는 [컬렉션](ch08-00-common-collections.md) 내의 연속적인 요소들을 참조하는 데 사용됩니다. 슬라이스는 참조의 한 종류이므로 소유권을 가지지 않습니다.

다음은 작은 프로그래밍 문제입니다. 공백으로 구분된 단어로 이루어진 문자열을 입력받아 문자열에서 첫 번째 단어를 반환하는 함수를 작성하는 것입니다. 함수가 문자열에서 공백을 찾지 못하면 전체 문자열이 하나의 단어이므로 전체 문자열을 반환해야 합니다.

슬라이스를 사용하지 않고 이 함수의 서명을 작성하는 방법을 살펴보겠습니다.

```rust,ignore
fn first_word(s: &String) -> ?
```

`first_word` 함수는 `&String`을 매개변수로 받습니다. 소유권을 원하지 않으므로 이는 괜찮습니다. 하지만 무엇을 반환해야 할까요? 문자열의 *부분*에 대해 이야기하는 방법이 없기 때문입니다. 그러나 공백으로 나타나는 단어의 끝 인덱스를 반환할 수 있습니다. Listing 4-7와 같이 시도해 보겠습니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:here}}
```

<span class=\"caption\">Listing 4-7: `first_word` 함수가 `String` 매개변수 내의 바이트 인덱스를 반환하는 경우</span>

문자열을 요소별로 순회하여 값이 공백인지 확인해야 하므로 `String`을 `as_bytes` 메서드를 사용하여 바이트 배열로 변환합니다.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:as_bytes}}
```

다음으로 `iter` 메서드를 사용하여 바이트 배열에 대한 이터레이터를 생성합니다.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:iter}}
```

이터레이터에 대해서는 [Chapter 13][ch13]에서 자세히 설명합니다. 현재로서는 `iter`가 컬렉션의 각 요소를 반환하는 메서드이며 `enumerate`는 `iter`의 결과를 감싸서 각 요소를 튜플의 일부로 반환한다는 것을 알아두세요. 튜플의 첫 번째 요소는 인덱스이고 두 번째 요소는 요소에 대한 참조입니다. `enumerate`를 사용하면 인덱스를 직접 계산하는 것보다 더 편리합니다.

`enumerate` 메서드가 튜플을 반환하기 때문에 패턴을 사용하여 해당 튜플을 분해할 수 있습니다. 패턴에 대해서는 [Chapter 6][ch6]에서 자세히 설명합니다. `for` 루프에서 `i`를 인덱스, `&item`을 튜플의 단일 바이트로 나타내는 패턴을 지정합니다. `iter().enumerate()`에서 요소에 대한 참조를 얻기 때문에 패턴에서 `&`를 사용합니다.

`for` 루프 내에서 공백을 나타내는 바이트를 찾아서 반환합니다. 공백을 찾으면 해당 위치를 반환합니다. 그렇지 않으면 `s.len()`을 사용하여 문자열의 길이를 반환합니다.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:inside_for}}
```

이제 문자열에서 첫 번째 단어의 끝 인덱스를 찾는 방법이 있지만 문제가 있습니다. `usize`을 단독으로 반환하지만, `String`의 맥락에서만 의미 있는 값입니다. 즉, `String`과 별개의 값이기 때문에 미래에 유효한지 보장할 수 없습니다. Listing 4-8과 같은 프로그램을 살펴보겠습니다. Listing 4-8은 Listing 4-7의 `first_word` 함수를 사용합니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-08/src/main.rs:here}}
```

<span class=\"caption\">Listing 4-8: `first_word` 함수의 결과를 저장하는 경우</span>

```
 `first_word` 함수와 그 후 `String` 내용을 변경하는 것</span>

이 프로그램은 오류 없이 컴파일되며, `word`를 사용하더라도 `s.clear()`를 호출한 후에도 그렇습니다.
`word`가 `s`의 상태와 연결되어 있지 않기 때문에 `word`는 여전히 값 `5`를 포함합니다. 우리는 그 값 `5`를 변수 `s`와 함께 사용하여 첫 번째 단어를 추출하려고 할 수 있지만, 이것은 `word`에 저장된 `5`가 `s`의 내용이 변경된 이후의 값이기 때문에 버그가 될 것입니다.

`word`의 인덱스가 `s`의 데이터와 동기화되지 않도록 관리하는 것은 번거롭고 오류 발생 가능성이 높습니다! `second_word` 함수를 작성하면 이러한 인덱스 관리가 더욱 복잡해집니다. 그 함수의 서명은 다음과 같아야 합니다.

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

이제 우리는 시작 *및* 끝 인덱스를 추적하고 있으며, 특정 상태의 데이터에서 계산되었지만 해당 상태와 연결되지 않은 값이 더 많습니다.

다행히 Rust는 이 문제에 대한 해결책을 제공합니다: 문자열 슬라이스.

### 문자열 슬라이스

*문자열 슬라이스*는 `String`의 일부에 대한 참조이며 다음과 같습니다.

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-17-slice/src/main.rs:here}}
```

`hello`는 전체 `String`에 대한 참조가 아니라 `String`의 일부에 대한 참조이며, 추가 `[0..5]` 부분에서 지정됩니다. 슬라이스는 `[시작 인덱스..끝 인덱스]` 범위 내에서 생성하며, `시작 인덱스`는 슬라이스의 첫 번째 위치이고 `끝 인덱스`는 슬라이스의 마지막 위치보다 1 크다는 것을 의미합니다. 슬라이스 데이터 구조는 내부적으로 슬라이스의 시작 위치와 슬라이스 길이를 저장하며, 이는 `끝 인덱스`에서 `시작 인덱스`를 뺀 값과 일치합니다. 따라서 `let world = &s[6..11];`의 경우 `world`는 `s`의 6번째 바이트에 대한 포인터와 길이 값이 5인 슬라이스가 됩니다.

그림 4-6은 이를 다이어그램으로 보여줍니다.

<img alt=\"Three tables: a table representing the stack data of s, which points
to the byte at index 0 in a table of the string data &quot;hello world&quot; on
the heap. The third table rep-resents the stack data of the slice world, which
has a length value of 5 and points to byte 6 of the heap data table.\"
src=\"img/trpl04-06.svg\" class=\"center\" style=\"width: 50%;\" />

<span class=\"caption\">Figure 4-6: String slice referring to part of a
`String`</span>

Rust의 `..` 범위 문법을 사용하면 인덱스 0부터 시작하려면 두 점 앞의 값을 생략할 수 있습니다. 즉, 다음은 같습니다.

```rust
let s = String::from(\"hello\");

let slice = &s[0..2];
let slice = &s[..2];
```

마찬가지로 슬라이스가 `String`의 마지막 바이트를 포함하는 경우, 숫자를 생략할 수 있습니다. 즉, 다음은 같습니다.

```rust
let s = String::from(\"hello\");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

두 값을 모두 생략하여 `String` 전체의 슬라이스를 가져올 수도 있습니다. 즉, 다음은 같습니다.

```rust
let s = String::from(\"hello\");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> 주의: 문자열 슬라이스 범위 인덱스는 유효한 UTF-8 문자 경계에 있어야 합니다. 중간 문자에 슬라이스를 만드려고 하면 프로그램이 오류를 출력하여 종료됩니다. 이 섹션에서는 문자열 슬라이스를 소개하기 위해 ASCII만 가정하고 있으며, 문자열 슬라이스의 자세한 내용은 제8장의 [\u201cUTF-8 인코딩된 텍스트를 문자열로 저장하기\u201d][strings]<!-- ignore --> 섹션에서 다룹니다.

이 모든 정보를 바탕으로 `first_word`를 재작성하여 슬라이스를 반환하는 함수로 변경할 수 있습니다.

슬라이스. 문자열 슬라이스를 나타내는 유형은 `&str`로 작성됩니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-18-first-word-slice/src/main.rs:here}}
```

우리는 Listing 4-7과 동일한 방식으로 단어의 끝 인덱스를 가져옵니다. 즉, 공백의 첫 번째 발생을 찾습니다. 공백을 찾으면 문자열의 시작과 공백의 인덱스를 시작 및 끝 인덱스로 사용하여 문자열 슬라이스를 반환합니다.

이제 `first_word`를 호출하면, 기본 데이터에 연결된 단일 값을 반환합니다. 이 값은 슬라이스의 시작 지점에 대한 참조와 슬라이스의 요소 수로 구성됩니다.

`second_word` 함수에도 슬라이스를 반환하는 것이 적합합니다.

```rust,ignore
fn second_word(s: &String) -> &str {
```

이제 훨씬 쉽게 사용할 수 있는 직관적인 API를 가지게 되었습니다. 컴파일러가 `String` 내부 참조의 유효성을 보장하기 때문에 오류를 범할 가능성이 훨씬 줄어듭니다. Listing 4-8의 프로그램에서 `first_word` 함수의 오류를 기억하세요? 첫 번째 단어의 끝 인덱스를 가져왔지만, 문자열을 비우면 인덱스가 무효가 되는 문제였습니다. 그 코드는 논리적으로 잘못되었지만 즉각적인 오류를 보이지 않았습니다. 문제는 문자열이 비워진 후에도 첫 번째 단어 인덱스를 계속 사용하려고 할 때 발생했습니다. 슬라이스를 사용하면 이러한 오류가 불가능해지고 코드에서 문제가 발생하는 것을 훨씬 빨리 알 수 있습니다. 슬라이스 버전의 `first_word`를 사용하면 컴파일 시간 오류가 발생합니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/src/main.rs:here}}
```

다음은 컴파일러 오류입니다.

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/output.txt}}
```

참고로, 불변 참조를 가진 경우 변수 참조를 가져올 수 없습니다. `clear`는 `String`을 잘라내기 위해 변수 참조가 필요합니다. `clear` 호출 후 `println!`은 `word`의 참조를 사용하기 때문에 불변 참조가 여전히 유효해야 합니다. Rust는 `clear`에서 변수 참조와 `word`에서 불변 참조가 동시에 존재하는 것을 허용하지 않으며, 컴파일이 실패합니다. Rust는 API를 사용하기 쉽게 만들었을 뿐만 아니라 컴파일 시간에 오류를 제거하는 데에도 도움이 되었습니다!

<!-- Old heading. Do not remove or links may break. -->
<a id=\"string-literals-are-slices\"></a>

#### 문자열 리터럴은 슬라이스입니다

문자열 리터럴이 바이너리 내부에 저장된다는 것을 기억하십시오. 이제 슬라이스에 대해 알고 있다면 문자열 리터럴을 제대로 이해할 수 있습니다.

```rust
let s = \"Hello, world!\";
```

여기서 `s`의 유형은 `&str`입니다. 즉, 바이너리의 특정 위치를 가리키는 슬라이스입니다. 문자열 리터럴이 불변이라는 점도 이유입니다. `&str`은 불변 참조입니다.

#### 문자열 슬라이스를 매개변수로 사용하기

리터럴과 `String` 값의 슬라이스를 가져올 수 있다는 것을 알면 `first_word` 함수를 개선할 수 있습니다. 즉, 함수의 형식을 다음과 같이 변경할 수 있습니다.

```rust,ignore
fn first_word(s: &String) -> &str {
```

경험이 많은 Rust 개발자는 Listing 4-9에 표시된 형식을 사용하여 함수를 작성할 것입니다. 이렇게 하면 `&String` 값과 `&str` 값 모두에 대해 동일한 함수를 사용할 수 있습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:here}}
```

<span class=\"caption\">Listing 4-9: `first_word` 함수를 개선하여 `s` 매개변수 유형에 문자열 슬라이스를 사용</span>

문자열 슬라이스가 있다면 직접 전달할 수 있습니다. `String`이 있다면 `String`의 슬라이스 또는 `String` 자체에 대한 참조를 전달할 수 있습니다. 이 유연성은 다음 장에서 다룰 예정인 *묵시적 Deref Coercions* 기능을 활용합니다.
## 슬라이스

슬라이스는 배열이나 문자열의 일부를 참조하는 데 사용되는 Rust 자료구조입니다. 슬라이스는 배열이나 문자열의 시작 인덱스와 길이를 저장하여 해당 부분에 대한 참조를 제공합니다.

### 문자열 슬라이스

문자열 슬라이스는 문자열의 일부를 참조하는 데 사용됩니다. 예를 들어, 다음과 같은 문자열이 있다고 가정해 보겠습니다.

```rust
let s = "Hello, world!";
```

이 문자열의 "Hello" 부분을 참조하려면 다음과 같이 문자열 슬라이스를 사용할 수 있습니다.

```rust
let s = "Hello, world!";
let hello = &s[..5];

println!("{}", hello); // 출력: Hello
```

이 코드에서 `&s[..5]`는 문자열 `s`의 처음부터 5번째 인덱스까지의 부분을 참조하는 슬라이스입니다. `hello` 변수는 이 슬라이스를 저장합니다.

### 슬라이스의 유용성

문자열 슬라이스는 문자열을 다룰 때 유용합니다. 예를 들어, 함수를 정의하여 문자열 슬라이스를 입력으로 받으면, 해당 함수는 문자열의 특정 부분을 처리할 수 있습니다.

```rust
fn greet(name: &[&str]) {
    for name in name {
        println!("Hello, {}!", name);
    }
}

let names = ["Alice