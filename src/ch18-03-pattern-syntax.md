## 패턴 구문

이 섹션에서는 패턴에서 유효한 모든 구문을 모아 각 구문을 사용하는 이유와
언제 사용해야 하는지에 대해 논의합니다.

### 리터럴과의 일치

제6장에서 보았듯이, 패턴을 리터럴과 직접 일치시킬 수 있습니다.
다음 코드는 몇 가지 예입니다:

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/no-listing-01-literals/src/main.rs:here}}
```

이 코드는 `x` 값이 1이기 때문에 `one`을 출력합니다. 이 구문은 특정 구체적인
값을 받으면 코드가 특정 작업을 수행하도록 하는 데 유용합니다.

### 이름 지정된 변수와의 일치

이름 지정된 변수는 부정확한 패턴이며, 책에서 여러 번 사용해 왔습니다.
그러나 `match` 표현식에서 이름 지정된 변수를 사용할 때는 복잡한 부분이 있습니다.
`match`가 새로운 범위를 시작하기 때문에 `match` 구문 내부의 패턴으로서
선언된 변수는 외부 `match` 구조에서 동일한 이름을 가진 변수를 가려
집니다. 모든 변수와 마찬가지로 이는 사실입니다. Listing 18-11에서 `x`라는
변수를 `Some(5)` 값으로 선언하고 `y`라는 변수를 `10` 값으로 선언합니다.
그런 다음 `x` 값에 대한 `match` 표현식을 만듭니다. match 팔의 패턴과
마지막 `println!`을 살펴보고 코드가 실행되기 전에 또는 더 읽기 전에
코드가 무엇을 출력할지 시도해 보세요.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-11/src/main.rs:here}}
```

<span class=\"caption\">Listing 18-11: 이름 지정된 변수 `y`를 가리는
`match` 표현식</span>

`match` 표현식이 실행될 때 발생하는 일을 살펴보겠습니다. 첫 번째 `match` 팔의
패턴은 정의된 `x` 값에 일치하지 않으므로 코드가 계속 진행됩니다.

두 번째 `match` 팔의 패턴은 `Some` 값 안의 모든 값에 일치하는 새로운 변수
`y`를 도입합니다. `match` 표현식 내부에 있는 새로운 범위에서 이를
하기 때문에 이는 외부에서 선언된 `y`가 아닌 새로운 `y` 변수입니다.
이 새로운 `y` 바인딩은 `Some` 안의 모든 값에 일치하므로 `x`에 있는
값과 일치합니다. 따라서 이 새로운 `y`는 `x`의 `Some` 안의 내부 값에 바인딩됩니다.
그 값은 `5`이므로 해당 팔의 표현식이 실행되고 `Matched, y = 5`가 출력됩니다.

만약 `x`가 `None` 값이었더라면 첫 번째 두 개의 팔의 패턴은 일치하지 않았을
것이므로 값은 언더스코어에 일치하게 됩니다. 언더스코어 팔의 패턴에
`x` 변수를 도입하지 않았기 때문에 표현식의 `x`는 여전히 외부 `x`이며
가려지지 않았습니다. 이 가정적인 경우 `match`는 `Default case, x = None`을
출력할 것입니다.

`match` 표현식이 실행되면 그 범위가 끝나고 내부 `y`의 범위도 끝납니다.
마지막 `println!`은 `at the end: x = Some(5), y = 10`을 출력합니다.

외부 `x`와 `y`의 값을 비교하는 `match` 표현식을 만드려면 `match` 구문
보다는 `match` 가드 조건을 사용해야 합니다. 이후에 `Extra Conditionals with
Match Guards` 섹션에서 `match` 가드에 대해 자세히 설명하겠습니다.

### 여러 패턴과의 일치

