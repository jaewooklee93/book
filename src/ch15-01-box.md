## `Box<T>`를 사용하여 힙에 데이터를 가리키기

가장 직관적인 스마트 포인터는 `Box<T>` 타입으로, `Box`라고 합니다. `Box`는 데이터를 스택 대신 힙에 저장할 수 있도록 합니다. 스택에는 힙 데이터를 가리키는 포인터만 남습니다. 스택과 힙의 차이점은 제4장을 참조하세요.

`Box`는 힙에 데이터를 저장하는 것 외에는 성능 오버헤드가 없습니다. 하지만 많은 추가 기능도 없습니다. 다음과 같은 상황에서 가장 자주 사용합니다.

* 컴파일 시에 크기가 알 수 없는 타입이 있을 때, 해당 타입의 값을 정확한 크기가 필요한 맥락에서 사용하고 싶을 때
* 많은 양의 데이터가 있고 소유권을 전달하지만 복사하지 않고 전달하고 싶을 때
* 특정 타입이 아니라 특정 트레이트를 구현하는 타입이라고만 알고 소유하고 싶을 때

`Box`를 사용하여 재귀적 타입을 가능하게 하는 부분은 [“재귀적 타입을 활성화하는 방법”](#enabling-recursive-types-with-boxes)<!-- ignore --> 섹션에서 보여줍니다. 두 번째 경우, 많은 양의 데이터의 소유권을 전달하는 것은 데이터가 스택에서 복사되기 때문에 오랜 시간이 걸릴 수 있습니다. 이 상황에서 성능을 향상시키려면 `Box`에 힙에 큰 양의 데이터를 저장할 수 있습니다. 그러면 스택에서 작은 양의 포인터 데이터만 복사되고, 데이터가 가리키는 데이터는 힙의 한 곳에 유지됩니다. 세 번째 경우는 *트레이트 객체*로 알려져 있으며, 제17장은 [“다른 타입의 값을 허용하는 트레이트 객체 사용”][trait-objects]<!-- ignore -->라는 전체 섹션을 할애했습니다. 따라서 여기에서 배운 내용은 제17장에서 다시 적용할 수 있습니다!

### `Box<T>`를 사용하여 힙에 데이터를 저장하기

`Box<T>`의 힙 저장 사용 사례를 설명하기 전에 구문과 `Box<T>` 내에 저장된 값과 상호 작용하는 방법을 살펴보겠습니다.

표 15-1은 `Box`를 사용하여 힙에 `i32` 값을 저장하는 방법을 보여줍니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-01/src/main.rs}}
```

표 15-1: `Box`를 사용하여 힙에 `i32` 값을 저장하기

`b` 변수를 정의하여 `5` 값을 가리키는 `Box`의 값을 할당합니다. 이 프로그램은 `b = 5`를 출력합니다. 이 경우, `Box` 내의 데이터에 액세스하는 방법은 데이터가 스택에 있었다면과 동일합니다. 다른 소유된 값과 마찬가지로, `Box`가 `main` 함수의 범위를 벗어나면(즉, `b`가 끝나면) 해당 `Box`와 가리키는 데이터가 모두 해제됩니다.

단일 값을 힙에 저장하는 것은 그리 유용하지 않으므로, 대부분의 경우 `Box`를 이러한 방식으로 사용하지 않습니다. 스택에 저장되는 단일 `i32` 값과 같은 값은 스택에 저장하는 것이 더 적절합니다. 재귀적 타입을 정의하여 `Box`가 가능하게 해주는 경우를 살펴보겠습니다.

### 재귀적 타입을 활성화하는 방법

*재귀적 타입*의 값은 같은 타입의 다른 값을 포함할 수 있습니다. 재귀적 타입은 컴파일 시점에 Rust가 타입이 얼마나 많은 공간을 차지하는지 알아야 하기 때문에 문제가 됩니다. 그러나 재귀적 타입의 값의 nesting은 이론적으로 무한히 계속될 수 있기 때문에 Rust는 얼마나 많은 공간이 필요한지 알 수 없습니다. `Box`는 알려진 크기를 가지기 때문에 재귀적 타입을 활성화하려면 `Box`를 재귀적 타입 정의에 삽입할 수 있습니다.

재귀적 타입의 예로서 *cons 리스트*를 살펴보겠습니다. 이는 함수형 프로그래밍 언어에서 흔히 볼 수 있는 데이터 유형입니다. 우리가 정의할 `cons 리스트` 유형은 재귀적이기 때문에 개념이 유용합니다.

#### cons 리스트에 대한 자세한 내용

*cons 리스트*는 Lisp 프로그래밍 언어에서 나온 데이터 구조입니다. "


그리고 그 다이어렉트리와는 달리, 컨스 리스트는 쌍으로 이루어진 쌍의 중첩으로 구성되며, Lisp 버전의 연결 리스트입니다. 이름은 Lisp의 `cons` 함수( "구성 함수"의 약자)에서 유래되었으며, 이 함수는 두 인수로부터 새로운 쌍을 생성합니다. 쌍으로 구성된 값과 다른 쌍에 `cons`를 호출하면 `cons` 리스트를 생성할 수 있습니다. 이 리스트는 재귀적인 쌍으로 구성됩니다.

예를 들어, 1, 2, 3를 포함하는 컨스 리스트의 가상 코드 표현은 다음과 같습니다. 각 쌍은 괄호로 묶여 있습니다.

```text
(1, (2, (3, Nil)))

