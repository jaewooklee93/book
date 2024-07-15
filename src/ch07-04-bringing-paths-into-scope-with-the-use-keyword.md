## use 키워드로 경로를 스코프에 가져오기

함수를 호출하기 위해 경로를 매번 작성해야 하는 것은 불편하고 반복적일 수 있습니다.
7-7번 목록에서 우리가 `add_to_waitlist` 함수에 절대 경로 또는 상대 경로를 선택했든, `add_to_waitlist`를 호출할 때마다 `front_of_house`와 `hosting`을 명시해야 했습니다. 다행히도 이 과정을 간소화하는 방법이 있습니다. 즉, `use` 키워드를 사용하여 한 번 경로에 대한 단축 경로를 만들고, 그 후에는 해당 스코프 내에서 짧은 이름을 사용하여 호출할 수 있습니다.

7-11번 목록에서는 `crate::front_of_house::hosting` 모듈을 `eat_at_restaurant` 함수의 스코프에 가져와 `eat_at_restaurant`에서 `hosting::add_to_waitlist`만 명시하여 `add_to_waitlist` 함수를 호출할 수 있도록 합니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-11/src/lib.rs}}
```

<span class=\"caption\">Listing 7-11: `use`를 사용하여 모듈을 스코프에 가져오기</span>

특정 스코프에서 `use`와 경로를 추가하는 것은 파일 시스템에서 심볼릭 링크를 만드는 것과 유사합니다. `crate` 루트에서 `use crate::front_of_house::hosting`를 추가하면 `hosting`이 해당 스코프에서 유효한 이름이 됩니다. 마치 `hosting` 모듈이 `crate` 루트에서 정의된 것처럼.
`use`로 스코프에 가져온 경로는 다른 경로와 마찬가지로 프라이버시를 확인합니다.

주의할 점은 `use`는 `use` 문이 있는 스코프에서만 단축 경로를 만드는 것입니다. 7-12번 목록은 `eat_at_restaurant` 함수를 `customer`라는 새로운 자식 모듈로 이동합니다. `customer` 모듈은 `use` 문이 있는 스코프와 다른 스코프이므로 함수 본문이 컴파일되지 않습니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground,test_harness,does_not_compile,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-12/src/lib.rs}}
```

<span class=\"caption\">Listing 7-12: `use` 문은 해당 스코프에서만 적용됩니다.</span>

컴파일러 오류는 단축 경로가 `customer` 모듈 내에서 더 이상 적용되지 않는다는 것을 보여줍니다.

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-12/output.txt}}
```

단축 경로가 해당 스코프에서 사용되지 않는다는 경고도 있습니다! 이 문제를 해결하려면 `customer` 모듈 내에서도 `use`를 이동하거나, 자식 `customer` 모듈에서 부모 모듈의 단축 경로를 `super::hosting`으로 참조해야 합니다.

### Idiomatic `use` 경로 만들기

7-11번 목록에서 `use crate::front_of_house::hosting`를 명시하고 `eat_at_restaurant`에서 `hosting::add_to_waitlist`를 호출하는 이유를 궁금해했을 수 있습니다. 7-13번 목록과 같이 `add_to_waitlist` 함수의 전체 경로를 `use`로 명시하여 동일한 결과를 얻을 수 있었습니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-13/src/lib.rs}}
```

<span class=\"caption\">Listing 7-13: `use`를 사용하여 `add_to_waitlist` 함수를 스코프에 가져오기(불 idiomatic)</span>

7-11번 목록과 7-13번 목록은 모두 동일한 작업을 수행하지만, 7-11번 목록은 `use`를 사용하여 함수를 스코프에 가져오는 idiomatic 방법입니다. 함수의 부모 모듈을 `use`로 가져오면 함수를 호출할 때 부모 모듈을 명시해야 합니다. 함수를 호출할 때 부모 모듈을 명시하면 함수가 로컬로 정의되지 않았음을 명확히 보여주면서 동시에 전체 경로의 반복을 최소화합니다. 7-13번 목록의 코드는 `add_to_waitlist` 함수가 정의된 위치가 명확하지 않습니다.

반대로, `use` 키워드로 구조체, 열거형 및 기타 항목을 가져올 때는 전체 경로를 지정하는 것이 전통적인 방법입니다.
7-14번 목록은 표준 라이브러리의 `HashMap` 구조체를 이진 저장소의 범위로 가져오는 전통적인 방법을 보여줍니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-14/src/main.rs}}
```

<span class=\"caption\">Listing 7-14: `HashMap`을 전통적인 방식으로 범위에 가져오기</span>

이 전통적인 방법 뒤에는 명확한 이유가 없습니다. 단지 등장한 관습이며 사람들은 이런 방식으로 Rust 코드를 읽고 작성하는 데 익숙해졌습니다.

예외는 동일한 이름을 가진 두 개의 항목을 `use` 문으로 가져올 때입니다. Rust는 이를 허용하지 않습니다. 7-15번 목록은 동일한 이름이지만 부모 모듈이 다른 두 개의 `Result` 유형을 가져오는 방법과 어떻게 참조하는지 보여줍니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-15/src/lib.rs:here}}
```