`match` 표현식에서 `|` 구문을 사용하여 여러 패턴을 일치시킬 수 있습니다.
이는 패턴 *또는* 연산자입니다. 다음 코드에서 `x` 값을 `match` 팔과 일치시키는데,
첫 번째 팔에는 *또는* 옵션이 있으므로 `x` 값이 해당 팔의 값 중 어느
쪽과도 일치하면 해당 팔의 코드가 실행됩니다.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/no-listing-02-multiple-patterns/src/main.rs:here}}
```

이 코드는 `one or two`를 출력합니다.

### `..=`를 사용하여 값 범위 일치

`..=` 구문은 값 범위에 일치하는 데 사용할 수 있습니다. 다음 코드에서 `x` 값을
`match` 팔과 일치시키는데, 첫 번째 팔은 `..=`를 사용하여 1부터 3까지의
범위에 일치하는 패턴을 사용합니다.

```rust
{{#rustdoc_include ../listings
listings/ch18-patterns-and-matching/no-listing-03-range-matching/src/main.rs:here}}
```

이 코드는 `Matched 2`를 출력합니다.

### 패턴과의 일치를 사용하는 방법

패턴은 Rust에서 코드를 작성하는 데 사용되는 강력한 도구입니다. 이
섹션에서 다룬 패턴 구문을 이해하면 코드를 더 명확하고 효율적으로
작성할 수 있습니다.

다음 코드는 패턴이 주어진 범위 내의 값 중 어느 하나와 일치하면 해당 팔을 실행합니다.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/no-listing-03-ranges/src/main.rs:here}}
```

`x`가 1, 2, 3, 4 또는 5이면 첫 번째 팔이 일치합니다. 이 문법은 `|` 연산자를 사용하여 동일한 개념을 표현하는 것보다 여러 개의 일치 값에 대해 더 편리합니다. `|`를 사용하려면 `1 | 2 | 3 | 4 | 5`와 같이 명시해야 합니다. 범위를 명시하는 것이 훨씬 짧으며, 예를 들어 1부터 1,000까지의 모든 숫자를 일치시키려면 특히 유용합니다.

컴파일러는 범위가 컴파일 시간에 비어 있는지 확인하며, Rust가 범위가 비어 있는지 여부를 알 수 있는 유일한 유형은 `char`과 숫자 값이기 때문에 범위는 숫자 또는 `char` 값과 함께만 사용할 수 있습니다.

다음은 `char` 값의 범위를 사용하는 예입니다.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/no-listing-04-ranges-of-char/src/main.rs:here}}
```

Rust는 `'c'`가 첫 번째 패턴의 범위 내에 있음을 알 수 있으며 `early ASCII letter`를 출력합니다.

### 구조 분해: 값을 분리하기

또한 패턴을 사용하여 구조체, 열거형 및 튜플을 분해하여 이러한 값의 다른 부분을 사용할 수 있습니다. 각 값을 살펴보겠습니다.

#### 구조체 분해

18-12번 목록은 `x`와 `y`라는 두 필드를 가진 `Point` 구조체를 보여줍니다. 이를 패턴과 함께 `let` 문을 사용하여 분해할 수 있습니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-12/src/main.rs}}
```

<span class=\"caption\">18-12번 목록: 구조체 필드를 분리하여 별도의 변수에 저장하기</span>

이 코드는 `p` 구조체의 `x`와 `y` 필드의 값을 일치시키는 `a`와 `b`라는 변수를 생성합니다. 이 예제는 패턴의 변수 이름이 구조체의 필드 이름과 일치하지 않아도 된다는 것을 보여줍니다. 그러나 변수 이름을 필드 이름과 일치시키는 것이 일반적이며, 어떤 변수가 어떤 필드에서 가져왔는지 기억하기 쉽기 때문입니다. 이러한 일반적인 사용법과 `let Point { x: x, y: y } = p;`가 중복을 많이 포함하기 때문에 Rust는 구조체 필드를 일치시키는 패턴에 대한 약속을 제공합니다. 패턴에서 구조체 필드 이름만 나열하면 변수가 생성되고, 이 변수 이름은 필드 이름과 동일합니다. 18-13번 목록은 18-12번 목록과 동일한 방식으로 작동하지만, `let` 패턴에서 생성되는 변수는 `x`와 `y`입니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-13/src/main.rs}}
```

<span class=\"caption\">18-13번 목록: 구조체 필드를 일치시키는 패턴을 사용하여 구조체 필드를 분해하기</span>

이 코드는 `p` 변수의 `x`와 `y` 필드를 일치시키는 `x`와 `y`라는 변수를 생성합니다. 결과적으로 `x`와 `y` 변수에는 `p` 구조체의 값이 저장됩니다.

구조체 패턴에서 문자열 값을 포함하여 분해할 수도 있습니다. 이를 통해 일부 필드를 특정 값으로 테스트하면서 다른 필드를 분해할 수 있습니다.

18-14번 목록에서는 `Point` 값을 세 가지 경우로 분리하는 `match` 표현식이 있습니다. `y = 0`일 때 `x` 축에 직접 위치한 점(즉, `y = 0`), `x = 0`일 때 `y` 축에 위치한 점 또는 그렇지 않은 점입니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-14/src/main.rs:here}}
```

