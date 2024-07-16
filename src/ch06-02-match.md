<!-- Old heading. Do not remove or links may break. -->
<a id=\"the-match-control-flow-operator\"></a>
## `match` 제어 흐름 연산자

Rust는 `match`라는 매우 강력한 제어 흐름 연산자를 가지고 있습니다. 이 연산자는 값을 여러 패턴과 비교하고, 어떤 패턴이 일치하는지에 따라 코드를 실행하도록 허용합니다. 패턴은 문자열, 변수 이름, 와일드카드, 그리고 많은 다른 것들로 구성될 수 있습니다. [Chapter
18][ch18-00-patterns]<!-- ignore --> 에서는 다양한 종류의 패턴과 그 기능에 대해 자세히 설명합니다. `match`의 강력함은 패턴의 표현력과 컴파일러가 모든 가능한 경우가 처리되었음을 확인하기 때문입니다.

`match` 표현식을 동전 분류기와 같다고 생각해 보세요. 동전은 크기가 다른 구멍이 있는 트랙을 따라 미끄러지며, 각 동전은 가장 먼저 맞는 구멍을 통해 떨어집니다. 마찬가지로, 값은 `match`의 각 패턴을 거치며, 값이 “fits”하는 첫 번째 패턴에 도달하면 해당 패턴과 연결된 코드 블록이 실행됩니다.

동전을 예시로 사용해 보겠습니다! `match`를 사용하여 알 수 없는 미국 동전을 입력받고, 동전의 종류를 판별하여 센트 단위로 가치를 반환하는 함수를 작성할 수 있습니다. Listing 6-3에서 보여주는 것처럼.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-03/src/main.rs:here}}
```

<span class=\"caption\">Listing 6-3: `match` 표현식을 사용하는 `enum`</span>

`value_in_cents` 함수의 `match`를 살펴보겠습니다. 먼저 `match` 키워드, 그리고 `coin`이라는 값이 있는 표현식이 있습니다. 이것은 `if` 문과 유사해 보이지만, `if` 문의 경우 조건이 불리언 값으로 평가되어야 하는 반면, `match`는 어떤 타입이든 될 수 있습니다. 이 예제에서 `coin`의 타입은 Listing 6-3의 첫 번째 줄에 정의된 `Coin` enum입니다.

다음은 `match` 팔입니다. 각 팔은 패턴과 코드로 구성됩니다. 첫 번째 팔은 `Coin::Penny`라는 패턴과 `=>` 연산자를 가지고 있습니다. `=>` 연산자는 패턴과 실행될 코드를 분리합니다. 각 팔은 쉼표로 구분됩니다.

`match` 표현식이 실행될 때, 결과 값이 각 팔의 패턴과 비교됩니다. 패턴이 값과 일치하면 해당 패턴과 연결된 코드가 실행됩니다. 패턴이 일치하지 않으면 다음 팔로 실행이 계속됩니다. 우리는 필요에 따라 팔을 여러 개 사용할 수 있습니다. Listing 6-3에서는 `match`가 네 개의 팔을 가지고 있습니다.

각 팔의 코드는 표현식이며, 일치하는 팔의 표현식 결과 값이 전체 `match` 표현식의 결과 값이 됩니다.

코드 블록이 짧은 경우 괄호를 사용하지 않습니다. Listing 6-3에서 각 팔은 단순히 값을 반환하기 때문에 괄호를 사용하지 않았습니다. 여러 줄의 코드를 팔에서 실행하려면 괄호를 사용해야 하며, 이 경우 쉼표는 선택 사항이 됩니다. 예를 들어, 다음 코드는 `Coin::Penny`가 호출될 때마다 “Lucky penny!”를 출력하지만, 여전히 블록의 마지막 값인 `1`을 반환합니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-08-match-arm-multiple-lines/src/main.rs:here}}
```

### 값에 바인딩되는 패턴