<span class=\"caption\">Listing 7-15: 동일한 이름을 가진 두 유형을 동일한 범위로 가져오려면 부모 모듈을 사용해야 합니다.</span>

보시다시피 부모 모듈을 사용하면 두 `Result` 유형을 구분할 수 있습니다. `use std::fmt::Result`와 `use std::io::Result`를 지정하면 `Result` 유형이 두 개 범위에 있게 되고, Rust는 어떤 `Result`를 의미하는지 알 수 없습니다.

### `as` 키워드를 사용하여 새 이름 제공

동일한 이름을 가진 두 유형을 `use` 문으로 동일한 범위로 가져올 때 해결책은 `as` 키워드를 사용하여 유형에 대한 새 이름 또는 *별칭*을 지정하는 것입니다. 7-16번 목록은 7-15번 목록의 코드를 `as` 키워드를 사용하여 하나의 `Result` 유형을 새로 이름 지어 작성하는 다른 방법을 보여줍니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-16/src/lib.rs:here}}
```

<span class=\"caption\">Listing 7-16: `use` 키워드로 유형을 가져올 때 `as` 키워드를 사용하여 유형을 새로 이름 지정하기</span>

두 번째 `use` 문에서 `std::io::Result` 유형에 대해 `IoResult`라는 새 이름을 선택했습니다. 이는 `std::fmt`에서 가져온 `Result`와 충돌하지 않습니다.
7-15번 목록과 7-16번 목록은 전통적인 방법으로 간주되므로 선택은 귀하의 것입니다!

### `pub use`를 사용하여 이름 재전달

`use` 키워드로 이름을 가져올 때, 새 범위에서 사용 가능한 이름은 private입니다. 우리 코드를 호출하는 코드가 해당 이름을 우리 코드의 범위에서 정의된 것처럼 참조하도록 하려면 `pub`과 `use`를 결합할 수 있습니다. 이 기술은 *재전달*이라고 불립니다. 왜냐하면 우리는 항목을 가져오지만 다른 사람이 자신의 범위로 가져올 수 있도록 합니다.

7-17번 목록은 7-11번 목록의 코드에서 루트 모듈의 `use`를 `pub use`로 변경한 것입니다.

<span class=\"filename\">Filename: src/lib.rs</span>

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-17/src/lib.rs}}
```

<span class=\"caption\">Listing 7-17: `pub use`를 사용하여 다른 코드에서 사용할 수 있는 이름 만들기</span>

이 변경 이전에는 외부 코드가 `add_to_waitlist` 함수를 호출하려면 `restaurant::front_of_house::hosting::add_to_waitlist()`와 같은 경로를 사용해야 했습니다. 또한 `front_of_house` 모듈을 `pub`으로 표시해야 했습니다. 이제 `pub use`를 사용하면 외부 코드는 `add_to_waitlist` 함수를 호출할 때 `restaurant::front_of_house::hosting` 경로를 사용하지 않고 `add_to_waitlist` 함수를 직접 호출할 수 있습니다.

use` 키워드를 사용하여 패스를 스코프에 가져오기

`use` 키워드는 루트 모듈에서 `hosting` 모듈을 다시 내보냈습니다. 외부 코드는 `restaurant::hosting::add_to_waitlist()` 경로를 사용할 수 있습니다.

다시 내보내는 것은 코드의 내부 구조가 코드를 호출하는 프로그래머가 생각하는 방식과 다를 때 유용합니다. 예를 들어, 이 레스토랑 메타포에서 레스토랑을 운영하는 사람들은 \u201c전면\u201d와 \u201c백면\u201d에 대해 생각합니다. 그러나 레스토랑을 방문하는 고객은 레스토랑의 부분을 그러한 용어로 생각하지 않을 가능성이 높습니다. `pub use`를 사용하면 하나의 구조로 코드를 작성할 수 있지만 다른 구조를 노출할 수 있습니다. 이렇게 하면 라이브러리를 개발하는 프로그래머와 라이브러리를 호출하는 프로그래머 모두에게 라이브러리가 잘 구성됩니다. `pub use`의 또 다른 예와 이것이 crate의 문서에 미치는 영향은 제14장의 \u201c`pub use`를 사용하여 편리한 공개 API를 내보내기\u201d[ch14-pub-use]<!-- ignore --> 섹션에서 살펴볼 것입니다.

### 외부 패키지 사용

제2장에서는 `rand`라는 외부 패키지를 사용하여 랜덤 숫자를 얻는 추측 게임 프로젝트를 프로그래밍했습니다. `rand`를 프로젝트에서 사용하려면 *Cargo.toml* 파일에 다음 줄을 추가했습니다.

<!-- `rand` 버전을 업데이트할 때, 이러한 파일에서 사용되는 `rand` 버전도 업데이트하여 모두 일치하도록 합니다:
* ch02-00-guessing-game-tutorial.md
* ch14-03-cargo-workspaces.md
-->

<span class=\"filename\">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:9:}}
```

