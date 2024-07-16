## 열거형 정의

구조체는 `Rectangle`과 같은 `width` 와 `height` 와 같은 관련된 필드와 데이터를 함께 그룹화하는 방법을 제공하는 반면, 열거형은 값이 가능한 값의 집합 중 하나임을 나타내는 방법을 제공합니다. 예를 들어, `Rectangle`이 `Circle`과 `Triangle`을 포함하는 가능한 모양의 집합 중 하나임을 나타낼 수 있습니다. 이를 위해 Rust는 이러한 가능성을 열거형으로 인코딩할 수 있도록 허용합니다.

코드로 표현하고 싶은 상황을 살펴보고 열거형이 구조체보다 유용하고 적합한 이유를 살펴보겠습니다. 예를 들어 IP 주소를 처리해야 할 수 있습니다. 현재 IP 주소에는 버전 4와 버전 6의 두 가지 주요 표준이 사용됩니다. 프로그램에서 처리할 IP 주소의 유일한 가능성이므로 IP 주소를 *열거*할 수 있습니다. 이것이 열거형이라는 이름의 기원입니다.

어떤 IP 주소든 버전 4 또는 버전 6 주소일 수 있지만 동시에 두 가지가 될 수 없습니다. IP 주소의 이러한 특성은 열거형 데이터 구조에 적합합니다. 열거형 값은 항상 자신의 변형 중 하나만을 가질 수 있기 때문입니다. 버전 4와 버전 6 주소는 모두 기본적으로 IP 주소이므로 코드가 모든 종류의 IP 주소에 적용되는 상황에서 동일한 유형으로 처리되어야 합니다.

`IpAddrKind` 열거형을 정의하고 IP 주소가 될 수 있는 가능한 종류인 `V4`와 `V6`을 나열하여 이 개념을 코드로 표현할 수 있습니다. 이것이 열거형의 변형입니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:def}}
```

`IpAddrKind`는 이제 코드의 다른 부분에서 사용할 수 있는 사용자 정의 데이터 유형입니다.

### 열거형 값

`IpAddrKind`의 두 가지 변형의 인스턴스를 생성할 수 있습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:instance}}
```

참고로 열거형의 변형은 식별자 아래에서 네임스페이스되며, 두 개의 콜론을 사용하여 두 개를 분리합니다. 이것은 `IpAddrKind::V4`와 `IpAddrKind::V6` 두 가지 값이 모두 `IpAddrKind` 유형이라는 점에서 유용합니다. 이제 `IpAddrKind`를 받는 함수를 정의할 수 있습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn}}
```

그리고 이 함수를 어떤 변형으로든 호출할 수 있습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn_call}}
```

열거형을 사용하는 것은 더 많은 장점이 있습니다. IP 주소 유형에 대해 더 생각해 보겠습니다. 현재는 실제 IP 주소 *데이터*를 저장하는 방법이 없습니다. *종류*만 알고 있습니다. 5장에서 구조체를 배웠기 때문에 Listing 6-1과 같이 구조체를 사용하여 이 문제를 해결하려고 할 수 있습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-01/src/main.rs:here}}
```

Listing 6-1: 구조체를 사용하여 IP 주소의 데이터와 `IpAddrKind` 변형을 저장하는 방법

여기서 `IpAddr` 구조체를 정의하여 `kind` 필드(이전에 정의한 열거형인 `IpAddrKind`)와 `address` 필드( `String` 유형)가 있습니다. 이 구조체의 두 가지 인스턴스가 있습니다. 첫 번째는 `home`이며, `kind` 필드에 `IpAddrKind::V4` 값을 가지고 `address` 데이터는 `127.0.0.1`입니다. 두 번째 인스턴스는 `loopback`이며, `kind` 필드에 다른 변형인 `IpAddrKind::V6` 값을 가지고 `address`는 `::1`입니다. `kind`와 `address` 값을 함께 묶어서 구조체를 사용하여 변형을 연결했습니다. 이제 변형이 값에 연결되었습니다.

그러나 열거형만으로 같은 개념을 나타내는 것이 더 간결합니다. 구조체 내에 열거형 대신 데이터를 직접 각 열거형 변형에 넣을 수 있습니다.
변형. `IpAddr` 열거형의 이 새로운 정의는 `V4` 와 `V6` 변형이 모두 `String` 값을 연결할 것이라고 말합니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-02-enum-with-data/src/main.rs:here}}
```

