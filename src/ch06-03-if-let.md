## `if let`을 사용한 간결한 제어 흐름

`if let` 구문은 `if`와 `let`를 결합하여 하나의 패턴에 맞는 값을 처리하면서 나머지를 무시하는 더 간결한 방법을 제공합니다. 6-6번 목록의 프로그램을 참조하여 `Option<u8>` 값인 `config_max` 변수에 대해 `match`를 사용하여 처리하는 것을 살펴보겠습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-06/src/main.rs:here}}
```

<span class=\"caption\">Listing 6-6: `Some`일 때만 코드를 실행하는 `match`</span>

만약 값이 `Some`이라면, 패턴에서 `max` 변수에 값을 바인딩하여 `Some` 변수 내의 값을 출력합니다. `None` 값에 대해서는 아무것도 하지 않습니다. `match` 표현식을 만족하기 위해 `_ => ()`를 추가해야 하는데, 이는 불필요한 boilerplate 코드입니다.

대신 `if let`을 사용하여 이를 더 간결하게 작성할 수 있습니다. 다음 코드는 6-6번 목록의 `match`와 동일한 동작을 수행합니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-12-if-let/src/main.rs:here}}
```

`if let` 구문은 패턴과 등호 기호로 연결된 표현식을 받습니다. `match`와 동일하게 작동하며, 표현식이 `match`에 주어지고 패턴이 첫 번째 팔입니다. 이 경우 패턴은 `Some(max)`이며, `max`는 `Some` 안의 값에 바인딩됩니다. 그런 다음 `if let` 블록의 코드를 사용하여 `max`를 `match`의 해당하는 `match` 팔에서 사용하는 것과 동일하게 사용할 수 있습니다. `if let` 블록의 코드는 값이 패턴에 맞지 않으면 실행되지 않습니다.

`if let`을 사용하면 입력량이 줄어들고 들여쓰기가 줄어들고 불필요한 boilerplate 코드가 줄어듭니다. 그러나 `match`가 강제하는 완전한 검사를 잃게 됩니다. `match`와 `if let` 중 선택은 특정 상황에서 작업하고 간결성을 얻는 것이 완전한 검사를 잃는 것과의 적절한 거래인지 여부에 따라 달라집니다.

즉, `if let`은 `match`에 대해 패턴에 맞는 값을 처리하고 나머지 값을 무시하는 문법 설탕으로 생각할 수 있습니다.

`if let`에 `else`를 포함할 수 있습니다. `else`와 함께 있는 코드 블록은 `match` 표현식의 `_` 사례와 동일한 코드 블록입니다. `if let`과 `else`에 해당하는 `match` 표현식을 생각해 보세요.

6-4번 목록에서 정의된 `Coin` enum을 다시 생각해 보겠습니다. `Quarter` 변수는 `UsState` 값도 포함하고 있습니다. `match` 표현식을 사용하여 모든 쿼터가 아닌 동전을 세면서 동시에 쿼터의 상태를 알리는 경우, 다음과 같이 작성할 수 있습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-13-count-and-announce-match/src/main.rs:here}}
```

또는 `if let`과 `else` 표현식을 사용하여 다음과 같이 작성할 수 있습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-14-count-and-announce-if-let-else/src/main.rs:here}}
```

프로그램의 논리가 `match`를 사용하여 표현하기에는 너무 복잡한 경우, `if let`도 Rust 도구 상자에 있습니다. 

## 요약

이제 enum을 사용하여 하나의 집합의 명시된 값 중 하나일 수 있는 사용자 정의 유형을 만드는 방법을 익혔습니다. 표준 라이브러리의 `Option<T>` 유형이 유형 시스템을 사용하여 오류를 방지하는 방법을 보여드렸습니다. enum 값에 데이터가 포함되어 있는 경우, `match` 또는 `if let`을 사용하여 해당 값을 추출하고 사용할 수 있습니다. 필요한 사례의 수에 따라 `match` 또는 `if let`을 사용할 수 있습니다.

이제 Rust 프로그램에서 도메인 개념을 구조체와 enum을 사용하여 표현할 수 있습니다. API에 사용자 정의 유형을 만들면 유형 안전성이 보장됩니다. 컴파일러는 각 함수가 기대하는 유형의 값만 받도록 합니다.

사용자에게 직관적이고 사용하기 쉬운 잘 정리된 API를 제공하기 위해 필요한 것만 노출하는 것은 중요합니다. 이를 위해 Rust의 모듈을 사용해 보겠습니다.

