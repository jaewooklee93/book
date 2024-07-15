## 객체지향 디자인 패턴 구현

*상태 패턴*은 객체지향 디자인 패턴입니다. 패턴의 핵심은 값이 내부적으로 가질 수 있는 상태의 집합을 정의하는 것입니다. 상태는 *상태 객체*의 집합으로 표현되며, 값의 동작은 상태에 따라 변경됩니다. 블로그 게시물 구조체의 예를 통해 상태 패턴을 이해해 보겠습니다. 이 구조체는 상태를 저장하는 필드를 가지며, 상태는 "초안게시물의 상태를 나타내는 `State` 트레이트를 정의합니다.

`Post`는 `Box<dyn State>`의 트레이트 객체를 `Option<T>` 안에 갖고 있는 `state`라는 프라이빗 필드에 저장합니다. 이유는 곧 알아갈 수 있습니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-12/src/lib.rs}}
```

<span class=\"caption\">Listing 17-12: `Post` 구조체와 `new` 함수의 정의, `State` 트레이트, 그리고 `Draft` 구조체</span>

`State` 트레이트는 다양한 게시물 상태가 공유하는 동작을 정의합니다. 상태 객체는 `Draft`, `PendingReview`, `Published`이며, 모두 `State` 트레이트를 구현합니다. 현재 트레이트에는 메서드가 없으며, `Draft` 상태만 정의하여 시작합니다. 게시물이 시작할 때 상태가 `Draft`가 되도록 합니다.

`Post`를 새로 생성할 때 `state` 필드를 `Some` 값으로 설정하여 `Box`를 갖습니다. 이 `Box`는 `Draft` 구조체의 새 인스턴스를 가리킵니다. 이렇게 하면 새 `Post` 인스턴스를 생성할 때마다 `Draft` 상태로 시작합니다. `Post`의 `state` 필드가 프라이빗이기 때문에 다른 상태로 `Post`를 생성할 수 없습니다! `Post::new` 함수에서 `content` 필드를 새로 생성된 `String`으로 설정합니다.

### 게시물 콘텐츠 텍스트 저장

Listing 17-11에서 보았듯이 `add_text`라는 메서드를 호출하여 `&str`을 전달하고 이것이 블로그 게시물의 텍스트 콘텐츠로 추가되도록 하려고 합니다. 이를 구현하는 것은 `content` 필드를 `pub`으로 노출하는 것보다 메서드로 구현하는 것이 좋습니다. 이렇게 하면 나중에 `content` 필드의 데이터가 읽는 방식을 제어하는 메서드를 구현할 수 있습니다. `add_text` 메서드는 매우 직관적이므로 Listing 17-13에 `impl Post` 블록에 구현을 추가합니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-13/src/lib.rs:here}}
```

<span class=\"caption\">Listing 17-13: `add_text` 메서드를 구현하여 게시물의 텍스트 콘텐츠에 텍스트를 추가합니다</span>

`add_text` 메서드는 `self`에 대한 가변 참조를 가져야 합니다. `add_text`를 호출하는 `Post` 인스턴스를 변경하기 때문입니다. `content`에 있는 `String`에 `push_str`을 호출하고 `text` 인수를 전달하여 저장된 `content`에 추가합니다. 이 동작은 게시물의 상태에 관계없으므로 상태 패턴의 일부가 아닙니다. `add_text` 메서드는 `state` 필드와 상호 작용하지 않습니다. 그러나 이는 우리가 지원하고자 하는 동작의 일부입니다.

### 게시물 초안 콘텐츠를 빈 문자열로 유지하는 것