## 패턴 구문"
}## 패턴 문법이 코드는 `Some numbers: 2, 8, 32`를 출력하고, 4와 16은 무시됩니다.

#### 사용하지 않는 변수를 `_`로 시작하여 무시하기

변수를 생성하지만 어디에서도 사용하지 않으면 Rust는 일반적으로 사용되지 않는 변수가 버그일 수 있기 때문에 경고를 냅니다. 그러나 때때로는 아직 사용하지 않을 변수를 생성하는 것이 유용할 수 있습니다. 예를 들어 프로토타입을 작성하거나 프로젝트를 시작할 때입니다. 이러한 상황에서는 Rust에게 사용되지 않는 변수에 대해 경고하지 않도록 `_`로 변수 이름을 시작하여 알 수 있습니다. Listing 18-20에서 두 개의 사용되지 않는 변수를 생성하지만 코드를 컴파일하면 하나에 대해서만 경고를 받습니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-20/src/main.rs}}
```

<span class=\"caption\">Listing 18-20: 사용되지 않는 변수 경고를 피하기 위해 변수 이름을 `_`로 시작하기</span>

여기서 `y` 변수를 사용하지 않아 경고를 받지만 `_x`를 사용하지 않아 경고를 받지 않습니다.

`_`만 사용하는 것과 `_`로 시작하는 이름을 사용하는 것 사이에는 미묘한 차이가 있습니다. `_x` 문법은 여전히 값을 변수에 바인딩하지만, `_`는 바인딩하지 않습니다. 이 차이가 중요한 경우를 보여주기 위해 Listing 18-21은 오류를 발생시킵니다.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-21/src/main.rs:here}}
```

<span class=\"caption\">Listing 18-21: `_`로 시작하는 사용되지 않는 변수는 여전히 값을 바인딩하여 값의 소유권을 가져갈 수 있습니다</span>

`s` 값이 `_s`로 이동되어 사용할 수 없게 되므로 오류가 발생합니다. 그러나 `_`만 사용하면 값에 결코 바인딩되지 않습니다. Listing 18-22는 오류 없이 컴파일됩니다. `s`가 `_`로 이동되지 않기 때문입니다.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-22/src/main.rs:here}}
```

<span class=\"caption\">Listing 18-22: `_`를 사용하면 값이 바인딩되지 않습니다</span>

이 코드는 `s`가 아무것도 바인딩되지 않기 때문에 괜찮게 작동합니다.



#### `..`를 사용하여 값의 나머지 부분 무시하기

여러 부분을 가진 값의 경우, 특정 부분을 사용하고 나머지를 무시하는 데 `..` 문법을 사용할 수 있습니다. `..` 패턴은 우리가 명시적으로 일치시키지 않은 값의 모든 부분을 무시합니다. Listing 18-23에서 `Point` 구조체는 3차원 공간에서의 좌표를 저장합니다. `match` 표현식에서 `x` 좌표만 작업하고 `y` 및 `z` 필드의 값을 무시하려고 합니다.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-23/src/main.rs:here}}
```

<span class=\"caption\">Listing 18-23: `..`를 사용하여 `Point`의 `x`만 일치시키고 나머지를 무시하기</span>

`x` 값을 나열한 후 `..` 패턴만 포함합니다. 이는 `y: _` 및 `z: _`를 나열하는 것보다 특히 많은 필드를 가진 구조체를 다룰 때 더 빠릅니다. 우리가 관심 있는 필드가 하나 또는 두 개일 때 유용합니다.

`..` 문법은 필요한 만큼의 값을 확장합니다. Listing 18-24는 `..`를 사용하여 튜플을 보여줍니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-24/src/main.rs}}
```

<span class=\"caption\">Listing 18-24: 튜플에서 첫 번째와 마지막 값만 일치시키기</span>

튜플이고 나머지 값은 무시합니다.

이 코드에서 첫 번째와 마지막 값은 `first`와 `last`로 매칭됩니다. `..`는 중간에 있는 모든 값을 매칭하고 무시합니다.

그러나 `..`를 사용하는 것은 모호해야 합니다. 매칭할 값과 무시할 값이 명확하지 않으면 Rust는 오류를 줍니다. 18-25번 목록은 `..`를 모호하게 사용하는 예시이므로 컴파일되지 않습니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-25/src/main.rs}}
```

