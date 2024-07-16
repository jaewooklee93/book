## 고급 함수 및 클로저

이 섹션에서는 함수 및 클로저와 관련된 몇 가지 고급 기능을 탐구합니다.
함수 포인터와 클로저 반환을 포함합니다.

### 함수 포인터

함수에 클로저를 전달하는 방법에 대해 이미 알아보았습니다. 정규 함수를 함수에 전달할 수도 있습니다! 이 기술은 이미 정의한 함수를 새 클로저를 정의하는 대신 전달하고 싶을 때 유용합니다. 함수는 `fn` (소문자 f) 유형으로 변환됩니다. `Fn` 클로저 트레이트와 혼동하지 않도록 주의해야 합니다. `fn` 유형은 *함수 포인터*라고 합니다. 함수 포인터를 사용하여 함수를 다른 함수의 인수로 전달할 수 있습니다.

함수 포인터 인수가 함수인지 여부를 나타내는 방법의 문법은 Listing 19-27에서 보여주는 것처럼 클로저와 유사합니다. `add_one`이라는 함수를 정의했는데, 이 함수는 인수에 1을 더합니다. `do_twice` 함수는 `i32` 인수를 받는 함수와 `i32` 값을 두 개의 인수로 받습니다. `do_twice` 함수는 `f` 함수를 두 번 호출하여 `arg` 값을 전달한 다음 두 함수 호출 결과를 더합니다. `main` 함수는 `do_twice` 함수를 `add_one`과 `5`라는 인수로 호출합니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-27/src/main.rs}}
```

<span class=\"caption\">Listing 19-27: `fn` 유형을 사용하여 함수 포인터를 인수로 받는 함수</span>

이 코드는 "The answer is: 12"를 출력합니다. `do_twice` 함수의 `f` 인수가 `i32` 인수를 받고 `i32`를 반환하는 `fn`임을 명시합니다. `do_twice` 함수의 본문에서 `f`를 호출할 수 있습니다. `main` 함수에서는 `add_one` 함수를 `do_twice` 함수의 첫 번째 인수로 전달할 수 있습니다.

클로저와 달리 `fn`은 트레이트가 아니라 유형이므로 `Fn` 트레이트 중 하나를 트레이트 경계로 선언하는 대신 직접 `fn` 유형을 인수 유형으로 명시합니다.

함수 포인터는 모든 세 개의 클로저 트레이트 (`Fn`, `FnMut`, `FnOnce`)를 구현하므로 클로저를 기대하는 함수에 함수 포인터를 항상 전달할 수 있습니다. 함수를 일반 유형과 클로저 트레이트 중 하나를 사용하여 작성하는 것이 가장 좋습니다. 이렇게 하면 함수가 함수 또는 클로저를 모두 받을 수 있습니다.

하지만 클로저가 없는 외부 코드와 상호 작용해야 하는 경우에는 `fn`만 받고 클로저를 받지 않고 싶을 수 있습니다. 예를 들어 C 함수는 함수를 인수로 받을 수 있지만 C에는 클로저가 없습니다.

클로저를 직접 정의하거나 명명된 함수를 사용할 수 있는 예를 들어 `Iterator` 트레이트에서 제공하는 `map` 메서드를 살펴보겠습니다. `map` 함수를 사용하여 숫자 벡터를 문자열 벡터로 변환하려면 클로저를 사용할 수 있습니다.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-15-map-closure/src/main.rs:here}}
```

또는 `map` 함수의 인수로 명명된 함수를 사용할 수도 있습니다.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-16-map-function/src/main.rs:here}}
```

이전에 [“고급 트레이트”][advanced-traits]<!-- ignore --> 섹션에서 언급한 것처럼, `to_string`과 같은 여러 함수가 있는 경우 완전히 명시된 문법을 사용해야 합니다. 여기서는 `ToString` 트레이트에서 구현된 `to_string` 함수를 사용하고 있습니다. `Display`를 구현하는 모든 유형에 대해 표준 라이브러리가 구현했습니다.

제6장의 [“Enum 값”][enum-values]<!-- ignore --> 섹션에서 기억하시면 우리가 정의한 각 `enum` 변이의 이름은 초기화 함수가 되기도 합니다. 이 초기화 함수를 함수 포인터로 사용하여 클로저를 기대하는 메서드의 인수로 지정할 수 있습니다.

```rust
## 고급 함수와 클로저

### 초기화 함수 사용

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-17-map-initializer/src/main.rs:here}}
```

여기서 `Status::Value` 인스턴스를 `map`이 호출되는 범위 내의 각 `u32` 값을 사용하여 생성합니다. 초기화 함수를 사용하는 방식을 선호하는 사람도 있고, 클로저를 사용하는 사람도 있습니다. 둘 다 동일한 코드로 컴파일되므로, 이해하기 쉬운 스타일을 사용하세요.

### 클로저 반환

클로저는 추상형을 나타내므로 직접 클로저를 반환할 수 없습니다. 대부분 클로저를 반환하려는 경우, 추상형을 구현하는 구체적인 유형을 함수의 반환 값으로 사용할 수 있습니다. 그러나 클로저는 반환 가능한 구체적인 유형이 없기 때문에 이를 할 수 없습니다. 예를 들어, 함수 포인터 `fn`을 반환 유형으로 사용할 수 없습니다.

다음 코드는 직접 클로저를 반환하려고 하지만 컴파일되지 않습니다.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-18-returns-closure/src/lib.rs}}
```

컴파일러 오류는 다음과 같습니다.

```console
{{#include ../listings/ch19-advanced-features/no-listing-18-returns-closure/output.txt}}
```

오류 메시지는 다시 `Sized` 추상형을 언급합니다! Rust는 클로저를 저장하기 위해 얼마나 많은 공간이 필요한지 알 수 없습니다. 이 문제에 대한 해결책은 앞서 살펴보았습니다. 추상형 객체를 사용할 수 있습니다.

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-19-returns-closure-trait-object/src/lib.rs}}
```

이 코드는 컴파일됩니다. 추상형 객체에 대한 자세한 내용은 제17장의 [“다양한 유형의 값을 허용하는 추상형 객체 사용”]<!--
ignore --> 섹션을 참조하세요.

다음으로 매크로를 살펴보겠습니다!

[advanced-traits]:
ch19-03-advanced-traits.html#advanced-traits
[enum-values]: ch06-01-defining-an-enum.html#enum-values
[using-trait-objects-that-allow-for-values-of-different-types]:
ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
