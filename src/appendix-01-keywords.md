## 부록 A: 키워드

다음 목록은 현재 또는 미래에 Rust 언어에서 사용될 예정인 키워드를 포함합니다. 따라서 이들은 식별자로 사용할 수 없습니다(단, [원시 식별자](raw-identifiers)로서 언급된 것처럼 원시 식별자로 사용할 수 있습니다).
식별자는 함수, 변수, 매개변수, 구조체 필드, 모듈, crate, 상수, 매크로, 정적 값, 속성, 유형, 트레이트 또는 라이프타임의 이름입니다.

[raw-identifiers]: #raw-identifiers

### 현재 사용 중인 키워드

다음은 현재 사용 중인 키워드 목록이며, 각 키워드의 기능이 설명되어 있습니다.

* `as` - 기본형 변환 수행, 특정 트레이트를 포함하는 항목을 구분하거나 `use` 문에서 항목을 다른 이름으로 바꾸기
* `async` - 현재 스레드를 차단하지 않고 `Future`를 반환하기
* `await` - `Future`의 결과가 준비될 때까지 실행을 일시 중지하기
* `break` - 루프를 즉시 종료하기
* `const` - 상수 항목 또는 상수 원시 포인터를 정의하기
* `continue` - 루프의 다음 반복으로 이동하기
* `crate` - 모듈 경로에서 crate 루트를 가리키는 것
* `dyn` - 트레이트 객체에 대한 동적 디스패치
* `else` - `if` 및 `if let` 제어 흐름 구조의 기본값
* `enum` - 열거형을 정의하기
* `extern` - 외부 함수 또는 변수에 연결하기
* `false` - 불린 값이 거짓인 리터럴
* `fn` - 함수 또는 함수 포인터 유형을 정의하기
* `for` - 이터레이터에서 항목을 반복하여 처리하거나 트레이트를 구현하거나 고등 순위 라이프타임을 지정하기
* `if` - 조건식의 결과에 따라 분기하기
* `impl` - 내재적 또는 트레이트 기능을 구현하기
* `in` - `for` 루프 구문의 일부
* `let` - 변수에 바인딩하기
* `loop` - 조건 없이 루프를 실행하기
* `match` - 값을 패턴과 일치시키기
* `mod` - 모듈을 정의하기
* `move` - 클로저가 모든 캡처를 소유하게 하기
* `mut` - 참조, 원시 포인터 또는 패턴 바인딩에서 변경 가능성을 나타내기
* `pub` - 구조체 필드, `impl` 블록 또는 모듈에서 공개 가시성을 나타내기
* `ref` - 참조로 바인딩하기
* `return` - 함수에서 반환하기
* `Self` - 현재 정의하거나 구현하고 있는 유형에 대한 유형 별칭
* `self` - 메서드 주체 또는 현재 모듈
* `static` - 프로그램 실행 전체 기간 동안 유효한 글로벌 변수 또는 라이프타임
* `struct` - 구조체를 정의하기
* `super` - 현재 모듈의 부모 모듈
* `trait` - 트레이트를 정의하기
* `true` - 불린 값이 참인 리터럴
* `type` - 유형 별칭 또는 연관 유형을 정의하기
* `union` - [union](../reference/items/unions.html)을 정의하기; union 선언에서만 키워드로 사용됩니다.
* `unsafe` - 불안전한 코드, 함수, 트레이트 또는 구현을 나타내기
* `use` - 기호를 스코프에 가져오기
* `where` - 유형을 제약하는 조항을 나타내기
* `while` - 표현식의 결과에 따라 조건적으로 루프를 실행하기

[union]: ../reference/items/unions.html

### 미래 사용을 위한 예약된 키워드

다음 키워드는 아직 기능이 없지만 Rust에서 미래 사용을 위해 예약되어 있습니다.

* `abstract`
* `become`
* `box`
* `do`
* `final`
* `macro`
* `override`
* `priv`
* `try`
* `typeof`
* `unsized`
* `virtual`
* `yield`

### 원시 식별자

*원시 식별자*는 키워드를 일반적으로 허용되지 않는 곳에서 사용할 수 있도록 하는 문법입니다. 원시 식별자를 사용하려면 키워드 앞에 `r#`를 붙입니다.

예를 들어, `match`는 키워드입니다. 다음 함수 이름으로 `match`를 사용하려고 시도하면 다음과 같은 오류가 발생합니다.

Filename: src/main.rs

```rust,ignore,does_not_compile
fn match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}
```

```text
error: expected identifier, found keyword `match`
 --> src/main.rs:4:4
  |
4 | fn match(needle: &str, haystack: &str) -> bool {
  |    ^^^^^ expected identifier, found keyword
```

오류 메시지에서 `match` 키워드를 함수 이름으로 사용할 수 없다는 것을 알 수 있습니다. `match`를 함수 이름으로 사용하려면 원시 식별자 문법을 사용해야 합니다.

```rust
fn r#match(needle: &str, haystack: &str) -> bool {
    haystack.contains(needle)
}

fn main() {
    assert!(r#match(\"foo\", \"foobar\"));
}
```

이 코드는 오류 없이 컴파일됩니다. 함수 정의에서 함수 이름과 `main`에서 함수를 호출하는 부분에서 모두 `r#` 접두사를 사용하는 것을 주의하십시오.

원시 식별자는 원하는 단어를 식별자로 사용할 수 있도록 허용합니다. 이 단어가 예약어라도 됩니다. 이는 식별자 이름을 선택하는 데 더 많은 자유를 제공하며, 이러한 단어가 예약어가 아닌 다른 언어로 작성된 프로그램과 통합할 수 있도록 합니다. 또한 원시 식별자는 다른 Rust 버전에서 작성된 라이브러리를 사용할 수 있도록 합니다. 예를 들어, `try`는 2015년판에서는 예약어가 아니지만 2018년판에서는 예약어입니다. 2015년판을 사용하는 라이브러리에 의존하고 `try` 함수가 있는 경우, 2018년판 코드에서 해당 함수를 호출하려면 원시 식별자 문법인 `r#try`를 사용해야 합니다. 자세한 내용은 [Appendix E][appendix-e]를 참조하십시오.

[appendix-e]: appendix-05-editions.html