<span class=\"caption\">Listing 18-25: `..`를 모호하게 사용하려는 시도</span>

이 코드를 컴파일하면 다음과 같은 오류가 발생합니다.

```console
{{#include ../listings/ch18-patterns-and-matching/listing-18-25/output.txt}}
```

Rust는 튜플에서 몇 개의 값을 무시해야 하는지 결정할 수 없습니다. `second`에 매칭하고 나머지 값을 무시하는지, 아니면 `second`에 매칭하고 나머지 값을 무시하는지 등 여러 가지 해석이 가능합니다. 변수 이름 `second`는 Rust에게 특별한 의미를 가지지 않기 때문에 `..`를 두 곳에서 사용하는 것은 모호하기 때문에 컴파일 오류가 발생합니다.

### Match Guard를 사용한 추가 조건

*match guard*는 `match` 팔레트에서 패턴 뒤에 지정된 추가적인 `if` 조건입니다. 이 조건도 선택될 때 만족해야 합니다. Match guard는 패턴만으로 표현할 수 없는 더 복잡한 아이디어를 표현하는 데 유용합니다.

조건은 패턴에서 생성된 변수를 사용할 수 있습니다. 18-26번 목록은 `match`에서 첫 번째 팔레트가 `Some(x)` 패턴과 `if x % 2 == 0` (숫자가 짝수인 경우)의 매치 가드를 가지고 있는 예입니다.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-26/src/main.rs:here}}
```

<span class=\"caption\">Listing 18-26: 패턴에 매치 가드 추가</span>

이 예제는 `The number 4 is even`을 출력합니다. `num`이 패턴과 비교될 때, `Some(4)`가 `Some(x)`에 맞기 때문에 매칭됩니다. 그런 다음 매치 가드는 `x`를 2로 나눈 나머지가 0인지 확인합니다. 0이므로 첫 번째 팔레트가 선택됩니다.

`num`이 `Some(5)`였더라면, 첫 번째 팔레트의 매치 가드는 5를 2로 나눈 나머지가 1이기 때문에 거짓이 됩니다. Rust는 매치 가드가 없는 두 번째 팔레트로 이동하여 `Some` 변형에 맞습니다.

패턴 내부에 `if x % 2 == 0` 조건을 표현하는 방법은 없습니다. 따라서 매치 가드는 이러한 논리를 표현할 수 있게 해줍니다. 추가적인 표현력의 단점은 컴파일러가 매치 가드 표현식이 포함될 때 완전성을 확인하지 않는다는 것입니다.

18-11번 목록에서 언급했듯이, 매치 가드를 사용하여 패턴-스키닝 문제를 해결할 수 있습니다. 패턴 표현식 내부에 새로운 변수를 만들었던 것을 기억하십시오. 이 새로운 변수는 `match` 외부의 변수에 대한 테스트를 할 수 없었습니다. 18-27번 목록은 매치 가드를 사용하여 이 문제를 해결하는 방법을 보여줍니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-27/src/main.rs}}
```

<span class=\"caption\">Listing 18-27: 매치 가드를 사용하여 외부 변수와의 등식을 테스트</span>

이 코드는 이제 `Default case, x = Some(5)`를 출력합니다. 두 번째 팔레트의 패턴은 `Some(x)`이고 매치 가드는 `x == 5`입니다. 이 조건이 참이므로 두 번째 팔레트가 선택됩니다.

match arm에서 `y`라는 새로운 변수를 선언하지 않기 때문에 외부의 `y`를 가리키는 변수를 덮어쓰지 않습니다.
즉, match guard에서 외부의 `y`를 사용할 수 있습니다. `Some(y)`와 같이 패턴을 지정하는 대신, `Some(n)`을 지정합니다. 이렇게 하면 `n`이라는 새로운 변수가 생성되며, `match` 외부에 `n`이라는 변수가 없기 때문에 어떤 것도 덮어쓰지 않습니다.

`if n == y`라는 match guard는 패턴이 아니기 때문에 새로운 변수를 선언하지 않습니다. 이 `y`는 외부의 `y`이며, `n`과 `y`를 비교하여 외부의 `y`와 같은 값을 가진 값을 찾을 수 있습니다.