`add_text`를 호출하고 게시물에 텍스트를 추가한 후에도, 게시물이 여전히 초안 상태이기 때문에 `content` 메서드가 빈 문자열 슬라이스를 반환해야 합니다. Listing 17-11의 7행과 같습니다. 현재 상태에서는 게시물이 `Draft` 상태일 때만 콘텐츠가 빈 문자열 슬라이스로 반환되어야 하므로, `content` 메서드를 구현하는 가장 간단한 방법은 항상 빈 문자열 슬라이스를 반환하는 것입니다. 이를 나중에 게시물의 상태를 변경하여 게시물을 게시할 수 있게 구현하면 변경할 수 있습니다. 현재까지 게시물은 `Draft` 상태만 있으므로 게시물 콘텐츠는 항상 빈 문자열이어야 합니다. Listing 17-14에 이 플레이스홀더 구현이 있습니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-14/src/lib.rs:here}}
```

<span class=\"caption\">Listing 17-14: `Post`의 `content` 메서드에 항상 빈 문자열 슬라이스를 반환하는 플레이스홀더 구현을 추가합니다</span>

## 게시글 상태 패턴 추가

이전에 추가한 `content` 메서드를 사용하면 Listing 17-11의 7행까지 원하는 방식으로 작동합니다.

### 게시글 검토 요청은 게시글 상태를 변경합니다

다음으로, 게시글 검토를 요청하는 기능을 추가해야 합니다. 이는 게시글 상태를 `Draft`에서 `PendingReview`로 변경해야 합니다. Listing 17-15에 이 코드가 나와 있습니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-15/src/lib.rs:here}}
```

<span class=\"caption\">Listing 17-15: `Post`에 `request_review` 메서드와 `State` 트레이트 구현</span>

`Post`에 `request_review`이라는 공개 메서드를 추가합니다. 이 메서드는 `self`에 대한 변경 가능한 참조를 받습니다. 그런 다음 현재 상태의 `request_review` 메서드를 호출하고, 이 두 번째 `request_review` 메서드는 현재 상태를 소비하여 새로운 상태를 반환합니다.

`State` 트레이트에 `request_review` 메서드를 추가합니다. 이 트레이트를 구현하는 모든 유형은 이제 `request_review` 메서드를 구현해야 합니다. `self`, `&self`, 또는 `&mut self`를 첫 번째 매개변수로 사용하는 대신 `self: Box<Self>`를 사용합니다. 이 구문은 메서드가 `Box`를 갖는 유형에만 유효하다는 것을 의미합니다. 이 구문은 `Box<Self>`의 소유권을 가져와서 이전 상태를 무효화하여 `Post`의 상태 값이 새로운 상태로 변환될 수 있도록 합니다.

오래된 상태를 소비하려면 `request_review` 메서드가 상태 값의 소유권을 가져야 합니다. 이때 `Post`의 `state` 필드의 `Option`이 유용하게 사용됩니다. `take` 메서드를 호출하여 `state` 필드에서 `Some` 값을 가져와서 `None`을 남겨둡니다. Rust에서는 구조체 필드가 비어 있을 수 없기 때문에 이렇게 합니다. 이를 통해 `Post`에서 상태 값을 이동시키는 것이 아니라 대여하는 것을 방지합니다. 그런 다음 `Post`의 `state` 값을 이 작업의 결과로 설정합니다.

`state` 값을 `None`으로 일시적으로 설정하는 것은 `self.state = self.state.request_review();`와 같은 코드로 직접 설정하는 것보다 좋습니다. 이렇게 하면 `Post`가 상태 값을 변환한 후에도 이전 상태 값을 사용할 수 없도록 합니다.

`Draft`의 `request_review` 메서드는 새로운 `PendingReview` 구조체의 박스 인스턴스를 반환합니다. 이 구조체는 게시글이 검토를 기다리는 상태를 나타냅니다. `PendingReview` 구조체는 `request_review` 메서드를 구현하지만, 어떤 변환도 수행하지 않습니다. 대신, 현재 상태를 반환합니다. 이는 이미 `PendingReview` 상태인 게시글에 검토를 요청하면 `PendingReview` 상태로 유지되어야 하기 때문입니다.

이제 상태 패턴의 장점을 볼 수 있습니다. `Post`의 `request_review` 메서드는 현재 상태에 관계없이 동일합니다. 각 상태는 자신의 규칙을 담당합니다.

`content` 메서드는 `Post`에서 그대로 유지하고, 빈 문자열 슬라이스를 반환합니다. 이제 `PendingReview` 상태뿐만 아니라 `Draft` 상태에서도 `Post`를 가질 수 있습니다. Listing 17-11은 이제 10행까지 작동합니다!

<!-- Old headings. Do not remove or links may break. -->
<a id=\"adding-the-approve-method-that-changes-the-behavior-of-content\"></a>

### `approve` 추가: `content`의 동작 변경