각 열거형 변형에 데이터를 직접 연결하므로 추가 구조체가 필요하지 않습니다. 여기서는 열거형이 작동하는 방식의 또 다른 세부 사항을 쉽게 볼 수 있습니다. 우리가 정의한 각 열거형 변형의 이름은 또한 해당 열거형의 인스턴스를 생성하는 함수가 됩니다. 즉, `IpAddr::V4()` 는 `String` 인수를 받아 `IpAddr` 유형의 인스턴스를 반환하는 함수 호출입니다. 열거형을 정의하는 결과로 자동으로 이 생성자 함수를 얻습니다.

구조체보다 열거형을 사용하는 데 또 다른 장점이 있습니다. 각 변형은 다른 유형과 연결된 데이터의 양을 가질 수 있습니다. 버전 4 IP 주소는 항상 0과 255 사이의 값을 가진 4개의 숫자 구성 요소를 가집니다. `V4` 주소를 4개의 `u8` 값으로 저장하고 여전히 `V6` 주소를 하나의 `String` 값으로 저장하고 싶다면 구조체로는 할 수 없습니다. 열거형은 이 경우를 쉽게 처리합니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-03-variants-with-different-data/src/main.rs:here}}
```

버전 4 및 버전 6 IP 주소를 저장하고 어떤 종류인지 표현하려는 다양한 방법을 보여주었습니다. 그러나 IP 주소를 저장하고 어떤 종류인지 나타내는 것이 얼마나 흔한지 알고 계시다면 표준 라이브러리에 사용할 수 있는 정의가 있습니다!][IpAddr]<!-- ignore --> 표준 라이브러리가 `IpAddr`를 정의하는 방식을 살펴보겠습니다. 정의된 열거형과 변형이 정확히 동일하지만, 각 변형 내부에 주소 데이터를 두 가지 다른 구조체 형태로 삽입합니다.

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

이 코드는 열거형 변형 내부에 어떤 종류의 데이터를 넣을 수 있는지 보여줍니다. 문자열, 숫자 유형 또는 구조체를 포함할 수 있습니다. 다른 열거형을 포함할 수도 있습니다! 또한 표준 라이브러리 유형은 종종 생각할 수 있는 것만큼 복잡하지 않습니다.

표준 라이브러리가 `IpAddr`를 정의하고 있더라도 우리가 정의한 우리의 정의를 사용할 수 있습니다. 우리가 표준 라이브러리의 정의를 our scope에 가져오지 않았기 때문입니다. Chapter 7에서 scope에 유형을 가져오는 방법에 대해 더 자세히 알아보겠습니다.

다음은 Listing 6-2에서 보여주는 또 다른 열거형의 예입니다. 이 열거형은 변형에 다양한 유형의 데이터가 포함되어 있습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

Listing 6-2: 각 변형이 다른 양과 유형의 값을 저장하는 `Message` 열거형

이 열거형은 다음과 같은 다양한 유형을 가진 네 가지 변형이 있습니다.

* `Quit` 는 데이터가 연결되어 있지 않습니다.
* `Move` 는 구조체와 같이 명명된 필드를 가지고 있습니다.
* `Write` 는 하나의 `String` 을 포함합니다.
* `ChangeColor` 는 세 개의 `i32` 값을 포함합니다.

Listing 6-2에서 정의된 `Message` 열거형과 같이 열거형을 정의하는 것은 구조체를 정의하는 것과 유사합니다. 단, 열거형은 `struct` 키워드를 사용하지 않으며 모든 변형이 `Message` 유형 아래에 함께 그룹화됩니다. 다음 구조체는 이전 열거형 변형에 있는 동일한 데이터를 저장할 수 있습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-04-structs-similar-to-message-enum/src/main.rs:here}}
```

그러나 각 유형이 있는 다양한 구조체를 사용했다면, 이러한 종류의 메시지를 모두 받는 함수를 정의하는 것이 열거형을 사용하는 것만큼 쉽지 않았을 것입니다.

