## 모듈 정의를 통한 범위 및 프라이버시 제어

이 섹션에서는 모듈과 모듈 시스템의 다른 부분, 즉 *경로*에 대해 논의합니다. 경로는 항목에 이름을 부여하는 데 사용됩니다. `use` 키워드는 경로를 범위로 가져오고, `pub` 키워드는 항목을 공개적으로 만듭니다. 또한 `as` 키워드, 외부 패키지 및 glob 연산자에 대해 설명합니다.

### 모듈 참조 가이드

모듈과 경로, `use` 키워드, `pub` 키워드가 컴파일러에서 어떻게 작동하고 대부분의 개발자가 코드를 구성하는지에 대한 빠른 참조를 제공합니다. 이 챕터에서 각 규칙의 예제를 살펴보겠지만, 모듈이 작동하는 방식을 상기시키는 데 유용합니다.

- **크레이트 루트에서 시작**: 크레이트를 컴파일할 때, 컴파일러는 먼저 크레이트 루트 파일(일반적으로 라이브러리 크레이트의 경우 *src/lib.rs* 또는 이진 크레이트의 경우 *src/main.rs*)에서 컴파일할 코드를 검색합니다.
- **모듈 선언**: 크레이트 루트 파일에서 `mod garden;`와 같이 새로운 모듈을 선언할 수 있습니다. 컴파일러는 모듈의 코드를 다음 위치에서 찾습니다.
  - `mod garden` 다음의 세미콜론을 대체하는 괄호 안에 있는 코드
  - *src/garden.rs* 파일
  - *src/garden/mod.rs* 파일
- **서브모듈 선언**: 크레이트 루트 파일 외의 모든 파일에서 서브모듈을 선언할 수 있습니다. 예를 들어, *src/garden.rs*에서 `mod vegetables;`를 선언할 수 있습니다. 컴파일러는 부모 모듈의 이름으로 지정된 디렉토리 내에서 서브모듈의 코드를 다음 위치에서 찾습니다.
  - 부모 모듈의 이름으로 지정된 디렉토리 내에서 `mod vegetables` 다음에 직접 있는 코드
  - *src/garden/vegetables.rs* 파일
  - *src/garden/vegetables/mod.rs* 파일
- **모듈 내 코드에 대한 경로**: 모듈이 크레이트의 일부가 되면, 부모 모듈의 프라이버시 규칙이 허용하는 한, 해당 크레이트의 다른 모든 곳에서 코드에 대한 경로를 사용하여 해당 모듈의 코드를 참조할 수 있습니다. 예를 들어, `garden::vegetables` 모듈의 `Asparagus` 유형은 `crate::garden::vegetables::Asparagus`에서 찾을 수 있습니다.
- **프라이빗 vs. 공개**: 모듈 내부의 코드는 기본적으로 부모 모듈에서 프라이빗합니다. 모듈을 공개적으로 만들려면 `pub mod` 대신 `mod`를 사용하여 선언합니다. 공개 모듈 내의 항목을 공개적으로 만들려면 선언 전에 `pub`를 사용합니다.
- **`use` 키워드**: 범위 내에서 `use` 키워드는 긴 경로를 반복하지 않고 항목에 대한 단축 경로를 만들어줍니다. `crate::garden::vegetables::Asparagus`에 참조할 수 있는 모든 범위에서 `use crate::garden::vegetables::Asparagus;`를 사용하면 해당 유형을 사용할 때 `Asparagus`만 작성하면 됩니다.

여기서 `backyard`라는 이름의 이진 크레이트를 만들어 이 규칙을 보여줍니다. 크레이트의 디렉토리도 `backyard`로 이름이 지정되어 있으며 다음과 같은 파일과 디렉토리를 포함합니다.

```text
backyard
\u251c\u2500\u2500 Cargo.lock
\u251c\u2500\u2500 Cargo.toml
\u2514\u2500\u2500 src
    \u251c\u2500\u2500 garden
    \u2502\u00a0\u00a0 \u2514\u2500\u2500 vegetables.rs
    \u251c\u2500\u2500 garden.rs
    \u2514\u2500\u2500 main.rs
```

이 경우 크레이트 루트 파일은 *src/main.rs*이며 다음과 같이 구성되어 있습니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/main.rs}}
```

`pub mod garden;` 라인은 *src/garden.rs*에서 찾을 수 있는 코드를 컴파일러에 알려줍니다. 이 코드는 다음과 같습니다.

<span class=\"filename\">Filename: src/garden.rs</span>

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden.rs}}
```