`approve` 메서드는 `request_review` 메서드와 유사합니다. 현재 상태가 승인될 때 가져야 하는 상태 값을 `state`로 설정합니다. Listing 17-16에 이 코드가 나와 있습니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-16/src/lib.rs:here}}
```

<span class=\"caption\">Listing 17-16: `Post`에 `approve` 메서드와 `State` 트레이트 구현</span>

우리는 `State` 트레이트에 `approve` 메서드를 추가하고, `State`를 구현하는 새로운 구조체인 `Published` 상태를 추가합니다.

`PendingReview`에서 `request_review`와 유사하게, `Draft`에서 `approve` 메서드를 호출하면 효과가 없기 때문에 `approve`는 `self`를 반환합니다. `PendingReview`에서 `approve`를 호출하면 `Published` 구조체의 새 박스 인스턴스가 반환됩니다. `Published` 구조체는 `State` 트레이트를 구현하며, `request_review` 메서드와 `approve` 메서드 모두 `self`를 반환합니다. 이는 게시물이 이러한 경우에 `Published` 상태로 유지되어야 하기 때문입니다.

이제 `Post`의 `content` 메서드를 업데이트해야 합니다. `content`에서 반환되는 값이 `Post`의 현재 상태에 따라 달라지도록 하기 위해 `Post`가 `state`에 정의된 `content` 메서드로 위임하도록 합니다. Listing 17-17을 참조하세요.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-oop/listing-17-17/src/lib.rs:here}}
```

<span class=\"caption\">Listing 17-17: `Post`의 `content` 메서드를 업데이트하여 `State`에 정의된 `content` 메서드로 위임</span>

목표는 `State`를 구현하는 구조체 내부에 모든 규칙을 유지하는 것이므로 `state`의 값에서 `content` 메서드를 호출하고 `post` 인스턴스(즉, `self`)를 인수로 전달합니다. 그런 다음 `state`의 값을 사용하여 `content` 메서드를 호출한 결과를 반환합니다.

`Option`에서 참조를 가져오려면 `as_ref` 메서드를 호출합니다. `state`가 `Option<Box<dyn State>>`이기 때문에 `as_ref`를 호출하면 `Option<&Box<dyn State>>`가 반환됩니다. `as_ref`를 호출하지 않으면 `state`를 함수 매개변수의 빌려진 `&self`에서 움직일 수 없기 때문에 오류가 발생합니다.

그런 다음 `unwrap` 메서드를 호출합니다. 이 메서드는 `None` 값이 절대 발생하지 않기 때문에 예외가 발생하지 않습니다. 이는 Chapter 9의 "컴파일러보다 더 많은 정보를 가지고 있는 경우"<!-- ignore --> 섹션에서 언급한 것과 같습니다. 컴파일러가 이해할 수 없더라도 `None` 값이 불가능하다는 것을 알고 있기 때문입니다.