enum과 struct의 또 다른 공통점은 `impl`을 사용하여 struct에 메서드를 정의할 수 있듯이, enum에도 메서드를 정의할 수 있다는 것입니다. 다음은 `Message` enum에 정의할 수 있는 `call`이라는 메서드입니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-05-methods-on-enums/src/main.rs:here}}
```

메서드의 본문에서는 `self`를 사용하여 메서드를 호출한 값을 가져옵니다. 이 예에서는 `m`이라는 변수가 `Message::Write(String::from("hello"))` 값을 가지고 있으며, `m.call()`이 실행될 때 `call` 메서드의 본문에서 `self`가 될 것입니다.

표준 라이브러리에 있는 또 다른 유용한 enum을 살펴보겠습니다. `Option`입니다.

### `Option` Enum과 Null 값 대신의 장점

이 섹션에서는 표준 라이브러리에서 정의된 또 다른 enum인 `Option`의 사례를 살펴봅니다. `Option` 유형은 값이 있을 수도 있고 없을 수도 있는 매우 일반적인 시나리오를 나타냅니다.

예를 들어, 비어 있지 않은 리스트에서 첫 번째 항목을 요청하면 값을 얻습니다. 비어 있는 리스트에서 첫 번째 항목을 요청하면 아무것도 얻지 않습니다. 이 개념을 형식 체계로 표현하면 컴파일러가 처리해야 하는 모든 경우를 확인할 수 있습니다. 이 기능은 다른 프로그래밍 언어에서 매우 흔한 오류를 방지하는 데 도움이 됩니다.

프로그래밍 언어 설계는 어떤 기능을 포함하는지에 대한 생각으로 자주 여겨지지만, 제외하는 기능도 중요합니다. Rust는 많은 다른 언어에 있는 null 기능이 없습니다. *Null*은 값이 없다는 것을 의미하는 값입니다. null을 가진 언어에서는 변수가 항상 두 가지 상태 중 하나일 수 있습니다: null 또는 not-null.

2009년 발표 "Null References: The Billion Dollar Mistake"에서 null의 발명자인 Tony Hoare는 다음과 같이 말했습니다.

> 나는 그것을 내가 만든 수십억 달러의 실수라고 부릅니다. 당시에는 객체 지향 언어에서 참조의 종합적인 형식 체계를 설계하고 있었습니다. 제 목표는 참조의 모든 사용이 컴파일러가 자동으로 검사하는 절대 안전해야 한다는 것이었습니다. 그러나 null 참조를 넣는 것에 저항할 수 없었습니다. 왜냐하면 그것이 구현하기 너무 쉬웠기 때문입니다. 이것은 지난 40년 동안 수십억 달러의 고통과 피해를 입힌 수많은 오류, 취약점 및 시스템 충돌로 이어졌습니다.

null 값의 문제는 null 값을 not-null 값으로 사용하려고 하면 오류가 발생하는 것입니다. 이 null 또는 not-null 속성이 널리 사용되기 때문에 이러한 종류의 오류를 쉽게 만들 수 있습니다.

그러나 null이 시도하는 개념은 여전히 유용합니다. null은 현재 일부 이유로 무효 또는 누락된 값입니다.

문제는 구체적인 구현이 아니라 개념 자체에 있습니다. 따라서 Rust는 null을 가지고 있지 않지만, 값이 현재 존재하거나 존재하지 않는다는 개념을 나타내는 enum을 가지고 있습니다. 이 enum은 `Option<T>`이며 표준 라이브러리에서 [정의되어 있습니다]<!-- ignore -->. 다음과 같습니다.

```rust
enum Option<T> {
    None,
    Some(T),
}
```

`Option<T>` enum은 매우 유용하여 prelude에 포함되어 있습니다. 즉, 명시적으로 스코프에 가져오지 않아도 됩니다. `Some`와 `None` 변형 또한 prelude에 포함되어 있습니다. `Some`와 `None`을 `Option::` 접두사 없이 직접 사용할 수 있습니다. `Option<T>` enum은 여전히 일반적인 enum이며, `Some(T)`와 `None`은 `Option<T>` 유형의 변형입니다.

`<T>` 문법은 아직 다루지 않은 Rust의 기능입니다. 이것은 일반 유형 매개변수이며, 10장에서 일반에 대해 자세히 다룹니다. 지금 당장은 `<T>`가 `Some` 변형의 `Option` enum이 어떤 유형의 데이터를 저장할 수 있는지 나타내는 것이며, `T`에 사용되는 각 구체적인 유형은 전체 `Option<T>` 유형을 다른 유형으로 만든다는 것을 알면 충분합니다. `Option`을 사용하여 숫자 유형과 문자열 유형을 저장하는 몇 가지 예입니다.

```rust
```

`some_number`의 유형은 `Option<i32>`입니다. `some_char`의 유형은
`Option<char>`이며, 이는 다른 유형입니다. Rust는 우리가 `Some` 변형 안에 값을 지정했기 때문에 이 유형을 추론할 수 있습니다. `absent_number`의 경우, Rust는 전체 `Option` 유형을 명시적으로 지정하도록 요구합니다. 컴파일러는 `None` 값만 보고 해당하는 `Some` 변형이 어떤 유형을 가질지 알 수 없습니다. 여기에서는 `absent_number`가 `Option<i32>` 유형이라는 것을 Rust에 알려줍니다.

`Some` 값을 가질 때는 값이 존재하고 그 값이 `Some` 안에 들어있다는 것을 알 수 있습니다. `None` 값을 가질 때는 어느 정도 `null`과 같은 의미입니다. 즉, 유효한 값이 없습니다. 그렇다면 `Option<T>`를 가질 때 `null`을 가질 때보다 어떤 것이 더 나을까요?

요약하자면, `Option<T>`와 `T` (여기서 `T`는 어떤 유형이든 될 수 있습니다)가 다른 유형이기 때문입니다. 컴파일러는 `Option<T>` 값을 확실히 유효한 값으로 사용하려고 하지 않습니다. 예를 들어, 이 코드는 `Option<i8>`에 `i8`을 더하려고 하기 때문에 컴파일되지 않습니다.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/src/main.rs:here}}
```

