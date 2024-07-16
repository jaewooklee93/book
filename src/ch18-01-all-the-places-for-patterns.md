## 패턴이 사용될 수 있는 모든 곳

패턴은 Rust에서 여러 곳에서 나타나며, 이미 많이 사용해왔을 것입니다.
이 섹션에서는 패턴이 유효한 모든 위치에 대해 설명합니다.

### `match` 팔

6장에서 설명했듯이, 우리는 `match` 표현식의 팔에 패턴을 사용합니다.
형식적으로, `match` 표현식은 `match` 키워드, 매칭할 값, 그리고 값이 해당 팔의 패턴과 일치하는 경우 실행할 패턴과 표현식으로 구성됩니다. 예를 들어 다음과 같습니다.

```text
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```

예를 들어, Listing 6-5에서 `Option<i32>` 값을 가진 변수 `x`에 대해 매칭하는 `match` 표현식을 살펴보겠습니다.

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

이 `match` 표현식의 패턴은 `None`과 `Some(i)`입니다.

`match` 표현식의 한 가지 요구 사항은 *완전성*이라는 것입니다. 즉, `match` 표현식의 값에 대한 모든 가능성을 처리해야 합니다. 모든 가능성을 다룬 것인지 확인하는 한 가지 방법은 마지막 팔에 모든 경우를 처리하는 catchall 패턴을 사용하는 것입니다. 예를 들어, 모든 값에 일치하는 변수 이름은 항상 실패하지 않고 모든 나머지 경우를 처리하기 때문에 유용합니다.

`_` 패턴은 모든 값에 일치하지만, 변수에 바인딩되지 않으므로 종종 마지막 팔에서 사용됩니다. `_` 패턴은 지정되지 않은 값을 무시하고 싶을 때 유용합니다. 이 챕터의 [“패턴에서 값 무시”][ignoring-values-in-a-pattern]<!-- ignore --> 섹션에서 `_` 패턴에 대해 자세히 설명합니다.

### 조건부 `if let` 표현식

6장에서 `if let` 표현식을 사용하여 `match`의 한 가지 경우만 매칭하는 더 간결한 방법으로 설명했습니다.
선택적으로, `if let`은 패턴이 일치하지 않을 경우 실행할 코드를 포함하는 `else`를 가질 수 있습니다.

Listing 18-1은 `if let`, `else if`, `else if let` 표현식을 혼합하여 사용할 수 있다는 것을 보여줍니다. 이렇게 하면 `match` 표현식보다 더 많은 유연성을 얻을 수 있습니다. `match` 표현식은 하나의 값만 비교할 수 있지만, `if let`, `else if`, `else if let` 팔은 서로 관련이 없는 조건을 표현할 수 있습니다.