또한 match guard에서 `|` 연산자를 사용하여 여러 패턴을 지정할 수 있습니다. match guard 조건은 모든 패턴에 적용됩니다. 18-28번 표는 `|`를 사용하는 패턴과 match guard를 결합할 때의 우선순위를 보여줍니다. 이 예제에서 중요한 부분은 `if y` match guard가 `4`, `5`, `6`에 모두 적용된다는 것입니다. 

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-28/src/main.rs:here}}
```

<span class=\"caption\">18-28번 표: match guard와 여러 패턴을 결합</span>

match 조건은 `x`의 값이 `4`, `5`, 또는 `6`이고 `y`가 `true`인 경우에만 해당하는 팔을 만듭니다. 이 코드가 실행되면 첫 번째 팔의 패턴이 `x`가 `4`이기 때문에 일치하지만, `if y` match guard가 `false`이기 때문에 첫 번째 팔은 선택되지 않습니다. 코드는 두 번째 팔로 이동하며, 이 팔이 일치하므로 프로그램은 `no`를 출력합니다. 이유는 `if` 조건이 `4 | 5 | 6` 전체 패턴에 적용되기 때문입니다. 즉, `|` 연산자로 지정된 값 목록의 마지막 값만 적용되는 것이 아니라, `if` 조건이 전체 패턴에 적용됩니다.

```text
(4 | 5 | 6) if y => ...
```

rather than this:

```text
4 | 5 | (6 if y) => ...
```

코드 실행 후 우선순위 동작이 명확해집니다. 만약 match guard가 `|` 연산자로 지정된 값 목록의 마지막 값에만 적용된다면, 팔이 일치했을 것이고 프로그램은 `yes`를 출력했을 것입니다.

### `@` 바인딩

`@` 연산자는 패턴 일치를 테스트하는 동시에 해당 값을 저장하는 변수를 만들 수 있도록 합니다. 18-29번 표에서는 `Message::Hello`의 `id` 필드가 `3..=7` 범위 내에 있는지 테스트하고, 동시에 해당 값을 `id_variable` 변수에 저장하고 싶습니다. 이 변수 이름은 `id`와 같을 수 있지만, 이 예제에서는 다른 이름을 사용합니다.

```rust
{{#rustdoc_include ../listings/ch18-patterns-and-matching/listing-18-29/src/main.rs:here}}
```

<span class=\"caption\">18-29번 표: 패턴에서 값을 바인딩하면서 동시에 테스트</span>

이 예제는 `Found an id in range: 5`를 출력합니다. `id_variable @`를 패턴 앞에 지정함으로써 `3..=7` 범위에 맞는 값을 저장하면서 동시에 해당 값을 테스트합니다.

두 번째 팔에서는 패턴에만 범위가 지정되어 있기 때문에 해당 팔의 코드에는 `id` 필드의 실제 값이 저장된 변수가 없습니다. `id` 필드의 값은 10, 11, 또는 12일 수 있지만, 해당 패턴의 코드는 어떤 값인지 알 수 없습니다. 패턴 코드는 `id` 필드의 값을 사용할 수 없기 때문입니다. 패턴 코드는 `id` 필드의 값을 저장하지 않았기 때문입니다.

마지막 팔에서는 변수만 지정되었기 때문에 `id` 변수에 `id` 필드의 값이 저장되어 있습니다. 해당 팔의 코드는 `id` 변수를 사용할 수 있습니다.

이유는 구조체 필드 간략 문법을 사용했기 때문입니다. 하지만 첫 번째 두 개의 팔과 같이 `id` 필드의 값에 대한 테스트를 이 팔에 적용하지 않았습니다. 어떤 값이든 이 패턴에 맞습니다.

`@`를 사용하면 값을 테스트하고 변수에 저장할 수 있습니다.

## 요약

Rust의 패턴은 다양한 종류의 데이터를 구분하는 데 매우 유용합니다. `match` 표현식에서 사용될 때 Rust는 패턴이 모든 가능한 값을 처리하도록 보장하거나 프로그램이 컴파일되지 않습니다. `let` 문과 함수 매개변수의 패턴은 이러한 구조를 더욱 유용하게 만들어 값을 더 작은 부분으로 분해하면서 동시에 변수에 할당할 수 있도록 합니다. 필요에 따라 간단하거나 복잡한 패턴을 만들 수 있습니다.

다음으로, 책의 마지막 두 장에서는 Rust의 다양한 기능의 고급 측면을 살펴보겠습니다