```

컨스 리스트의 각 항목에는 현재 항목의 값과 다음 항목이 포함됩니다. 리스트의 마지막 항목에는 `Nil`이라는 값만 포함되어 있으며 다음 항목이 없습니다. 컨스 리스트는 `cons` 함수를 재귀적으로 호출하여 생성됩니다. 재귀의 기본 사례를 나타내는 표준 이름은 `Nil`입니다. 이것이 제6장의 "null" 또는 "nil" 개념과 동일하지 않음을 유의하십시오. "null" 또는 "nil"은 유효하지 않거나 없는 값입니다.

컨스 리스트는 Rust에서 일반적으로 사용되는 데이터 구조는 아닙니다. 대부분의 경우 Rust에서 항목 목록을 가질 때 `Vec<T>`가 더 나은 선택입니다. 다른, 더 복잡한 재귀 데이터 유형은 다양한 상황에서 유용하지만, 이 장에서 컨스 리스트를 시작함으로써 상자를 사용하여 재귀 데이터 유형을 정의하는 방법을 탐구할 수 있습니다.

표 15-2에는 컨스 리스트 데이터 구조를 나타내는 `enum` 정의가 포함되어 있습니다. 이 코드는 아직 컴파일되지 않을 것입니다. `List` 유형은 알려진 크기를 가지지 않기 때문입니다.

Filename: src/main.rs

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-02/src/main.rs:here}}
```

표 15-2: `i32` 값의 컨스 리스트를 나타내는 `enum`을 정의하는 첫 번째 시도

> 참고: 이 예제에서는 `i32` 값만 저장하는 컨스 리스트를 구현했습니다. 제10장에서 논의했듯이, 일반화를 사용하여 어떤 유형의 값을 저장할 수 있는 컨스 리스트 유형을 정의할 수도 있습니다.

`List` 유형을 사용하여 `1, 2, 3` 리스트를 저장하는 것은 표 15-3과 같습니다.

