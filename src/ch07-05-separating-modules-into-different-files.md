## 모듈을 다른 파일로 분리하기

지금까지 이 장의 모든 예제는 하나의 파일에서 여러 모듈을 정의했습니다.
모듈이 커지면 코드를 탐색하기 쉽게 하기 위해 정의를 별도의 파일에 이동하는 것이 좋습니다.

예를 들어, 7-17번 목록의 여러 레스토랑 모듈을 가진 코드에서 시작해 보겠습니다. 우리는 모든 모듈이 크레이트 루트 파일에서 정의되는 대신 파일로 모듈을 추출할 것입니다. 이 경우 크레이트 루트 파일은 *src/lib.rs* 이지만, *src/main.rs* 가 크레이트 루트 파일인 이진 크레이트에도 이 절차가 적용됩니다.

먼저 `front_of_house` 모듈을 자신의 파일에 추출합니다. `front_of_house` 모듈의 중괄호 안의 코드를 제거하고, *src/lib.rs* 에 7-21번 목록에 나와 있는 코드만 남겨둡니다. 이때, *src/front_of_house.rs* 파일을 만들기 전까지는 컴파일되지 않습니다.

Filename: src/lib.rs

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/lib.rs}}
```

Listing 7-21: `front_of_house` 모듈을 선언하는데, 본문은 *src/front_of_house.rs* 에 있을 것입니다.

다음으로, 중괄호 안에 있던 코드를 *src/front_of_house.rs* 라는 새 파일로 옮깁니다. 7-22번 목록에 나와 있습니다. 컴파일러는 크레이트 루트에서 `front_of_house` 라는 이름의 모듈 선언을 발견했기 때문에 해당 파일을 찾을 수 있습니다.

Filename: src/front_of_house.rs

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/front_of_house.rs}}
```

Listing 7-22: `front_of_house` 모듈 내부의 정의가 *src/front_of_house.rs* 에 있습니다.

모듈 트리에서 `mod` 선언을 사용하여 파일을 로드해야 하는 것은 한 번만입니다. 컴파일러가 프로젝트에 해당 파일이 포함되어 있고 (모듈 트리에서 코드가 어디에 있는지 `mod` 문을 사용하여 선언한 위치를 알기 때문에) 프로젝트의 다른 파일은 해당 파일의 코드를 `mod` 문에서 선언된 경로를 사용하여 참조해야 합니다. 이는 [“모듈 트리에서 항목을 참조하는 경로”][paths]<!-- ignore --> 섹션에서 설명된 내용과 같습니다. 즉, `mod` 는 다른 프로그래밍 언어에서 볼 수 있는 “include” 연산이 아닙니다.

다음으로 `hosting` 모듈을 자신의 파일에 추출합니다. `hosting` 이 루트 모듈의 자식 모듈이 아니라 `front_of_house` 의 자식 모듈이기 때문에 이 과정은 조금 다릅니다. `hosting` 에 대한 파일을 모듈 트리의 조상 이름으로 지정된 새 디렉토리에 배치할 것입니다. 이 경우 *src/front_of_house* 입니다.

`hosting` 을 옮기기 시작하려면 *src/front_of_house.rs* 를 `hosting` 모듈의 선언만 포함하도록 변경합니다.

Filename: src/front_of_house.rs

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house.rs}}
```

그런 다음 *src/front_of_house* 디렉토리와 *hosting.rs* 파일을 만들어 `hosting` 모듈에서 정의된 코드를 포함합니다.

Filename: src/front_of_house/hosting.rs

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house/hosting.rs}}
```

만약 *hosting.rs* 를 *src* 디렉토리에 넣었다면, 컴파일러는 *hosting.rs* 코드가 크레이트 루트에서 선언된 `hosting` 모듈에 속한다고 기대할 것입니다.
## 모듈을 여러 파일로 분리하기

이전 챕터에서 우리는 `front_of_house` 모듈 내부에 `hosting` 모듈을 정의했습니다. 이제 이 모듈들을 여러 파일로 분리하여 코드를 더욱 조직적으로 관리해 보겠습니다.

### 파일 구조

모듈의 코드를 담당하는 파일은 모듈 이름과 동일하거나, `mod.rs` 파일을 사용하여 모듈을 정의할 수 있습니다. 예를 들어, `front_of_house` 모듈의 코드는 `src/front_of_house.rs` 또는 `src/front_of_house/mod.rs` 파일에서 찾을 수 있습니다. `hosting` 모듈의 코드는 `src/front_of_house/hosting.rs` 또는 `src/front_of_house/hosting/mod.rs` 파일에서 찾을 수 있습니다.

이러한 파일 구조는 Rust 컴파일러가 모듈의 코드를 찾는 방식을 결정합니다. 컴파일러는 일반적으로 모듈 이름과 일치하는 파일을 찾습니다. 예를 들어, `front_of_house` 모듈의 코드는 `src/front_of_house.rs` 파일에서 찾을 수 있습니다.

### 모듈 분리

모듈을 여러 파일로 분리하면 코드를 더욱 관리하기 쉽습니다. 예를 들어, `front_of_house` 모듈의 코드를 `src/front_of_house.rs` 파일로, `hosting` 모듈의 코드를 `src/front_of_house/hosting.rs` 파일로 분리할 수 있습니다.

이렇게 하면 `eat_at_restaurant` 함수에서 `front_of_house` 모듈의 `hosting` 모듈을 사용할 때, 코드가 더욱 명확하고 읽기 쉽습니다.

### `use` 문

`use` 문은 모듈 내부의 항목을 다른 모듈에서 사용할 때 사용됩니다. 예를 들어, `front_of_house` 모듈의 `hosting` 모듈에서 `take_order` 함수를 사용하려면 다음과 같이 `use` 문을 사용할 수 있습니다.

```rust
use front_of_house::hosting; 

fn eat_at_restaurant() { 
    hosting::take_order(); 
}
```

### 요약

Rust는 여러 파일로 분리된 모듈을 사용하여 코드를 조직적으로 관리할 수 있도록 합니다. `use` 문을 사용하여 모듈 내부의 항목을 다른 모듈에서 사용할 수 있습니다. 모듈의 코드는 `mod.rs` 파일 또는 모듈 이름과 동일한 파일에서 찾을 수 있습니다.

다음 챕터에서는 Rust 표준 라이브러리에서 제공하는 다양한 자료구조를 살펴보겠습니다.

