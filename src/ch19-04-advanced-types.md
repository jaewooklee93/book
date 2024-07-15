## 고급 타입

Rust 타입 시스템에는 아직 논의하지 않았지만, 앞으로 다룰 특징들이 있습니다. 먼저, 새로운 타입(newtype)의 유용성을 살펴보며 새로운 타입을 사용하는 이유를 살펴보겠습니다. 그런 다음 타입 별칭(type alias)으로 넘어가겠습니다. 타입 별칭은 새로운 타입과 유사하지만 의미가 약간 다릅니다. 또한 `!` 타입과 동적으로 크기가 조절되는 타입에 대해서도 논의하겠습니다.

### 타입 안전성과 추상화를 위한 새로운 타입 패턴 사용

>참고: 이 섹션은 이전 섹션 [\u201c외부 타입에 대한 외부 트레이트를 구현하기 위한 새로운 타입 패턴 사용\u201d][using-the-newtype-pattern]<!-- ignore --> 에 대해 읽은 것을 가정합니다.

새로운 타입 패턴은 이전에 논의한 것 외에도 유용합니다. 예를 들어, 값이 절대 혼동되지 않도록 정적으로 보장하고 값의 단위를 나타내는 데 사용할 수 있습니다. Listing 19-15에서 새로운 타입을 사용하여 단위를 나타내는 예를 보았습니다. `Millimeters` 와 `Meters` 구조체는 `u32` 값을 새로운 타입으로 감싸었습니다. 만약 `Millimeters` 타입의 매개변수를 가진 함수를 작성하면, `Meters` 타입 또는 단순히 `u32` 값으로 함수를 호출하려고 시도하는 프로그램을 컴파일할 수 없습니다.

또한 새로운 타입 패턴은 타입의 일부 구현 세부 사항을 추상화하는 데 사용할 수 있습니다. 새로운 타입은 내부 타입의 API와 다른 공개 API를 제공할 수 있습니다.

새로운 타입은 내부 구현을 숨길 수도 있습니다. 예를 들어, `People` 타입을 `HashMap<i32, String>`으로 감싸서 사람의 ID를 이름과 연결하여 저장할 수 있습니다. `People`을 사용하는 코드는 `People` 컬렉션에 이름 문자열을 추가하는 등 우리가 제공하는 공개 API와만 상호 작용할 것입니다. 해당 코드는 내부적으로 ID가 이름에 할당되는 것을 알 필요가 없습니다. 새로운 타입 패턴은 구현 세부 사항을 숨기기 위한 가벼운 방법이며, 이는 Chapter 17의 [\u201c구현 세부 사항을 숨기는 캡슐화\u201d][encapsulation-that-hides-implementation-details]<!-- ignore --> 섹션에서 논의했습니다.

### 타입 별칭으로 타입 동의어 만들기

Rust는 기존 타입에 다른 이름을 부여하기 위해 *타입 별칭*을 선언할 수 있습니다. 이를 위해 `type` 키워드를 사용합니다. 예를 들어, `Kilometers`라는 별칭을 `i32`에 만들 수 있습니다.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-04-kilometers-alias/src/main.rs:here}}
```

이제 `Kilometers`라는 별칭은 `i32`의 *동의어*입니다. `Millimeters`와 `Meters` 타입이 Listing 19-15에서 만든 것과 달리 `Kilometers`는 별도의 새로운 타입이 아닙니다. `Kilometers` 타입의 값은 `i32` 타입의 값과 동일하게 처리됩니다.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-04-kilometers-alias/src/main.rs:there}}
```

`Kilometers`와 `i32`가 동일한 타입이기 때문에 `Kilometers` 값과 `i32` 값을 모두 추가할 수 있으며 `Kilometers` 값을 `i32` 매개변수를 받는 함수에 전달할 수 있습니다. 그러나 이 방법을 사용하면 Listing 19-15에서 논의한 새로운 타입 패턴에서 얻는 타입 검사 이점을 얻을 수 없습니다. 즉, `Kilometers`와 `i32` 값을 어딘가에서 혼동하면 컴파일러는 오류를 알려주지 않습니다.

타입 동의어의 주요 사용 사례는 반복을 줄이는 것입니다. 예를 들어, 다음과 같은 긴 타입이 있을 수 있습니다.

```rust,ignore
Box<dyn Fn() + Send + 'static>
```