`match` 팔의 또 다른 유용한 기능은 값의 부분에 바인딩할 수 있다는 것입니다. 이를 통해 enum 변수에서 값을 추출할 수 있습니다.

예를 들어, enum 변수 중 하나에 데이터를 추가해 보겠습니다. 1999년부터 2008년까지 미국은 각 주의 디자인을 한쪽에 새긴 동전을 만들었습니다. 다른 동전에는 주 디자인이 없었기 때문에, 이 추가 정보는 `Quarter` 변수에 포함되어 있습니다. `enum`을 변경하여 `UsState` 값을 포함하도록 변경할 수 있습니다.
내부에 저장되어 있는 값, 즉 Listing 6-4에서 한 것처럼

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-04/src/main.rs:here}}
```

<span class=\"caption\">Listing 6-4: `Coin` enum에서 `Quarter` 변형이 `UsState` 값을도 포함하는 경우</span>

친구가 모든 주 쿼터를 수집하려고 한다고 가정해 보겠습니다. 우리가 동전을 종류별로 분류하는 동안, 각 쿼터에 해당하는 주 이름을 외치면 친구가 가지고 있지 않은 주를 추가할 수 있습니다.

이 코드의 `match` 표현식에서 `state`라는 변수를 `Coin::Quarter` 변형에 일치하는 패턴에 추가했습니다. `Coin::Quarter`가 일치하면 `state` 변수는 해당 쿼터의 주 값에 바인딩됩니다. 그런 다음 `state`를 해당 팔의 코드에서 사용할 수 있습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-09-variable-in-pattern/src/main.rs:here}}
```

만약 `value_in_cents(Coin::Quarter(UsState::Alaska))`을 호출한다면, `coin`은 `Coin::Quarter(UsState::Alaska)`가 됩니다. 각 `match` 팔과 비교할 때, `None`이 아닌 `Some`을 일치시키기 위해 `Coin::Quarter(state)` 패턴에 도달합니다. 그때 `state`의 바인딩은 `UsState::Alaska` 값이 됩니다. 그러므로 `Coin` enum 변형의 내부 주 값을 `Quarter`에서 가져올 수 있습니다.

### `Option<T>`와의 일치

이전 섹션에서는 `Option<T>`를 사용할 때 `Some` 케이스에서 내부 `T` 값을 가져오려고 했습니다. `match`를 사용하여 `Option<T>`를 처리할 수도 있습니다. `Coin` enum과 마찬가지로 `Option<T>`의 변형을 비교하지만, `match` 표현식이 작동하는 방식은 동일합니다.

`Option<i32>`을 입력으로 받고, 값이 있다면 그 값에 1을 더하는 함수를 작성해 보겠습니다. 값이 없다면 함수는 `None` 값을 반환하고 어떤 연산도 수행하지 않습니다.

`match` 덕분에 이 함수를 쉽게 작성할 수 있으며 Listing 6-5와 같습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:here}}
```

<span class=\"caption\">Listing 6-5: `Option<i32>`에 대한 `match` 표현식을 사용하는 함수</span>

Listing 6-5의 `plus_one` 함수의 첫 번째 실행을 자세히 살펴보겠습니다. `plus_one(five)`를 호출하면 `plus_one` 함수의 `x` 변수에는 `Some(5)` 값이 있습니다. `match`를 실행하고 각 일치 팔과 비교합니다.

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

`Some(5)` 값은 `None` 패턴과 일치하지 않으므로 다음 팔로 이동합니다.

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:second_arm}}
```

`Some(5)`는 `Some(i)` 패턴과 일치합니까? 맞습니다! 동일한 변형을 가지고 있습니다. `i`는 `Some`에 포함된 값에 바인딩되므로 `i`는 `5` 값을 가집니다. 일치 팔의 코드가 실행되므로 `i` 값에 1을 더하고 `6`을 포함하는 새로운 `Some` 값을 만듭니다.