Filename: src/main.rs

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-03/src/main.rs:here}}
```

표 15-3: `List` `enum`을 사용하여 `1, 2, 3` 리스트를 저장하는 방법

첫 번째 `Cons` 값은 `1`과 다른 `List` 값을 포함합니다. 이 `List` 값은 또 다른 `Cons` 값을 포함하며 `2`와 다른 `List` 값을 포함합니다. 이 `List` 값은 `3`을 포함하는 또 다른 `Cons` 값이며 마지막 `List` 값은 `Nil`입니다. `Nil`은 리스트의 끝을 나타내는 재귀적이지 않은 변형입니다.

표 15-3의 코드를 컴파일하려고 하면 표 15-4에 표시된 오류가 발생합니다.

```console
{{#include ../listings/ch15-smart-pointers/listing-15-03/output.txt}}
```

표 15-4: 재귀 `enum`을 정의하려고 할 때 발생하는 오류

오류 메시지에서 이 유형은 "무한 크기"를 가지고 있음을 알 수 있습니다. 이는 `List`를 재귀적으로 정의했기 때문입니다. 즉, `List` 값이 직접 다른 `List` 값을 포함하기 때문입니다. 따라서 Rust는 `List` 값을 저장할 공간을 얼마나 필요로 하는지 알 수 없습니다. 이 오류가 발생하는 이유를 자세히 살펴보기 위해 Rust가 비재귀 유형의 크기를 계산하는 방식을 먼저 살펴보겠습니다.

#### 비재귀 유형의 크기를 계산하는 방법

제6장에서 유효하지 않거나 없는 값을 나타내는 `enum`을 정의할 때 `Message` `enum`을 사용했습니다.

```rust
```

## Box를 사용하여 알 수 있는 크기의 재귀형 만들기

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-05/src/main.rs}}
```

Rust는 `Message` 값에 얼마나 많은 공간을 할당해야 하는지 결정하기 위해 각 변형을 살펴봅니다. Rust는 `Message::Quit`가 공간을 필요로 하지 않는다는 것을 알고, `Message::Move`는 두 개의 `i32` 값을 저장할 만큼의 공간이 필요하며, 이와 같이 계속합니다. 단 하나의 변형만 사용될 것이기 때문에 `Message` 값이 필요로 하는 최대 공간은 그 변형 중 가장 큰 변형이 차지하는 공간과 같습니다.

Listing 15-2의 `List` enum과 같은 재귀형의 경우 Rust가 얼마나 많은 공간을 할당해야 하는지 결정하려고 할 때는 컴파일러가 오류를 발생시키고 다음과 같은 도움말을 제공합니다.

<!-- manual-regeneration
after doing automatic regeneration, look at listings/ch15-smart-pointers/listing-15-03/output.txt and copy the relevant line
-->

```text
help: insert some indirection (e.g., a `Box`, `Rc`, or `&`) to break the cycle
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +
```

이 제안에서 “indirection”은 값을 직접 저장하는 대신, 포인터를 사용하여 값을 간접적으로 저장하는 것을 의미합니다.

`Box<T>`는 포인터이기 때문에 Rust는 `Box<T>`가 필요로 하는 공간을 항상 알고 있습니다. 즉, `Box<T>`가 가리키는 데이터의 양에 관계없이 포인터의 크기는 변하지 않습니다. 이는 `Cons` 변형 안에 `Box<T>`를 넣을 수 있음을 의미합니다. `Box<T>`는 힙에 있는 다음 `List` 값을 가리키는 포인터가 될 것입니다. 즉, `Cons` 변형 내부에 값이 들어있는 것처럼 보이지만, 실제로는 값들이 서로 인접해 있는 것과 같습니다.

Listing 15-2의 `List` enum 정의와 Listing 15-3의 `List` 사용을 Listing 15-5와 같이 변경하면 컴파일이 가능합니다.

Filename: src/main.rs

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-05/src/main.rs}}
```

Listing 15-5: `Box<T>`를 사용하여 알 수 있는 크기를 가진 `List`의 정의

`Cons` 변형은 `i32`의 크기와 `Box`의 포인터 데이터 크기를 필요로 합니다. `Nil` 변형은 값을 저장하지 않으므로 `Cons` 변형보다 적은 공간을 차지합니다. 이제 `List` 값이 `i32`의 크기와 `Box`의 포인터 데이터 크기만큼 공간을 차지한다는 것을 알 수 있습니다. `Box`를 사용함으로써 무한한 재귀적인 연결을 끊었기 때문에 컴파일러가 `List` 값을 저장하기 위해 필요한 크기를 알 수 있습니다. Figure 15-2는 `Cons` 변형이 이제 어떻게 보이는지 보여줍니다.

<img alt=\"A finite Cons list\" src=\"img/trpl15-02.svg\" class=\"center\" />

<h1>박스</h1>

그림 15-2: `Cons`가 `Box`를 갖고 있기 때문에 무한한 크기를 가진 것이 아닌 `List`

박스는 단지 간접 지정과 힙 할당을 제공합니다. 다른 특별한 기능은 없습니다. 이러한 특별한 기능은 나중에 다른 스마트 포인터 유형에서 볼 수 있습니다. 또한 이러한 특별한 기능이 발생시키는 성능 오버헤드가 없기 때문에, 간접 지정만이 필요한 경우(예: `cons` 리스트)에 유용할 수 있습니다. 박스의 다른 사용 사례는 제17장에서도 살펴볼 것입니다.

`Box<T>` 유형은 `Deref` 트레이트를 구현하기 때문에 스마트 포인터입니다. 이 트레이트는 `Box<T>` 값이 참조와 같이 처리될 수 있도록 합니다. `Box<T>` 값이 범위를 벗어날 때, 박스가 가리키는 힙 데이터도 `Drop` 트레이트 구현 덕분에 함께 정리됩니다. 이 두 트레이트는 이 장에서 나중에 논의할 다른 스마트 포인터 유형의 기능에 더욱 중요합니다. 두 트레이트를 자세히 살펴보겠습니다.

[trait-objects]: ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types