함수 서명과 타입 지정으로 이러한 긴 타입을 사용하는 것은 코드 전체에서 반복적이고 오류 발생 가능성이 높습니다. Listing 19-24와 같이 이러한 코드가 많은 프로젝트를 상상해 보세요.

```rust
{{#rustdoc_include ../listings/ch19-advanced-features/listing-19-24/src/main.rs:here}}
```

<span class=\"caption\">Listing 19-24: 여러 곳에서 긴 타입 사용</span>

타입 별칭은 `Thunk`라는 이름으로 이 긴 타입을 줄여서 코드를 더 관리하기 쉽게 합니다. Listing 19-25에서 `Thunk`라는 별칭을 도입하여 긴 타입을 대체하여 모든 사용을 짧은 별칭 `Thunk`로 바꿀 수 있습니다.

```rust
```

<span class=\"caption\">Listing 19-25: `Thunk` 타입 별칭을 사용하여 반복을 줄이기</span>

이 코드는 읽고 작성하는 데 훨씬 쉬워집니다! 유의미한 이름을 타입 별칭에 지정하면 의도를 효과적으로 전달하는 데 도움이 됩니다 (*thunk*는 나중에 평가될 코드를 나타내는 단어이므로 닫힘이 저장되는 타입에 적절한 이름입니다).

타입 별칭은 `Result<T, E>` 타입과 함께 반복을 줄이기 위해 일반적으로 사용됩니다. 표준 라이브러리의 `std::io` 모듈을 생각해 보세요. I/O 작업은 종종 작업이 실패할 때 처리하기 위해 `Result<T, E>`를 반환합니다. 이 라이브러리에는 `std::io::Error` 구조체가 모든 가능한 I/O 오류를 나타냅니다. `std::io`의 많은 함수는 `Result<T, E>`를 반환하며 `E`가 `std::io::Error`로 채워집니다. 예를 들어 `Write` 트레이트의 함수는 다음과 같습니다.

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-05-write-trait/src/lib.rs}}
```

`Result<..., Error>`가 반복적으로 사용됩니다. 따라서 `std::io`는 다음과 같은 타입 별칭 선언을 가지고 있습니다.

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-06-result-alias/src/lib.rs:here}}
```

이 선언이 `std::io` 모듈에 있기 때문에 `std::io::Result<T>`라는 완전한 별칭을 사용할 수 있습니다. 즉, `E`가 `std::io::Error`로 채워진 `Result<T, E>`입니다. `Write` 트레이트 함수 선언은 다음과 같이 보입니다.

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-06-result-alias/src/lib.rs:there}}
```

타입 별칭은 코드 작성을 쉽게 하고 일관된 인터페이스를 제공하는 데 도움이 됩니다. 별칭이기 때문에 `Result<T, E>`와 동일하므로 `Result<T, E>`에 대해 작동하는 모든 메서드와 `?` 연산자와 같은 특수 문법을 사용할 수 있습니다.

### 절대 돌아오지 않는 `Never` 타입

Rust에는 `!`라는 특수 타입이 있으며, 타입 이론 용어로는 *빈 타입*이라고 불립니다. 이 타입은 값이 없기 때문에 빈 타입입니다. 우리는 *절대 타입*이라고 부르는 것을 선호합니다. 이 타입은 함수가 절대 돌아오지 않을 때 반환 타입을 나타냅니다. 예를 들어 보세요.

```rust,noplayground
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-07-never-type/src/lib.rs:here}}
```

이 코드는 \u201c함수 `bar`는 절대 돌아오지 않습니다\u201d라고 읽습니다. 절대 돌아오지 않는 함수는 *발산 함수*라고 합니다. `!` 타입의 값을 만들 수 없으므로 `bar`는 절대 돌아올 수 없습니다.

하지만 값을 만들 수 없는 타입은 어떤 용도가 있을까요? 2장의 코드에서 숫자 맞추기 게임을 살펴보았습니다. 19장의 코드에서 다시 살펴보겠습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:ch19}}
```

<span class=\"caption\">Listing 19-26: `continue` 문으로 끝나는 `match`</span>