여기서 `pub mod vegetables;`는 *src/garden/vegetables.rs*의 코드가 포함된다는 것을 의미합니다. 해당 코드는 다음과 같습니다.

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden/vegetables.rs}}
```

이제 이 규칙의 세부 사항을 살펴보고 실제로 작동하는 방식을 보여드리겠습니다!

### 모듈을 사용하여 관련 코드 그룹화

*모듈*은 crate 내에서 코드를 읽기 쉽고 재사용하기 쉽게 조직하는 데 사용됩니다.
모듈은 또한 코드 내부의 항목이 기본적으로 프라이빗이기 때문에 *프라이버시*를 제어할 수 있습니다.
프라이빗 항목은 외부에서 사용할 수 없는 내부 구현 세부 사항입니다.
우리는 모듈과 그 안의 항목을 공개로 만들어 외부 코드가 사용하고 의존할 수 있도록 선택할 수 있습니다.

예를 들어, 레스토랑의 기능을 제공하는 라이브러리 crate를 작성해 보겠습니다.
함수의 서명을 정의하지만 구현은 비워두어 코드의 조직에 집중하고 레스토랑의 구현에는 집중하지 않습니다.

레스토랑 업계에서는 레스토랑의 일부를 *전면*과 *백면*이라고 합니다.
전면은 고객이 있는 곳이며, 호스트가 고객을 배치하고 서버가 주문을 받고 결제를 처리하고 바텐더가 음료를 만들어내는 곳입니다.
백면은 요리사와 조리사가 주방에서 일하는 곳, 식기세척자가 청소를 하고 관리자가 행정 업무를 하는 곳입니다.

이러한 방식으로 crate를 구조화하려면 함수를  pertanian 모듈로 정리할 수 있습니다.
`cargo new restaurant --lib`를 실행하여 `restaurant`라는 이름의 새로운 라이브러리를 만들고, *src/lib.rs* 파일에 Listing 7-1에 있는 코드를 입력하여 전면 부분의 모듈과 함수 서명을 정의합니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-01/src/lib.rs}}
```

<span class=\"caption\">Listing 7-1: `front_of_house` 모듈이 다른 모듈을 포함하고 있는 모듈</span>

`mod` 키워드를 사용하여 모듈을 정의하고 모듈 이름(이 경우 `front_of_house`)을 붙입니다.
모듈의 몸체는 괄호 안에 들어갑니다.
모듈 내에서는 다른 모듈을 배치할 수 있습니다. Listing 7-1과 같이 다른 모듈을 배치할 수 있습니다.

모듈을 사용하면 관련 정의를 함께 그룹화하고 그룹화된 이유를 명명할 수 있습니다.
이 코드를 사용하는 프로그래머는 그룹을 기반으로 코드를 탐색할 수 있으며, 모든 정의를 읽어야 하는 대신 관련된 정의만 찾을 수 있습니다.
새로운 기능을 추가하는 프로그래머는 프로그램을 정리된 상태로 유지하기 위해 코드를 어디에 배치해야 하는지 알 수 있습니다.

앞서 언급했듯이 `src/main.rs`와 `src/lib.rs`는 각각 crate의 루트라고 불립니다.
이러한 이름의 이유는 이 두 파일의 내용이 crate의 모듈 구조의 뿌리에 있는 `crate`라는 이름의 모듈을 형성하기 때문입니다.
이를 *모듈 트리*라고 합니다.

Listing 7-2는 Listing 7-1의 구조의 모듈 트리를 보여줍니다.

```text
crate
 \u2514\u2500\u2500 front_of_house
     \u251c\u2500\u2500 hosting
     \u2502   \u251c\u2500\u2500 add_to_waitlist
     \u2502   \u2514\u2500\u2500 seat_at_table
     \u2514\u2500\u2500 serving
         \u251c\u2500\u2500 take_order
         \u251c\u2500\u2500 serve_order
         \u2514\u2500\u2500 take_payment
```

<span class=\"caption\">Listing 7-2: Listing 7-1의 코드에 대한 모듈 트리</span>

이 트리는 어떤 모듈이 다른 모듈 안에 포함되는지 보여줍니다. 예를 들어 `hosting`은 `front_of_house` 안에 포함됩니다.
트리는 또한 모듈이 *형제*라는 것을 보여줍니다. 즉, `front_of_house` 안에 정의된 `hosting`과 `serving`은 형제입니다.
모듈 A가 모듈 B 안에 포함되어 있다면, 모듈 A를 모듈 B의 *자식*이라고 하고 모듈 B를 모듈 A의 *부모*라고 합니다.
모든 모듈 트리가 암시적으로 `crate`라는 이름의 모듈 아래에 뿌리 놓여 있는 것을 알아두세요.

모듈 트리는 컴퓨터의 파일 시스템의 디렉토리 트리와 유사할 수 있습니다.
이것은 매우 적절한 비교입니다! 파일 시스템의 디렉토리와 마찬가지로 모듈을 사용하여 코드를 정리합니다.
그리고 디렉토리의 파일과 마찬가지로 모듈을 찾는 방법이 필요합니다.