이제 Listing 6-5의 두 번째 `plus_one` 호출을 살펴보겠습니다. `x`는 `None`입니다. `match`를 실행하고 첫 번째 팔과 비교합니다.

```rust,ignore
"


## 6.2. `match` 사용하기

`match` 문은 Rust에서 값을 패턴에 맞춰 처리하는 데 사용됩니다. 이는 `if-else if-else` 문과 유사하지만, 더욱 유연하고 강력합니다.

예를 들어, `Option<T>` 타입의 값을 처리하는 경우, `match` 문을 사용하여 `Some(value)` 또는 `None` 중 어느 경우에 해당하는지 확인하고 각 경우에 맞는 코드를 실행할 수 있습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

이 코드는 `Option<i32>` 타입의 값을 받아서 `Some(value)`인 경우 `value`를 1 증가시키고, `None`인 경우 `None`을 반환합니다.

`match` 문은 패턴 매칭을 통해 값을 분류하고 각 분류에 대한 코드를 실행합니다. 이는 Rust에서 널값을 처리하는 데 유용하게 사용됩니다.

### `match`와 `enum`의 조합

`match` 문과 `enum` 타입을 함께 사용하면 더욱 강력한 기능을 얻을 수 있습니다. `enum` 타입은 여러 가지 값을 나타내는 데 사용되며, `match` 문을 사용하여 각 값에 대한 코드를 실행할 수 있습니다.

예를 들어, `Shape`라는 `enum` 타입을 정의하고, 각 값에 대한 코드를 실행하는 `match` 문을 사용할 수 있습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-06/src/main.rs:here}}
```

### `match`의 완전성

`match` 문은 모든 가능한 패턴을 처리해야 합니다. 즉, `match` 문의 모든 팔 (arm)은 모든 가능한 값을 커버해야 합니다.

만약 `match` 문이 모든 가능한 값을 커버하지 않으면 Rust는 오류를 발생시킵니다.

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/output.txt}}
```

### 모든 값을 처리하는 패턴 (`_`) 

`match` 문에서 모든 값을 처리하는 패턴으로 `_`를 사용할 수 있습니다. `_`는 특정 값을 바인딩하지 않고 모든 값을 매칭합니다.

예를 들어, 위에서 설명한 게임 예시에서 3 또는 7이 아닌 모든 값을 처리하는 경우 `_`를 사용할 수 있습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-15-binding-catchall/src/main.rs:here}}
```

`_`를 사용하면 `match` 문이 모든 가능한 값을 처리하는 것을 보장하면서도, 특정 값을 사용하지 않을 때 코드를 간결하게 만들 수 있습니다.

`match` 문은 Rust에서 값을 처리하는 데 매우 유용한 도구입니다. `match` 문과 `enum` 타입을 함께 사용하면 더욱 강력한 기능을 얻을 수 있습니다.



```

이 예제 또한 완전성 요구 사항을 충족합니다. 왜냐하면 마지막 arm에서 모든 다른 값을 명시적으로 무시하기 때문입니다. 아무것도 잊지 않았습니다.

마지막으로 게임 규칙을 한 번 더 변경하여 3 또는 7을 굴리지 않으면 턴에 아무런 일이 일어나지 않도록 하겠습니다. `_` arm에 있는 코드로 이를 표현할 수 있습니다.

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-17-underscore-unit/src/main.rs:here}}
```

여기서 `_` arm에서 이전 arm의 패턴에 맞지 않는 다른 모든 값을 Rust에 명시적으로 사용하지 않을 것이라고 알리고 있습니다. 이 경우 코드를 실행하지 않으려고 합니다.

패턴과 매칭에 대한 자세한 내용은 [Chapter 18][ch18-00-patterns]<!-- ignore -->에서 다룹니다. 지금은 `if let` 구문으로 넘어가겠습니다. `match` 표현식이 조금 복잡한 경우 유용할 수 있습니다.

[ch18-00-patterns]: ch18-00-patterns.html