당시에는 이 코드의 몇 가지 세부 사항을 건너뜁니다. 6장의 \u201cThe `match` Control Flow Operator\u201d 섹션에서 `match` 팔레트는 모두 동일한 타입을 반환해야 한다고 설명했습니다. 따라서 다음과 같은 코드는 작동하지 않습니다.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-08-match-arms-different-types/src/main.rs:here}}
```

이 코드의 `guess` 유형은 정수이자 문자열이어야 하며, Rust은 `guess`가 하나의 유형만 가져야 한다고 요구합니다. 그렇다면 `continue`는 무엇을 반환하는가? 19-26번 목록에서 `u32`를 하나의 팔에서 반환하고 `continue`로 끝나는 다른 팔이 있었던 것은 어떻게 허용되었는가?

짐작하셨을 것 같습니다. `continue`는 `!` 값을 가지고 있습니다. 즉, Rust가 `guess`의 유형을 계산할 때, 전자는 `u32` 값을 가지고 있는 `match` 팔과 후자는 `!` 값을 가지고 있는 `match` 팔을 모두 살펴봅니다. `!`는 값을 가질 수 없기 때문에 Rust는 `guess`의 유형이 `u32`라고 결정합니다.

이러한 동작을 공식적으로 설명하면 `!` 유형의 표현식은 다른 모든 유형으로 강제 변환될 수 있다는 것입니다. 우리는 `continue`로 이 `match` 팔을 끝낼 수 있는 이유는 `continue`가 값을 반환하지 않기 때문입니다. 대신, 제어를 루프의 맨 위로 되돌려서 `Err` 경우에는 `guess`에 값을 할당하지 않습니다.

`never` 유형은 `panic!` 매크로와 함께 유용합니다. 우리가 `Option<T>` 값에 대해 호출하는 `unwrap` 함수를 기억하세요. 이 함수는 값을 생성하거나 다음과 같은 정의로 panic을 일으킵니다.

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-09-unwrap-definition/src/lib.rs:here}}
```

이 코드에서 `match`에서 19-26번 목록과 같은 일이 발생합니다. Rust는 `val`가 유형 `T`이고 `panic!`가 유형 `!`이므로 전체 `match` 표현식의 결과가 `T`임을 알 수 있습니다. 이 코드가 작동하는 이유는 `panic!`가 값을 생성하지 않기 때문입니다. 프로그램을 종료합니다. `None` 경우에는 `unwrap`에서 값을 반환하지 않으므로 이 코드는 유효합니다.

마지막으로 `!` 유형을 가진 표현식은 `loop`입니다.

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-10-loop-returns-never/src/main.rs:here}}
```

여기서 루프는 끝나지 않으므로 `!`가 표현식의 값입니다. 그러나 `break`를 포함하면 이것은 사실이 아닙니다. 루프가 `break`에 도달하면 종료되기 때문입니다.

### 동적 크기 유형과 `Sized` 트레이트

Rust는 유형의 특정 세부 사항을 알아야 합니다. 예를 들어, 특정 유형의 값에 할당할 메모리 공간의 크기를 알아야 합니다. 이로 인해 유형 시스템의 한 구석이 처음에는 약간 혼란스러울 수 있습니다. 동적 크기 유형이라는 개념입니다.

때로는 DST 또는 불규칙 크기 유형이라고도 하며, 이 유형은 런타임에만 알 수 있는 크기를 가진 값을 작성할 수 있도록 합니다.

이제까지 책에서 사용해 온 `str`이라는 동적 크기 유형에 대해 자세히 살펴보겠습니다. 바로 `str` 자체입니다. `&str`이 아니라 `str`입니다. 우리는 문자열의 길이를 런타임에만 알 수 있기 때문에 `str` 유형의 변수를 만들 수도 없고 `str` 유형의 인수를 받을 수도 없습니다. 다음과 같은 코드를 고려해 보세요. 이 코드는 작동하지 않습니다.

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-11-cant-create-str/src/main.rs:here}}
```

Rust는 특정 유형의 모든 값에 대해 메모리 할당량을 알아야 하며, 모든 유형의 값은 동일한 메모리 공간을 사용해야 합니다. Rust가 이 코드를 허용하면 `s1`과 `s2` 두 개의 `str` 값은 동일한 메모리 공간을 차지해야 합니다. 하지만 그들은 다른 길이를 가지고 있습니다. `s1`은 12바이트의 메모리를 사용해야 하며 `s2`는 15바이트를 사용해야 합니다. 이것이 `str` 유형의 변수를 만들 수 없는 이유입니다.