Listing 18-1의 코드는 사용자 입력에서 받은 값을 기반으로 배경 색상을 결정하는 방법을 보여줍니다. 이 예제에서는 실제 프로그램이 사용자 입력에서 받을 수 있는 값을 하드코딩하여 만들었습니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-01/src/main.rs}}
```

<span class=\"caption\">Listing 18-1: `if let`, `else if`, `else if let`,
`else`를 혼합</span>

사용자가 좋아하는 색상을 지정하면 해당 색상이 배경 색상으로 사용됩니다. 좋아하는 색상이 지정되지 않았고 오늘이 화요일이라면 배경 색상은 초록색입니다. 그렇지 않으면 사용자가 나이를 문자열로 지정하고 성공적으로 숫자로 변환할 수 있다면, 숫자의 값에 따라 보라색 또는 주황색이 배경 색상이 됩니다. 위의 조건이 모두 적용되지 않으면 배경 색상은 파란색입니다.

이 조건부 구조는 복잡한 요구 사항을 지원할 수 있습니다. 여기서 사용한 하드코딩된 값으로는 이 예제가 `보라색을 배경 색상으로 사용합니다.`라고 출력됩니다.

`if let`은 `match` 팔과 마찬가지로 동일한 범위 내에서 변수를 가리키는 것을 막습니다. `if let Ok(age) = age` 라는 줄은 `Ok` 변수 안의 값을 담는 새로운 `age` 변수를 만들어 냅니다. 따라서 `if age > 30` 조건을 해당 블록 내부에 두어야 합니다. 즉, `if let Ok(age) = age && age > 30`과 같이 두 조건을 결합할 수 없습니다. 비교하려는 `age`는 괄호 안의 새 범위가 시작될 때까지 유효하지 않습니다.

 `if let` 표현을 사용하는 단점은 컴파일러가 완전성을 확인하지 않는다는 것입니다. `match` 표현과 달리 컴파일러는 완전성을 확인합니다. 마지막 `else` 블록을 생략하고 일부 케이스를 처리하지 않으면 컴파일러가 가능한 논리 오류에 대해 알려주지 않습니다.

### `while let` 조건 루프

`if let`과 유사하게 구축된 `while let` 조건 루프는 패턴이 계속해서 일치하는 동안 `while` 루프가 실행되도록 허용합니다. 18-2번 목록에서 벡터를 스택으로 사용하여 푸시된 순서와 반대 순서로 벡터의 값을 출력하는 `while let` 루프를 코딩합니다.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-02/src/main.rs:here}}
```

<span class=\"caption\">Listing 18-2: `while let` 루프를 사용하여 `stack.pop()`이 `Some`을 반환하는 동안 값을 출력</span>

이 예제는 3, 2, 그리고 1을 출력합니다. `pop` 메서드는 벡터에서 마지막 요소를 제거하고 `Some(value)`를 반환합니다. 벡터가 비어 있으면 `pop`은 `None`을 반환합니다. `while` 루프는 `pop`이 `Some`을 반환하는 동안 코드 블록을 계속 실행합니다. `pop`이 `None`을 반환하면 루프가 중단됩니다. `while let`을 사용하여 스택에서 모든 요소를 팝할 수 있습니다.

### `for` 루프