이 코드를 실행하면 다음과 같은 오류 메시지가 표시됩니다.

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/output.txt}}
```

강렬하죠! 실제로 이 오류 메시지는 Rust가 `i8`과 `Option<i8>`를 더하는 방법을 이해하지 못하기 때문에 발생합니다. 왜냐하면 그들은 다른 유형이기 때문입니다. Rust에서 `i8`와 같은 유형의 값을 가질 때 컴파일러는 항상 유효한 값이 있다는 것을 보장합니다. 그 값을 사용하기 전에 null을 확인할 필요 없이 자신감 있게 진행할 수 있습니다. `Option<i8>` (또는 작업 중인 값의 유형)를 가질 때만 유효하지 않을 수 있는 값에 대해 걱정해야 하며, 컴파일러는 해당 경우를 처리하도록 압박합니다. 

즉, `Option<T>`에서 `T` 값을 가져와 `T` 연산을 수행하려면 `Option<T>`를 `T`로 변환해야 합니다. 일반적으로 이는 null과 관련된 가장 흔한 문제 중 하나인, 값이 null이 아닐 것이라고 잘못 가정하는 것을 방지하는 데 도움이 됩니다.

null을 잘못 가정하는 위험을 제거하면 코드에 대한 자신감을 높일 수 있습니다. 값이 null일 수 있도록 하려면 `Option<T>` 유형으로 값의 유형을 명시적으로 선택해야 합니다. 그런 다음 값을 사용할 때는 값이 null일 수 있는 경우를 명시적으로 처리해야 합니다. `Option<T>`가 아닌 유형의 모든 값은 null이 아니라고 안전하게 가정할 수 있습니다. 이는 Rust에서 null의 범위를 제한하고 Rust 코드의 안전성을 높이기 위해 의도적으로 내려진 결정입니다.

그렇다면 `Option<T>` 값에서 `Some` 변형의 `T` 값을 가져와 `T`를 사용하려면 어떻게 하나요? `Option<T>` 유형의 값을 사용하려면 `Some` 변형이 있는 경우에만 실행되는 코드와 `None` 변형이 있는 경우에만 실행되는 코드가 있어야 합니다. `match` 표현식은 이러한 경우에 유용한 제어 흐름 구문입니다. `enum`과 함께 사용할 때 다양한 변형에 따라 다른 코드를 실행합니다.

enum의 어떤 변형을 가지고 있는지, 그리고 코드가 일치하는 값 안의 데이터를 사용할 수 있다는 것을 알 수 있습니다.

[IpAddr]: ../std/net/enum.IpAddr.html
[option]: ../std/option/enum.Option.html
[docs]: ../std/option/enum.Option.html