*Cargo.toml* 파일에 `rand`를 의존성으로 추가하면 Cargo가 `rand` 패키지와 모든 의존성을 [crates.io](https://crates.io/)에서 다운로드하고 `rand`를 프로젝트에 제공합니다.

그런 다음 패키지의 스코프에 `rand` 정의를 가져오려면 `use` 문을 사용하여 crate의 이름인 `rand`로 시작하고 스코프에 가져올 항목을 나열합니다. 제2장의 \u201c랜덤 숫자 생성\u201d[rand]<!-- ignore --> 섹션에서 `Rng` 트레이트를 스코프에 가져와 `rand::thread_rng` 함수를 호출하는 방법을 살펴보았습니다.


Rust 커뮤니티의 구성원들은 [crates.io](https://crates.io/)에 많은 패키지를 제공했으며, 이러한 패키지를 패키지에 포함하는 것은 다음과 같은 단계를 포함합니다. 패키지의 *Cargo.toml* 파일에 이러한 패키지를 나열하고 `use`를 사용하여 그들의 crate에서 항목을 스코프에 가져옵니다.

참고로 표준 `std` 라이브러리 또한 패키지에서 외부인 crate입니다. 표준 라이브러리는 Rust 언어와 함께 배포되기 때문에 `std`을 포함하는 *Cargo.toml*을 변경할 필요가 없습니다. 그러나 `use`를 사용하여 표준 라이브러리에서 항목을 패키지의 스코프에 가져오는 것은 여전히 필요합니다. 예를 들어 `HashMap`을 사용하려면 다음과 같은 줄을 사용해야 합니다.

```rust
use std::collections::HashMap;
```

이는 표준 라이브러리 crate인 `std`로 시작하는 절대 경로입니다.

### 큰 `use` 목록을 정리하기 위한 중첩 경로 사용

같은 crate 또는 같은 모듈에서 정의된 여러 항목을 사용하는 경우 각 항목을 개별 줄에 나열하면 파일에서 수직 공간이 많이 차지될 수 있습니다. 예를 들어, 표 2-4에서 추측 게임에서 사용했던 두 개의 `use` 문은 `std`에서 항목을 스코프에 가져옵니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-01-use-std-unnested/src/main.rs:here}}
```

대신, 공통 경로 부분을 지정한 후 두 개의 콜론과 curly braces를 사용하여 경로의 다른 부분을 묶어서 하나의 줄로 동일한 항목을 스코프에 가져올 수 있습니다. 이를 표 7-18에서 보여줍니다.

<span class=\"filename\">Filename: src/main.rs</span>

```rust,ignore
## 경로를 사용 키워드로 사용하여 경로를 현재 범위에 가져오기

프로그램이 커질수록 동일한 crate 또는 모듈에서 많은 항목을 가져오는 것은 `use` 문을 줄이는 데 도움이 될 수 있습니다. 

`use` 문을 사용하여 여러 항목을 가져올 때, 중첩된 경로를 사용하면 코드를 더욱 간결하게 만들 수 있습니다. 예를 들어, `std::io`와 `std::io::Write`를 가져오는 두 개의 `use` 문을 살펴보겠습니다. 

```rust
use std::io; 
use std::io::Write; 
```

두 경로의 공통 부분은 `std::io`입니다. 이 두 경로를 하나의 `use` 문으로 통합하려면 `self`를 중첩된 경로에 사용할 수 있습니다. 

```rust
use std::io::{self, Write}; 
```

이 줄은 `std::io`와 `std::io::Write`를 현재 범위에 가져옵니다. 

### Glob 연산자

특정 경로에서 정의된 모든 공개 항목을 현재 범위에 가져오려면 해당 경로 뒤에 `*` 글로브 연산자를 지정할 수 있습니다. 

```rust
use std::collections::*; 
```

이 `use` 문은 `std::collections`에서 정의된 모든 공개 항목을 현재 범위에 가져옵니다. 글로브 연산자를 사용할 때 주의해야 할 점은 코드의 가독성이 떨어질 수 있다는 것입니다. 

글로브 연산자는 테스트에서 모든 항목을 테스트 모듈에 가져오는 데 자주 사용됩니다. 