`for` 루프에서 `for` 키워드 뒤에 직접적으로 나오는 값은 패턴입니다. 예를 들어, `for x in y`에서 `x`는 패턴입니다. 18-3번 목록은 `for` 루프에서 패턴을 사용하여 튜플을 분해하거나, 튜플을 분해하는 방법을 보여줍니다.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-03/src/main.rs:here}}
```

<span class=\"caption\">Listing 18-3: `for` 루프에서 패턴을 사용하여 튜플을 분해</span>

18-3번 목록의 코드는 다음과 같은 출력을 생성합니다.

```console
{{#include ../listings/ch18-patterns-and-matching/listing-18-03/output.txt}}
```

`enumerate` 메서드를 사용하여 이터레이터를 적용하여 값과 그 값의 인덱스를 생성하여 튜플로 묶습니다. 생성된 첫 번째 값은 `(0, 'a')` 튜플입니다. 이 값이 패턴 `(index, value)`에 일치할 때, `index`는 `0`이고 `value`는 `'a'`이므로 출력의 첫 번째 줄이 출력됩니다.

### `let` 문

이 장에서까지는 `match`와 `if let`과 함께 패턴을 사용했다고 명시적으로 논의했지만, 실제로는 `let` 문에서도 다른 곳에서 패턴을 사용해 왔습니다. 예를 들어, 다음과 같은 간단한 변수 할당을 `let`으로 고려해 보세요.

```rust
let x = 5;
```

`let` 문과 같은 문장을 사용할 때마다 패턴을 사용해 왔지만, 그렇게 생각하지 않았을 수도 있습니다! 더 정확하게 말하면, `let` 문은 다음과 같이 보입니다.

```text
let PATTERN = EXPRESSION;
```

`let x = 5;`와 같은 문장에서 `PATTERN` 슬롯에 변수 이름이 있을 때, 변수 이름은 패턴의 특별히 간단한 형태입니다. Rust는 `EXPRESSION`을 `PATTERN`과 비교하고, 찾은 모든 이름을 할당합니다. 따라서 `let x = 5;` 예제에서 `x`는 “이 부분에 일치하는 것을 변수 `x`에 바인딩하라”라는 패턴을 의미합니다. `x`라는 이름이 전체 패턴이기 때문에 이 패턴은 “어떤 값이든 `x`라는 변수에 바인딩하라”라는 의미입니다.

`let`에서 패턴을 일치시키는 측면을 더 명확하게 보려면 18-4번 목록을 참조하십시오. 이 목록은 패턴을 사용하여 튜플을 분해하는 `let`을 보여줍니다.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-04/src/main.rs:here}}
```

<span class=\"caption\">Listing 18-4: 패턴을 사용하여 튜플을 분해하고 한 번에 세 개의 변수를 생성</span>

여기서 우리는 튜플을 패턴에 맞춥니다. Rust는 `(1, 2, 3)` 값을 패턴과 비교합니다.
Rust는 패턴을 사용하여 값을 분해하고 변수에 할당할 수 있습니다. 패턴은 값의 구조를 나타내는 표현식입니다. 예를 들어, 튜플 `(1, 2, 3)`에 대한 패턴 `(x, y, z)`를 사용하면 Rust는 값이 패턴에 맞는지 확인합니다. 만약 맞으면 `1`을 `x`에, `2`를 `y`에, `3`을 `z`에 할당합니다. 이 튜플 패턴은 세 개의 개별 변수 패턴을 중첩한 것과 같다고 생각할 수 있습니다.

튜플 패턴의 요소 수가 튜플의 요소 수와 일치하지 않으면 전체 유형이 일치하지 않아 컴파일러 오류가 발생합니다. 예를 들어, 18-5번 목록은 세 개의 요소를 가진 튜플을 두 개의 변수로 해체하려는 시도를 보여줍니다. 이것은 작동하지 않습니다.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-05/src/main.rs:here}}
```

<span class=\"caption\">18-5번 목록: 튜플의 요소 수와 일치하지 않는 패턴으로 인해 오류가 발생하는 코드</span>

이 코드를 컴파일하려고 하면 다음과 같은 유형 오류가 발생합니다.

```console
{{#include ../listings/ch18-patterns-and-matching/listing-18-05/output.txt}}
```

오류를 수정하려면 튜플에서 하나 이상의 값을 `_` 또는 `..`를 사용하여 무시하거나, 변수의 수가 튜플의 요소 수와 일치하도록 변수를 제거할 수 있습니다.

### 함수 매개변수

함수 매개변수에도 패턴을 사용할 수 있습니다. 18-6번 목록은 `i32` 유형의 하나의 매개변수 `x`를 가진 함수 `foo`를 선언합니다. 이제는 익숙하실 것입니다.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-06/src/main.rs:here}}
```

<span class=\"caption\">18-6번 목록: 함수 선언문에서 패턴이 사용되는 매개변수</span>

`x` 부분은 패턴입니다! `let`과 마찬가지로 함수의 인수에 튜플을 매칭할 수 있습니다. 18-7번 목록은 함수에 전달되는 튜플의 값을 분리합니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-07/src/main.rs}}
```

<span class=\"caption\">18-7번 목록: 매개변수가 튜플을 해체하는 함수</span>

이 코드는 `Current location: (3, 5)`를 출력합니다. `&(3, 5)` 값이 패턴 `&(x, y)`에 맞기 때문에 `x`는 값 `3`이고 `y`는 값 `5`입니다.

마찬가지로, 13장에서 논의된 것처럼 클로저 매개변수 목록에도 함수 매개변수와 동일한 방식으로 패턴을 사용할 수 있습니다.

이제 패턴을 사용하는 여러 가지 방법을 보았지만, 모든 위치에서 패턴이 동일하게 작동하는 것은 아닙니다. 일부 위치에서는 패턴이 확실해야 하며, 다른 상황에서는 불확실할 수 있습니다. 다음으로 이 두 개념을 논의할 것입니다.