이제 `&Box<dyn State>`에서 `content`를 호출하면 `&`와 `Box`에 대한 디레프 암시적 변환이 발생하여 `content` 메서드가 마침내 `State` 트레이트를 구현하는 유형에 호출됩니다. 즉, `State` 트레이트 정의에 `content`를 추가해야 하며, Listing 17-18에 표시된 것처럼 현재 상태에 따라 어떤 콘텐츠를 반환해야 하는지에 대한 논리를 넣어야 합니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-18/src/lib.rs:here}}
```

<span class=\"caption\">Listing 17-18: `State` 트레이트에 `content` 메서드 추가</span>

`content` 메서드에 대한 기본 구현을 추가하여 빈 문자열 슬라이스를 반환합니다. 즉, `Draft`와 `PendingReview` 구조체에서 `content`를 구현할 필요가 없습니다. `Published` 구조체는 `content` 메서드를 재정의하여 `post.content`의 값을 반환합니다.

이 메서드에 대한 라이프타임 어노테이션이 필요하다는 점에 유의하세요. Chapter 10에서 논의했듯이, `post`에 대한 참조를 가져오고 `post`의 일부에 대한 참조를 반환하기 때문에 반환된 참조의 라이프타임은 `post` 인수의 라이프타임과 관련이 있습니다.

이제 Listing 17-11이 모두 작동합니다! 상태 패턴을 구현하고 블로그 게시물 워크플로우의 규칙을 구현했습니다. 모든 논리는 상태를 구현하는 구조체 내부에 있습니다.

규칙은 `Post` 전체에 흩어져 있는 것이 아니라 상태 객체에 저장됩니다.

> #### enum을 왜 사용하지 않았을까요?
>
> 여러분은 왜 `enum`을 사용하지 않았는지 궁금해하셨을 것입니다. 다양한
> 가능한 게시물 상태를 변형으로 가진 `enum`을 사용하는 것은 분명히 가능한
> 해결책입니다. 해당 방법을 시도해보고 결과를 비교하여 어떤 방법을 선호하는지
> 확인해보세요! `enum`을 사용하는 단점 중 하나는 상태 값을 확인하는 모든
> 위치에서 `match` 표현식이나 유사한 것을 사용해야 한다는 것입니다. 이는
> 이 트레이트 객체 솔루션보다 반복적인 작업이 될 수 있습니다.

### 상태 패턴의 장단점

우리는 Rust가 객체 지향 상태 패턴을 구현하여 게시물이 각 상태에서 가져야 하는
다양한 동작을 캡슐화할 수 있다는 것을 보여주었습니다. `Post`의 메서드는
다양한 동작에 대해 아무것도 모릅니다. 우리가 코드를 구성한 방식에서는
게시물이 게시된 상태에서 어떻게 동작할 수 있는지 알고 싶다면 `State` 트레이트의
`Published` 구조체에 대한 구현만 살펴봐야 합니다.

대안으로 상태 패턴을 사용하지 않는 경우, `Post`의 메서드에서 `match` 표현식을 사용하거나
`main` 코드에서 게시물의 상태를 확인하고 동작을 변경하는 경우가 있습니다.
그렇다면 게시물이 게시된 상태라는 것을 이해하기 위해 여러 곳을 살펴봐야 합니다!
더 많은 상태를 추가할수록 이러한 `match` 표현식이 필요해지며, 각 `match` 표현식은
또 다른 팔을 필요로 합니다.

상태 패턴을 사용하면 `Post` 메서드와 `Post`를 사용하는 위치에서 `match` 표현식이
필요하지 않습니다. 새로운 상태를 추가하려면 새로운 구조체를 추가하고 해당 구조체의
트레이트 메서드를 구현하는 것만으로도 충분합니다.

상태 패턴을 사용하는 구현은 더 많은 기능을 추가하기 쉽습니다. 상태 패턴을 사용하는
코드를 유지하는 간편성을 확인하려면 다음과 같은 몇 가지 제안을 시도해보세요.

* `reject` 메서드를 추가하여 게시물의 상태를 `PendingReview`에서 `Draft`로
  돌려보냅니다.
* `approve`를 두 번 호출해야만 상태가 `Published`로 변경될 수 있도록 합니다.
* 사용자가 게시물이 `Draft` 상태일 때만 텍스트 콘텐츠를 추가할 수 있도록 합니다.
  힌트: 상태 객체가 콘텐츠에 대한 변경 사항을 담당하지만 `Post`를 수정하지 않습니다.

상태 패턴의 단점 중 하나는 상태가 상태 간의 전환을 구현하기 때문에
일부 상태가 서로 연결되어 있다는 것입니다. `PendingReview`와 `Published` 사이에
`Scheduled`과 같은 새로운 상태를 추가하면 `PendingReview`에서 `Scheduled`로
전환하는 코드를 변경해야 합니다. 새로운 상태가 추가될 때 `PendingReview`가
변경되지 않도록 하는 것이 더 쉬웠겠지만, 그렇게 하려면 다른 디자인 패턴을
사용해야 합니다.

또 다른 단점은 일부 논리의 중복입니다. 중복을 제거하려면 `State` 트레이트의
`request_review` 및 `approve` 메서드에 `self`를 반환하는 기본 구현을 정의할 수
있습니다. 그러나 이렇게 하면 객체 안전성을 위반하게 됩니다. 트레이트가
정확히 어떤 구체적인 `self`가 될지 모르기 때문입니다. `State`를 트레이트 객체로
사용하려면 메서드가 객체 안전해야 합니다.

다른 중복은 `Post`의 `request_review` 및 `approve` 메서드의 유사한 구현입니다.
두 메서드 모두 `Option`의 `state` 필드의 값에 대한 동일한 메서드의 구현을
위임하고 새로운 `state` 필드 값을 설정합니다. `Post`에 이러한 패턴을 따르는
메서드가 많다면 중복을 제거하기 위해 매크로를 정의하는 것을 고려할 수 있습니다.
(19장의 [\u201c매크로\u201d][macros]<!-- ignore --> 섹션을 참조하세요).

상태 패턴을 객체 지향 언어에서 정의된 것처럼 정확하게 구현하면
Rust의 강점을 최대한 활용하지 못합니다. 다음은 `blog` crate에 대한 몇 가지 변경 사항을
보여드리며, 잘못된 상태와 전환을 컴파일 시간 오류로 만들 수 있습니다.

#### 상태와 동작을 유형으로 인코딩

17-11 목록의 `main`의 첫 번째 부분을 고려해 보겠습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch17-oop/listing-17-11/src/main.rs:here}}
```