그렇다면 어떻게 해야 할까요? 이 경우, 이미 답을 알고 계시죠. 유형을 `&str`으로 만듭니다. 4장의 [\u201c문자열 슬라이스\u201d][string-slices]<!-- ignore --> 섹션에서 기억하시겠지만 슬라이스 데이터 구조는 단지 슬라이스의 시작 위치와 길이만 저장합니다. 따라서 `&T`는 특정 유형의 메모리 주소를 저장하는 단일 값입니다. 따라서 `&str`은 문자열의 길이를 알 수 있기 때문에 `str`과 달리 동적 크기 유형이 아닙니다.


 `T` 가 위치한 곳에서 `&str`은 *두* 값입니다: `str`의 주소와 길이입니다. 따라서 컴파일 시간에 `&str` 값의 크기를 알 수 있습니다: `usize`의 길이의 두 배입니다. 즉, `&str`의 크기는 참조하는 문자열의 길이와 상관없이 항상 알 수 있습니다. 일반적으로 동적으로 크기가 조절되는 유형이 Rust에서 사용되는 방식입니다. 동적으로 크기가 조절되는 유형은 동적 정보의 크기를 저장하는 추가 메타데이터가 있습니다. 동적으로 크기가 조절되는 유형의 황금 규칙은 항상 어떤 종류의 포인터 뒤에 동적으로 크기가 조절되는 유형의 값을 넣어야 한다는 것입니다. 

`str`을 모든 종류의 포인터와 결합할 수 있습니다. 예를 들어 `Box<str>` 또는 `Rc<str>`입니다. 사실, 이전에 다른 동적으로 크기가 조절되는 유형과 함께 본 적이 있지만, 추상화된 개념은 동일합니다. 추상화된 개념은 모든 동적으로 크기가 조절되는 유형과 동일합니다. 모든 추상화된 개념은 추상화된 개념의 이름을 사용하여 참조할 수 있는 동적으로 크기가 조절되는 유형입니다. 제17장의 [\u201c다양한 유형의 값을 허용하는 추상화된 개념 객체를 사용하는 방법\u201d][using-trait-objects-that-allow-for-values-of-different-types]<!--
ignore --> 섹션에서 추상화된 개념을 추상화된 개념 객체로 사용하려면 `&dyn Trait` 또는 `Box<dyn Trait>` (`Rc<dyn Trait>`도 작동합니다)와 같은 포인터 뒤에 넣어야 한다고 언급했습니다. 

Rust는 유형의 크기가 컴파일 시간에 알려져 있는지 여부를 확인하기 위해 `Sized` 추상화된 개념을 제공합니다. 이 추상화된 개념은 컴파일 시간에 크기가 알려진 모든 것에 대해 자동으로 구현됩니다. 또한 Rust는 모든 일반적인 함수에 `Sized`에 대한 암묵적인 경계를 추가합니다. 즉, 다음과 같은 일반적인 함수 정의는 다음과 같이 작성한 것으로 간주됩니다. 

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-12-generic-fn-definition/src/lib.rs}}
```

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-13-generic-implicit-sized-bound/src/lib.rs}}
```

 기본적으로 일반적인 함수는 컴파일 시간에 크기가 알려진 유형에서만 작동합니다. 그러나 다음과 같은 특수한 구문을 사용하여 이 제한을 완화할 수 있습니다. 

```rust,ignore
{{#rustdoc_include ../listings/ch19-advanced-features/no-listing-14-generic-maybe-sized/src/lib.rs}}
```

`?Sized` 추상화된 개념 경계는 \u201c`T`는 `Sized`일 수도 있고 없을 수도 있습니다\u201d를 의미하며, 이 구문은 일반적인 유형이 컴파일 시간에 크기가 알려져 있어야 한다는 기본값을 무시합니다. 이러한 의미의 `?Trait` 구문은 `Sized`에만 사용할 수 있으며 다른 추상화된 개념에는 사용할 수 없습니다. 

 또한 `t` 매개변수의 유형을 `T`에서 `&T`로 변경했다는 점에 주목하십시오. 유형이 `Sized`가 아니기 때문에 어떤 종류의 포인터 뒤에 넣어야 합니다. 이 경우에는 참조를 선택했습니다. 

 다음으로 함수와 폐쇄를 다룰 것입니다! 

[encapsulation-that-hides-implementation-details]:
ch17-01-what-is-oo.html#encapsulation-that-hides-implementation-details
[string-slices]: ch04-03-slices.html#string-slices
[the-match-control-flow-operator]:
ch06-02-match.html#the-match-control-flow-operator
[using-trait-objects-that-allow-for-values-of-different-types]:
ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[using-the-newtype-pattern]: ch19-03-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types