`Post::new`을 사용하여 여전히 초안 상태의 새 게시물을 만들 수 있으며, 게시물의 내용에 텍스트를 추가할 수 있습니다. 하지만 초안 게시물의 `content` 메서드가 빈 문자열을 반환하는 대신, 초안 게시물에는 `content` 메서드가 전혀 없도록 하겠습니다. 그렇게 하면 초안 게시물의 내용을 가져하려고 하면 `content` 메서드가 존재하지 않음을 알리는 컴파일러 오류가 발생합니다. 결과적으로, 컴파일러 오류가 발생하기 때문에 생산에서 초안 게시물 내용을 실수로 표시할 수 없습니다.
17-19번 목록은 `Post` 구조체와 `DraftPost` 구조체의 정의, 그리고 각 구조체의 메서드를 보여줍니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-19/src/lib.rs}}
```

<span class=\"caption\">Listing 17-19: `content` 메서드가 있는 `Post`와 `content` 메서드가 없는 `DraftPost`</span>

`Post`와 `DraftPost` 구조체 모두 게시물 텍스트를 저장하는 `content` 필드를 가지고 있습니다. 구조체에는 더 이상 `state` 필드가 없으며, 구조체의 유형으로 상태를 인코딩하고 있습니다. `Post` 구조체는 게시된 게시물을 나타내며, `content` 메서드를 통해 `content`를 반환합니다.

`Post::new` 함수는 여전히 있지만, `Post` 인스턴스를 반환하는 대신 `DraftPost` 인스턴스를 반환합니다. `content`가 private이며 `Post`를 반환하는 함수가 없기 때문에, 지금은 `Post` 인스턴스를 직접 만들 수 없습니다.

`DraftPost` 구조체에는 `add_text` 메서드가 있으므로 `content`에 텍스트를 추가할 수 있습니다. 그러나 `DraftPost`에는 `content` 메서드가 정의되지 않았다는 점에 유의하세요! 따라서 프로그램은 모든 게시물이 초안 게시물로 시작하고, 초안 게시물의 내용은 표시할 수 없습니다. 이러한 제약 조건을 우회하려는 모든 시도는 컴파일러 오류로 이어집니다.

#### 유형 시스템으로 변환된 상태를 구현하는 방법

그렇다면 게시된 게시물을 어떻게 얻을 수 있을까요? 우리는 초안 게시물이 검토 및 승인을 받아야만 게시될 수 있다는 규칙을 강제해야 합니다. 검토 중인 게시물 상태의 게시물은 여전히 내용을 표시해서는 안 됩니다. `PendingReviewPost`라는 또 다른 구조체를 추가하여 `DraftPost`에서 `request_review` 메서드를 호출하여 `PendingReviewPost`를 생성하고, `PendingReviewPost`에서 `approve` 메서드를 호출하여 `Post`로 변환하는 방법을 구현해 보겠습니다. 17-20번 목록을 참조하세요.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-20/src/lib.rs:here}}
```

<span class=\"caption\">Listing 17-20: `DraftPost`에서 `request_review`를 호출하여 `PendingReviewPost`를 생성하고, `PendingReviewPost`에서 `approve` 메서드를 호출하여 게시된 `Post`로 변환하는 방법</span>

`request_review`와 `approve` 메서드는 `self`를 소유하므로 `DraftPost`와 `PendingReviewPost` 인스턴스를 소유하고 `PendingReviewPost`와 게시된 `Post`로 변환합니다. 이렇게 하면 `request_review`를 호출한 후 `DraftPost` 인스턴스가 남지 않고, 마찬가지로 `PendingReviewPost` 인스턴스가 남지 않습니다. `PendingReviewPost` 구조체에는 `content` 메서드가 정의되지 않았으므로, `PendingReviewPost`의 내용을 읽으려고 하면 컴파일러 오류가 발생합니다. `DraftPost`와 마찬가지로.
`approve` 메서드를 `PendingReviewPost`에 호출하여 `content` 메서드가 정의된 게시된 `Post` 인스턴스를 얻을 수 있는 유일한 방법이기 때문에, `Post::new` 함수를 통해 `DraftPost`를 생성하고, `request_review` 메서드를 통해 `PendingReviewPost`를 생성하고, `approve` 메서드를 통해 `Post`를 생성하는 유일한 방법이므로, 이제 블로그 게시물 워크플로우를 유형 시스템에 인코딩했습니다.

하지만 `main`에서도 몇 가지 작은 변경 사항이 필요합니다. `request_review` 및
`approve` 메서드는 호출된 구조체를 수정하는 대신 새 인스턴스를 반환하므로,
더 많은 `let post =` 그림자 할당을 추가하여 반환된 인스턴스를 저장해야 합니다.
또한 초안 및 검토 중 게시물의 내용에 대한 주장이 빈 문자열이거나 필요하지
않다는 점도 알아야 합니다. 이제 더 이상 이러한 상태의 게시물의 내용을
사용하려고 하는 코드를 컴파일할 수 없습니다.
`main`에서 업데이트된 코드는 17-21번 목록에 나와 있습니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch17-oop/listing-17-21/src/main.rs}}
```

<span class=\"caption\">17-21번 목록: `blog` 작업 흐름의 새 구현을 사용하기 위한 `main`의
변경 사항</span>

`post`를 재할당하기 위해 `main`에서 수행해야 하는 변경 사항은 이
구현이 객체 지향 상태 패턴을 완벽하게 따르지 않는다는 것을 의미합니다.
상태 간의 변환이 `Post` 구현 내부에서 완전히 캡슐화되지 않습니다.
그러나 유효하지 않은 상태를 컴파일 시점의 타입 체크와 타입 시스템 덕분에
이제 불가능하게 되었습니다! 이는 초안 게시물의 내용을 표시하는
과 같은 특정 오류가 생산에 도달하기 전에 발견되는 것을 보장합니다.

이 책의 17-21번 목록 이후 `blog` crate에서 시작 부분에 제시된 작업을 시도해
보세요. 이 버전의 코드의 디자인에 대해 어떻게 생각하는지 확인하십시오.
일부 작업은 이미 이 디자인에서 완료되었을 수 있습니다.

이제까지 Rust에서 객체 지향 디자인 패턴을 구현할 수 있다는 것을
보았습니다. 그러나 Rust에서 객체 지향 패턴과 같은 다른 패턴, 예를 들어
타입 시스템에 상태를 인코딩하는 패턴도 사용할 수 있습니다. 이러한
패턴은 다른 장단점을 가지고 있습니다. 객체 지향 패턴에 익숙하더라도
Rust의 기능을 활용하기 위해 문제를 다시 생각하면 컴파일 시점에
일부 오류를 방지하는 등의 이점을 얻을 수 있습니다. 소유권과 같은
특정 기능은 객체 지향 언어에서 없는 Rust의 특징입니다. 따라서
객체 지향 패턴이 항상 Rust의 강점을 활용하는 가장 좋은 방법은 아닙니다.
하지만 사용 가능한 옵션입니다.

## 요약

이 장을 읽고 Rust가 객체 지향 언어인지 여부에 대해 생각하는 것은
어떤 것이든 괜찮습니다. 하지만 이제는 trait objects를 사용하여 Rust에서
일부 객체 지향 기능을 얻을 수 있다는 것을 알고 있습니다. 동적 디스패치는
코드에 약간의 유연성을 제공할 수 있지만, 실행 시간 성능은 약간 저하될 수
있습니다. 이 유연성을 사용하여 코드 유지 관리성을 높이는 데 도움이 되는
객체 지향 패턴을 구현할 수 있습니다. Rust에는 소유권과 같은 객체
지향 언어에서 없는 다른 기능도 있습니다. 객체 지향 패턴이 항상 Rust의
강점을 활용하는 가장 좋은 방법은 아니지만, 사용 가능한 옵션입니다.

다음으로 Rust의 또 다른 기능인 패턴을 살펴보겠습니다. 이는 책
전반적으로 간략하게 살펴보았지만, 아직 그 전체 능력을 보여주지 않았습니다.
자, 시작해 봅시다!

[more-info-than-rustc]: ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler
[macros]: ch19-06-macros.html#macros
